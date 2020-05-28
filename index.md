## A bit about me

My name is Steve and I am a developer living just outside of Edinburgh, Scotland.
I have been writing C# for the better part of 15 years, and F# on the side for about 10 years.
I have also become particularly interested in Rust and WASM.

## Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
