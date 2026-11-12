# Runalith Linux Sandbox

A Linux-only process and workspace mutation boundary extracted from Runalith.

The crate combines three related capabilities:

- Bubblewrap-based command execution with no direct-process fallback, network namespace isolation, read-only executable roots, one writable workspace, cleared environment, bounded arguments/input/output, timeouts, and cancellation.
- Mandatory per-execution cgroup v2 CPU, memory, swap, and PID limits, with kernel-observed process-tree cleanup evidence.
- Plan-scoped, expiring, one-use workspace-write capabilities with descriptor-anchored path checks, atomic replacement, content-addressed receipts, and crash recovery.

## Security boundary

This is security-sensitive infrastructure, not a security certification or a substitute for an independent sandbox audit. It must not be treated as safe for untrusted workloads merely because unit tests pass.

The host remains responsible for:

1. Authenticating the caller and verifying workflow state, approval, immutable plan binding, task identity, and role before issuing a `WorkspaceWriteCapability`.
2. Provisioning and monitoring a systemd-delegated cgroup v2 hierarchy with `cpu`, `memory`, and `pids` controllers enabled. The constructor probes delegation and the implementation may enable child controllers within the configured delegated root.
3. Provisioning Bubblewrap and the executable roots as trusted host files. The sandbox binary must be a regular executable named `bwrap`.
4. Enforcing filesystem/project quotas on the writable workspace. `scratch_max_bytes` bounds the sandbox tmpfs mounts; it does not bound workspace disk usage.
5. Giving each execution a distinct workspace and independently enforcing host-side ownership, quotas, cleanup, and retention.
6. Treating `CgroupCleanupEvidence` as local process-tree evidence only. It does not prove remote provider work stopped, release a lease, or settle a workflow task.
7. Handling all host configuration, service-manager delegation, kernel compatibility, and operator recovery.

No unsandboxed execution fallback exists. Unsupported hosts or missing Bubblewrap/cgroup delegation must fail closed. `WorkspaceWriteCapability::issue` accepts claims that the caller asserts are already verified; this crate is not an approval authority.

## Host integration tests are opt-in

The default test suite never writes to the current host's delegated cgroup and does not start a real Bubblewrap sandbox. Tests use temporary filesystem fixtures and fake cgroup filesystems.

To run the cgroup/Bubblewrap integration tests, use a disposable Linux VM or dedicated CI worker with explicit systemd cgroup v2 delegation, Bubblewrap, and required utilities installed:

```sh
RUNALITH_SANDBOX_RUN_HOST_CGROUP_TESTS=1 cargo test --locked
```

These tests create and remove temporary child cgroups and launch constrained processes inside them. Do not enable this variable on a production host.

## Dependencies

The only Runalith crate dependency is [`content-artifacts`](https://github.com/nplongx/content-artifacts), pinned to an immutable Git revision. It stores workspace-write receipts and journals as content-addressed artifacts.

The dependency is a Git dependency rather than a crates.io dependency. This crate currently targets Cargo Git dependencies; crates.io packaging requires `content-artifacts` to be published or vendored.

## Build and test

Requires Linux for the Bubblewrap backend and Rust 1.85 or newer.

```sh
cargo test --locked
cargo fmt -- --check
cargo clippy --all-targets --locked -- -D warnings
RUSTDOCFLAGS='-D warnings' cargo doc --locked --no-deps
```

GitHub Actions checks Rust 1.85 and stable. Host cgroup integration tests remain opt-in.

## Provenance

Extracted from `nplongx/runalith` at source commit `29c24cfa2b6ead5704d26cf2746bc384f2938b80`, from `rust/crates/runalith-sandbox`. The source dependency on `runalith-artifacts` is replaced with the pinned public `content-artifacts` repository. The standalone test default is additionally hardened to avoid host cgroup mutations unless explicitly opted in.

## License

MIT. See [LICENSE](LICENSE).
