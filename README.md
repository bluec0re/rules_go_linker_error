Static PIE builds aren't working with rules_go:

```shell
$ mkdir -p /tmp/build_output && docker run -e USER="$(id -u)" -u="$(id -u)" -v "$PWD:/src/workspace" -v /tmp/build_output:/tmp/build_output -w /src/workspace gcr.io/bazel-public/bazel:latest --output_user_root=/tmp/build_output \
build :hello

Starting local Bazel server and connecting to it...
Loading: 
Loading: 0 packages loaded
Analyzing: target //:hello (1 packages loaded, 0 targets configured)
INFO: Analyzed target //:hello (51 packages loaded, 9138 targets configured).
 checking cached actions
INFO: Found 1 target...
[1 / 11] [Prepa] BazelWorkspaceStatusAction stable-status.txt
ERROR: /src/workspace/BUILD.bazel:3:10: GoLink hello_/hello failed: (Exit 1): builder failed: error executing command (from target //:hello) bazel-out/k8-opt-exec-2B5CBBC6/bin/external/go_sdk/builder_reset/builder link -sdk external/go_sdk -installsuffix linux_amd64 -buildmode pie -package_list ... (remaining 17 arguments skipped)

Use --sandbox_debug to see verbose messages from the sandbox and retain the sandbox build root for debugging
external/go_sdk/pkg/tool/linux_amd64/link: running /usr/bin/gcc failed: exit status 1
/usr/bin/ld.gold: --no-dynamic-linker: unknown option
/usr/bin/ld.gold: use the --help option for usage information
collect2: error: ld returned 1 exit status

link: error running subcommand external/go_sdk/pkg/tool/linux_amd64/link: exit status 2
Target //:hello failed to build
Use --verbose_failures to see the command lines of failed build steps.
INFO: Elapsed time: 4.047s, Critical Path: 0.49s
INFO: 2 processes: 2 internal.
FAILED: Build did NOT complete successfully
```

This works:

```shell
$ mkdir -p /tmp/build_output/.cache && docker run -e USER="$(id -u)" -u="$(id -u)" -v "$PWD:/src/workspace" -v /tmp/build_output:/tmp/build_output -v /tmp/build_output/.cache:/.cache -w /src/workspace gcr.io/bazel-public/bazel:latest --output_user_root=/tmp/build_output \
run @go_sdk//:bin/go -- build -o /tmp/build_output/hello -ldflags '-extldflags -static-pie' /src/workspace/hello.go

Starting local Bazel server and connecting to it...
Loading: 
Loading: 0 packages loaded
Analyzing: target @go_sdk//:bin/go (1 packages loaded, 0 targets configured)
INFO: Analyzed target @go_sdk//:bin/go (1 packages loaded, 1 target configured).
INFO: Found 1 target...
[0 / 1] [Prepa] BazelWorkspaceStatusAction stable-status.txt
WARNING: /tmp/build_output/a08c2e4811c846650b733c6fc815a920/external/go_sdk/BUILD.bazel:47:7: @go_sdk//:bin/go is a source file, nothing will be built for it. If you want to build a target that consumes this file, try --compile_one_dependency
INFO: Elapsed time: 2.117s, Critical Path: 0.03s
INFO: 1 process: 1 internal.
INFO: Build completed successfully, 1 total action
INFO: Running command line: /tmp/build_output/a08c2e4811c846650b733c6fc815a920/external/go_sdk/bin/go build -o /tmp/build_output/hello -ldflags '-extldflags -static-pie' /src/workspace/hello.go

$ /tmp/build_output/hello     
Hello world!
```