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

