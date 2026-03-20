# Tetragon Tracing Policies

This directory contains a reusable set of Tetragon `TracingPolicy` manifests for Linux filesystem and network observability.

The policies are split into two profiles:

- `tracing_policies/prod/` for lower-noise baseline coverage
- `tracing_policies/verbose/` for higher-volume network discovery and tuning

## Included policies

| Policy | File | Purpose |
|--------|------|---------|
| Filesystem writes | `tracing_policies/prod/10-fs-write-user-space.yaml` | Emits on successful write permission checks inside selected user-space paths. |
| Config reads | `tracing_policies/prod/20-fs-read-config-artifacts.yaml` | Emits on successful reads of common config-style files. |
| Scratch file creation/truncation | `tracing_policies/prod/30-fs-temp-scratch.yaml` | Captures `openat` / `openat2` calls with `O_CREAT`, `O_TRUNC`, or `O_TMPFILE`. |
| Rename/delete/symlink lifecycle | `tracing_policies/prod/40-fs-rename-delete.yaml` | Captures rename, unlink, and symlink syscall arguments. |
| DNS outbound | `tracing_policies/prod/65-net-dns-outbound.yaml` | Captures outbound DNS traffic on UDP/TCP port 53. |
| Remote outbound TCP | `tracing_policies/verbose/60-net-outbound-remote.yaml` | Captures outbound TCP connects to non-private, non-loopback destinations. |
| Loopback outbound TCP | `tracing_policies/verbose/61-net-outbound-loopback.yaml` | Captures outbound TCP connects to loopback destinations. |
| Listening sockets | `tracing_policies/verbose/70-net-inbound-listen.yaml` | Emits when a process enters TCP `LISTEN`. |
| Accepted connections | `tracing_policies/verbose/71-net-inbound-accept.yaml` | Emits when a TCP socket transitions to `TCP_ESTABLISHED`. |

## Notes

- The filesystem read/write policies use `security_file_permission` so Tetragon can resolve the canonical file path and the kernel allow/deny result from the same hook.
- The scratch and lifecycle policies use syscall tracepoints because they need raw syscall arguments such as flag bitmasks and paired source/destination paths.
- The scratch and lifecycle manifests are intentionally close to the extracted originals. If you want tighter scoping, add `Prefix` selectors to the relevant path arguments.
- The read/write manifests currently scope to `/home/` and `/root/`. Adjust those `Prefix` selectors for your environment.
- A project-specific binary exclusion that previously existed in the loopback policy was removed here to keep the manifest portable.
- When changing syscall tracepoint argument declarations, verify indexes and widths against `/sys/kernel/debug/tracing/events/syscalls/<tracepoint>/format`.
- When changing kprobe or LSM hook argument declarations, verify prototypes against `/sys/kernel/btf/vmlinux` with `bpftool btf dump`.
