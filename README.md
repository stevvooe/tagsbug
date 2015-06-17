This package demonstrates a possible bug with listing and testing go packages
that use tags to enable or disable all files. This only occurs when doing
relative listings (ie "./...").

The behavior has been broken down into a few example packages. This can be
confirmed by retrieving this repo with the following command:

```
go get github.com/stevvooe/tagsbug
```

Each package provides an example of the different behavior.

## __`pkg1`__

This is out "control" package. It should always build, list and test as a
normal package.

## __`pkg2`__

This demonstrates the main issue. All of the Go files are excluded by default
using build tags. With or without build tags specified, `pkg2` can never be
tested or listed.

You'll have to change directory into the source directory to reproduce this
issue.

Without build tags:

```console
$ go list ./...
github.com/stevvooe/tagsbug
github.com/stevvooe/tagsbug/pkg1
github.com/stevvooe/tagsbug/pkg3
```

```console
$ go test ./...
?     github.com/stevvooe/tagsbug   [no test files]
?     github.com/stevvooe/tagsbug/pkg1 [no test files]
?     github.com/stevvooe/tagsbug/pkg3 [no test files]
```

This behavior is somewhat expected. `pkg2` is not really enabled, according
to our tags. The problem appears when we run the same, including the build
tag:

```console
$ go list -tags pkg2 ./...
github.com/stevvooe/tagsbug
github.com/stevvooe/tagsbug/pkg1
github.com/stevvooe/tagsbug/pkg3
$ go test -tags pkg2 ./...
?     github.com/stevvooe/tagsbug   [no test files]
?     github.com/stevvooe/tagsbug/pkg1 [no test files]
?     github.com/stevvooe/tagsbug/pkg3 [no test files]
```

In both of the above cases, we would expect `pkg2` to be enabled and the tests
should work.

Note that this works fine with a fully qualified path:

```
$ go list -tags pkg2 github.com/stevvooe/tagsbug/...
github.com/stevvooe/tagsbug
github.com/stevvooe/tagsbug/pkg1
github.com/stevvooe/tagsbug/pkg2
github.com/stevvooe/tagsbug/pkg3
$ go test -tags pkg2 github.com/stevvooe/tagsbug/...
?     github.com/stevvooe/tagsbug   [no test files]
?     github.com/stevvooe/tagsbug/pkg1 [no test files]
ok    github.com/stevvooe/tagsbug/pkg2 0.003s
?     github.com/stevvooe/tagsbug/pkg3 [no test files]
```

The above is roughly the behavior we expect, except using a the relative
package path.

`go build` is also affected:

```
$ go build -v -tags pkg2 ./...
github.com/stevvooe/tagsbug
github.com/stevvooe/tagsbug/pkg1
github.com/stevvooe/tagsbug/pkg3
```

And works correctly with the full import path:

```
$ go build -v -tags pkg2 github.com/stevvooe/tagsbug/...
github.com/stevvooe/tagsbug
github.com/stevvooe/tagsbug/pkg1
github.com/stevvooe/tagsbug/pkg2
github.com/stevvooe/tagsbug/pkg3
```

## __`pkg3`__

This demonstrates the workaround. We simply add a Go source file with a
package declaration and everything works as expected.

```
$ go list -tags pkg3 ./...
github.com/stevvooe/tagsbug
github.com/stevvooe/tagsbug/pkg1
github.com/stevvooe/tagsbug/pkg3
$ go test -tags pkg3 ./...
?     github.com/stevvooe/tagsbug   [no test files]
?     github.com/stevvooe/tagsbug/pkg1 [no test files]
ok    github.com/stevvooe/tagsbug/pkg3 0.003s
```

Note that the `pkg3` tests are now running.