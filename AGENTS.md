# AGENTS.md

Technical notes for agents working on this Obsidian vault, which is published as
a static site with Quartz 5. Human-facing orientation lives in `README.md`.

Most of this file was carried over from the sibling vault
`~/Obsidian/marlboro-digital-culture`, which is built from the same Quartz
template. Entries verified against this vault are marked; the rest are
structural to the template and expected to hold, but were not re-tested here.

## Layout

```
./              Obsidian vault root (this is what Obsidian actually opens)
  content/      what Quartz builds from. Notes and their assets go here
  .quartz/      the Quartz 5 install, hidden from Obsidian
  public/       build output, gitignored, safe to delete
```

This vault is a fresh copy of the template and is barely populated. `content/`
holds only `index.md` and `img/`. The syllabus and topic markdown files sit at
the vault root, outside `content/`, so they are not part of the site. Moving
one into `content/` is what publishes it.

## Running it

Use the scripts, never `npx quartz`. The repo's own bin is not linked into
`node_modules/.bin`, so npx falls through to an unrelated `quartz` package on
the registry (v0.0.1, a transmission-daemon client).

| Task            | Command                           |
| --------------- | --------------------------------- |
| Serve with HMR  | `./dev.sh` (8080, ws 3003)        |
| Build to public | `./build.sh`                      |
| Override ports  | `PORT=8081 WS_PORT=3004 ./dev.sh` |

Both scripts work from any directory.

`.quartz/.node-version` pins `v22.16.0`, which nodenv does not have installed
(verified here). `./dev.sh` then dies immediately with
`nodenv: version 'v22.16.0' is not installed` and never reaches Quartz. Work
around it per-run with `NODENV_VERSION=22.22.2 ./dev.sh`. The permanent fixes
are `nodenv install 22.16.0` or editing the pin, but that file belongs to the
vendored Quartz tree, so prefer the env var unless the user wants it changed.
Quartz only requires node >= 22.

## Gotchas that cost real time

Check these before diagnosing anything else. Each records the symptom as well
as the cause, because most of them presented as a different problem.

Everything the site serves must live under `content/`. Quartz only globs the
directory passed via `-d`. A file above it is never copied to `public/`, and a
`../` path does not escape: Quartz normalises it away, so `../img/x.jpg` emits
as `src="././img/x.jpg"` and 404s. Subdirectories at any depth are fine, and
the folder name does not matter. `content/img/` is the convention here.

Adding a new asset file needs a server restart. The incremental rebuild picks
up markdown edits but does not copy newly added non-markdown files. The symptom
is an `<img>` that renders while the file itself 404s. Restart `./dev.sh` and
it is copied. Editing markdown afterwards hot reloads normally.

A YAML frontmatter error kills the whole dev server, not just the page. It
prints `Failed to process markdown` and the process exits, so the port goes
dead. Body-text errors do not do this, only frontmatter. Before investigating
hot reload, check the server is still alive:
`lsof -nP -iTCP:8080 -sTCP:LISTEN`. The recurring trigger is an unquoted colon
in a title, which YAML reads as a nested mapping. Quote it:
`title: "VM641-02: Language of the Media Arts"`.

The hot-reload client never reconnects. Quartz emits
`new WebSocket("ws://localhost:3003").addEventListener("message", () => location.reload(true))`
with no `onclose` handler. Any tab open across a server restart or crash holds a
dead socket forever, rebuilds correctly server-side, and never refreshes. Always
hard-reload the page after restarting the server.

Deleting a file while the server runs can poison the rebuild. The pending delete
is retried on every subsequent rebuild and fails with
`ENOENT: no such file or directory, unlink '../public/...'`, which blocks all
further rebuilds. Restart to clear it. Relevant when cleaning up scratch files.

Never run `build.sh` while `dev.sh` is running. Both write to `public/` and
`build.sh` cleans it first, which deletes the files the dev server is serving
and poisons its rebuild loop. The symptom is a server that still answers 200
with stale content but silently stops picking up edits. Restart to clear it.

Obsidian's vault root is the project root, not `content/`. The template README
says to point Obsidian at `content/`, but `.obsidian/` sits at the root here,
so that is not what happened. Consequence: in the sidebar `content` is just one
folder among others, and anything created at the top level lands beside it
rather than inside it, where Quartz cannot see it. `.obsidian/app.json` sets
`newFileFolderPath: wiki/concepts`, a path that does not exist under `content/`,
and has no `attachmentFolderPath`, so pasted images default to the vault root
and will 404. Setting it to `content/img` fixes that at the source.

## Authoring

Images. Standard markdown works but has no width control. Use the wikilink form
to size, where the number is pixels and height stays auto:

```markdown
![[img/photo.jpg|500]]
![[img/photo.jpg|500x300]]
```

Callouts. All 13 Obsidian types render, plus a 14th `custom` defined by this
theme in `custom.scss`. A blank line ends a callout, there is no closing marker;
use a bare `>` for a blank line inside one. Append `-` to start collapsed, `+`
for expanded but foldable.

```markdown
> [!custom] An explicit title
> Body.
>
> Second paragraph, same box.
```

Always give `[!custom]` a title. With none, Quartz falls back to the capitalised
type name and the header reads "Custom". An empty title and `&nbsp;` both fall
back too; only a literal zero-width space renders blank, which is not worth the
invisible character. To drop the header entirely, hide
`.callout[data-callout="custom"] > .callout-title` in `custom.scss`, accepting
that it removes the icon and titles from every custom callout.

YouTube. An image embed pointed at a watch URL becomes an iframe:

```markdown
![](https://www.youtube.com/watch?v=VIDEO_ID)
```

The iframe is a fixed `width="600px"` with no aspect-ratio rule, so it does not
scale on narrow screens. No responsive CSS exists for it yet in any vault here.

## Configuration

Prefer `.quartz/quartz.config.yaml` over patching vendored Quartz source, since
config survives an upgrade. Sidebar component labels take a `title` option:

```yaml
  - source: "@quartz-community/explorer"
    enabled: true
    options:
      title: Pages
```

Note that the explorer's mobile toggle `aria-label` is hardcoded in the compiled
plugin under `node_modules` and is not reachable from config or from the i18n
strings in `.quartz/quartz/i18n/locales/en-US.ts`.

Everything the YAML cannot express lives in `.quartz/quartz/styles/custom.scss`:
the self-hosted font, a tighter heading scale, wrapped code blocks, a card grid
for folder listings, and the `[!custom]` callout.

Departure Mono is not on Google Fonts. Every build logs a failed fetch for it
(here: `Failed to fetch font Departure Mono with weight 700, got Bad Request`).
This is expected and harmless; the `@font-face` in `custom.scss` is what
actually loads it. Pinning `weights: [400]` in the config stops the request
asking for a weight that does not exist anywhere, but does not silence the
warning.

`baseUrl` is `mroberts1.github.io/lang-media-arts`, including the subpath,
because Pages serves this repo under a path rather than at a domain root.
Dropping the subpath breaks every generated link.

The `@quartz-community/cname` plugin is disabled. It writes `public/CNAME` from
`baseUrl`, and any CNAME file makes Pages try to serve a custom domain, which
fails on a `github.io` subpath. Re-enable it only alongside a real domain.

Local deviations from the stock template, all in `quartz.config.yaml`:
`page-title` and `graph` are disabled, the explorer is titled `Pages`,
`table-of-contents` is capped at `maxDepth: 2` (h1 and h2 only), and
`content-meta` has `showReadingTime: false`. The site title lives in
`configuration.pageTitle`, not in the disabled `page-title` component, and it
still reaches the page through the `og:site_name` meta tag.

## Deployment

The site publishes to GitHub Pages from `main` via `.github/workflows/deploy.yml`:
`npm ci` in `.quartz/`, then the same build command as `build.sh`, then
`upload-pages-artifact` on `public/`. Pushing to `main` is the whole deploy
step; there is nothing to run locally first.

Because a push is a deploy, do not commit or push while iterating. Work
locally against `./dev.sh` and let the user decide when to publish.

Live at https://mroberts1.github.io/lang-media-arts/

CI resolves node from `.quartz/.node-version` via `setup-node`, which installs
`v22.16.0` on demand. Only local nodenv lacks that version, so the workaround
above is a local concern and must not be "fixed" by editing the pin, which
would change what CI builds with.

`.quartz/` is vendored, not a git clone. Its own `.git` was removed so the
outer repo could track the files, so `git pull` from upstream Quartz is not
available. It sits at `jackyzha0/quartz` branch `v5`, commit
`754058ad54665fe80de8bcaaaa1626e1d0fcf118`. To upgrade, clone that repo fresh
and reapply the local changes, which are `quartz.config.yaml`, `.node-version`,
`quartz/styles/custom.scss`, and `quartz/static/fonts/`.

`.obsidian/` is gitignored. Plugin `data.json` files hold live credentials, and
one plugin ships a 59MB binary. The site build never reads it. `.smart-env/`
(Smart Connections embeddings) is gitignored for the same reason.

## Keeping this file current

Update this file when a change invalidates something above, or when a new
non-obvious behaviour costs time to diagnose. Record the symptom alongside the
cause.

## Plugins installed from git

`quartz-image-zoom` (lightbox on click) is installed from
`github:vazome/quartz-image-zoom`, pinned in `.quartz/quartz.lock.json`.

Install with the bootstrap CLI, not npx:
`node ./quartz/bootstrap-cli.mjs plugin add github:<owner>/<repo>` from
`.quartz/`.

Git plugins install to `.quartz/.quartz/plugins/`, not `.quartz/plugins/`. The
loader joins `process.cwd()` with `.quartz/plugins` and the CLI already runs
from `.quartz/`, so the path doubles up. That directory is gitignored as a
cache, and `bootstrap-cli.mjs build` does not fetch missing plugins, so CI runs
`npm run install-plugins` before building. Dropping that step yields a green
build with the plugin silently missing.
