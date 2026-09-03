Linux x86_64 `buck2` with the RE client timeouts from
https://github.com/facebook/buck2/pull/1488 plus the Execute-stream wait
follow-up (`timeout-oss-execute-stream`, https://github.com/facebook/buck2/pull/1489).

- version: `buck2 13f91e8ab4c33151e60a8f5fa2077c61e4524156`
- built from `timeout-oss-execute-stream` @ `812bc63604dae34aab20cb6fc67f766aebb45afc`
- file: `buck2.zst` (zstd of the binary; sha256 `87c6fd610cb0bb39f0f2e7bc9e6aa7b215da1f9b8e9fb4c74953e4b8b0e6805d`)

Public download (no auth):

https://raw.githubusercontent.com/njskalski/buck2/ci-binaries/buck2.zst

Defaults in this binary, all under `[buck2_re_client]`:

| key | default | what it bounds |
| --- | --- | --- |
| `grpc_timeout_secs` | 60 | unary CAS/AC RPCs |
| `grpc_stream_timeout_secs` | 600 | ByteStream Read/Write open |
| `grpc_execute_without_action_timeout_secs` | 1800 | Execute stream wait, action has no `timeout_seconds` |
| `grpc_execute_after_action_timeout_secs` | 300 | added to `timeout_seconds` for the Execute stream wait |

Plus `[buck2] materializer_ensure_timeout_secs` (60) for the buck2-dm Ensure oneshot.

New since `3a8f22ed`: an expired `grpc-timeout` is retryable. tonic reports it
as `TimeoutExpired`, which `Status` maps to `Cancelled` rather than
`DeadlineExceeded`, so the 5-attempt retry used to skip it and CAS/AC timeouts
failed the build outright.

CI: set `BUCK2_BINARY_URL` to the raw URL above. Local:

```
curl -fsSL -o buck2.zst https://raw.githubusercontent.com/njskalski/buck2/ci-binaries/buck2.zst
zstd -d buck2.zst -o buck2 && chmod +x buck2
./buck2 --version
```

This branch holds only the binary. Do not open a PR from it to facebook/buck2.
