# Nexus Studio: public assets

This repository is the public face of Nexus Studio. It carries **output only**:
compiled installers, the announcements file, and public media. **No source code
of Nexus Studio is ever pushed here.** The private repository
`evolution-foundation/nexus-studio` remains the only home for code, and its CI
is what publishes releases here.

This file is written to be pushed as the public repository's `README.md`. It
lives in the private repository at `apps/studio/distribution/`, which is where
it is edited.

## Layout, by purpose

| where | what | who writes it |
|---|---|---|
| GitHub **Releases** | installers, `latest.json`, `.sig` files | CI only, from a `v*` tag |
| `announcements/` | `announcements-v1.json`, read by the app | a person, by hand |
| `media/` | images and other public files | a person, by hand |

## The one rule that must not be broken

**`latest.json` lives as a release asset, never as a file in the tree.**

Every installed copy of Nexus Studio reads `latest.json` to decide whether to
update. This repository also receives images and hand-edited JSON, so if the
update channel read a path in the tree, one careless push could break updating
for everyone who has the app. Reading it from the latest release's assets means
only **cutting a release** can move the update channel, and pushing a file never
can.

The app reads:

```
https://github.com/evolution-foundation/nexus-studio-assets/releases/latest/download/latest.json
```

## Announcements

`announcements/announcements-v1.json` is the one file meant to be edited by
hand: publishing an announcement is a commit, with no server anywhere. The app
fetches it at most once a day and works from cache when offline.

The app treats this file as **untrusted input**, because it is remote content
that changes what the app shows. Plain text only, no HTML and no markdown; a
link is accepted only if it parses as `https`. Anything malformed is dropped
rather than rendered, per announcement, so one bad entry does not cost the
others.

Each entry takes:

| field | required | notes |
|---|---|---|
| `id` | yes | permanent, lowercase, 3 to 64 chars of `a-z0-9-`. Changing it re-shows the announcement |
| `title` | yes | plain text, up to 80 characters |
| `body` | yes | plain text, up to 400 characters, one paragraph |
| `published` | yes | `YYYY-MM-DD`. A future date hides it until then |
| `priority` | yes | integer 0 to 100. Highest surfaces first |
| `expires` | no | `YYYY-MM-DD`. After this day it disappears |
| `minVersion`, `maxVersion` | no | `x.y.z` or `x.y.z-N` |
| `platforms` | no | any of `macos`, `windows`, `linux`. Omit for everyone |
| `link` | no | must be `https`. Anything else drops the whole entry |

Only the single most important unread announcement surfaces on its own, once.
The rest stay in the app's Announcements list.

## Downloads

Installers are attached to each release under fixed names that never carry a
version, so the landing page can link to
`releases/latest/download/<name>` and never need editing:

- `nexus-studio-macos-apple-silicon.dmg`
- `nexus-studio-macos-intel.dmg`
- `nexus-studio-windows-x64-setup.exe`
- `nexus-studio-linux-x86_64.AppImage`
- `nexus-studio-linux-amd64.deb`

The updater artifacts (`.app.tar.gz`, `.AppImage.tar.gz`, the NSIS installer)
and their `.sig` files are also attached. They are what the updater installs,
and the landing page ignores them when it counts downloads.
