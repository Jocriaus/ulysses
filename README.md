# Ulysses docs

**[Play Ulysses in your browser](play/index.html ':ignore')**: mouse or touch, drag to steer, Space to fire. Best in portrait on a phone.

The documentation library for **Ulysses**, the flood rescue game in `ulysses-godot/`.

![Ulysses start screen](manual/images/start-screen.png ':size=180')

## Guides

| Guide | For | Start here |
|---|---|---|
| **The Ulysses manual** | Web backend developers and gamers learning to make a game in Godot 4 | [Introduction](manual/README.md) |

## Using this library

- **Sidebar**: every chapter, in reading order. **Search** (top of the sidebar) looks through every chapter.
- **Pictures**: click a screenshot to zoom in; click again to close. The editor screenshots have numbered yellow boxes, explained in the list under each one.
- **Code**: GDScript, shader and config blocks are highlighted; each has a **Copy** button.
- **Previous / Next** links at the bottom of each page follow the chapter order.

## Opening it

The library is a static page, `docs/index.html`, that reads the Markdown files in `docs/` as they are (it uses [Docsify](https://docsify.js.org), kept in `docs/lib/` so it works offline). It needs a web server, because browsers block pages opened as files from loading other files.

| Way | Steps | Address |
|---|---|---|
| **XAMPP** (this machine) | Start Apache in the XAMPP Control Panel | http://localhost/First-Game/docs/ |
| **Python** | `python -m http.server 8000 --directory docs` from the repo root | http://localhost:8000/ |
| **Node** | `npx serve docs` from the repo root | the address it prints |
| **Online** | Published with the game by `deploy/publish_site.ps1` (see the repo README) | https://jocriaus.github.io/ulysses/ |

The Markdown also reads fine on GitHub or in VS Code's preview, without the library.

## Adding a guide

1. Make a folder in `docs/` (for example `docs/art-pipeline/`) with a `README.md` and its pages; keep images in its own `images/` folder and link them relatively (`images/x.png`).
2. Add the pages to `docs/_sidebar.md`.
3. Add a row to the table above.
