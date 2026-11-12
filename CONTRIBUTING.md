# Contributing

Keep host authority outside this crate. Do not add approval, authentication, lease settlement, remote-provider terminality, or workflow scheduler logic.

For workspace-write changes, preserve path/symlink race protections, one-use capability semantics, durable before/after journal binding, receipt immutability, and fail-closed recovery. Add deterministic tests using temporary directories and `content-artifacts` fixtures.

For cgroup/Bubblewrap changes, keep host integration tests behind `RUNALITH_SANDBOX_RUN_HOST_CGROUP_TESTS=1`. Never run them against a production host. Unit tests must use synthetic cgroup filesystems and disposable paths.

Run `cargo test --locked`, `cargo fmt -- --check`, `cargo clippy --all-targets --locked -- -D warnings`, and rustdoc with warnings denied.
