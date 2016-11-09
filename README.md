# The Go Cloudflare Handbook

A collection of Go style and code review notes.

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
