# Instructions for AI agents

This file is for coding agents (Claude Code, Codex, Copilot and others) that
work in this repository. Humans should read [CONTRIBUTING.md](CONTRIBUTING.md)
first; everything there applies to agents too.

## What this repository is

The trust root of `pkg`, the PureC OS package manager. Every installed PureC OS
system downloads `registry.conf` and `repository.conf` from here, checks their
signatures and installs, with root rights, only what they list. A mistake here
breaks or compromises every machine that runs `pkg -Sy`. Treat each change as a
security change.

There is no source code to build here. The packages are built in the PureC OS
tree (`mk/pkg/`) and in the repositories listed in `registry.conf`.

## Layout

| Path | Meaning |
|---|---|
| `registry.conf` | GitHub URLs of official repositories, one per line; `pkg` takes packages from the assets of their releases |
| `repository.conf` | `NAME PATH DESCRIPTION` for packages stored here; a name listed here wins over a release |
| `bin/NAME/` | Exactly one archive `NAME-VERSION-REL-ARCH.pkg.tar.gz` and its `.sig` |
| `script/publish.py` | Older copy of `mk/pkg/publish.py`; always run the one in the PureC OS tree, it is newer and finds `pkgsign.py` next to itself |
| `.github/ISSUE_TEMPLATE/` | Template for package requests; keep its checklist in sync with CONTRIBUTING.md |

## Hard rules

1. **Never edit a signed file by hand after signing.** `registry.conf` and
   `repository.conf` end with `#sig HEX`, an ECDSA P-256 signature over SHA-512
   of everything above that line. Changing a single byte, a trailing space or a
   line ending makes `pkg` refuse the whole registry. Change them only through
   `mk/pkg/publish.py`, or edit and then re-sign.
2. **Never sign anything yourself unless the user asks you to and the key is
   present.** The private key lives only on the maintainer machine
   (`~/.config/purec-pkg/signing.pem`). Never print it, copy it, commit it or
   send it anywhere. If it is missing, stop and say so; do not generate a new
   key, that would lock out every installed system.
3. **Never add a package without its `.sig`.** Installed systems use
   `SigLevel = Required`.
4. **Never add a URL to `registry.conf` that the user did not name.** Each URL
   grants that repository's release owner root on every machine.
5. **No Linux binaries.** Packages are built with the PureC toolchain
   (`x86_64-elf-gcc`, PureC libc, `linker-userspace.ld`) or are data-only with
   arch `any`.
6. **Do not commit or push unless the user asks.** Do not force-push `main`.
7. **No hardware identifiers** (MAC addresses, serial numbers) in commits,
   descriptions or docs.

## Publishing a package

From the root of the PureC OS tree, with the registry checked out as `Trusted/`:

    python3 mk/pkg/publish.py --registry Trusted path/to/NAME-VERSION-REL-x86_64.pkg.tar.gz

The script copies the archive and its signature to `bin/NAME/`, deletes the
older version, rewrites `repository.conf` (keeping existing descriptions) and
re-signs both `.conf` files. The description of a package is taken from
`pkgdesc` in its `.PKGINFO`.

Before you report the work as done, go through the whole checklist, not just
"the archive is there":

1. **Old versions removed.** `bin/` holds one archive per package, every archive
   has a `.sig`, and nothing in `bin/` is missing from `repository.conf`.
2. **Removed packages removed everywhere:** folder in `bin/`, line in
   `repository.conf`, row in the README table.
3. **Descriptions.** Every line in `repository.conf` has a description (from
   `pkgdesc` in `.PKGINFO`). If `pkgdesc` is empty, fix the package, not the
   file.
4. **registry.conf.** A package published through releases needs the GitHub URL
   of its repository here; a repository that no longer publishes is removed.
5. **Names.** No stale prefixes (for example `qt-` for something that is no
   longer Qt). A renamed package declares `conflict` with its old name, and
   every reference is updated: units, `.desktop` files, README, the image
   manifest in the PureC OS tree.
6. **Signatures verified**, not just written (see below).
7. **README** package table matches `repository.conf` and `registry.conf`.

## Checking signatures

Installed systems trust the public keys in `/etc/pkg/keys/`; the official one
is `rootfs/etc/pkg/keys/purec.pub` in the PureC OS tree (an uncompressed P-256
point in hex). `pkg` checks signatures in `programs/purec-os-pkg/src/sig.c`. To check on the host, verify the bytes above
the `#sig` line, and each archive against its `.sig`, with the public key, for
example with Python `cryptography`:

    ec.EllipticCurvePublicKey.from_encoded_point(ec.SECP256R1(), point)
        .verify(sig, sha512(data).digest(), ec.ECDSA(utils.Prehashed(hashes.SHA512())))

For a `.conf` file, `data` is everything before the `#sig ` line (including the
final newline) and `sig` is `bytes.fromhex` of the digits after it.

A real end-to-end check is `pkg -Sy` on a running PureC OS against a local copy
(see "Testing a change locally" in the README).

## Style of changes

- One logical change per commit: "Add NAME VERSION", "Update NAME to VERSION",
  "Remove NAME". The signature refresh of the `.conf` files goes in the same
  commit as the change that needed it.
- Docs are in English, short and factual.
- Large archives are fine in `bin/`, but do not add build directories, logs,
  caches or anything other than archives and signatures.
