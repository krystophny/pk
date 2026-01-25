# Integrating with pk

This document describes how to integrate pk with build systems like lfs-infra.

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PK_ROOT` | `/` | Installation root. Use `/mnt/lfs/` for chroot, empty for local/safe mode |
| `PK_PACKAGES_DIR` | (search) | Override packages directory location |
| `PK_FORCE_ROOT` | unset | Set to `1` to allow installing to host root |

## Directory Paths

When `PK_ROOT=/`:
- `DB_DIR=/var/lib/pk` - Package database
- `CACHE_DIR=/var/cache/pk` - Built packages
- `SOURCES_DIR=/usr/src` - Source tarballs

When `PK_ROOT=/mnt/lfs/`:
- All paths are prefixed with `/mnt/lfs/`

## Package Directory Search Order

pk searches for package definitions in this order:
1. `$PK_PACKAGES_DIR` (if set)
2. `./packages` (relative to current dir)
3. `$SCRIPT_DIR/../packages` (relative to pk script)
4. `/etc/pk/packages` (system install)
5. `/usr/share/pk/packages` (system install)

## System Installation Layout

When pk is installed system-wide:
```
/usr/bin/pk                    # Main script
/usr/lib/pk/                   # Subcommands (pk-build, pk-install, etc.)
/etc/pk/packages/*.toml        # Package definitions
/etc/pk/build.conf             # Optimization flags
/var/lib/pk/<pkgname>/         # Package database
/var/cache/pk/                 # Built .pk.tar.xz packages
```

## Using pk from Another Project

### Source pk-common for Parsing

To use pk's package parsing functions in your scripts:

```bash
#!/bin/bash
# Source pk-common from pk repo or system install
PK_COMMON="${PK_COMMON:-../pk/pk.d/pk-common}"
[ -f "$PK_COMMON" ] || PK_COMMON="/usr/lib/pk/pk-common"
. "$PK_COMMON"

# Now you can use:
version=$(get_pkg_version "gcc")
url=$(get_pkg_url "gcc")
stage=$(get_pkg_stage "gcc")
depends=$(get_pkg_depends "gcc")
```

### Available Functions (from pk-common)

**Package info:**
- `get_pkg_version <pkg>` - Package version
- `get_pkg_url <pkg>` - Download URL
- `get_pkg_git_url <pkg>` - Git repository URL
- `get_pkg_stage <pkg>` - Build stage number
- `get_pkg_build_order <pkg>` - Priority within stage
- `get_pkg_build_system <pkg>` - autotools, meson, cmake, make, or custom
- `get_pkg_source_pkg <pkg>` - Source package (for split packages)
- `get_pkg_source_dir <pkg>` - Override source directory name
- `get_pkg_configure_flags <pkg>` - Autotools configure flags
- `get_pkg_meson_flags <pkg>` - Meson flags
- `get_pkg_cmake_flags <pkg>` - CMake flags
- `get_pkg_depends <pkg>` - Dependencies (array)
- `get_pkg_build_commands <pkg>` - Custom build commands (array)
- `get_pkg_provides <pkg>` - Provided files (array)

**Low-level:**
- `extract_field <pkg> <field>` - Extract any field
- `extract_array <pkg> <field>` - Extract array field
- `cat_packages` - Output all package definitions
- `find_packages_dir` - Set PACKAGES_DIR variable

### Call pk Directly

For build operations, call pk commands:

```bash
# Download and build
pk d gcc        # Download source
pk b gcc        # Build package

# Install
pk i /var/cache/pk/gcc-14.2.0.pk.tar.xz

# Build entire stage
pk bs 4         # Build all stage 4 packages
```

## Integration Example (lfs-infra)

```bash
#!/bin/bash
# Find pk repo (adjacent directory)
PK_REPO="${SCRIPT_DIR}/../pk"

# Set packages directory for scripts
export PK_PACKAGES_DIR="${PK_REPO}/packages"

# Use pk for installation
PK_ROOT="${LFS}" "${PK_REPO}/pk" i "$pkg_file"
```

## Build Configuration

The `build.conf` file provides optimization flags:

```bash
# Source build.conf
. /etc/pk/build.conf

# Available flag sets:
# CFLAGS_PERF   - Aggressive optimizations (fast-math, LTO)
# CFLAGS_SAFE   - Safe optimizations (no fast-math)
# CFLAGS_BOOTSTRAP - Conservative for cross-compilation

# Helper functions:
export_perf_flags      # Set aggressive flags
export_safe_flags      # Set safe flags
export_bootstrap_flags # Set conservative flags
```
