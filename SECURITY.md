# Security policy and limitations

This project implements security-sensitive Linux process isolation and filesystem mutation primitives. It has not undergone an independent security audit. Do not rely on it as the sole boundary protecting a production host or sensitive data.

Report suspected vulnerabilities privately to the repository maintainer before publishing exploit details. Do not include credentials, production workspace data, or other secrets in reports.

## Trust assumptions

- The kernel, Bubblewrap binary, cgroup v2 implementation, delegated cgroup root, executable roots, and host filesystem are trusted.
- The host authenticates users, approves commands, verifies plan/task/role identity, provisions workspaces, and controls resource quotas.
- The writable workspace is unique to one execution. The workspace-write receipt store is durable and protected from untrusted replacement.
- Cgroup cleanup proves only that the local cgroup process tree became empty. It says nothing about remote effects.

## Known limitations

- Host cgroup/Bubblewrap integration tests are opt-in because they create/remove delegated child cgroups and launch constrained processes.
- Workspace capacity is not enforced by the sandbox; the host must provision filesystem or project quota.
- `WorkspaceWriteCapability::issue` does not verify approval or plan authority. The caller must do so before issuance.
- This crate does not provide a complete defense against kernel, Bubblewrap, filesystem, or host-configuration vulnerabilities.
