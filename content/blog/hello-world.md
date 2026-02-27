+++
title = "Hello, World — A First Post"
date = 2025-02-01
description = "An example post showing what this site can do: text, images, code, and more."
[taxonomies]
tags = ["meta", "writing"]
+++

This is the first post on this site. It's also a reference for how to
write future posts, so it covers most of the building blocks you'll use.

## Images

Images in posts are placed using a shortcode. On wide screens they float
into a right-hand column alongside the text; on mobile they flow inline.

{{ img(src="example.webp", alt="A placeholder image", caption="Caption goes here if you want one", w=800, h=533) }}

Just drop your image files into `static/img/` and reference them by
filename. If you have both a `.webp` and a `.jpg` version the browser
will pick the best one automatically.

## Writing

Plain paragraphs are the workhorse. Keep sentences clear and direct.
Long walls of text are hard to read — break things up with headings,
short paragraphs, or a list when it genuinely helps.

> A blockquote looks like this. Good for a pull quote or an excerpt
> you want to highlight.

## Lists

Use a list when order or enumeration genuinely matters:

1. First item
2. Second item
3. Third item

Or an unordered list for a collection of things:

- One thing
- Another thing
- A third thing

## Code

Inline code looks like `this`. A fenced block:

```rust
fn main() {
    println!("Hello, maravexa.us!");
}
```

## Links and footnotes

Link to [external sites](https://example.com) or to
[other posts on this site](/blog/) normally.

That's about everything you need. Write the content, run `zola build`,
push to your repo. Done.
