# jellyfin_am

A personal [AM (AppMan)](https://github.com/ivan-hc/AM)-compatible database
for installing and auto-updating the **Jellyfin Desktop nightly AppImage**
through `am`.

The upstream `jellyfin-desktop` project (https://github.com/jellyfin/jellyfin-desktop)
has no first-class releases — its only Linux build channel is the
[`nightly.link`](https://nightly.link/jellyfin/jellyfin-desktop/workflows/build-linux-appimage/main/linux-appimage-x86_64.zip)
zip, regenerated on every push to `main`. `am` can't consume raw `.zip`
files, but it *can* read a third-party database via `am newrepo`. This
repo is that database.

## Repo layout

```
programs/
  x86_64/
    jellyfin-desktop-nightly   # install + AM-updater script
  x86_64-apps                  # one-line entry per app (◆ name : description)
```

The layout is deliberately minimal so you can drop in more apps later
(just add another `programs/x86_64/<appname>` script + a `◆` line in
`x86_64-apps`).

## Usage

### 1. Install `am` first

```sh
wget -q https://raw.githubusercontent.com/ivan-hc/AM/main/AM-INSTALLER && \
chmod a+x ./AM-INSTALLER && ./AM-INSTALLER && rm ./AM-INSTALLER
```

Pick option **1** (system-wide `am`) or option **2** (rootless
`appman`).

### 2. Point `am` at this database

After pushing this repo to GitHub (it lives at
`https://github.com/aypro/jellyfin_am` by default), tell `am` to use
it:

```sh
am newrepo add https://raw.githubusercontent.com/aypro/jellyfin_am/master
am newrepo on
am newrepo info   # confirm the source switched
```

(Fork it first if you want to maintain your own additions — `newrepo`
just needs any reachable raw URL serving `programs/x86_64/<appname>`
and `programs/x86_64-apps`.)

### 3. Install

```sh
am install jellyfin-desktop-nightly
# or, with AppMan, rootless:
am -i --user jellyfin-desktop-nightly
```

### 4. Update

```sh
am update                       # update every managed app
am update jellyfin-desktop-nightly
```

The `AM-updater` script re-scrapes the latest workflow run on the
[jellyfin-desktop Actions page](https://github.com/jellyfin/jellyfin-desktop/actions/workflows/build-linux-appimage.yml)
and only re-downloads when the commit SHA changes, so updates are
silent when there's nothing new.

## How the version is detected

The nightly zip is named
`JellyfinDesktop-<version>-dev+<short-sha>-x86_64.AppImage`. The
AM-updater walks the Actions workflow history for
`build-linux-appimage.yml`, grabs the most recent commit SHA from the
run header, and compares it to the SHA stored in
`/opt/jellyfin-desktop-nightly/version`. No GitHub API key needed for
this scrape — it just reads the HTML of the actions list page.

## Adding more apps later

1. Drop a new install script at `programs/x86_64/<appname>` (use the
   existing one or the AM
   [template guide](https://github.com/ivan-hc/AM/blob/main/docs/guides-and-tutorials/template.md)
   as a reference).
2. Append a line to `programs/x86_64-apps`:
   `◆ <appname> : <one-sentence description>.`
3. Commit + push. `am` will pick it up on the next `am sync` / install.

## Why this exists

- `nightly.link` is a flat zip, not a tagged release — `am` has no
  built-in way to track it.
- The AUR `jellyfin-desktop-git` package exists, but AUR helpers
  aren't useful on non-Arch systems.
- This repo lets the same `am` workflow that handles the rest of your
  AppImages also handle Jellyfin, with the same `am update` /
  `am -r` / `am --launcher` semantics as everything else.