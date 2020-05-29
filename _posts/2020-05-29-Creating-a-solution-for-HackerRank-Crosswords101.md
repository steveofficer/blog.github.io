---
layout: post
tags: F# DynamicProgramming Graphs
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
 represents the crossword grid 
It is then a dynamic programming problem, where we are trying to map a set of words to a set of spaces, trying each word against each space.
One optimization we can make is that because we know the length of each word, and we know the length of each space we can compare then lengths and only consider pairs with the same length.
Another constraint we need to apply is that Rows and Columns can intersect and share common characters. So we need to consider the characters already present in the grid when we try to match a word to a space.