# Contributing to the PureC OS registry

This repository decides what `pkg` trusts. Every package listed here is
installed on PureC OS machines with root rights, so the rules are strict.

If you are an AI agent, also read [AGENTS.md](AGENTS.md).

## Contents

- [Ways to contribute](#ways-to-contribute)
- [Requirements for a package](#requirements-for-a-package)
- [Building a package](#building-a-package)
- [Signing](#signing)
- [How a package gets in](#how-a-package-gets-in)
- [Community repositories](#community-repositories)
- [Updating and removing packages](#updating-and-removing-packages)
- [Testing](#testing)
- [Checklist before a pull request](#checklist-before-a-pull-request)
- [Commits and pull requests](#commits-and-pull-requests)
- [Review](#review)
- [Security](#security)

## Ways to contribute

- **Propose a package.** Open an issue with the "Add or update a package"
  template. You do not need the signing key: a maintainer checks the archive,
  signs it and lists it.
- **Update a package.** Same template, with the new version and what changed.
- **Report a broken package.** Open an issue with the package name, version,
  the `pkg` command and its full output.
- **Improve the docs** (README, this file, AGENTS.md, the issue template).
  Pull requests are welcome and do not touch the signed files.

## Requirements for a package

1. **Signed.** Next to every archive lies its signature
   `NAME-VERSION-REL-x86_64.pkg.tar.gz.sig` (ECDSA P-256 over SHA-512, made by
   `mk/pkg/pkgsign.py` with the PureC OS key). `pkg` refuses unsigned packages
   with `SigLevel = Required`, the default of installed systems.
2. **Built for PureC OS.** Architecture `x86_64`, built with the PureC
   toolchain (`x86_64-elf-gcc`, PureC libc, `linker-userspace.ld`) or as a
   data-only package with arch `any`. Linux binaries do not run here.
3. **Named correctly.** `NAME-VERSION-REL-ARCH.pkg.tar.gz`, made by
   `mk/pkg/mkpkg.py`. `.PKGINFO` carries `pkgname`, `pkgver`, `pkgdesc`, `arch`
   and every `depend`; a renamed package declares `conflict` with its old name.
4. **Clean file list.** Files go under `usr/` (units under
   `usr/lib/init/system/`). A package never overwrites files of another one.
5. **Tested.** Installed with `pkg -U` on a running PureC OS before it is
   listed.

### Names and versions

- `NAME`: lowercase letters, digits and `.`, `_`, `+`, `-`. It says what the
  package is, not how it was made: no `qt-` prefix for a program that no longer
  uses Qt, no `purec-` prefix (everything here is for PureC OS).
- `VERSION`: the upstream version (`9.2`, `2.56.1`), or your own semantic
  version for PureC OS programs (`MAJOR.MINOR.PATCH`).
- `REL`: the package release, starting at `1`. Raise it when the package
  changes but the program does not (new build flags, a fixed file list, a new
  unit). Reset it to `1` with a new `VERSION`.
- `ARCH`: `x86_64` for anything with machine code, `any` for data only (fonts,
  icons, wallpapers, docs).

### Dependencies and conflicts

- Declare every package whose files you need at run time with `depend`.
  Static libraries linked into your binary are not run-time dependencies.
- When you rename a package, the new one declares `conflict` with the old name,
  and the old name is removed from the registry in the same change.
- Two packages never ship the same path. If they must, they conflict.

### Configuration files and hooks

- Files under `etc/` that users may edit are listed with `--backup`; `pkg`
  keeps the user's copy on upgrade.
- Hooks (`post-install`, `post-upgrade`, `pre-remove`, `post-remove`) are short
  shell scripts for PureC OS `sh`. They must be idempotent, must not download
  anything and must not touch files of other packages.

## Building a package

Packages are made with `mk/pkg/mkpkg.py` from the PureC OS tree:

    python3 mk/pkg/mkpkg.py \
        --name hello --version 1.0.0-1 --arch x86_64 \
        --desc "Example program" \
        --depend libgui \
        --file usr/bin/hello=build/hello \
        --tree usr/share/hello=assets/hello \
        -o out/

`--version` takes `VERSION-REL`. Other options: `--conflict NAME`,
`--backup etc/FILE`, `--link DEST=TARGET`, `--hook post-install=FILE`.

Check the result before going further:

    tar -tzf out/hello-1.0.0-1-x86_64.pkg.tar.gz       # file list, only usr/
    tar -xzOf out/hello-1.0.0-1-x86_64.pkg.tar.gz .PKGINFO

Make sure the binaries are PureC OS binaries (static, linked with
`linker-userspace.ld`), and that every symbol they need resolved at link time.

## Signing

Only maintainers hold the private key. A maintainer signs with

    python3 mk/pkg/pkgsign.py sign ~/.config/purec-pkg/signing.pem \
        out/hello-1.0.0-1-x86_64.pkg.tar.gz out/hello-1.0.0-1-x86_64.pkg.tar.gz.sig

Installed systems trust the keys listed in `keyring.conf` of this repository.
It is signed by the master keys (`/etc/pkg/keys/master/`), not by the registry
key, and holds `serial N`, `key FP HEXPOINT official` for the PureC OS key,
`key FP HEXPOINT repo=URL names=GLOBS` for community keys and
`revoke FP` for withdrawn keys. Sign it with

    python3 mk/pkg/pkgsign.py keyring-sign --key MASTER.pem keyring.conf
    python3 mk/pkg/pkgsign.py keyring-verify --masters DIR keyring.conf

and raise `serial` on every change; `pkg` refuses a lower serial. Without a
keyring (old installations) the keys in `/etc/pkg/keys/` are trusted. A
signature made with a key the keyring does not allow is refused.

`registry.conf` and `repository.conf` carry their own signature on the last
line (`#sig HEX`), over every byte above it. Any change to these files, even
whitespace, needs a new signature; `publish.py` does it.

## How a package gets in

- **From a release:** add the GitHub URL of your repository to
  `registry.conf`; its releases carry the archive and the `.sig` as assets.
  Use this for programs with their own repository and release cycle.
- **Stored here:** run, from the PureC OS tree,

      python3 mk/pkg/publish.py --registry Trusted out/NAME-VERSION-REL-x86_64.pkg.tar.gz

  It copies the archive and its signature to `bin/NAME/`, removes the older
  version, takes the description from `pkgdesc`, rewrites `repository.conf`
  and signs `repository.conf` and `registry.conf`. Use the copy in
  `mk/pkg/`, not `script/publish.py`, which is older.

A name in `repository.conf` wins over the same name in a release, so a package
must not live in both places.

## Community repositories

A third-party repository can be listed without becoming official. Its packages
are installed only by users who set `AllowCommunity = yes`, and are marked
`community` by `pkg`.

To onboard a repository, open a pull request that adds

1. the line `community https://github.com/OWNER/REPO` to `registry.conf`;
2. a `key` line to `keyring.conf`, made by the maintainer of the repository
   from the public half of their signing key:

       python3 mk/pkg/pkgsign.py keyring-entry their.pub \
           repo=https://github.com/OWNER/REPO names=tool,tool-*

3. a new `serial` (one more than the current) in `keyring.conf`.

The key is bound to that repository and to the package names matched by
`names`; packages signed by it for any other repository or name are refused.
Official names (everything in `repository.conf` and in the official
repositories) are reserved: a community package with such a name is dropped.
The maintainers sign the changed `keyring.conf` with the master key(s) and
`registry.conf` with the registry key. To drop a repository, remove its lines
and add `revoke FINGERPRINT reason` for its key.

## Updating and removing packages

**Update:** publish the new archive with `publish.py`. It deletes the old
archive and its `.sig`; check that nothing else of the old version is left in
`bin/NAME/`.

**Remove:**

1. Delete `bin/NAME/`.
2. Delete its line from `repository.conf`, or its URL from `registry.conf` if
   the whole repository goes away.
3. Re-sign both files (run `publish.py` with any current package, or sign them
   by hand with `pkgsign.py`).
4. Remove the row from the package table in README.md.

Installed systems keep a removed package until the user removes it; they only
stop getting updates.

## Testing

1. Install the archive directly on a running PureC OS:

       pkg -U /root/hello-1.0.0-1-x86_64.pkg.tar.gz

   Run the program, then `pkg -R hello` and check that nothing is left behind.
2. Test the registry itself. Serve your checkout and point a test config at it
   (see "Testing a change locally" in the README):

       pkg --config /root/pkg.conf -Sy
       pkg --config /root/pkg.conf -S hello

   `-Sy` must accept both signatures; `-S` must download, verify and install.
3. Upgrade test: with the previous version installed, `-Sy` and `-Su` must
   move to the new one and keep files listed with `--backup`.

## Checklist before a pull request

- [ ] Every archive in `bin/` has its `.sig`, and only the current version is
      there.
- [ ] `repository.conf` lists only files that exist, each with a description.
- [ ] `registry.conf` lists every repository that publishes through releases,
      and only those.
- [ ] `keyring.conf` has a higher `serial` when keys changed, and enough master `#sig` lines.
- [ ] Both `.conf` files end with a valid `#sig` line, checked with the public
      key, not only written.
- [ ] Renamed packages declare `conflict` with the old name, and the old name
      is gone from the registry.
- [ ] README.md package table matches the registry.
- [ ] The package was installed and run on PureC OS.

## Commits and pull requests

- One logical change per commit: `Add NAME VERSION`, `Update NAME to VERSION`,
  `Remove NAME`, `Docs: ...`.
- The re-signed `.conf` files go in the same commit as the change that needed
  them, so `main` never holds a file with a stale signature.
- Do not commit build directories, logs, private keys or anything except
  archives, signatures, the two `.conf` files and docs.
- Do not include hardware identifiers (MAC addresses, serial numbers) in
  commits, issues or logs you paste; mask them.
- Pull requests target `main`. Describe what changed, the version, and how you
  tested it.

## Review

A maintainer checks for every package change:

- the archive and the source it was built from;
- the file list and `.PKGINFO`;
- the signatures of the archive and of both `.conf` files;
- a test install on PureC OS.

A new URL in `registry.conf` gets the strictest review: it lets the owner of
that repository ship root code to every PureC OS machine through its releases.

## Security

If you find a package that is malicious, a signature that does not verify, or a
way to make `pkg` accept something it should not, do not open a public issue.
Contact the maintainers privately through GitHub (a private security advisory
on this repository) and wait for a fix before publishing details.
