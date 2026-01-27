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

## Multilib Build Order (32-bit support)

Building 32-bit support requires this order:

1. **lib32-glibc** (stage 3, build_order 26) - Built using `gcc -m32` from existing 64-bit GCC. glibc's configure detects cross-compilation and doesn't run test programs.

2. **gcc** (stage 3, build_order 27) - Rebuilt with `--enable-multilib` after lib32-glibc provides the 32-bit runtime.

3. **Other lib32-* packages** (stage 13) - Built after multilib GCC is available.
