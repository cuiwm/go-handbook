# The Go Cloudflare Handbook

A collection of Go style and code review notes.

## Flags

The flags package lends itself to a lot of misuse of global state.

In a main package, flags SHOULD be defined and registered in the `main()` function. This enforces proper argument passing, which eventually improves testability and readability.

Libraries MUST NOT register flags in the global context or in the `init()` function. Since flags are registered in a global context, the user wouldn't be able to import two libraries that define a `-n` flag. Also, a user should be able to import a library without exposing its flags at all.

Instead, libraries SHOULD expose a `RegisterFlags(*flag.FlagSet)` function that registers the flags on the passed FlagSet, or on the global one of `fs` is nil.
