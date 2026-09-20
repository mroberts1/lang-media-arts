## Language of Media Arts

Course site for VM641-02, Language of Media Arts. An Obsidian vault published
as a static site with Quartz 5.

Live at https://mroberts1.github.io/lang-media-arts/

```
./           Obsidian vault root, which is what Obsidian opens
  content/   what the site is built from. Notes and their assets go here
  .quartz/   the Quartz 5 install, hidden from Obsidian
  public/    build output, gitignored
```

Notes at the vault root are drafts and reference material. Only what is under
`content/` is published.

## Working on it

```
./dev.sh
```

Then open http://localhost:8080. `./build.sh` writes the static site to
`public/` without serving it. Both scripts work from any directory. If a port is
already taken, override it: `PORT=8081 WS_PORT=3004 ./dev.sh`.

Pushing to `main` deploys; there is nothing to build locally first.

Agent-facing notes, including the gotchas worth reading before debugging
anything, are in [AGENTS.md](AGENTS.md).

## The theme

Colours come from the letterpress (light) and cyanotype (dark) palettes of
[jzhao.xyz](https://jzhao.xyz/): navy ink and vermilion on warm paper, and
light blue and vermilion on prussian blue.

| Role      | Light     | Dark      |
| --------- | --------- | --------- |
| light     | `#f5eedd` | `#06182f` |
| lightgray | `#e3d9c0` | `#122845` |
| gray      | `#9a8e76` | `#7191b8` |
| darkgray  | `#2d4673` | `#caddf4` |
| dark      | `#16294e` | `#eef4fc` |
| secondary | `#284d78` | `#8fb9de` |
| tertiary  | `#c8482b` | `#e0552f` |

Vermilion (`tertiary`) is the accent for hovers, callout borders and
selection. Callout backgrounds use the faint vermilion `highlight` wash.

Type is Helvetica Neue throughout, matching the other course vaults and the
personal site. It is a system face rather than a webfont, so `fontOrigin` is
`local` and nothing is fetched at build time; the fallback stack for machines
without it lives in `.quartz/quartz/styles/custom.scss`.

Everything the YAML config can't express lives in that `custom.scss`: the
self-hosted font, a tighter heading scale, wrapped code blocks, a card grid for
folder listings, and an extra `> [!custom]` callout type using the mint accent.

## Updating Quartz

`.quartz/` is vendored, not a clone, so `npx quartz update` is not available.
See the deployment section of [AGENTS.md](AGENTS.md) for the upgrade procedure
and the pinned upstream commit.
