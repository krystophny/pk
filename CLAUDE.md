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

## Multilib Bootstrap (32-bit support)

Building GCC with `--enable-multilib` requires 32-bit libc to be present first. Bootstrap procedure:

1. **Download 32-bit libc from Debian:**
   ```sh
   curl -LO "http://deb.debian.org/debian/pool/main/g/glibc/libc6_2.42-11_i386.deb"
   curl -LO "http://deb.debian.org/debian/pool/main/g/glibc/libc6-dev_2.42-11_i386.deb"
   ```

2. **Extract and install to /usr/lib32:**
   ```sh
   mkdir extract && cd extract && ar x ../libc6*.deb && tar xf data.tar.xz
   sudo cp -a usr/lib/i386-linux-gnu/* /usr/lib32/
   sudo ln -sf /usr/lib32/ld-linux.so.2 /lib/ld-linux.so.2
   sudo ln -sf libc.so.6 /usr/lib32/libc.so
   ```

3. **Also create /lib/cpp symlink:**
   ```sh
   sudo ln -sf /usr/bin/cpp /lib/cpp
   ```

4. **Then build GCC with multilib.**

After GCC multilib is built, rebuild lib32-glibc from source to replace the Debian bootstrap.
