# 1 查看 build 命令手册

在 Linux/Windows/macOS 中，大多数命令都有多种选项，这些选项可以实现命令的个性化需求。我们可以利用 go help build 来查看 go build 命令的选项，具体的命令及输出如下

> go help build  
> usage: go build [-o output] [build flags] [packages]

Build compiles the packages named by the import paths,
along with their dependencies, but it does not install the results.

If the arguments to build are a list of .go files from a single directory,
build treats them as a list of source files specifying a single package.

When compiling packages, build ignores files that end in '\_test.go'.

When compiling a single main package, build writes the resulting
executable to an output file named after the last non-major-version
component of the package import path. The '.exe' suffix is added
when writing a Windows executable.
So 'go build example/sam' writes 'sam' or 'sam.exe'.
'go build example.com/foo/v2' writes 'foo' or 'foo.exe', not 'v2.exe'.

When compiling a package from a list of .go files, the executable
is named after the first source file.
'go build ed.go rx.go' writes 'ed' or 'ed.exe'.

When compiling multiple packages or a single non-main package,
build compiles the packages but discards the resulting object,
serving only as a check that the packages can be built.

The -o flag forces build to write the resulting executable or object
to the named output file or directory, instead of the default behavior described
in the last two paragraphs. If the named output is an existing directory or
ends with a slash or backslash, then any resulting executables
will be written to that directory.

The build flags are shared by the build, clean, get, install, list, run,
and test commands:

        -C dir
                Change to dir before running the command.
                Any files named on the command line are interpreted after
                changing directories.
                If used, this flag must be the first one in the command line.
        -a
                force rebuilding of packages that are already up-to-date.
        -n
                print the commands but do not run them.
        -p n
                the number of programs, such as build commands or
                test binaries, that can be run in parallel.
                The default is GOMAXPROCS, normally the number of CPUs available.
        -race
                enable data race detection.
                Supported only on linux/amd64, freebsd/amd64, darwin/amd64, darwin/arm64, windows/amd64,
                linux/ppc64le and linux/arm64 (only for 48-bit VMA).
        -msan
                enable interoperation with memory sanitizer.
                Supported only on linux/amd64, linux/arm64, linux/loong64, freebsd/amd64
                and only with Clang/LLVM as the host C compiler.
                PIE build mode will be used on all platforms except linux/amd64.
        -asan
                enable interoperation with address sanitizer.
                Supported only on linux/arm64, linux/amd64, linux/loong64.
                Supported on linux/amd64 or linux/arm64 and only with GCC 7 and higher
                or Clang/LLVM 9 and higher.
                And supported on linux/loong64 only with Clang/LLVM 16 and higher.
        -cover
                enable code coverage instrumentation.
        -covermode set,count,atomic
                set the mode for coverage analysis.
                The default is "set" unless -race is enabled,
                in which case it is "atomic".
                The values:
                set: bool: does this statement run?
                count: int: how many times does this statement run?
                atomic: int: count, but correct in multithreaded tests;
                        significantly more expensive.
                Sets -cover.
        -coverpkg pattern1,pattern2,pattern3
                For a build that targets package 'main' (e.g. building a Go
                executable), apply coverage analysis to each package matching
                the patterns. The default is to apply coverage analysis to
                packages in the main Go module. See 'go help packages' for a
                description of package patterns.  Sets -cover.
        -v
                print the names of packages as they are compiled.
        -work
                print the name of the temporary work directory and
                do not delete it when exiting.
        -x
                print the commands.
        -asmflags '[pattern=]arg list'
                arguments to pass on each go tool asm invocation.
        -buildmode mode
                build mode to use. See 'go help buildmode' for more.
        -buildvcs
                Whether to stamp binaries with version control information
                ("true", "false", or "auto"). By default ("auto"), version control
                information is stamped into a binary if the main package, the main module
                containing it, and the current directory are all in the same repository.
                Use -buildvcs=false to always omit version control information, or
                -buildvcs=true to error out if version control information is available but
                cannot be included due to a missing tool or ambiguous directory structure.
        -compiler name
                name of compiler to use, as in runtime.Compiler (gccgo or gc).
        -gccgoflags '[pattern=]arg list'
                arguments to pass on each gccgo compiler/linker invocation.
        -gcflags '[pattern=]arg list'
                arguments to pass on each go tool compile invocation.
        -installsuffix suffix
                a suffix to use in the name of the package installation directory,
                in order to keep output separate from default builds.
                If using the -race flag, the install suffix is automatically set to race
                or, if set explicitly, has _race appended to it. Likewise for the -msan
                and -asan flags. Using a -buildmode option that requires non-default compile
                flags has a similar effect.
        -ldflags '[pattern=]arg list'
                arguments to pass on each go tool link invocation.
        -linkshared
                build code that will be linked against shared libraries previously
                created with -buildmode=shared.
        -mod mode
                module download mode to use: readonly, vendor, or mod.
                By default, if a vendor directory is present and the go version in go.mod
                is 1.14 or higher, the go command acts as if -mod=vendor were set.
                Otherwise, the go command acts as if -mod=readonly were set.
                See https://golang.org/ref/mod#build-commands for details.
        -modcacherw
                leave newly-created directories in the module cache read-write
                instead of making them read-only.
        -modfile file
                in module aware mode, read (and possibly write) an alternate go.mod
                file instead of the one in the module root directory. A file named
                "go.mod" must still be present in order to determine the module root
                directory, but it is not accessed. When -modfile is specified, an
                alternate go.sum file is also used: its path is derived from the
                -modfile flag by trimming the ".mod" extension and appending ".sum".
        -overlay file
                read a JSON config file that provides an overlay for build operations.
                The file is a JSON struct with a single field, named 'Replace', that
                maps each disk file path (a string) to its backing file path, so that
                a build will run as if the disk file path exists with the contents
                given by the backing file paths, or as if the disk file path does not
                exist if its backing file path is empty. Support for the -overlay flag
                has some limitations: importantly, cgo files included from outside the
                include path must be in the same directory as the Go package they are
                included from, and overlays will not appear when binaries and tests are
                run through go run and go test respectively.
        -pgo file
                specify the file path of a profile for profile-guided optimization (PGO).
                When the special name "auto" is specified, for each main package in the
                build, the go command selects a file named "default.pgo" in the package's
                directory if that file exists, and applies it to the (transitive)
                dependencies of the main package (other packages are not affected).
                Special name "off" turns off PGO. The default is "auto".
        -pkgdir dir
                install and load all packages from dir instead of the usual locations.
                For example, when building with a non-standard configuration,
                use -pkgdir to keep generated packages in a separate location.
        -tags tag,list
                a comma-separated list of additional build tags to consider satisfied
                during the build. For more information about build tags, see
                'go help buildconstraint'. (Earlier versions of Go used a
                space-separated list, and that form is deprecated but still recognized.)
        -trimpath
                remove all file system paths from the resulting executable.
                Instead of absolute file system paths, the recorded file names
                will begin either a module path@version (when using modules),
                or a plain import path (when using the standard library, or GOPATH).
        -toolexec 'cmd args'
                a program to use to invoke toolchain programs like vet and asm.
                For example, instead of running asm, the go command will run
                'cmd args /path/to/asm <arguments for asm>'.
                The TOOLEXEC_IMPORTPATH environment variable will be set,
                matching 'go list -f {{.ImportPath}}' for the package being built.

The -asmflags, -gccgoflags, -gcflags, and -ldflags flags accept a
space-separated list of arguments to pass to an underlying tool
during the build. To embed spaces in an element in the list, surround
it with either single or double quotes. The argument list may be
preceded by a package pattern and an equal sign, which restricts
the use of that argument list to the building of packages matching
that pattern (see 'go help packages' for a description of package
patterns). Without a pattern, the argument list applies only to the
packages named on the command line. The flags may be repeated
with different patterns in order to specify different arguments for
different sets of packages. If a package matches patterns given in
multiple flags, the latest match on the command line wins.
For example, 'go build -gcflags=-S fmt' prints the disassembly
only for package fmt, while 'go build -gcflags=all=-S fmt'
prints the disassembly for fmt and all its dependencies.

For more about specifying packages, see 'go help packages'.
For more about where packages and binaries are installed,
run 'go help gopath'.
For more about calling between Go and C/C++, run 'go help c'.

Note: Build adheres to certain conventions such as those described
by 'go help gopath'. Not all projects can follow these conventions,
however. Installations that have their own conventions or that use
a separate software build system may choose to use lower-level
invocations such as 'go tool compile' and 'go tool link' to avoid
some of the overheads and design decisions of the build tool.

See also: go install, go get, go clean.

# 2 查看编译的详细过程

可以看到，go build 的选项非常多，其中有 3 个需要我们特别关注：
· -n：打印编译的详细过程，但是并不实际执行，相当于预览功能。
· -x：执行编译，同时也打印编译的详细过程。
· --work：这是一个追加选项，在 go build 的编译过程中会生成部分中间目录和文件，编译结束后，会将生成的临时目录和文件删除，--work 用于保留生成的临时目录和文件，方便开发者查看和诊断问题。

我们可以直接利用-x 和--work 选项来查看编译的详细过程，具体的命令及输出如下：

```shell
> go build -x -work  first.go
WORK=/var/folders/m6/k3xdyk6n7358y_h13350m9gm0000gn/T/go-build3155358895
mkdir -p $WORK/b001/
cat >/var/folders/m6/k3xdyk6n7358y_h13350m9gm0000gn/T/go-build3155358895/b001/importcfg.link << 'EOF' # internal
packagefile command-line-arguments=/Users/allenwork/Library/Caches/go-build/2d/2d1eda912c63b8ff1e92884bc1d30a778d69d5d085961024d0b2a954459962e7-d
packagefile fmt=/Users/allenwork/Library/Caches/go-build/6e/6e509e1755ebee294a41cab8c1c4604cac8fc4bf9331d0ecfa331e6e4cc6e293-d
packagefile runtime=/Users/allenwork/Library/Caches/go-build/82/82092428a6b54f2ddb3f77dc337ca0a4ea4505f09c42712106c6f4b361bd275d-d
packagefile errors=/Users/allenwork/Library/Caches/go-build/37/37221ce4852a18d3b3768dc061ae72875554205a9c9e5cb68e94e531a836fdf0-d
packagefile internal/fmtsort=/Users/allenwork/Library/Caches/go-build/e1/e132c0d8b1db08b80911ba447b0326829dda3aaaa43909569810b48c67226796-d
packagefile io=/Users/allenwork/Library/Caches/go-build/2d/2dd5fa579e4caff798a00e9b2b6b2e9d0b2bedf5a743b303f10adaa0981a96c9-d
packagefile math=/Users/allenwork/Library/Caches/go-build/86/86a0e2de2d511143b7f9ea9b363a7eed27518b8f0d512d96e047d96a2f4f738a-d
packagefile os=/Users/allenwork/Library/Caches/go-build/19/19f5300ac1cdb16eae520243416ff7f32ef8ea6da871833768e0519bffaec1a4-d
packagefile reflect=/Users/allenwork/Library/Caches/go-build/6a/6a71072b77f9b3f997c5187f12c18ac918d39af35ac1353acd2b1d5f497748cf-d
packagefile sort=/Users/allenwork/Library/Caches/go-build/f7/f7a53152d976df2de02bd97587a08a0f7e3044c0b74dc9beb14fb6fe125edf10-d
packagefile strconv=/Users/allenwork/Library/Caches/go-build/3b/3be5428bedf691b583133efb2ec50f6f10cb6913ea6ece765209beb419717974-d
packagefile sync=/Users/allenwork/Library/Caches/go-build/6b/6b17f3191b3c628c425f73fe2787ffe32613b8b6b2ca393a3c49b886c4504966-d
packagefile unicode/utf8=/Users/allenwork/Library/Caches/go-build/5c/5c541fc27df039b3aac53c27180fcb2d628d0a9121bd772bf3b43549b4bc958a-d
packagefile internal/abi=/Users/allenwork/Library/Caches/go-build/61/617acc8c39bed0ff05c5a7f72d8d687525539c8aaf5f461d93d83230aa433a0d-d
packagefile internal/bytealg=/Users/allenwork/Library/Caches/go-build/79/796d65faaa25bf22b56dcaec7fbd58fe781086ef027a946976f33cece1eceb26-d
packagefile internal/chacha8rand=/Users/allenwork/Library/Caches/go-build/6f/6fcf61f35b39cd42b83c850429f2b2d6f4bfdddb519a1fe051ea1b462504d248-d
packagefile internal/coverage/rtcov=/Users/allenwork/Library/Caches/go-build/99/99b5b0d1e3fdbdc993423d31c45e022a2e8813c2c37dbb14133f5a96e498ca3c-d
packagefile internal/cpu=/Users/allenwork/Library/Caches/go-build/8f/8f0ee18966b29c537bd2bc38b25ae5a589a357f53fa1158839bf721d7197e69e-d
packagefile internal/goarch=/Users/allenwork/Library/Caches/go-build/ed/ed3c34c7e2670b75473e0b195a68f993afc489be6680c9523e3810a75a542bff-d
packagefile internal/godebugs=/Users/allenwork/Library/Caches/go-build/07/07a392dfc3003ceef83b3fcba0e4bd51708c036e22e93a816fb7a592078e79ea-d
packagefile internal/goexperiment=/Users/allenwork/Library/Caches/go-build/cb/cbb3f94030a93e93cf64ec23a4e181dfaa4610b85b1711c480b6abe2383f05e3-d
packagefile internal/goos=/Users/allenwork/Library/Caches/go-build/62/62b1549bec43acf85a04c9cdebcb810504c38ab67d57b6c38b8eb1b4da368fbd-d
packagefile runtime/internal/atomic=/Users/allenwork/Library/Caches/go-build/93/93d1d99616cc2695a2c1c2a8b6df835278e45d24af73baa3e9ad33e231854f5b-d
packagefile runtime/internal/math=/Users/allenwork/Library/Caches/go-build/f6/f68d0c21cc30c3fc7f51d6e7abb1c8525fe03a25031a4e70579aa31a1dd64d7e-d
packagefile runtime/internal/sys=/Users/allenwork/Library/Caches/go-build/ff/ffa10bf88c00a8348103b0f993ead865d283ef7a1b7798405e285ae48a27cc8a-d
packagefile internal/reflectlite=/Users/allenwork/Library/Caches/go-build/24/24d1a364249ec5405ed8079f6b3722bb836811a78f951a86484fdeabb6f8478d-d
packagefile math/bits=/Users/allenwork/Library/Caches/go-build/0e/0e5ebe7f84bc400861ff7b820b9b21e11a0305b1443dfadc48427f6c8449975d-d
packagefile internal/itoa=/Users/allenwork/Library/Caches/go-build/2e/2e894d349ff45b3344e7dec7884f6f314ba8aef033314c5d476215a7696a181f-d
packagefile internal/poll=/Users/allenwork/Library/Caches/go-build/ad/ad3bcfeabd8a287cbaad208de370f5772832b78e9c56077849f860492444b5c6-d
packagefile internal/safefilepath=/Users/allenwork/Library/Caches/go-build/b5/b5c41a056a1de73b469e3ed2bea77ebfe2c9604c26d54abdd6d4075db20369b6-d
packagefile internal/syscall/execenv=/Users/allenwork/Library/Caches/go-build/51/518fce92cbf36dd615d84d7b33ad155095fd8f86e55068bccc9a6ffbce1608f5-d
packagefile internal/syscall/unix=/Users/allenwork/Library/Caches/go-build/21/21d4cba0aa8fe68495950f51d1f9acc4f66c17ae257f9f6f3f50b251951e1e85-d
packagefile internal/testlog=/Users/allenwork/Library/Caches/go-build/11/11b3d811ef5fb78a00922c8ea8c02553425d10d8c180e7ac6691ea925c3c2ce5-d
packagefile io/fs=/Users/allenwork/Library/Caches/go-build/da/da28b2319003ccf6cbca0dd87074f0d611cfc3c51f46134d3e837ee1f9eddae5-d
packagefile sync/atomic=/Users/allenwork/Library/Caches/go-build/4c/4c429a25512a2ad3dc4108b712b12093f1b91019e8c1dc9f4eaac1b60c9eb6de-d
packagefile syscall=/Users/allenwork/Library/Caches/go-build/4e/4e9bc7dfb588dcbd9156ddc3f8bc067ed1c656b702db91809b63834383aa4eaa-d
packagefile time=/Users/allenwork/Library/Caches/go-build/26/2672a68ea544617dd85fe2eb20ce748d31fd8acf2c924d356608d979d321b13d-d
packagefile internal/unsafeheader=/Users/allenwork/Library/Caches/go-build/b1/b193d022a6e48a2345a0fe93108490107accfd253b08c8a2d1e6ca71279b519c-d
packagefile unicode=/Users/allenwork/Library/Caches/go-build/ce/ce27308404956d025d7f3b40b385c4193b07cc01e7d16c76de57f1fea5a99919-d
packagefile slices=/Users/allenwork/Library/Caches/go-build/8d/8d4d58a8dd620076728ac7af9fdd98366e5fd16d82da9d4f8e8d7ad37b91bbe2-d
packagefile internal/race=/Users/allenwork/Library/Caches/go-build/b1/b1fa43e51e999396e8e8cdbd3efd63b7196691b30088ba00456f2b05bdced6d4-d
packagefile internal/oserror=/Users/allenwork/Library/Caches/go-build/69/69dc4421c5318d81d8ac7ac146d5a4420cf0d159d0477286147d465a093dafb0-d
packagefile path=/Users/allenwork/Library/Caches/go-build/17/17c763f7a70b961bd374a6b471c1aae622e0d7b94ccd596d87cb313c64faef3e-d
packagefile cmp=/Users/allenwork/Library/Caches/go-build/24/2467db35d5a4358e366c0067cb8dda26e16a3871acdfa0f32ebb195b8a4e9c61-d
modinfo "0w\xaf\f\x92t\b\x02A\xe1\xc1\a\xe6\xd6\x18\xe6path\tcommand-line-arguments\nbuild\t-buildmode=exe\nbuild\t-compiler=gc\nbuild\tCGO_ENABLED=1\nbuild\tCGO_CFLAGS=\nbuild\tCGO_CPPFLAGS=\nbuild\tCGO_CXXFLAGS=\nbuild\tCGO_LDFLAGS=\nbuild\tGOARCH=amd64\nbuild\tGOOS=darwin\nbuild\tGOAMD64=v1\n\xf92C1\x86\x18 r\x00\x82B\x10A\x16\xd8\xf2"
EOF
mkdir -p $WORK/b001/exe/
cd .
/usr/local/Cellar/go/1.22.2/libexec/pkg/tool/darwin_amd64/link -o $WORK/b001/exe/a.out -importcfg $WORK/b001/importcfg.link -buildmode=pie -buildid=H9r3bRFEiXeviYo87CmR/J0l7I8QSHwk2s6nur7_0/n4HVGaSyRbggdi6Easje/H9r3bRFEiXeviYo87CmR -extld=cc /Users/allenwork/Library/Caches/go-build/2d/2d1eda912c63b8ff1e92884bc1d30a778d69d5d085961024d0b2a954459962e7-d
/usr/local/Cellar/go/1.22.2/libexec/pkg/tool/darwin_amd64/buildid -w $WORK/b001/exe/a.out # internal
mv $WORK/b001/exe/a.out first
```

从 go build 命令的输出可以看出，该命令实际执行了若干操作系统指令。各指令说明如下

(1) WORK=/var/folders/28/w3v87nrn6fsd06mkl9v21vnr0000gn/T/go-build3761933774：用于指定 WORK 变量的值。WORK 就是 go build 的工作目录，该目录的前半部分“/var/folders/28/w3v87nrn6fsd06mkl9v21vnr0000gn/T”来源于操作系统的临时目录；后半部分“go-build3761933774”是编译时自动追加的子文件夹。在 macOS 中，操作系统临时目录利用变量$TMPDIR 进行定义；而在 Windows 操作系统中，该值来源于环境变量 TMP。
(2) mkdir -p $WORK/b001/：用于递归创建工作目录。它保证了工作目录一定存在。
(3) cat >$WORK/b001/importcfg.link << 'EOF' # internal，以及其后的多条 packagefile 语句，直至 EOF：用于生成名为 importcfg.link 的文件。最后，将所有需要链接的文件（.a 文件）写入 importcfg.link 中。
(4) mkdir -p $WORK/b001/exe/：用于在工作目录下创建一个名为 exe 的文件夹，该文件夹用于存放生成的可执行文件。
(5) /usr/local/go/pkg/tool/darwin_amd64/link -o $WORK/b001/exe/a.out -importcfg...：该指令执行链接操作，并生成 a.out 文件。
(6) /usr/local/go/pkg/tool/darwin_amd64/buildid -w $WORK/b001/exe/a.out：该指令用于更新 a.out 文件的 build ID。-w 选项代表要重写 a.out 中的 build ID。build ID 是可执行文件的独一无二的身份标识。
(7) mv $WORK/b001/exe/a.out first：将 a.out 重命名为 first，first 文件就是最终生成的可执行文件。

🔥：其中，命令选项-buildmode=exe 指定构建模式为 exe。在其中找到 buildmode=exe 的解释，可以看到，在该模式下会将 main 包以及所有导入包构建为一个可执行文件。这其实就是静态链接的过程。，在其中找到 buildmode=exe 的解释，可以看到，在该模式下会将 main 包以及所有导入包构建为一个可执行文件。这其实就是静态链接的过程。
