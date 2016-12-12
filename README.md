# The Cloudflare Go Handbook

A collection of Go style, code review and project management notes.

NOTE: this is still a very early work in progress.

## GOPATH / go get / Makefiles

Projects SHOULD use [hellogopher](https://gist.github.com/FiloSottile/7e0cceb56d4c96ade45ca3deab2ae444) to be both `go get`-able and buildable with `make` from anywhere on the filesystem.

Libraries MUST be `go get`-able.

## Vendoring

Use `gvt` or `glide`. DO check into git the entire `vendor/` folder for projects, DO NOT check in `vendor/` for libraries (but you're free to check in `vendor/manifest` if you like).

See also the *Vendoring* section of the hellogopher README.

## go vet

Run `go vet` as part of testing or building. All `go vet` errors MUST be considered as test/build failures and fixed, including false positives.

Hellogopher will run `go vet` as part of `make test`, and fail on errors.

## Flags

The flags package lends itself to a lot of misuse of global state.

In a main package, flags SHOULD be defined and registered in the `main()` function. This enforces proper argument passing, which eventually improves testability and readability.

Libraries MUST NOT register flags in the global context or in the `init()` function. Since flags are registered in a global context, the user wouldn't be able to import two libraries that define a `-n` flag, which is surprising and uncontrollable. Also, a user should be able to import a library without exposing its flags at all.

Instead, libraries SHOULD expose a `RegisterFlags(*flag.FlagSet)` function that registers the flags on the passed FlagSet, or on the global one of `fs` is nil. The flag variables SHOULD be public, so that in case of a conflict (or at will) the user can register or set them differently.

### Examples

```go
package handbook

import "flag"

// Variables that the flags will populate if RegisterFlags is used.
var (
    Kill *bool // violently enforce the handbook
)

func RegisterFlags(fs *flag.FlagSet) {
    if fs == nil {
        fs = flag.CommandLine
    }
    Kill = flag.Bool("kill", false, "violently enforce the handbook")
}
```

```go
package handbook

import "flag"

type Handbook struct {
    // Fields that the flags will populate if RegisterFlags is used.
    Kill bool // violently enforce the handbook
    
    // ...
}

func (h *Handbook) RegisterFlags(fs *flag.FlagSet) {
    if fs == nil {
        fs = flag.CommandLine
    }
    flag.BoolVar(&h.Kill, "kill", false, "violently enforce the handbook")
}
```
