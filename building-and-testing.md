# 🏗 Building & Testing

WS4SQlite is distributed with binaries for the major combinations of OSs and architectures, or better, the ones I can access ;-). I'm trying to do portable builds as much as possible. For these reasons, building ws4sqlite is generally not required, but it could be useful for those archs I don't have access to, and to ensure that the binary matches the sources.

### Supported platforms

These are platforms for which we'll provide binaries at the time of the release.

| OS             | Arch    | Notes                                                                              |
| -------------- | ------- | ---------------------------------------------------------------------------------- |
| Linux          | x86-64  | Static build for cross-distro compatibility                                        |
| Linux          | armv7hf | Static build for cross-distro compatibility                                        |
| Linux          | aarch64 | Static build for cross-distro compatibility                                        |
| Windows        | x86-64  |                                                                                    |
| MacOS (darwin) | x86-64  |                                                                                    |
| MacOS (darwin) | arm64   | Availability may take some days after release - credits to D. Quinto for packaging |

If you have access to an unsupported architecture, and would like to contribute the binary, please write me (oss /AT/ germanorizzo /DOT/ it).

### Building

ws4sqlite is a Go(lang) program, that uses Go 1.17 and CGO (to compile an embedded SQLite). If you are not familiar with this, please read a guide on [compiling with CGO](https://go.dev/doc/install/gccgo); the Go toolset makes it very convenient, but there are some prerequisites.

* Go 1.17
* Make
* A compile toolchain (e.g. GCC)

I included a Make file to script the building under Linux, so if you have all the prerequisites it should be a matter of:

```bash
git clone https://github.com/proofrock/ws4sqlite
cd ws4sqlite
make build
# You will find the binary in the bin/ directory.
```

For MacOS replace Line 3 with:

```bash
make build-nostatic
```

For Windows, instead of the step at Line 3 do:

```bash
cd src
go build
```

And this should be all.

### Testing

The code includes unit tests. Use the appropriate target for make:

```bash
make do-test
```

If you don't use this method, beware that there is a test that takes two minutes to complete; be sure to provide a suitable timeout value.
