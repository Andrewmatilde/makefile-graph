# makefile-graph

[![Build Status](https://github.com/dnaeon/makefile-graph/actions/workflows/test.yaml/badge.svg)](https://github.com/dnaeon/makefile-graph/actions/workflows/test.yaml/badge.svg)
[![Go Reference](https://pkg.go.dev/badge/github.com/dnaeon/makefile-graph.svg)](https://pkg.go.dev/github.com/dnaeon/makefile-graph)
[![Go Report Card](https://goreportcard.com/badge/github.com/dnaeon/makefile-graph)](https://goreportcard.com/report/github.com/dnaeon/makefile-graph)
[![codecov](https://codecov.io/gh/dnaeon/makefile-graph/branch/master/graph/badge.svg)](https://codecov.io/gh/dnaeon/makefile-graph)

`makefile-graph` is a Go module and CLI application, which parses
[GNU Make](https://www.gnu.org/software/make/)'s internal database and generates a
graph representing the relationships between the discovered Makefile targets.

## Requirements

* [GNU Make](https://www.gnu.org/software/make/)
* Go version 1.21.x or later

## Installation

You can install the CLI application using one of the following ways.

If you have cloned the repository you can build the CLI app using the provided
Makefile target.

``` shell
make build
```

The resulting binary will be located in `bin/makefile-graph`.

Install the CLI application using `go install`.

``` shell
go install github.com/Andrewmatilde/makefile-graph/cmd/makefile-graph@latest
```

In order to install the parser package and use it in your own Go code run the
following command within your Go module.

``` shell
go get -v github.com/Andrewmatilde/makefile-graph/pkg/parser
```

## Usage

Let's use the following [example
Makefile](https://www.gnu.org/software/make/manual/html_node/Simple-Makefile.html)
from [GNU make's documentation](https://www.gnu.org/software/make/manual/make.html).

``` makefile
edit : main.o kbd.o command.o display.o \
       insert.o search.o files.o utils.o
	cc -o edit main.o kbd.o command.o display.o \
		   insert.o search.o files.o utils.o

main.o : main.c defs.h
	cc -c main.c
kbd.o : kbd.c defs.h command.h
	cc -c kbd.c
command.o : command.c defs.h command.h
	cc -c command.c
display.o : display.c defs.h buffer.h
	cc -c display.c
insert.o : insert.c defs.h buffer.h
	cc -c insert.c
search.o : search.c defs.h buffer.h
	cc -c search.c
files.o : files.c defs.h buffer.h command.h
	cc -c files.c
utils.o : utils.c defs.h
	cc -c utils.c
clean :
	rm edit main.o kbd.o command.o display.o \
	   insert.o search.o files.o utils.o
```

You can also find this example Makefile in the [examples](./examples/) directory
of this repo.

Running the following command will generate the [Dot
representation](https://graphviz.org/doc/info/lang.html) for the Makefile
targets and their dependencies from our example Makefile.

``` shell
makefile-graph --makefile examples/Makefile --direction TB
```

In order to render the graph you can pipe it directly to the
[dot command](https://graphviz.org/doc/info/command.html), e.g.

``` makefile
makefile-graph --makefile examples/Makefile --direction TB | dot -Tsvg -o graph.svg
```

This is what the graph looks like when we render it using `dot(1)`.

![Example Makefile Graph](./images/image-1.svg)

Sometimes when rendering large graphs it may not be obvious at first glance what
are the dependencies for a specific target. In order to help with such
situations `makefile-graph` supports a flag which allows you to highlight
specific targets and their dependencies.

The following command will highlight the `files.o` target along with it's
dependencies.

``` makefile
makefile-graph \
    --makefile examples/Makefile \
    --direction TB \
    --target files.o \
    --highlight \
    --highlight-color lightgreen
```

When we render the output from above command we will see this graph
representation.

![Example Makefile Graph Highlighted](./images/image-2.svg)

If we want to focus on a specific target and it's dependencies only we can use
the following command, which will generate a graph only for the target and it's
dependencies.

``` makefile
makefile-graph \
    --makefile examples/Makefile \
    --direction TB \
    --target files.o \
    --related-only
```

This is what the resulting graph looks like.

![Example Makefile Graph Related Only](./images/image-3.svg)

The `--direction` option is used for specifying the [direction of graph
layout](https://graphviz.org/docs/attrs/rankdir/). You can set it to `TB`, `BT`,
`LR` or `RL`.

The `--format` option is used for specifying the output format for the graph. By
default it will produce the `dot` representation for the graph.

You can also view the [topological
order](https://en.wikipedia.org/wiki/Topological_sorting) for a given target by
setting the format to `tsort`, e.g.

``` makefile
makefile-graph \
    --makefile examples/Makefile \
    --target files.o \
    --format csv
```

Running above command produces the following output, which represents the
topological order for the `files.o` target.

``` text
images/dev-env/.dockerbuilt , images/dev-env/Dockerfile
GOLOBAL , images/dev-env/Dockerfile
tidy , images/dev-env/.dockerbuilt
GOLOBAL , images/dev-env/.dockerbuilt
GOLOBAL , tidy
chaos-build , bin/chaos-builder
GOLOBAL , bin/chaos-builder
generate-deepcopy , chaos-build
GOLOBAL , chaos-build
GOLOBAL , generate-deepcopy
e2e-test/image/e2e/chaos-mesh , helm/chaos-mesh
GOLOBAL , helm/chaos-mesh
GOLOBAL , e2e-test/image/e2e/chaos-mesh
GOLOBAL , e2e-test/image/e2e/bin/ginkgo
GOLOBAL , clean-image-built
images/build-env/.dockerbuilt , images/build-env/Dockerfile
GOLOBAL , images/build-env/Dockerfile
images/chaos-daemon/bin/pause , images/build-env/.dockerbuilt
GOLOBAL , images/build-env/.dockerbuilt
images/chaos-daemon/bin/pause , hack/pause.c
GOLOBAL , hack/pause.c
GOLOBAL , images/chaos-daemon/bin/pause
pkg/time/fakeclock/fake_gettimeofday.o , pkg/time/fakeclock/fake_gettimeofday.c
GOLOBAL , pkg/time/fakeclock/fake_gettimeofday.c
watchmaker , pkg/time/fakeclock/fake_gettimeofday.o
GOLOBAL , pkg/time/fakeclock/fake_gettimeofday.o
pkg/time/fakeclock/fake_clock_gettime.o , pkg/time/fakeclock/fake_clock_gettime.c
GOLOBAL , pkg/time/fakeclock/fake_clock_gettime.c
watchmaker , pkg/time/fakeclock/fake_clock_gettime.o
GOLOBAL , pkg/time/fakeclock/fake_clock_gettime.o
GOLOBAL , watchmaker
e2e-test/cmd/e2e_helper/.dockerbuilt , e2e-test/cmd/e2e_helper/Dockerfile
GOLOBAL , e2e-test/cmd/e2e_helper/Dockerfile
image-e2e-helper , e2e-test/cmd/e2e_helper/.dockerbuilt
GOLOBAL , e2e-test/cmd/e2e_helper/.dockerbuilt
GOLOBAL , image-e2e-helper
ui , pnpm_install_dependencies
GOLOBAL , pnpm_install_dependencies
images/chaos-dashboard/bin/chaos-dashboard , ui
GOLOBAL , ui
images/chaos-dashboard/bin/chaos-dashboard , image-build-env
GOLOBAL , image-build-env
GOLOBAL , images/chaos-dashboard/bin/chaos-dashboard
GOLOBAL , swagger_spec
GOLOBAL , e2e-test/image/e2e/bin/e2e.test
schedule-migration.tar.gz , schedule-migration
GOLOBAL , schedule-migration
GOLOBAL , schedule-migration.tar.gz
clean , clean-binary
GOLOBAL , clean-binary
GOLOBAL , clean
GOLOBAL , help
images/chaos-dashboard/.dockerbuilt , images/chaos-dashboard/Dockerfile
GOLOBAL , images/chaos-dashboard/Dockerfile
image-chaos-dashboard , images/chaos-dashboard/.dockerbuilt
GOLOBAL , images/chaos-dashboard/.dockerbuilt
image , image-chaos-dashboard
GOLOBAL , image-chaos-dashboard
images/chaos-mesh/.dockerbuilt , images/chaos-mesh/Dockerfile
GOLOBAL , images/chaos-mesh/Dockerfile
images/chaos-mesh/.dockerbuilt , images/chaos-mesh/bin/chaos-controller-manager
GOLOBAL , images/chaos-mesh/bin/chaos-controller-manager
image-chaos-mesh , images/chaos-mesh/.dockerbuilt
GOLOBAL , images/chaos-mesh/.dockerbuilt
image , image-chaos-mesh
GOLOBAL , image-chaos-mesh
images/chaos-daemon/.dockerbuilt , images/chaos-daemon/Dockerfile
GOLOBAL , images/chaos-daemon/Dockerfile
images/chaos-daemon/.dockerbuilt , images/chaos-daemon/bin/cdh
GOLOBAL , images/chaos-daemon/bin/cdh
images/chaos-daemon/.dockerbuilt , images/chaos-daemon/bin/chaos-daemon
GOLOBAL , images/chaos-daemon/bin/chaos-daemon
image-chaos-daemon , images/chaos-daemon/.dockerbuilt
GOLOBAL , images/chaos-daemon/.dockerbuilt
image , image-chaos-daemon
GOLOBAL , image-chaos-daemon
GOLOBAL , image
GOLOBAL , chaosctl
GOLOBAL , image-dev-env
generate , generate-ctrl
GOLOBAL , generate-ctrl
manifests/crd.yaml , config
GOLOBAL , config
generate , manifests/crd.yaml
GOLOBAL , manifests/crd.yaml
GOLOBAL , generate
GOLOBAL , proto
e2e , e2e-build
GOLOBAL , e2e-build
GOLOBAL , e2e
GOLOBAL , failpoint-enable
images/chaos-dlv/.dockerbuilt , images/chaos-dlv/Dockerfile
GOLOBAL , images/chaos-dlv/Dockerfile
image-chaos-dlv , images/chaos-dlv/.dockerbuilt
GOLOBAL , images/chaos-dlv/.dockerbuilt
image-all , image-chaos-dlv
GOLOBAL , image-chaos-dlv
images/chaos-kernel/.dockerbuilt , images/chaos-kernel/Dockerfile
GOLOBAL , images/chaos-kernel/Dockerfile
image-chaos-kernel , images/chaos-kernel/.dockerbuilt
GOLOBAL , images/chaos-kernel/.dockerbuilt
image-all , image-chaos-kernel
GOLOBAL , image-chaos-kernel
e2e-test/image/e2e/.dockerbuilt , e2e-test/image/e2e/Dockerfile
GOLOBAL , e2e-test/image/e2e/Dockerfile
e2e-test/image/e2e/manifests , manifests
GOLOBAL , manifests
e2e-test/image/e2e/.dockerbuilt , e2e-test/image/e2e/manifests
GOLOBAL , e2e-test/image/e2e/manifests
image-chaos-mesh-e2e , e2e-test/image/e2e/.dockerbuilt
GOLOBAL , e2e-test/image/e2e/.dockerbuilt
image-all , image-chaos-mesh-e2e
GOLOBAL , image-chaos-mesh-e2e
GOLOBAL , image-all
multithread_tracee , test/cmd/multithread_tracee/main.c
GOLOBAL , test/cmd/multithread_tracee/main.c
test-utils , multithread_tracee
GOLOBAL , multithread_tracee
test-utils , timer
GOLOBAL , timer
test , test-utils
GOLOBAL , test-utils
GOLOBAL , test
GOLOBAL , bin/chaos-controller-manager
GOLOBAL , generate-makefile
GOLOBAL , gosec-scan
GOLOBAL , enter-buildenv
GOLOBAL , all
GOLOBAL , local/chaos-dashboard
GOLOBAL , enter-devenv
check , helm-values-schema
GOLOBAL , helm-values-schema
check , install.sh
GOLOBAL , install.sh
check , fmt
GOLOBAL , fmt
check , lint
GOLOBAL , lint
check , vet
GOLOBAL , vet
GOLOBAL , check
GOLOBAL , failpoint-disable
GOLOBAL , clean-local-binary
GOLOBAL , local/chaos-controller-manager
GOLOBAL , boilerplate
GOLOBAL , bin/chaos-dashboard
GOLOBAL , coverage
GOLOBAL , e2e-image
```

## Tests

Run the tests.

``` shell
make test
```

Run test coverage.

``` shell
make test-cover
```

## Contributing

`makefile-graph` is hosted on
[Github](https://github.com/dnaeon/makefile-graph). Please contribute by
reporting issues, suggesting features or by sending patches using pull requests.

## License

`makefile-graph` is Open Source and licensed under the [BSD
License](http://opensource.org/licenses/BSD-2-Clause).
