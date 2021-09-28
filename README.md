# Lon[eng] Go Club

### Table of contents
- [Getting Started](#getting-started)
  - [Installation](#installation)
  - [Workspace](#workspace)
    - [Environment variables](#environment-variables)
      - [Important environment variables](#important-environment-variables)
      - [Example configurations](#example-configurations)
- [Packages & Modules](#packages--modules)  
  - [Definitions](#definitions)
  - [Workflow](#workflow)
    - [Initialise a module](#initialise-a-module)
      - [Naming conventions](#naming-conventions)
    - [Build & Run your program](#build--run-your-program)
      - [Order of execution](#order-of-execution)
    - [Add a dependency](#add-a-dependency)
      - [Useful commands](#useful-commands)
  - [Additional features](#additional-features)
    - [Versioning](#versioning)
    - [The init() function](#the-init-function)
    - [Imports](#imports)
    - [Exports](#exports)

## Getting started

### Installation 
On macOS, you can install Go with Homebrew by running:
```bash
$ brew install go
```

To test the installation, restart your terminal and run:
```bash
$ which go
```
This should print the location of the Go binary: `/usr/local/bin/go`.

`/usr/local/bin` being in your `$PATH`, you can now check your Go version by running: 
```bash
$ go version
```

To learn more about the various functionalities of the Go CLI, running:
```bash
$ go help
$ go help <command_name>
```

### Workspace
Before the introduction of `modules`, Go worked with a strict directory hierarchy which formed the **workspace**. This allows Go to manage our projects' source files, compiled binaries and downloaded dependencies. These directories are found under the `$GOPATH`, which defaults to `$HOME/go` but can but updated to your liking. 3 directories form the workspace:
- `src`: where all your `.go` files are located, as well as the projects' downloaded dependencies
- `bin`: where Go puts your programs' compiled binaries
- `pkg`: where Go stores pre-compiled object files to speed up subsequent compiling of your programs

As we'll see in Packages & Modules, Go modules make this framework much less rigid and introduce better dependency management.

#### Environment variables
Go uses a set of environment variables you can see by running: 
```bash
$ go env
```

##### Important environment variables
| Name | Description |
| ---- | ------------|
| **GOROOT** | Location of the Go SDK. |
| **GOPATH** | Location of the Go workspace, used to resolve imports. It contains: a `/src` directory where all the `.go` files and downloaded packages are stored; a `/pkg` folder where the compiled output from code present in `/src` is stored, under subdirectories based on OS and Architecture targets; a `/bin` directory contains the executable binaries compiled by linking packages from `/pkg`. |
| **GOBIN** | Location of the Go executable binaries. Add it to your `$PATH` so installed binaries can be run without specifying the full path. |
| **GO111MODULE** | Determines how Go should import packages: using modules if set to `on`, or gopath if `off`.
| **GOMODCACHE** | Location of the downloaded packages source code when using Go modules. |

##### Example configurations
Place this in your `~/.zshrc` or `~/.bashrc` file:
```bash
export GOPATH="$HOME/go"
export GOBIN="$GOPATH/bin"
export GO111MODULE=on
export GOMODCACHE="$GOPATH/pkg/mod"
export PATH="$PATH:$GOBIN"
```

## Packages & Modules

### Definitions
A `package` is a collection of `.go` files starting with the same package name declaration at the top of the file, and living under the same directory:
```go
package goclub
```
> Each directory can only contain 1 package. But packages can be nested in a directory structure.

There are 2 types of packages:
- **Executable**: Only the package called `main` is executable, because it contains the special `main()` function. This is the function that is run when the package is compiled and run.
- **Utility**: All other packages are not self-executable and are meant to be imported by other packages to provide functions, variables etc..

Introduced in Go 1.11, `modules` are Go support for dependency management. A module is a collection of packages with a `go.mod` file at its root. This file defines 3 things:
1. the module's `import path` used by other modules to import it and use its packages
2. the `go version` used to compile the module
3. the module's `dependencies` requirements (including versions)

```go
module github.com/bcgdv/school

go 1.17

require (
  rsc.io/quote v1.5.2
)
```

> Similar to packages, **executable** modules are those containing the `main package`, other modules are simply **utility** modules.

> The module import path is the prefix that will be used by other modules when importing your module's packages (i.e., `import module_import_path/package_name`).

### Workflow

#### Initialise a module

Outside of your `$GOPATH`, run:
```bash
$ mkdir school && cd school
$ go mod init github.com/bcgdv/school
```
This will generate the `go.mod` file for you.

You can now create a simple executable package that prints a message:
```bash
$ mkdir goclub && touch goclub/main.go 
```

```go
package main

import "fmt"

func main() {
  fmt.Println("Hello world")
}
```

> It's worth noting that with Go modules, the name of the directories and files don't matter - what's important is the name of the package. So the `main package` could live in a file/directory of a different name.

#### Naming conventions
`Packages` are named with 1 word with no underscore or camelcase (more info [here](https://go.dev/blog/package-names)).

For `modules`, there are 2 usecases:
- If it's a utility module meant to be published (for others to import): the module import path should match the URL of the Version Control System where it is hosted (e.g., github.com/bcgdv/school), so Go can download it.
- If it's a utility module not meant to be published, or an executable module, the name doesn't matter as much.

#### Build & Run your program
Simply run:
```bash
$ go run goclub/main.go
```
This will compile and run your program straight away.

If you only want to create an executable binary without running your program, you can either run `go install` which will place the binary in your `$GOBIN` directory, or `go build` which will place the binary in your current directory. With the former you can run your program from anywhere by its name (as `$GOBIN` should be in your `$PATH`) like `goclub`. With the latter, you need to use the relative path to run your program like `./goclub`.

##### Order of execution
1. Starts with the `main` package
2. Initialises imported packages (recursively, starting from the bottom of the import tree)
3. Initialises global variables
4. Runs all `init()` functions (seen below)
5. Runs `main()` function in `main` package

#### Add a dependency
Let's add a package that exports a function to generate random quotes:
```bash
$ go get github.com/rsc/quote
```
This will add the requirement to your `go.mod` file, and add the dependency checksum to a `go.sum` file at the root of your module.

The downloaded dependency is placed in your `$GOMODCACHE` directory, under the `/cache` directory.

You can now use it in your code:
```go
package main

import (
  "fmt"
  
  "rsc.io/quote"
)

func main() {
  fmt.Println(quote.Hello())
}
```

##### Useful commands
| Command | Description |
| ------- | ----------- |
| **go get -u <package_name>** | Upgrades the specific package to its latest minor version |
| **go get ./...** | Upgrades all dependencies to their latest minor version |
| **go get <package_name>@version** | Downloads a package to a specific version |
| **go mod tidy** | Looks at your module's `.go` files, downloads all missing imports and remove unused dependencies from the `go.mod` file |
| **go mod download** | Looks at the `go.mod` file and downloads all the dependencies (e.g., after pulling your team's project) |

### Additional features

#### Versioning
Go modules support semantic versioning. To increase the `minor` or `patch` version of your module, rename its import path in your `go.mod` file as follows:
```go
module github.com/bcgdv/school v1.2.0
```

When upgrading a project dependencies, Go only downloads the latest `minor` and `patch` versions (e.g., from v1.1.0 to v1.2.0, even if there is a v2.0.0 available). This is because Go considers `major` versions as entirely separate modules. So if you want to increase it, rename your module as follows:
```go
module github.com/bcgdv/school/v2 v2.0.0
```
Other projects will import your v2 module with `github.com/bcgdv/school/v2`. 

For the 1st version of a module, it is common not to specify it in the import path.

#### The init() function
You can add 1 or more `init()` function(s) in your packages' `.go` files. They will always be run automatically at package initialisation and are often use to set global variables that can't be hardcoded (e.g., need a network request). This function takes no argument and cannot be called explicitely.

#### Imports
You can use `aliases` when importing a package. This is useful when 2 packages have the same method names:
```go
import bestgoclub "github.com/bcgdv/school/goclub"
```

You can also use `blank identifiers`, often used because Go doesn't allow unused variables, but sometimes you need to import a package just so its `init()` functions run and sets global variables (i.e., side effects).
```go
import _ "github.com/go-sql-driver/mysql"
```

#### Exports
Go doesn't have `public`, `private`, `protected` keywords (because it doesn't have classes in the OOP sense). 

Exported variables, functions and structs hence need to be capitalised to be called outside of their package.

> It's worth noting that inside the same package, variables, functions, structs can be lowercased and used without imports even if split in different files.
