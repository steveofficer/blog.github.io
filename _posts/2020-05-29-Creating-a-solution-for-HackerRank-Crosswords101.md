---
layout: post
tags: F# DynamicProgramming
---

I was wanting to exercise my functional programming muscles, so I thought I would tackle a HackerRank puzzle as they have a wide variety of problems that have already been thought out and are easy to just pick up and chew on.

I wanted something a bit meatier than an easy problem and scrolling through the list I came across an Advanced level problem that sounded interesting.

[The problem](https://www.hackerrank.com/challenges/crosswords-101/problem)

Even though I use F#, which supports mutation, I wanted to set myself the limitation of being purely functional.

## Interpreting the problem

We have a 10 x 10 grid which is sparsely populated.
- `+` means we have a non-fillable block
- `-` means we have a block that needs filled
- We need to find the adjacent `-` blocks, going top to bottom, and left to right. With each contiguous sequence representing a space for a word.
- Rows and Columns can overlap such that 2 spaces share a single block.


## Deciding on data structures.
The words are line segments, they have a start point, an end point and a length.
The grid itself is a `Map<(int*int), char>` which maps a co-ordinate in the grid to a letter that occupies that point.

I considered 2 different approaches on how to model the problem:
 - We could use a graph traversal algorithm to create disconnected set of graphs that we then try and solve.
 - We could break the grid up into rows and columns, and then find the spaces that we need to fill.
 
I first considered a graph based model algorithm, but I thought that the second approach would be less time consuming as I was getting too hung up on how to represent the graph. However, I would like to revisit the graph based approach at a later time.

## The steps
Because we read the grid in row by row, it makes sense to build the rows of the grid while we receive the data.
Then from the rows, we can find all the columns.

Graphically and at a high level this is what we are trying to do:

Read in a textual representation of the crossword grid:
```
+-++++++++
+-++++++++
+-------++
+-++++++++
+-++++++++
+------+++
+-+++-++++
+++++-++++
+++++-++++
++++++++++
```

which represents the crossword grid 

![Blank crossword](/assets/images/posts/2020-05-29/EmptyCrossWord.png)

We then want to find all the contiguous regions that represent the placeholders of where we want to place the words.

![Find Rows and Columns](/assets/images/posts/2020-05-29/ColumnsAndRowsIdentified.png)

We have found 4 contiguous regions.
1. Starts at (1,0). Ends at (1,6).
1. Starts at (5,5). Ends at (5,8).
1. Starts at (1,2). Ends at (7,2).
1. Starts at (1,5). Ends at (6,5).

Which is great because we have 4 words that we are trying to place, namely `AGRA, NORWAY, ENGLAND, GWALIOR`

We have the following constraints when we match words to regions:
1. The length of the word vs the length of the region. We can only match a word to a space if they have the same length.
1. The blocks that are already filled. If the region overlaps with other regions, we can only match words that have the same characters in the overlapped blocks.

Perhaps one immediate optimization we can make is to sort the words and regions by length frequency so that we can make sure we can quickly shrink the search space. This is because we can quickly eliminate the regions that only have a few possibilities, and then do trial and error on the spaces that have many possibilities.

In this case, we know that `AGRA` is the only word that can appear at `(5,5), (5,8)` and we know that `NORWAY` is the only word that can appear at `(1,5), (6,5)`.
So what we are left with is deciding how to match `ENGLAND` and `GWALIOR`, which gives us a search space of `2` vs an original search space of `24`. Nice, 12 times performance boost (ignoring the cost of creating the frequency map and the sort) :)

![Match Exact Matches](/assets/images/posts/2020-05-29/PartiallyFilled.png)


This is a dynamic programming problem, where we can break the problem down by matching 1 word to 1 space, removing them from the search space, and then trying to solve the problem with the remaining words and regions.

## The code

```fsharp
// Just for testing purposes we have these hard coded.
let rows = [| 
    "+-++++++++"
    "+-++++++++"
    "+-------++"
    "+-++++++++"
    "+-++++++++"
    "+------+++"
    "+-+++-++++"
    "+++++-++++"
    "+++++-++++"
    "++++++++++"
|]
let words = [ "AGRA"; "NORWAY"; "ENGLAND"; "GWALIOR" ]

// The code
type Region = (int*int) list

type Grid = Map<int*int, char>

let parse_row (y: int) (row: string) = 
    let (grid, regions) =
        row.ToCharArray()
        |> Array.mapi (fun x c -> x,c)
        |> Array.filter (snd >> ((=) '-'))
        |> Array.fold 
            (fun (grid, regions) (x, _) -> 
                let next_grid = Map.add (x,y) ' ' grid
                let next_regions =
                    match regions with
                    | h::t when x - fst (List.head h) = 1 -> ((x,y)::h)::t
                    | t -> [(x,y)]::t
            
                (next_grid, next_regions)
            )
            (Map.empty, [])
    (grid, regions |> List.filter (fun r -> r.Length > 1) |> List.map List.rev)

let calculate_columns (m: Grid) = 
    Seq.collect (fun x -> seq {for y in 0..9 -> (x,y)}) (seq {0..9})
    |> Seq.filter (fun (x,y) -> m.ContainsKey(x,y)) 
    |> Seq.fold
        (fun regions (x,y) -> 
            match regions with
            | h::t when y - snd (List.head h) = 1 -> ((x,y)::h)::t
            | t -> [(x,y)]::t
        ) 
        []
    |> Seq.filter (fun r -> r.Length > 1) |> Seq.map List.rev
    |> List.ofSeq

let merge_grids (gridA: Grid) (gridB: Grid) =
    let itemsA = Map.toList gridA
    let itemsB = Map.toList gridB
    itemsA@itemsB |> Map.ofList

let draw (grid: Grid) =
    [0..9]
    |> List.iter (fun y -> printfn "%s" ([0..9] |> List.map (fun x -> match grid.TryFind((x,y)) with | Some c -> c | _ -> '+') |> String.Concat))

let fits_in (region: Region) (grid: Grid) (word: char list) =
    if (region.Length <> word.Length)
    then false
    else 
        word
        |> List.zip region
        |> List.forall (fun (coord, c) -> match grid.Item coord with | ' ' -> true | x when x = c -> true | _ -> false)

let rec permutations = function
    | []      -> seq [List.empty]
    | x :: xs -> Seq.collect (insertions x) (permutations xs)
and insertions x = function
    | []             -> [[x]]
    | (y :: ys) as xs -> (x::xs)::(List.map (fun x -> y::x) (insertions x ys))

let rec solve (grid: Grid) (regions: Region list) (words: char list list) =
    permutations words
    |> Seq.choose (fun w ->
        match (regions, w) with
        | ([],[]) -> Some grid
        | (rh::rt, wh::wt) when wh |> fits_in rh grid -> 
            let new_grid = 
                wh
                |> List.zip rh
                |> List.fold (fun (g: Grid) (coord, c) -> g.Add(coord, c)) grid
            solve new_grid rt wt
        | _ -> None
    )
    |> Seq.tryItem 0
    
let (grid, rows) =
       [ for i in 0..9 -> rows.[i] |> parse_row i ] 
       |> List.fold (fun (grid, regions) (g,r) -> (merge_grids grid g, r@regions )) (Map.empty, [])

let regions = 
    List.append
        rows
        (calculate_columns grid)

match solve grid regions words with
| Some g -> draw g
| _ -> printfn "no solution"
```
