# Setting up your Go environment

## Installing the Go Tools

To write Go code, you first need to [download](https://golang.org/dl) and install the Go development tools.

```shell
tar -C /usr/local -xzf go1.15.2.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> $HOME/.profile
source $HOME/.profile
```

## The Go Workspace

You are free to organize your projects as you see fit.
However, Go still expects there to be a single workspace for third-party Go tools installed via ``go install``.

By default, this workspace is located in ``$HOME/go``, with source code with these tools stored in ``$HOME/go/src`` and
the compiled binaries in ``$HOME/go/bin``. You can use this default or specify a different workspace by setting the
``$GOPATH`` environment variable.

> Some online resources tell you to set the ``$GOROOT`` environment variable.
This variable specifies the location where your Go development environment is installed.
This is no longer necessary; the go tool figures this out automatically.

Whether or not you use the default location, it’s a good idea to explicitly define ``GOPATH`` and to put the
``$GOPATH/bin`` directory in your executable path. Explicitly defining ``GOPATH`` makes it clear where your
Go workspace is located and adding ``$GOPATH/bin`` to your executable path makes it easier to run third-party tools
installed via ``go install``.

If you are on a Unix-like system using bash , add the following lines to your .profile.

```shell
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

You can get a complete list, along with a brief description of each variable, using the ``go env`` command.

## The Go Command

There are two similar commands available via Go: ``go run`` and ``go build``. Each takes
either a single Go file, a list of Go files, or the name of a package.

### go run

The ``go run`` command does in fact compile your code into a binary. However, the binary is built in a temporary directory.
The ``go run`` command builds the binary, executes the binary from that temporary directory, and then deletes the binary
after your program finishes.

```shell
go run hello.go
```

### go build

If you want a different name for your application, or if you want to store it in a different location, use the `-o` flag.

```shell
go build -o hello_world hello.go
```

## Getting Third-Party Go Tools

Go developers don’t rely on a centrally hosted service, like Maven Central for Java or the NPM registry for JavaScript.
Instead, they share projects via their source code repositories.
The `go install` command takes an argument, which is the location of the source code repository of the project,
followed by an @ and the version of the tool you want.


```shell
go install github.com/rakyll/hey@0.1.4
```

This downloads hey and all of its dependencies, builds the program, and installs the binary in your `$GOPATH/bin` directory.

If you want to update a tool to a newer version, rerun `go install` with the newer version specified or with `@latest`.

As we’ll talk about in “Module Proxy Servers” on page 200, the contents of Go repositories are cached in proxy servers.
Depending on the repository and the values in your `GOPROXY` environment variable,
`go install` may download from a proxy or directly from a repository.
If `go install` downloads directly from a repository, it relies on command-line tools being installed on your computer.
For example, you must have Git installed to download from GitHub.
