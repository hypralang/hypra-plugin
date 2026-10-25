---
name: hypra
description: Write and fix Hypra pages (.hypra files), a markup language that compiles to HTML, Markdown for AI agents and WebMCP tools. Use when creating, editing or reviewing .hypra files, or when a project uses the hypra compiler.
---

# Writing Hypra

Hypra is HTML written another way, checked by a compiler. Every HTML element and attribute works, so design a
page exactly as you would in HTML, with the same layout, classes and CSS or Tailwind. Hypra writes no CSS of its
own. The full reference is https://hypralang.dev/llms-full.txt; read it when something here isn't enough.

## The shape of everything

`name(attributes) "text" { children }`. Every part is optional.

```hypra
page(title: "Our menu", description: "Today's bread and pastries, baked this morning, with prices and allergens.") {
  h1 "Our menu"
  p { "Open " b "every day" " from 7." }
  div(class: "card", id: "rye") {
    img("/img/rye.jpg", alt: "A rye loaf on a wooden board", width: 600, height: 400)
    a("/allergens") "Allergens in our bread"
  }
}
```

From HTML:

- `<h1>Welcome</h1>` → `h1 "Welcome"`
- `<a href="/guide" class="button">Read the guide</a>` → `a("/guide", class: "button") "Read the guide"`
- `<p>Open <b>daily</b>.</p>` → `p { "Open " b "daily" "." }`
- `<img src="/x.svg" alt="">` → `img("/x.svg", decorative: true)`
- `<input type="checkbox" checked>` → `input(type: "checkbox", checked: true)`
- `<button @click="open = !open">` → `button("@click": "open = !open")` (quote unusual attribute names)
- `<html lang>`, `<title>` → `page(lang: "de", title: "…", description: "…")`
- `<body class="…">` → `body(class: "…")` with no content, inside the page or layout
- `<link>`, `<script src>`, `<style>` → inside `head { }`: `link(rel: "stylesheet", href: "/app.css")`,
  `script("/app.js")`, `style """ … """`
- `<!-- comment -->` → `// comment`

## Pages, data, layouts and components

- Every page file has exactly one `page(...)` with a `title` and a `description` (50 to 160 characters wide).
- Data the page uses is declared: `props(article: Article)`, then `"{article.title}"` in any quoted text or
  attribute, and `img(article.photo, alt: article.photo_alt)` for a value on its own.
- Loops and conditions: `for item in list { li "{item.name}" }`, `if user.vip { b "VIP" }`.
- A `layout` holds what every page shares (head, header, footer) and a `slot` for the page; a page uses it with
  `page(layout: Site, ...)`. A `component` is a piece used more than once. Keep both in their own files, like
  `layouts/site.hypra`, and bring them in with `use "../layouts/site.hypra"` at the top of the page. Files with
  only layouts and components have no page; pages build them.

```hypra
layout Site(lang: "en", title_suffix: " | Crumb & Co") {
  head { link(rel: "stylesheet", href: "/styles.css") }
  header { nav(aria-label: "Main") { a("/") "Home"  a("/menu") "Menu" } }
  main { slot }
  footer { p "Baked in Leeds since 2019." }
}

component Item(name: Text, price: Text) {
  article(class: "item") {
    h3 "{name}"
    p "{price}"
  }
}

page(layout: Site, title: "Menu", description: "Today's bread and pastries at Crumb & Co, baked this morning.") {
  props(items: List)
  h1 "Today's menu"
  h2 "Bread"
  for item in items { Item(name: item.name, price: item.price) }
}
```

## Forms and actions

Use `field` inside a `form`: it writes the label and the input together and checks the value's type. A form
with a `name` is also a tool for AI agents. `action` is a button that does one thing on the server.

```hypra
page(title: "Order bread", description: "Order bread from Crumb & Co for pickup tomorrow morning, from 7 to 11.") {
  h1 "Order bread"
  form("/orders", name: "order", description: "Order bread for pickup tomorrow") {
    field("email", type: Email, label: "Email")
    field("loaves", type: Int(1..10), label: "Loaves")
    field("pickup", type: Choice(early: "7 to 9", late: "9 to 11"), label: "Pickup time")
    button "Order"
  }
  action("newsletter.join") "Get the menu by email"
}
```

Give a field `description: "..."` when its label alone doesn't tell an agent what to enter, e.g.
`field("notes", type: Text(max: 300), label: "Notes", optional: true, description: "Allergies or a sliced loaf")`.
Add `autosubmit: true` to a form only when it's safe to send without the visitor checking it, like a search;
never to a form that orders, pays or deletes. Both need hypra 0.5 or later.

## What the compiler checks

Errors stop the build:

- An image needs `alt` that says what it shows, or `decorative: true`.
- Something people click is a `button` or a link, never a `div` or `span` with `onclick`.
- Inputs written by hand (`input`, `select`, `textarea`) need a label; prefer `field`.
- A page has one `h1`.
- `html`, `title` and a `body` with content aren't written by hand.

Warnings are worth fixing too: headings that skip a level, link text like "click here", alt text like "image".

## Work like this

1. Write or change the `.hypra` files.
2. Run `hypra check` on them (`npx hypra check views` with the npm package; folders work). It exits with 1 on an
   error, and every message has a code, a line, what to change and a link to a page about the rule.
3. Fix every error and warning. Don't silence one with `allow("CODE") "reason"` unless the page really needs it.
4. Build with `hypra build views --target js --out dist` for Node.js, or let `hypra/laravel` compile them in
   Laravel.

For how a whole project is laid out (pages, a layout, components, a stylesheet and a server), see
https://github.com/hypralang/hypra/tree/main/starter. Build what was asked for in the same way; don't copy it.
