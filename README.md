# PureC-OS-Trusted-PKG-URL

The package registry of PureC OS. `registry.conf` lists the official
repositories, one GitHub URL per line; `pkg -Sy` downloads it
(`Registry = https://github.com/PureC-OS/PureC-OS-Trusted-PKG-URL` in
`/etc/pkg/pkg.conf`) and trusts only the packages from the releases of these
repositories.

To add a repository: add its URL here and commit. Its releases must carry
assets named `NAME-VERSION-REL-x86_64.pkg.tar.gz` (one repository may ship
several packages) and, when packages are signed, `….pkg.tar.gz.sig`.

Optional: sign the list itself so that a changed file is refused
(`registry.conf.sig` next to it is checked when present):

    python3 mk/pkg/pkgsign.py sign ~/.config/purec-pkg/signing.pem registry.conf registry.conf.sig

## repository.conf and bin/

Besides the releases of other repositories this repository can hold packages
itself. `repository.conf` lists them, one per line: the package name, the path
of its archive relative to the file and an optional description.

    tpkg bin/tpkg/tpkg-1.2-1-x86_64.pkg.tar.gz Test package

Every package lives in its own folder with two files, the archive and its
signature:

    bin/NAME/NAME-VERSION-REL-x86_64.pkg.tar.gz
    bin/NAME/NAME-VERSION-REL-x86_64.pkg.tar.gz.sig

`pkg -Sy` downloads `repository.conf` (and `repository.conf.sig` when it
exists), `pkg -S NAME` downloads the archive and the `.sig` from the listed
path and checks the signature with the keys from `/etc/pkg/keys`. A name that
appears both here and in a release of a repository from `registry.conf` is
taken from here. Without `repository.conf` nothing changes.

To publish packages from the PureC OS tree:

    python3 mk/pkg/publish.py --registry Trusted bin/repo/NAME-VERSION-REL-x86_64.pkg.tar.gz

It copies the archive and its `.sig` into `bin/NAME/`, replaces the older
version in that folder and rewrites the line in `repository.conf`
(`--keep N` keeps N versions). Commit and push `Trusted` afterwards.
