# Evolution Data Server on Windows

This branch (`windows-native-port`) contains the Evolution Data Server changes
needed by the native Windows GNOME Calendar port. It is based on the upstream
`master` commit used by the port and is intended to be built with MSYS2 UCRT64
and MinGW-w64.

The companion GNOME Calendar branch and the full build/packaging guide live at:

- <https://github.com/vasil1729/gnome-calendar/tree/windows-native-port>
- <https://github.com/vasil1729/gnome-calendar/blob/windows-native-port/WINDOWS.md>

## What changed

The branch fixes the parts of EDS that the calendar client and its factories
need on Windows:

- **File metadata:** use `GStatBuf` where GLib's `g_stat()` writes 64-bit
  `struct _stat64` data, and keep the plain `struct stat` for `fstat()` calls.
  This avoids writing past the end of the buffer on MinGW.
- **Networking:** include `winsock2.h` before NSPR, and link Camel against
  `ws2_32`.
- **Regular expressions:** link against MSYS2's `libsystre`/`libtre` for the
  POSIX `regcomp()` family used by Camel's search code.
- **Time functions:** provide Windows fallbacks for `localtime_r()` where the
  C runtime does not have it.
- **Build system:** use forward-slash native paths in the iconv and OAuth helper
  checks, and avoid `file(TO_NATIVE_PATH)` escaping problems on Windows.
- **UUID:** the build uses a small `libuuid` compatibility shim backed by
  Windows `rpcrt4.dll`; see the GNOME Calendar repository's
  `windows/uuid-shim/`.
- **Camel and library portability:** `guint`/`GStatBuf` cleanups, missing GLib
  includes, and smaller Windows guards in the calendar, addressbook, and Camel
  code paths.

The full patch is mirrored in the GNOME Calendar repository at
`windows/patches/evolution-data-server-windows.patch`.

## Building

The recommended path is the GNOME Calendar repository's build script, which
installs the UUID shim, configures EDS with the required Windows options, and
installs it into `/ucrt64`:

```bash
git clone -b windows-native-port https://github.com/vasil1729/evolution-data-server.git
git clone -b windows-native-port https://github.com/vasil1729/gnome-calendar.git
cd gnome-calendar
bash windows/setup-msys2.sh
bash windows/build-eds.sh
```

The important CMake options are:

```text
-DENABLE_GTK=OFF
-DENABLE_GTK4=ON
-DENABLE_GOA=OFF
-DENABLE_OAUTH2_WEBKITGTK=OFF
-DENABLE_OAUTH2_WEBKITGTK4=OFF
-DENABLE_IPV6=OFF
-DWITH_LIBDB=OFF
-DWITH_OPENLDAP=OFF
-DWITH_SUNLDAP=OFF
-DWITH_KRB5=OFF
-DSOCKET_LDFLAGS=ws2_32
-DREGEX_LDFLAGS=systre
```

## Scope

The target is the calendar/addressbook data server used by GNOME Calendar. The
mail-specific Camel code is patched enough to build, but this branch is not a
general-purpose Windows Evolution mail port. GNOME Online Accounts, WebKit-based
OAuth prompters, and GeoClue are out of scope for the native calendar bundle.

## License

Evolution Data Server is licensed under the GNU Lesser General Public License,
version 2.1 or later, with some parts under the GNU General Public License. The
Windows port changes keep the same licensing as the files they modify.
