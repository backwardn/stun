[![Master status](https://tc.gortc.io/app/rest/builds/buildType:(id:stun_MasterStatus)/statusIcon.svg)](https://tc.gortc.io/project.html?projectId=stun&tab=projectOverview&guest=1)
[![GoDoc](https://godoc.org/gortc.io/stun?status.svg)](http://godoc.org/gortc.io/stun)
[![codecov](https://codecov.io/gh/gortc/stun/branch/master/graph/badge.svg)](https://codecov.io/gh/gortc/stun)
# STUN
Package stun implements Session Traversal Utilities for NAT (STUN)
[[RFC5389](https://tools.ietf.org/html/rfc5389)] protocol with no
external dependencies and zero allocations in hot paths.Complies to
[gortc principles](https://gortc.io/#principles) as core package.

See [stun server](https://github.com/gortc/stund) for simple usage. Also see
[gortc/turn](https://github.com/gortc/turn) for TURN
[[RFC5766](https://tools.ietf.org/html/rfc5766)] implementation and
[gortcd](https://github.com/gortc/gortcd) for TURN and STUN server. This
repo was merged to [pion/stun](https://github.com/pion/stun) at version
`v1.19.0`.

Please use `v1` version for stun agent and client.

# Example
You can get your current IP address from any STUN server by sending
binding request. See more idiomatic example at `cmd/stun-client`.
```go
package main

import (
	"fmt"

	"gortc.io/stun/v2"
)

func main() {
	// Creating a "connection" to STUN server.
	c, err := stun.Dial("udp", "stun.l.google.com:19302")
	if err != nil {
		panic(err)
	}
	// Building binding request with random transaction id.
	message := stun.MustBuild(stun.TransactionID, stun.BindingRequest)
	// Sending request to STUN server, waiting for response message.
	if err := c.Do(message, func(res stun.Event) {
		if res.Error != nil {
			panic(res.Error)
		}
		// Decoding XOR-MAPPED-ADDRESS attribute from message.
		var xorAddr stun.XORMappedAddress
		if err := xorAddr.GetFrom(res.Message); err != nil {
			panic(err)
		}
		fmt.Println("your IP is", xorAddr.IP)
	}); err != nil {
		panic(err)
	}
}
```

## Supported RFCs
- [x] [RFC 5389](https://tools.ietf.org/html/rfc5389) — Session Traversal Utilities for NAT
- [x] [RFC 5769](https://tools.ietf.org/html/rfc5769) — Test Vectors for STUN
- [x] [RFC 7064](https://tools.ietf.org/html/rfc7064) — STUN URI

# Stability [![stability-mature](https://img.shields.io/badge/stability-mature-008000.svg)](https://github.com/mkenney/software-guides/blob/master/STABILITY-BADGES.md#mature) ![GitHub tag](https://img.shields.io/github/tag/gortc/stun.svg)

The v1 is currently stable, no backward incompatible changes are
expected with exception of critical bugs or security fixes.

Additional attributes are unlikely to be implemented in scope of stun package,
the only exception is constants for attribute or message types.

# RFC 3489 notes
RFC 5389 obsoletes RFC 3489, so implementation was ignored by purpose, however,
RFC 3489 can be easily implemented as separate package.

# Requirements
Go 1.13 or better is required.

# Testing
Client behavior is tested and verified in many ways:
  * End-To-End with long-term credentials
    * **coturn**: The coturn [server](https://github.com/coturn/coturn/wiki/turnserver) (linux)
  * Bunch of code static checkers (linters)
  * Standard unit-tests with coverage reporting (linux {amd64, **arm**64}, windows and darwin)
  * Explicit API backward compatibility [check](https://github.com/gortc/api), see `api` directory

See [TeamCity project](https://tc.gortc.io/project.html?projectId=stun&guest=1) and `e2e` directory
for more information. Also the Wireshark `.pcap` files are available for e2e test in
artifacts for build.

# Benchmarks

Intel(R) Core(TM) i7-8700K:

```
version: 1.16.5
goos: linux
goarch: amd64
pkg: github.com/gortc/stun
PASS
benchmark                                         iter       time/iter      throughput   bytes alloc        allocs
---------                                         ----       ---------      ----------   -----------        ------
BenchmarkMappedAddress_AddTo-12               30000000     36.40 ns/op                        0 B/op   0 allocs/op
BenchmarkAlternateServer_AddTo-12             50000000     36.70 ns/op                        0 B/op   0 allocs/op
BenchmarkAgent_GC-12                            500000   2552.00 ns/op                        0 B/op   0 allocs/op
BenchmarkAgent_Process-12                     50000000     38.00 ns/op                        0 B/op   0 allocs/op
BenchmarkMessage_GetNotFound-12              200000000      6.90 ns/op                        0 B/op   0 allocs/op
BenchmarkMessage_Get-12                      200000000      7.61 ns/op                        0 B/op   0 allocs/op
BenchmarkClient_Do-12                          2000000   1072.00 ns/op                        0 B/op   0 allocs/op
BenchmarkErrorCode_AddTo-12                   20000000     67.00 ns/op                        0 B/op   0 allocs/op
BenchmarkErrorCodeAttribute_AddTo-12          30000000     52.20 ns/op                        0 B/op   0 allocs/op
BenchmarkErrorCodeAttribute_GetFrom-12       100000000     12.00 ns/op                        0 B/op   0 allocs/op
BenchmarkFingerprint_AddTo-12                 20000000    102.00 ns/op     430.08 MB/s        0 B/op   0 allocs/op
BenchmarkFingerprint_Check-12                 30000000     54.80 ns/op     948.38 MB/s        0 B/op   0 allocs/op
BenchmarkBuildOverhead/Build-12                5000000    333.00 ns/op                        0 B/op   0 allocs/op
BenchmarkBuildOverhead/BuildNonPointer-12      3000000    536.00 ns/op                      100 B/op   4 allocs/op
BenchmarkBuildOverhead/Raw-12                 10000000    181.00 ns/op                        0 B/op   0 allocs/op
BenchmarkMessageIntegrity_AddTo-12             1000000   1053.00 ns/op      18.98 MB/s        0 B/op   0 allocs/op
BenchmarkMessageIntegrity_Check-12             1000000   1135.00 ns/op      28.17 MB/s        0 B/op   0 allocs/op
BenchmarkMessage_Write-12                    100000000     27.70 ns/op    1011.09 MB/s        0 B/op   0 allocs/op
BenchmarkMessageType_Value-12               2000000000      0.49 ns/op                        0 B/op   0 allocs/op
BenchmarkMessage_WriteTo-12                  100000000     12.80 ns/op                        0 B/op   0 allocs/op
BenchmarkMessage_ReadFrom-12                  50000000     25.00 ns/op     801.19 MB/s        0 B/op   0 allocs/op
BenchmarkMessage_ReadBytes-12                100000000     18.00 ns/op    1113.03 MB/s        0 B/op   0 allocs/op
BenchmarkIsMessage-12                       2000000000      1.08 ns/op   18535.57 MB/s        0 B/op   0 allocs/op
BenchmarkMessage_NewTransactionID-12           2000000    673.00 ns/op                        0 B/op   0 allocs/op
BenchmarkMessageFull-12                        5000000    316.00 ns/op                        0 B/op   0 allocs/op
BenchmarkMessageFullHardcore-12               20000000     88.90 ns/op                        0 B/op   0 allocs/op
BenchmarkMessage_WriteHeader-12              200000000      8.18 ns/op                        0 B/op   0 allocs/op
BenchmarkMessage_CloneTo-12                   30000000     37.90 ns/op    1795.32 MB/s        0 B/op   0 allocs/op
BenchmarkMessage_AddTo-12                    300000000      4.77 ns/op                        0 B/op   0 allocs/op
BenchmarkDecode-12                           100000000     22.00 ns/op                        0 B/op   0 allocs/op
BenchmarkUsername_AddTo-12                    50000000     23.20 ns/op                        0 B/op   0 allocs/op
BenchmarkUsername_GetFrom-12                 100000000     17.90 ns/op                        0 B/op   0 allocs/op
BenchmarkNonce_AddTo-12                       50000000     34.40 ns/op                        0 B/op   0 allocs/op
BenchmarkNonce_AddTo_BadLength-12            200000000      8.29 ns/op                        0 B/op   0 allocs/op
BenchmarkNonce_GetFrom-12                    100000000     17.50 ns/op                        0 B/op   0 allocs/op
BenchmarkUnknownAttributes/AddTo-12           30000000     48.10 ns/op                        0 B/op   0 allocs/op
BenchmarkUnknownAttributes/GetFrom-12        100000000     20.90 ns/op                        0 B/op   0 allocs/op
BenchmarkXOR-12                               50000000     25.80 ns/op   39652.86 MB/s        0 B/op   0 allocs/op
BenchmarkXORSafe-12                            3000000    515.00 ns/op    1988.04 MB/s        0 B/op   0 allocs/op
BenchmarkXORFast-12                           20000000     73.40 ns/op   13959.30 MB/s        0 B/op   0 allocs/op
BenchmarkXORMappedAddress_AddTo-12            20000000     56.70 ns/op                        0 B/op   0 allocs/op
BenchmarkXORMappedAddress_GetFrom-12          50000000     37.40 ns/op                        0 B/op   0 allocs/op
ok  	github.com/gortc/stun	76.868s
```

## Build status
[![Build Status](https://travis-ci.com/gortc/stun.svg)](https://travis-ci.com/gortc/stun)
[![Build status](https://ci.appveyor.com/api/projects/status/fw3drn3k52mf5ghw/branch/master?svg=true)](https://ci.appveyor.com/project/ernado/stun-j08g0/branch/master)

## License
BSD 3-Clause License
