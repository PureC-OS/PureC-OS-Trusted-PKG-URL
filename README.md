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
