---
# Cartouche v1
title: "go-cli-holonization — CLI Adoption SDK"
author:
  name: "B. ALTER"
  copyright: "© 2026 Benoit Pereira da Silva"
created: 2026-02-25
revised: 2026-02-25
lang: en-US
origin_lang: en-US
translation_of: null
translator: null
access:
  humans: true
  agents: true
status: draft
---

# go-cli-holonization

**Patterns, documentation, and examples for adopting existing CLI tools
as holons in Go.**

This is not a standalone library. All code here builds on top of
[go-holons](https://github.com/organic-programming/go-holons), the core
Go SDK for building holons. **go-cli-holonization** focuses exclusively on
**CLI tools** — command-line programs that are invoked from a shell with
flags and arguments, read from `stdin`, write to `stdout`/`stderr`, and
communicate results through exit codes. Tools like `ffmpeg`, `git`, `curl`,
`jq`, or `kubectl` are typical examples.

This SDK captures the specific knowledge, conventions, and reusable patterns
needed to wrap such programs as holons, where the domain logic is not
written in Go but **delegated to an existing external binary**.

---

## Vision

The command-line ecosystem — `ffmpeg`, `git`, `curl`, `jq`, `imagemagick`,
`pandoc`, `kubectl` — represents decades of battle-tested, domain-specific
software. Rewriting any of it would be wasteful. Adopting it is the right move.

**go-cli-holonization** provides the Go primitives to wrap any CLI program
(or individual subcommand) as a full holon — with a `.proto` contract,
a gRPC server, `serve`/`dial` mechanics, and the complete facet set defined
by the Organic Programming constitution.

### Why a dedicated SDK?

Holonizing a CLI is structurally different from writing a holon from scratch:

| Concern | Native holon | Adopted CLI holon |
|---|---|---|
| **Domain logic** | Written in Go | Delegated to the external binary |
| **I/O** | In-process function calls | `exec.Command` → stdout/stderr parsing |
| **Error model** | Go errors / gRPC status | Exit codes + stderr patterns |
| **Streaming** | Native gRPC streams | Line-by-line stdout capture |
| **Lifecycle** | Direct control | Process spawning, signal forwarding |

These concerns recur identically for every adopted tool. **go-cli-holonization**
extracts them into reusable building blocks so that each adoption reduces to:

1. Define the `.proto` contract.
2. Implement a thin mapping layer (request → flags, stdout → response).
3. Done. Transport, reflection, lifecycle, and error translation are handled.

### The bridge to the old world

The CLI facet is an inheritance from Unix philosophy. It serves as a
**compatibility bridge** — connecting holons to the broader ecosystem of CI
pipelines, shell scripts, and human operators who work in terminals.

For every `rpc` service defined in a holon's contract, and every function
exposed through the external code facet, there is a **strictly aligned CLI
subcommand**. This 1:1 alignment is enforced: what the contract declares,
the CLI exposes — no more, no less.

As the organic ecosystem matures and holons compose directly at runtime
through `serve`/`dial`, the CLI facet will naturally become less
critical. But today it remains essential for interoperability with the
existing world.

---

## Scope

**go-cli-holonization** provides:

### Process execution

- **Command builder** — translate a gRPC request message into CLI flags,
  arguments, and environment variables.
- **Process runner** — spawn the target binary, capture stdout/stderr,
  forward signals (`SIGTERM`, `SIGINT`), enforce timeouts.
- **Stream adapter** — convert line-by-line stdout into a gRPC
  `stream` response for long-running commands (`git log`, `tail -f`,
  `docker logs`).

### Output parsing

- **JSON mapper** — for tools that support structured output
  (`ffprobe -print_format json`, `kubectl -o json`), parse directly
  into protobuf messages.
- **Text parser** — for tools that produce tabular or free-text output,
  provide pattern-based extraction helpers.
- **Exit code translation** — map process exit codes and stderr patterns
  to appropriate gRPC status codes.

### Lifecycle and transport

- **Serve/Dial integration** — adopted holons get the same `serve`
  and `dial` mechanics as native holons, using the standard
  [go-holons](https://github.com/organic-programming/go-holons) SDK
  for transport.
- **Reflection** — server reflection is enabled by default, as
  required by the constitution.
- **Graceful shutdown** — `SIGTERM` handling with clean process
  teardown of spawned binaries.

### Testing helpers

- **Round-trip harness** — spin up the adopted holon, call its RPCs,
  verify responses against expected output.
- **Binary mock** — swap the real binary for a test double that
  returns controlled stdout/stderr/exit codes.

---

## Intended usage

```go
package main

import (
    "github.com/organic-programming/go-cli-holonization/adopt"
    "github.com/organic-programming/go-cli-holonization/exec"
)

// The adoption layer: map gRPC requests to CLI invocations.
func (s *FFProbeServer) Probe(ctx context.Context, req *pb.ProbeRequest) (*pb.MediaInfo, error) {
    cmd := exec.NewCommand("ffprobe",
        exec.Flag("-v", "quiet"),
        exec.Flag("-print_format", "json"),
        exec.Flag("-show_format", req.ShowFormat),
        exec.Flag("-show_streams", req.ShowStreams),
        exec.Arg(req.Path),
    )

    var info pb.MediaInfo
    if err := cmd.RunJSON(ctx, &info); err != nil {
        return nil, adopt.TranslateError(err) // exit code → gRPC status
    }
    return &info, nil
}
```

> **Note**: This is an illustrative sketch. The actual API will be designed
> alongside the first real adoptions.

---

## Relationship to the holonizer

The long-term vision is an **automatic holonizer** — a tool that introspects
a CLI's `--help` output, documentation, and behavior, then generates the
`.proto`, wrapper, and tests automatically.

**go-cli-holonization** is the manual, pragmatic predecessor.
It provides the runtime primitives that both hand-written adoption layers
and the future holonizer will use. When the holonizer arrives, it will
generate code that imports this SDK — the holonizer produces the mapping;
this SDK provides the execution engine.

```
Manual adoption (today)          Automatic adoption (future)
─────────────────────            ──────────────────────────
Human writes .proto              Holonizer generates .proto
Human writes wrapper             Holonizer generates wrapper
  ↓ both import ↓                  ↓ both import ↓
┌────────────────────────────────────────────────────┐
│          go-cli-holonization (this SDK)             │
│  exec · parse · translate · serve · test           │
└────────────────────────────────────────────────────┘
```

---

## Status

🚧 **Not implemented yet** — this repository currently contains only the
vision and design direction. Upcoming content will include:

- **Documentation** — step-by-step guides for adopting a CLI tool as a holon.
- **Examples** — reference adoptions (e.g. `ffprobe`, `git`) with full
  `.proto`, wrapper, and tests.
- **Patterns** — reusable recipes for common adoption scenarios
  (JSON output, streaming, error mapping).

All implementation code will use
[go-holons](https://github.com/organic-programming/go-holons) as its
foundation.

---

## License

See [LICENSE](LICENSE).
