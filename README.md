# PureC OS package registry

The list of packages that `pkg`, the PureC OS package manager, trusts.

Installed systems point at this repository in `/etc/pkg/pkg.conf`:

    [options]
    Registry = https://github.com/PureC-OS/PureC-OS-Trusted-PKG-URL

`pkg -Sy` downloads `registry.conf` and `repository.conf` from here, checks
their signatures, and `pkg -S NAME` installs a package only if one of them
lists it.

## Files

| File | What it holds |
|---|---|
| `registry.conf` | Official repositories, one GitHub URL per line. Packages come from the assets of their releases. |
| `repository.conf` | Packages stored in this repository: `NAME PATH DESCRIPTION`, the path is relative to the file. A name listed here wins over a release. |
| `bin/NAME/` | The archive `NAME-VERSION-REL-x86_64.pkg.tar.gz` and its signature `.sig`. |

Both `.conf` files end with `#sig HEX`, the ECDSA signature of everything above
it. The file and its signature travel together, so the five-minute cache of
raw.githubusercontent.com never pairs a new file with an old signature. A file
that does not match its signature is refused.

## Packages

| Package | Description |
|---|---|
| `kernel` | The PureC OS kernel and its modules (releases of PureC-OS/PureC-OS) |
| `pkg` | The package manager itself |
| `video-backend` | Display server `videod` and its client library |
| `libgui` | GUI toolkit on top of videod (static library and headers) |
| `userspace` | Desktop: panels, start menu, applications |
| `login` | Graphical login screen |
| `terminal` | Terminal emulator |
| `settings` | System settings |
| `vim` | Vi IMproved text editor |
| `git` | Distributed version control (https through curl and wolfSSL) |
| `wolfssl` | wolfSSL TLS library (development files) |
| `tcc` | Tiny C compiler with its runtime and headers |
| `make` | GNU make |
| `lua` | Lua scripting language |
| `micropython` | Python for small systems |

## Testing a change locally

Serve this folder and point a test config at it:

    python3 -m http.server 8099 --bind HOST_IP        # on the build machine

    [options]                                         # /root/pkg.conf on PureC OS
    Architecture = x86_64
    SigLevel = Required
    Registry = http://HOST_IP:8099/registry.conf

    pkg --config /root/pkg.conf -Sy

## Contributing

Packages must be signed and built for PureC OS x86_64. See
[CONTRIBUTING.md](CONTRIBUTING.md) and the issue template. AI agents follow
[AGENTS.md](AGENTS.md) (Claude Code also reads [CLAUDE.md](CLAUDE.md)).

## License

See [LICENSE](LICENSE).
