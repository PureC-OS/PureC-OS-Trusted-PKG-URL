# Contributing to the PureC OS registry

This repository decides what `pkg` trusts. Every package listed here is
installed on PureC OS machines with root rights, so the rules are strict.

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

## How a package gets in

- **From a release:** add the GitHub URL of your repository to
  `registry.conf`; its releases carry the archive and the `.sig` as assets.
- **Stored here:** run, from the PureC OS tree,

      python3 mk/pkg/publish.py --registry Trusted bin/NAME/NAME-VERSION-REL-x86_64.pkg.tar.gz

  It copies the archive and its signature to `bin/NAME/`, removes the older
  version, rewrites `repository.conf` and signs `repository.conf` and
  `registry.conf`.

Before opening a pull request:

- old versions are gone from `bin/`, and `repository.conf` lists only files
  that exist, each with its `.sig`;
- every line of `repository.conf` has a description;
- `registry.conf` and `repository.conf` end with a valid `#sig` line (edit
  them by hand only before signing; any later change breaks the signature).

Open an issue with the "Add or update a package" template, or a pull request
that follows the same checklist.
