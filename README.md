# Atomic's Official BRXRP Repo

The official plugin registry and first-party resource-plugin source for
[BRX Editor](https://brxeditor.com). This repo is specifically meant to be a safe, public place for
both loading essential packs such as Brick Rigs assets, as well as serving
as a model for others to host their own BRXRP source repositories. GitHub
content served through (`cdn.jsdelivr.net`) and
`raw.githubusercontent.com` both send `Access-Control-Allow-Origin: *`, so
plugin repos must be public repositories that support this feature, it
can't just be a link to something like a google drive folder.

This specific repo does two jobs-

## (`registry/`)

The curated lists the editor reads to decide what to show and trust. These features are only enabled on this repo, if you attempt to recreate it in your own, it will simply not work. I repeat, this is uniquely only for this repo, as it is the only default, unremovable, verified repo.

* **`verified.json`** - the list of trusted source repos. A repo is *Verified*
by being in this list. This means the repo **link** is trusted, not that
every file has been audited in real time. This is only awarded to BRXRP distributors who are known to be reputable, partake in good personal data security (to ensure a verified repo cannot fall into the hands of hackers) and personally trusted by me. Each entry is `{ owner, repo, ref, root, name }`
where `root` is the folder that contains the mod folders.
* **`featured.json`** - mods pinned to the top of the Browse tab, in order. 

Both are edited by hand. Adding a third-party repo link to `verified.json` is the
only thing that makes it Verified; everything else a user adds themselves shows
up as *User-Added*.

## (`mods/`)

`mods/` is this repo's own mods root (see its `verified.json` entry). Any repo
can be a source by following the same layout, then being added to a curated
list (or added by a user as a custom repo source).

```
mods/
  <mod-id>/
    mod.json          cross-version metadata
    <version>.brxrp   one file per released version
```

### `mod.json`

```json
{
  "id": "my-very-awesome-plugin",
  "name": "My Very Awesome Plugin",
  "author": "John Brick Rigs",
  "description": "le very funny description, ligma balls",
  "icon": null,
  "versions": [
    {
      "version": "1.0.0",
      "file": "1.0.0.brxrp",
      "brickRigs": "1.10",
      "sha256": "<hex sha-256 of the .brxrp file>"
    }
  ]
}
```

* `versions` is newest-last; the editor selects the newest by default and lets
the user pick an older one.
* `brickRigs` is the Brick Rigs version the plugin targets.
* `sha256` is verified before the plugin is registered, so a tampered or
truncated download is rejected.

### How the editor loads a repo

1. Read `verified.json` (and any user-added repos) fresh from
`raw.githubusercontent.com` (short cache, so curation updates are not stuck
behind jsDelivr's long edge cache).
2. Crawl each repo's `root` via the jsDelivr data API
(`data.jsdelivr.com/v1/packages/gh/<owner>/<repo>@<ref>`), inferring one mod
per folder, and read each `mod.json`.
3. Fetch a chosen `.brxrp` from jsDelivr (immutable, the version is in the
path, so the long CDN cache is fine), verify its `sha256`, then register it.

All `.brxrp` files are decoded by the editor's bounds-checked decoder, which
protects the end user against a malicious file, failing to load rather than harming the editor.

