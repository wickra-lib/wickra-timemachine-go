# Wickra TimeMachine — Go

[![CI](https://github.com/wickra-lib/wickra-timemachine/actions/workflows/ci.yml/badge.svg)](https://github.com/wickra-lib/wickra-timemachine/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/wickra-lib/wickra-timemachine/branch/main/graph/badge.svg)](https://codecov.io/gh/wickra-lib/wickra-timemachine)
[![Go module](https://raw.githubusercontent.com/wickra-lib/.github/main/profile/badges/go.svg)](https://pkg.go.dev/github.com/wickra-lib/wickra-timemachine-go)
[![License: MIT OR Apache-2.0](https://img.shields.io/badge/license-MIT_OR_Apache--2.0-blue)](https://github.com/wickra-lib/wickra-timemachine#license)

**Go bindings for the Wickra Time Machine over its C ABI hub via cgo. A `TimeMachine` is built from a spec JSON and driven over a JSON boundary, so a seek reconstructs the byte-identical market snapshot as every other Wickra Time Machine binding.**

## Install

Use the published **`wickra-timemachine-go`** module, which bundles the prebuilt C ABI
library for every platform, so `go get` + `go build` works with no extra steps
(a C compiler is still required, as the binding uses cgo):

```bash
go get github.com/wickra-lib/wickra-timemachine-go
```

```go
import wickra "github.com/wickra-lib/wickra-timemachine-go"
```

`wickra-timemachine-go` is generated from the [`bindings/go`](https://github.com/wickra-lib/wickra-timemachine/tree/main/bindings/go)
directory of [wickra-timemachine](https://github.com/wickra-lib/wickra-timemachine) by the release
pipeline: it mirrors the Go sources, the vendored C ABI header (`include/wickra_timemachine.h`)
and the prebuilt libraries under `lib/<goos>_<goarch>/`. On Linux/macOS the
library path is baked in via rpath; on Windows the DLL must be discoverable at
run time (next to the executable or on `PATH`).

### Building from this repository (contributors)

The `bindings/go` directory in the main repository is the development source. To
build against a locally compiled C ABI, build the hub and stage the library into
the per-platform directory cgo links against:

```bash
cargo build -p wickra-timemachine-c --release
mkdir -p lib/linux_amd64                          # match your GOOS_GOARCH
cp target/release/libwickra_timemachine.so    lib/linux_amd64/    # Linux
cp target/release/libwickra_timemachine.dylib lib/darwin_arm64/   # macOS (arm64)
cp target/release/wickra_timemachine.dll      lib/windows_amd64/  # Windows
```

## Quick start

```go
package main

import (
	"encoding/json"
	"fmt"

	wickra "github.com/wickra-lib/wickra-timemachine-go"
)

func main() {
	tm, err := wickra.New("{}")
	if err != nil {
		panic(err)
	}
	defer tm.Close()

	feed := `{"ts":10,"symbol":"BTC-USDT","feed":{"kind":"market","type":"trade","symbol":{"base":"BTC","quote":"USDT"},"price":"100","quantity":"1","aggressor":"Buy","timestamp":10}}` + "\n" +
		`{"ts":20,"symbol":"BTC-USDT","feed":{"kind":"market","type":"trade","symbol":{"base":"BTC","quote":"USDT"},"price":"110","quantity":"2","aggressor":"Sell","timestamp":20}}`

	data, _ := json.Marshal(feed)
	if _, err := tm.Command(`{"cmd":"load","data":` + string(data) + `}`); err != nil {
		panic(err)
	}
	resp, err := tm.Command(`{"cmd":"seek","ts":20}`)
	if err != nil {
		panic(err)
	}
	fmt.Println(resp) // the market snapshot reconstructed at ts=20
}
```

## Documentation

The full guides, quickstarts and API reference live in the main repository and
documentation site:

- **Repository:** <https://github.com/wickra-lib/wickra-timemachine>
- **Docs:** <https://docs.wickra.org>

Wickra ships native bindings for Python, Node.js, WASM and Rust, plus a C ABI hub
that any C-capable language (C, C++, C#, Go, Java, R) links against — all exposing
the same core from the shared, `unsafe`-forbidden Rust core.

## Security

Found a security issue? **Please don't open a public issue.** Report it privately
via the affected repository's *Security* tab (*"Report a vulnerability"*) or email
**support@wickra.org** with a subject line starting `[wickra security]`. Full
policy: <https://github.com/wickra-lib/wickra-timemachine/blob/main/SECURITY.md>.

## Disclaimer

Wickra TimeMachine is research and analytics software. Its outputs are
deterministic transforms of the input data — they are not financial advice and do
not predict the market. Any use in a live trading context is at your own risk. The
software is provided **as is**, without warranty of any kind.

## License

Licensed under either of [Apache-2.0](https://github.com/wickra-lib/wickra-timemachine/blob/main/LICENSE-APACHE)
or [MIT](https://github.com/wickra-lib/wickra-timemachine/blob/main/LICENSE-MIT) at your option.
