# go-chainlets

Chainlets is a tool that allows you to look up the mod graph (mostly generated by `go mod graph`) for dependency chains.

[![Go Report Card](https://goreportcard.com/badge/github.com/saedx1/go-chainlets)](https://goreportcard.com/report/github.com/saedx1/go-chainlets)
[![GoDoc](https://godoc.org/github.com/saedx1/go-chainlets?status.svg)](https://godoc.org/github.com/saedx1/go-chainlets)

This can be useful in multiple scenarios:

1. Discovering all the packages that has a certain dependency.
2. Tracking dependencies that might have security vulnerabilities.
3. Other scenarios we are not familiar; feel free to create a pull request to **enhance** this list.

## Installation

```sh
go get github.com/saedx1/go-chainlets
```

A binary should be created in your $GOPATH/bin

## Usage

First, you will need a file that contains the output of a `go mod graph`; for example:

```sh
go mod graph > mod.graph
```

Then, you have a couple ways of using the tool:

### Normal Mode (Detailed and More Verbose)

This is the default mode. Use it if you want to display the full chains that link your own packages to a certain package you specify. This can be more output than you expect; you might end up with hunderds of chains to the same package.

#### Example

Let's say you want to know the chains that end up with `golang.org/x/crypto@v0.0.0-20200622213623-75b288015ac9`, you would the following command (assuming you stored the graph in `mod.graph`):

```sh
go-chainlets mod.graph golang.org/x/crypto@v0.0.0-20190605123033-f99c8df09eb5
```

This will output something like this:

```sh
1) 1) github.com/saedx1/go-chainlets -> github.com/spf13/cobra@v1.1.3 -> github.com/spf13/viper@v1.7.0 -> github.com/bketelsen/crypt@v0.0.3-0.20200106085610-5cbc8cc4026c -> golang.org/x/crypto@v0.0.0-20190605123033-f99c8df09eb5
2) github.com/saedx1/go-chainlets -> github.com/spf13/cobra@v1.1.3 -> github.com/spf13/viper@v1.7.0 -> github.com/bketelsen/crypt@v0.0.3-0.20200106085610-5cbc8cc4026c -> cloud.google.com/go/firestore@v1.1.0 -> cloud.google.com/go@v0.46.3 -> cloud.google.com/go/datastore@v1.0.0 -> google.golang.org/appengine@v1.6.1 -> golang.org/x/crypto@v0.0.0-20190605123033-f99c8df09eb5
3) ...
```

### Quick Mode (Concice and Less Verbose)

If you don't care about knowing the full paths, and you want to know which packages you're directly **require**-ing in your `go.mod` that lead to a certain dependency (you only care about the first package after yours in the chain). The output is a set of unique packages (no duplicates). You would run it with the `--direct` flag (or `-d`):

```sh
go-chainlets mod.graph golang.org/x/crypto@v0.0.0-20190605123033-f99c8df09eb5 --direct
```

The output for this would be:

```sh
1) github.com/saedx1/go-chainlets
```

which makes sense, because that's the root package in all chains. To get around that you need to use the exclude flag `--exclude` (or `-e`) and specify pkg (or part of its name). All the edges where the specified package is at the left (dependent) will be removed; so, usually you would only use this with the root package, otherwise, if you exclude a package in the middle, you will have a unexpected results due to a disconnected graph (2 or more individual graphs).

```sh
go-chainlets mod.graph golang.org/x/crypto@v0.0.0-20190605123033-f99c8df09eb5 --direct -e go-chainlets
```

and the output would be like this:

```sh
1) github.com/spf13/cobra@v1.1.3
```

Which means that this version of `cobra` is causing you to have `golang.org/x/crypto@v0.0.0-20190605123033-f99c8df09eb5`. In other words, removing `cobra` from your dependencies will remove `x/crypto@...` from the dependencies as well.