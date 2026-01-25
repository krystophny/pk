# pk - Minimal POSIX Package Manager

A minimal package manager written in POSIX shell. Zero dependencies beyond standard Unix tools.

## Features

- **Install/remove** packages from `.pk.tar.xz` archives
- **Build** packages from source using `packages.toml` definitions
- **Download** source tarballs with variable expansion
- **Track** all installed files for clean removal
- **Check** for untracked system files
- Git-like modular architecture (subcommands in `pk.d/`)

## Usage

```bash
pk i <pkg.tar.xz>    # Install package
pk r <pkgname>       # Remove package
pk l                 # List installed packages
pk q <pkgname>       # Query package info
pk f <pkgname>       # List package files
pk b <pkgname>       # Build package from packages.toml
pk d <pkgname>       # Download package source
pk c [dir...]        # Check for untracked files
pk v                 # Show version
```

## Installation

```bash
# Install pk using pk (bootstrap)
tar cJf /tmp/pk-0.0.2.pk.tar.xz -C /path/to/pk ./pk ./pk.d
./pk i /tmp/pk-0.0.2.pk.tar.xz

# Or manual install
cp pk /usr/bin/
cp -r pk.d /usr/lib/pk/
```

## Package Format

Packages are simple `.pk.tar.xz` archives containing files relative to root:
```
./usr/bin/foo
./usr/lib/libfoo.so
./usr/share/man/man1/foo.1
```

## Database

Package metadata stored in `/var/lib/pk/<pkgname>/`:
- `info` - name and version
- `files` - list of installed files

## Environment Variables

- `PK_ROOT` - Install prefix (for chroot builds)

## Build System (packages.toml)

pk can build packages from TOML definitions:
```toml
[packages.example]
version = "1.0.0"
url = "https://example.com/pkg-${version}.tar.xz"
build_system = "autotools"  # or "meson", "cmake", "make"
configure_flags = "--enable-foo"
```

See [lfs-infra](https://github.com/krystophny/lfs-infra) for a complete example.

## License

MIT
