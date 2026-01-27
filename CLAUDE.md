# pk - Custom Linux Package Manager

## Build Commands in TOML Files

When writing build commands that generate multi-line files, **never use `printf` with `\n` escape sequences**. TOML string escaping causes `\\n` to become literal `\n` text instead of newlines.

**Wrong:**
```toml
build_commands = [
    "printf '#!/bin/sh\\nexec foo\\n' > ${PKG}/usr/bin/script"
]
```

**Correct:**
```toml
build_commands = [
    "echo '#!/bin/sh' > ${PKG}/usr/bin/script",
    "echo 'exec foo' >> ${PKG}/usr/bin/script"
]
```

Use multiple `echo` commands with `>` (first line) and `>>` (subsequent lines) instead.
