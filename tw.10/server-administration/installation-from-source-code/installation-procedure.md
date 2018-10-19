---
description: 版木：11
---

# 16.4. 安裝流程

## 1. **組態設定**

安裝過程的第一步是設定原始碼編譯時所需的選項。這是透過執行 configure 腳本完成的。對於預設安裝而言，只需輸入：

```text
./configure
```

此腳本將執行許多測試以確定各種系統相關變數的值，並檢測操作業系統的所有特性，最後將在編譯樹中建立多個檔案以記錄它找到的內容。如果要將編譯目錄分開，也可以在原始碼以外的目錄中執行 configure。此過程也稱為 VPATH 編譯。可以這樣做：

```text
mkdir build_dir
cd build_dir
/path/to/source/tree/configure [options go here]
make
```

預設配置將編譯伺服器和工具程式，以及僅需要 C 編譯器的所有用戶端應用程式和介存取介面。預設情況下，所有檔案都將安裝在 /usr/local/pgsql 下。

您可以透過提供以下一個或多個命令列選項來自訂編譯的過程：

`--prefix=`_`PREFIX`_

安裝所以檔案到到目錄 PREFIX 下而不是 /usr/local/pgsql。實際檔案將安裝到各個子目錄中；任何檔案都不會直接安裝到 PREFIX 目錄中。

如果您有特殊需求，還可以使用以下選項自訂各個子目錄。但是，如果保留這些預設值，則安裝結果將是可重新配置的，這意味著您可以在安裝後移動目錄。（man 和 doc 路徑不受此影響。）

對於可重新配置的安裝，您可能希望使用 configure 的 --disable-rpath 選項。此外，您需要告訴作業系統如何尋找共享函式庫。

`--exec-prefix=`_`EXEC-PREFIX`_

您可以在與 PREFIX 設定的前綴不同的前綴 EXEC-PREFIX 下安裝相依於系統結構的檔案。這對於在主機之間共享與系統結構無關的檔案非常有用。如果省略這一點，則 EXEC-PREFIX 設定為等於 PREFIX，並且相依於系統結構的檔案和獨立檔案都將安裝在同一個樹下，這可能就是您想要的。

`--bindir=`_`DIRECTORY`_

指定可執行程式的目錄。預設值為 EXEC-PREFIX/bin，通常為 /usr/local/pgsql/bin。

`--sysconfdir=`_`DIRECTORY`_

預設設定各種組態配置檔案的目錄，PREFIX/etc。

`--libdir=`_`DIRECTORY`_

設定安裝函式庫和動態模組的位置。預設值為 EXEC-PREFIX/lib。

`--includedir=`_`DIRECTORY`_

設定安裝 C 和 C++ 標頭檔案的目錄。預設值為 PREFIX/include。

`--datarootdir=`_`DIRECTORY`_

設定各種類型的唯讀資料檔案的根目錄。這僅設定以下某些選項的預設值。預設值為 PREFIX/share。

`--datadir=`_`DIRECTORY`_

設定安裝好的程式所使用的唯讀資料檔案目錄。預設值為 DATAROOTDIR。請注意，這與放置資料庫檔案的位置無關。

`--localedir=`_`DIRECTORY`_

設定用於安裝區域設定資料的目錄，像是訊息翻譯的目錄檔案。預設值為 DATAROOTDIR/locale。

`--mandir=`_`DIRECTORY`_

PostgreSQL 附帶的手冊頁面將安裝在此目錄下的各自 manx 子目錄中。預設值為 DATAROOTDIR/man。

`--docdir=`_`DIRECTORY`_

設定安裝文件檔案的根目錄，“man” 頁面除外。這僅設定以下選項的預設值。此選項的預設值為 DARAROOTDIR/doc/postgresql。

`--htmldir=`_`DIRECTORY`_

PostgreSQL 的 HTML 格式文件檔案將安裝在此目錄下。預設值為 DATAROOTDIR。

**注意**  
可以將 PostgreSQL 安裝到共享安裝位置（例如 /usr/local/include），而不會干擾系統其餘部分的命名空間。首先，字串 “/postgresql” 會自動附加到 datadir，sysconfdir 和docdir，除非完全展開的目錄名已包含字串 “postgres” 或 “pgsql”。例如，如果選擇 /usr/local 作為前綴，則檔案將安裝在 /usr/local/doc/postgresql 中，但如果前綴為 /opt/postgres，則它將位於 /opt/postgres/doc 中。用戶端介面的公用 C 標頭檔案安裝在 includedir 中，並且命名空間是清楚的。內部標頭檔案和伺服器標頭檔案安裝在 includedir 下的私有目錄中。有關如何存取其標頭檔案的訊息，請參閱每個介面的文件檔案。最後，如果可以的話，還將在 libdir 下為可動態載入的模組建立一個私有的子目錄。

`--with-extra-version=`_`STRING`_

Append _`STRING`_ to the PostgreSQL version number. You can use this, for example, to mark binaries built from unreleased Git snapshots or containing custom patches with an extra version string such as a `git describe` identifier or a distribution package release number.

`--with-includes=`_`DIRECTORIES`_

_`DIRECTORIES`_ is a colon-separated list of directories that will be added to the list the compiler searches for header files. If you have optional packages \(such as GNU Readline\) installed in a non-standard location, you have to use this option and probably also the corresponding 

`--with-libraries` option.

Example: `--with-includes=/opt/gnu/include:/usr/sup/include`.

`--with-libraries=`_`DIRECTORIES`_

_`DIRECTORIES`_ is a colon-separated list of directories to search for libraries. You will probably have to use this option \(and the corresponding `--with-includes` option\) if you have packages installed in non-standard locations.

Example: `--with-libraries=/opt/gnu/lib:/usr/sup/lib`.

`--enable-nls[=`_`LANGUAGES`_\]

Enables Native Language Support \(NLS\), that is, the ability to display a program's messages in a language other than English. _`LANGUAGES`_ is an optional space-separated list of codes of the languages that you want supported, for example `--enable-nls='de fr'`. \(The intersection between your list and the set of actually provided translations will be computed automatically.\) If you do not specify a list, then all available translations are installed.

To use this option, you will need an implementation of the Gettext API; see above.

`--with-pgport=`_`NUMBER`_

Set _`NUMBER`_ as the default port number for server and clients. The default is 5432. The port can always be changed later on, but if you specify it here then both server and clients will have the same default compiled in, which can be very convenient. Usually the only good reason to select a non-default value is if you intend to run multiple PostgreSQL servers on the same machine.

`--with-perl`

Build the PL/Perl server-side language.

`--with-python`

Build the PL/Python server-side language.

`--with-tcl`

Build the PL/Tcl server-side language.

`--with-tclconfig=`_`DIRECTORY`_

Tcl installs the file `tclConfig.sh`, which contains configuration information needed to build modules interfacing to Tcl. This file is normally found automatically at a well-known location, but if you want to use a different version of Tcl you can specify the directory in which to look for it.

`--with-gssapi`

Build with support for GSSAPI authentication. On many systems, the GSSAPI \(usually a part of the Kerberos installation\) system is not installed in a location that is searched by default \(e.g.,`/usr/include`, `/usr/lib`\), so you must use the options `--with-includes` and `--with-libraries` in addition to this option. `configure` will check for the required header files and libraries to make sure that your GSSAPI installation is sufficient before proceeding.

`--with-krb-srvnam=`_`NAME`_

The default name of the Kerberos service principal used by GSSAPI. `postgres` is the default. There's usually no reason to change this unless you have a Windows environment, in which case it must be set to upper case `POSTGRES`.

`--with-llvm`

Build with support for LLVM based JIT compilation \(see [Chapter 32](https://www.postgresql.org/docs/current/static/jit.html)\). This requires the LLVM library to be installed. The minimum required version of LLVM is currently 3.9.

`llvm-config` will be used to find the required compilation options. `llvm-config`, and then `llvm-config-$major-$minor` for all supported versions, will be searched on `PATH`. If that would not yield the correct binary, use `LLVM_CONFIG` to specify a path to the correct `llvm-config`. For example

```text
./configure ... --with-llvm LLVM_CONFIG='/path/to/llvm/bin/llvm-config'
```

LLVM support requires a compatible `clang` compiler \(specified, if necessary, using the `CLANG` environment variable\), and a working C++ compiler \(specified, if necessary, using the `CXX`environment variable\).

`--with-icu`

Build with support for the ICU library. This requires the ICU4C package to be installed. The minimum required version of ICU4C is currently 4.2.

By default, pkg-config will be used to find the required compilation options. This is supported for ICU4C version 4.6 and later. For older versions, or if pkg-config is not available, the variables `ICU_CFLAGS` and `ICU_LIBS` can be specified to `configure`, like in this example:

```text
./configure ... --with-icu ICU_CFLAGS='-I/some/where/include' ICU_LIBS='-L/some/where/lib -licui18n -licuuc -licudata'
```

\(If ICU4C is in the default search path for the compiler, then you still need to specify a nonempty string in order to avoid use of pkg-config, for example, `ICU_CFLAGS=' '`.\)

`--with-openssl`

Build with support for SSL \(encrypted\) connections. This requires the OpenSSL package to be installed. `configure` will check for the required header files and libraries to make sure that your OpenSSL installation is sufficient before proceeding.

`--with-pam`

Build with PAM \(Pluggable Authentication Modules\) support.

`--with-bsd-auth`

Build with BSD Authentication support. \(The BSD Authentication framework is currently only available on OpenBSD.\)

`--with-ldap`

Build with LDAP support for authentication and connection parameter lookup \(see [Section 34.17](https://www.postgresql.org/docs/current/static/libpq-ldap.html) and [Section 20.10](https://www.postgresql.org/docs/current/static/auth-ldap.html) for more information\). On Unix, this requires the OpenLDAP package to be installed. On Windows, the default WinLDAP library is used. `configure` will check for the required header files and libraries to make sure that your OpenLDAP installation is sufficient before proceeding.

`--with-systemd`

Build with support for systemd service notifications. This improves integration if the server binary is started under systemd but has no impact otherwise; see [Section 18.3](https://www.postgresql.org/docs/current/static/server-start.html) for more information. libsystemd and the associated header files need to be installed to be able to use this option.

`--without-readline`

Prevents use of the Readline library \(and libedit as well\). This option disables command-line editing and history in psql, so it is not recommended.

`--with-libedit-preferred`

Favors the use of the BSD-licensed libedit library rather than GPL-licensed Readline. This option is significant only if you have both libraries installed; the default in that case is to use Readline.

`--with-bonjour`

Build with Bonjour support. This requires Bonjour support in your operating system. Recommended on macOS.

`--with-uuid=`_`LIBRARY`_

Build the [uuid-ossp](https://www.postgresql.org/docs/current/static/uuid-ossp.html) module \(which provides functions to generate UUIDs\), using the specified UUID library. _`LIBRARY`_ must be one of:

* `bsd` to use the UUID functions found in FreeBSD, NetBSD, and some other BSD-derived systems
* `e2fs` to use the UUID library created by the `e2fsprogs` project; this library is present in most Linux systems and in macOS, and can be obtained for other platforms as well
* `ossp` to use the [OSSP UUID library](http://www.ossp.org/pkg/lib/uuid/)

`--with-ossp-uuid`

Obsolete equivalent of `--with-uuid=ossp`.

`--with-libxml`

Build with libxml \(enables SQL/XML support\). Libxml version 2.6.23 or later is required for this feature.

Libxml installs a program `xml2-config` that can be used to detect the required compiler and linker options. PostgreSQL will use it automatically if found. To specify a libxml installation at an unusual location, you can either set the environment variable `XML2_CONFIG` to point to the `xml2-config` program belonging to the installation, or use the options `--with-includes` and `--with-libraries`.`--with-libxslt`

Use libxslt when building the [xml2](https://www.postgresql.org/docs/current/static/xml2.html) module. xml2 relies on this library to perform XSL transformations of XML.

`--disable-float4-byval`

Disable passing float4 values “by value”, causing them to be passed “by reference” instead. This option costs performance, but may be needed for compatibility with old user-defined functions that are written in C and use the “version 0” calling convention. A better long-term solution is to update any such functions to use the “version 1” calling convention.

`--disable-float8-byval`

Disable passing float8 values “by value”, causing them to be passed “by reference” instead. This option costs performance, but may be needed for compatibility with old user-defined functions that are written in C and use the “version 0” calling convention. A better long-term solution is to update any such functions to use the “version 1” calling convention. Note that this option affects not only float8, but also int8 and some related types such as timestamp. On 32-bit platforms, `--disable-float8-byval` is the default and it is not allowed to select `--enable-float8-byval`.

`--with-segsize=`_`SEGSIZE`_

Set the _segment size_, in gigabytes. Large tables are divided into multiple operating-system files, each of size equal to the segment size. This avoids problems with file size limits that exist on many platforms. The default segment size, 1 gigabyte, is safe on all supported platforms. If your operating system has “largefile” support \(which most do, nowadays\), you can use a larger segment size. This can be helpful to reduce the number of file descriptors consumed when working with very large tables. But be careful not to select a value larger than is supported by your platform and the file systems you intend to use. Other tools you might wish to use, such as tar, could also set limits on the usable file size. It is recommended, though not absolutely required, that this value be a power of 2. Note that changing this value requires an initdb.

`--with-blocksize=`_`BLOCKSIZE`_

Set the _block size_, in kilobytes. This is the unit of storage and I/O within tables. The default, 8 kilobytes, is suitable for most situations; but other values may be useful in special cases. The value must be a power of 2 between 1 and 32 \(kilobytes\). Note that changing this value requires an initdb.

`--with-wal-blocksize=`_`BLOCKSIZE`_

Set the _WAL block size_, in kilobytes. This is the unit of storage and I/O within the WAL log. The default, 8 kilobytes, is suitable for most situations; but other values may be useful in special cases. The value must be a power of 2 between 1 and 64 \(kilobytes\). Note that changing this value requires an initdb.

`--disable-spinlocks`

Allow the build to succeed even if PostgreSQL has no CPU spinlock support for the platform. The lack of spinlock support will result in poor performance; therefore, this option should only be used if the build aborts and informs you that the platform lacks spinlock support. If this option is required to build PostgreSQL on your platform, please report the problem to the PostgreSQLdevelopers.

`--disable-strong-random`

Allow the build to succeed even if PostgreSQL has no support for strong random numbers on the platform. A source of random numbers is needed for some authentication protocols, as well as some routines in the [pgcrypto](https://www.postgresql.org/docs/current/static/pgcrypto.html) module. `--disable-strong-random` disables functionality that requires cryptographically strong random numbers, and substitutes a weak pseudo-random-number-generator for the generation of authentication salt values and query cancel keys. It may make authentication less secure.

`--disable-thread-safety`

Disable the thread-safety of client libraries. This prevents concurrent threads in libpq and ECPG programs from safely controlling their private connection handles.

`--with-system-tzdata=`_`DIRECTORY`_

PostgreSQL includes its own time zone database, which it requires for date and time operations. This time zone database is in fact compatible with the IANA time zone database provided by many operating systems such as FreeBSD, Linux, and Solaris, so it would be redundant to install it again. When this option is used, the system-supplied time zone database in _`DIRECTORY`_ is used instead of the one included in the PostgreSQL source distribution. _`DIRECTORY`_ must be specified as an absolute path. `/usr/share/zoneinfo` is a likely directory on some operating systems. Note that the installation routine will not detect mismatching or erroneous time zone data. If you use this option, you are advised to run the regression tests to verify that the time zone data you have pointed to works correctly with PostgreSQL.

This option is mainly aimed at binary package distributors who know their target operating system well. The main advantage of using this option is that the PostgreSQL package won't need to be upgraded whenever any of the many local daylight-saving time rules change. Another advantage is that PostgreSQL can be cross-compiled more straightforwardly if the time zone database files do not need to be built during the installation.

`--without-zlib`

Prevents use of the Zlib library. This disables support for compressed archives in pg\_dump and pg\_restore. This option is only intended for those rare systems where this library is not available.

`--enable-debug`

Compiles all programs and libraries with debugging symbols. This means that you can run the programs in a debugger to analyze problems. This enlarges the size of the installed executables considerably, and on non-GCC compilers it usually also disables compiler optimization, causing slowdowns. However, having the symbols available is extremely helpful for dealing with any problems that might arise. Currently, this option is recommended for production installations only if you use GCC. But you should always have it on if you are doing development work or running a beta version.

`--enable-coverage`

If using GCC, all programs and libraries are compiled with code coverage testing instrumentation. When run, they generate files in the build directory with code coverage metrics. See[Section 33.5](https://www.postgresql.org/docs/current/static/regress-coverage.html) for more information. This option is for use only with GCC and when doing development work.

`--enable-profiling`

If using GCC, all programs and libraries are compiled so they can be profiled. On backend exit, a subdirectory will be created that contains the `gmon.out` file for use in profiling. This option is for use only with GCC and when doing development work.

`--enable-cassert`

Enables _assertion_ checks in the server, which test for many “cannot happen” conditions. This is invaluable for code development purposes, but the tests can slow down the server significantly. Also, having the tests turned on won't necessarily enhance the stability of your server! The assertion checks are not categorized for severity, and so what might be a relatively harmless bug will still lead to server restarts if it triggers an assertion failure. This option is not recommended for production use, but you should have it on for development work or when running a beta version.

`--enable-depend`

Enables automatic dependency tracking. With this option, the makefiles are set up so that all affected object files will be rebuilt when any header file is changed. This is useful if you are doing development work, but is just wasted overhead if you intend only to compile once and install. At present, this option only works with GCC.

`--enable-dtrace`

Compiles PostgreSQL with support for the dynamic tracing tool DTrace. See [Section 28.5](https://www.postgresql.org/docs/current/static/dynamic-trace.html) for more information.

To point to the `dtrace` program, the environment variable `DTRACE` can be set. This will often be necessary because `dtrace` is typically installed under `/usr/sbin`, which might not be in the path.

Extra command-line options for the `dtrace` program can be specified in the environment variable `DTRACEFLAGS`. On Solaris, to include DTrace support in a 64-bit binary, you must specify `DTRACEFLAGS="-64"` to configure. For example, using the GCC compiler:

```text
./configure CC='gcc -m64' --enable-dtrace DTRACEFLAGS='-64' ...
```

Using Sun's compiler:

```text
./configure CC='/opt/SUNWspro/bin/cc -xtarget=native64' --enable-dtrace DTRACEFLAGS='-64' ...
```

`--enable-tap-tests`

Enable tests using the Perl TAP tools. This requires a Perl installation and the Perl module `IPC::Run`. See [Section 33.4](https://www.postgresql.org/docs/current/static/regress-tap.html) for more information.

If you prefer a C compiler different from the one `configure` picks, you can set the environment variable `CC` to the program of your choice. By default, `configure` will pick `gcc` if available, else the platform's default \(usually `cc`\). Similarly, you can override the default compiler flags if needed with the `CFLAGS` variable.

You can specify environment variables on the `configure` command line, for example:

```text
./configure CC=/opt/bin/gcc CFLAGS='-O2 -pipe'
```

Here is a list of the significant variables that can be set in this manner:

`BISON`

Bison program

`CC`

C compiler

`CFLAGS`

options to pass to the C compiler

`CLANG`

path to `clang` program used to process source code for inlining when compiling with `--with-llvmCPP`

C preprocessor

`CPPFLAGS`

options to pass to the C preprocessor

`CXX`

C++ compiler

`CXXFLAGS`

options to pass to the C++ compiler

`DTRACE`

location of the `dtrace` program

`DTRACEFLAGS`

options to pass to the `dtrace` program

`FLEX`

Flex program`LDFLAGS`

options to use when linking either executables or shared libraries

`LDFLAGS_EX`

additional options for linking executables only

`LDFLAGS_SL`

additional options for linking shared libraries only

`LLVM_CONFIG`

`llvm-config` program used to locate the LLVM installation.

`MSGFMT`

`msgfmt` program for native language support

`PERL`

Full path name of the Perl interpreter. This will be used to determine the dependencies for building PL/Perl.

`PYTHON`

Full path name of the Python interpreter. This will be used to determine the dependencies for building PL/Python. Also, whether Python 2 or 3 is specified here \(or otherwise implicitly chosen\) determines which variant of the PL/Python language becomes available. See [Section 46.1](https://www.postgresql.org/docs/current/static/plpython-python23.html) for more information.

`TCLSH`

Full path name of the Tcl interpreter. This will be used to determine the dependencies for building PL/Tcl, and it will be substituted into Tcl scripts.

`XML2_CONFIG`

`xml2-config` program used to locate the libxml installation.

Sometimes it is useful to add compiler flags after-the-fact to the set that were chosen by `configure`. An important example is that gcc's `-Werror` option cannot be included in the `CFLAGS`passed to `configure`, because it will break many of `configure`'s built-in tests. To add such flags, include them in the `COPT` environment variable while running `make`. The contents of`COPT` are added to both the `CFLAGS` and `LDFLAGS` options set up by `configure`. For example, you could do

```text
make COPT='-Werror'
```

or

```text
export COPT='-Werror'
make
```

**Note**

When developing code inside the server, it is recommended to use the configure options `--enable-cassert` \(which turns on many run-time error checks\) and `--enable-debug`\(which improves the usefulness of debugging tools\).

If using GCC, it is best to build with an optimization level of at least `-O1`, because using no optimization \(`-O0`\) disables some important compiler warnings \(such as the use of uninitialized variables\). However, non-zero optimization levels can complicate debugging because stepping through compiled code will usually not match up one-to-one with source code lines. If you get confused while trying to debug optimized code, recompile the specific files of interest with `-O0`. An easy way to do this is by passing an option to make: 

`make PROFILE=-O0 file.o`.

The `COPT` and `PROFILE` environment variables are actually handled identically by the PostgreSQL makefiles. Which to use is a matter of preference, but a common habit among developers is to use `PROFILE` for one-time flag adjustments, while `COPT` might be kept set all the time.

## **2. Build**

To start the build, type either of:

```text
make
make all
```

\(Remember to use GNU make.\) The build will take a few minutes depending on your hardware. The last line displayed should be:

```text
All of PostgreSQL successfully made. Ready to install.
```

If you want to build everything that can be built, including the documentation \(HTML and man pages\), and the additional modules \(`contrib`\), type instead:

```text
make world
```

The last line displayed should be:

```text
PostgreSQL, contrib, and documentation successfully made. Ready to install.
```

If you want to invoke the build from another makefile rather than manually, you must unset `MAKELEVEL` or set it to zero, for instance like this:

```text
build-postgresql:
        $(MAKE) -C postgresql MAKELEVEL=0 all
```

Failure to do that can lead to strange error messages, typically about missing header files.

## **3. Regression Tests**

If you want to test the newly built server before you install it, you can run the regression tests at this point. The regression tests are a test suite to verify that PostgreSQL runs on your machine in the way the developers expected it to. Type:

```text
make check
```

\(This won't work as root; do it as an unprivileged user.\) See [Chapter 33](https://www.postgresql.org/docs/current/static/regress.html) for detailed information about interpreting the test results. You can repeat this test at any later time by issuing the same command.

## **4. Installing the Files**

**Note**

If you are upgrading an existing system be sure to read [Section 18.6](https://www.postgresql.org/docs/current/static/upgrading.html), which has instructions about upgrading a cluster.

To install PostgreSQL enter:

```text
make install
```

This will install files into the directories that were specified in [Step 1](https://www.postgresql.org/docs/current/static/install-procedure.html#CONFIGURE). Make sure that you have appropriate permissions to write into that area. Normally you need to do this step as root. Alternatively, you can create the target directories in advance and arrange for appropriate permissions to be granted.

To install the documentation \(HTML and man pages\), enter:

```text
make install-docs
```

If you built the world above, type instead:

```text
make install-world
```

This also installs the documentation.

You can use `make install-strip` instead of `make install` to strip the executable files and libraries as they are installed. This will save some space. If you built with debugging support, stripping will effectively remove the debugging support, so it should only be done if debugging is no longer needed. `install-strip` tries to do a reasonable job saving space, but it does not have perfect knowledge of how to strip every unneeded byte from an executable file, so if you want to save all the disk space you possibly can, you will have to do manual work.

The standard installation provides all the header files needed for client application development as well as for server-side program development, such as custom functions or data types written in C. \(Prior to PostgreSQL 8.0, a separate `make install-all-headers` command was needed for the latter, but this step has been folded into the standard install.\)

**Client-only installation:**  If you want to install only the client applications and interface libraries, then you can use these commands:

```text
make -C src/bin install
make -C src/include install
make -C src/interfaces install
make -C doc install
```

`src/bin` has a few binaries for server-only use, but they are small.

**Uninstallation:**  To undo the installation use the command `make uninstall`. However, this will not remove any created directories.

**Cleaning:**  After the installation you can free disk space by removing the built files from the source tree with the command `make clean`. This will preserve the files made by the `configure` program, so that you can rebuild everything with `make` later on. To reset the source tree to the state in which it was distributed, use `make distclean`. If you are going to build for several platforms within the same source tree you must do this and re-configure for each platform. \(Alternatively, use a separate build tree for each platform, so that the source tree remains unmodified.\)

If you perform a build and then discover that your `configure` options were wrong, or if you change anything that `configure` investigates \(for example, software upgrades\), then it's a good idea to do `make distclean` before reconfiguring and rebuilding. Without this, your changes in configuration choices might not propagate everywhere they need to.

