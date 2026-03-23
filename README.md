# go-cli-holonization

Status : placeholder, not implemented.


**Patterns, documentation, and examples for adopting existing CLI tools
as holons in Go.**

This is not a language sdk. All code here builds on top of
[go-holons](https://github.com/organic-programming/go-holons), the core
Go SDK for building holons. **go-cli-holonization** focuses exclusively on
**CLI tools** — command-line programs that are invoked from a shell with
flags and arguments, read from `stdin`, write to `stdout`/`stderr`, and
communicate results through exit codes. Tools like `ffmpeg`, `git`, `curl`,
`jq`, or `kubectl` are typical examples.

This SDK captures the specific knowledge, conventions, and reusable patterns
needed to wrap such programs as holons, where the domain logic is not
written in Go but **delegated to an existing external binary**.
