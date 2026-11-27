# Contributing

Each chapter of the reference is one Markdown file in `reference/`, and
[croftlang.com](https://croftlang.com/learn) turns it into a page.

## Layout

```
reference/
├── 01-hello-world.md            → croftlang.com/learn/hello-world
├── 02-contexts.md               → croftlang.com/learn/contexts
└── ...
```

- The number is the reading order. The rest of the file name is the page's address.
- The first line is the chapter's title: `# Contexts`.
- Link to another chapter by its file name, `[contexts](02-contexts.md)`. That works here on
  GitHub, and the website turns it into `/learn/contexts`.
- Croft code goes in ` ```croft ` blocks, so the site can highlight it.

## Writing

Keep it short and plain, as if explaining it to a colleague. One idea per paragraph, and
several small examples rather than one big one. A joke is allowed now and then, as long as the
page still makes sense without it.

Check every code sample with the compiler before it goes in (`croft check`), and quote real
error messages, not ones you remember.

## Language changes

A change to the language in [croftlang/lang](https://github.com/croftlang/lang) that changes
how Croft behaves needs a matching pull request here.

## Seeing your changes

Check out [croftlang/website](https://github.com/croftlang/website) next to this repository
and run `pnpm dev:docs` there. It reads the chapters from here and reloads when one changes.
The live site picks up `main` on its next deploy.
