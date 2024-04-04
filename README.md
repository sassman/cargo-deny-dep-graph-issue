# Illustration of cargo-deny-advisories operating on a wrong dependency graph

## `cargo tree` setting the baseline

```shell
❯ cargo tree -i sqlx
sqlx v0.7.4
└── cargo-deny-dep-graph-issue v0.1.0 (/Users/me/cargo-deny-dep-graph-issue)

❯ cargo tree -i sqlx-mysql
warning: nothing to print.

To find dependencies that require specific target platforms, try to use option `--target all` first, and then narrow your search scope accordingly.
```

## `cargo-deny-advisories` shows a different picture

```shell
❯ cargo deny check advisories -s
2024-04-04 13:02:58 [WARN] unable to find a config path, falling back to default config
error[A001]: Marvin Attack: potential key recovery through timing sidechannels
   ┌─ /Users/me/cargo-deny-dep-graph-issue/Cargo.lock:96:1
   │
96 │ rsa 0.9.6 registry+https://github.com/rust-lang/crates.io-index
   │ --------------------------------------------------------------- security vulnerability detected
   │
   = ID: RUSTSEC-2023-0071
   = Advisory: https://rustsec.org/advisories/RUSTSEC-2023-0071
   = ### Impact
     Due to a non-constant-time implementation, information about the private key is leaked through timing information which is observable over the network. An attacker may be able to use that information to recover the key.

     ### Patches
     No patch is yet available, however work is underway to migrate to a fully constant-time implementation.

     ### Workarounds
     The only currently available workaround is to avoid using the `rsa` crate in settings where attackers are able to observe timing information, e.g. local use on a non-compromised computer is fine.

     ### References
     This vulnerability was discovered as part of the "[Marvin Attack]", which revealed several implementations of RSA including OpenSSL had not properly mitigated timing sidechannel attacks.

     [Marvin Attack]: https://people.redhat.com/~hkario/marvin/
   = Announcement: https://github.com/RustCrypto/RSA/issues/19#issuecomment-1822995643
   = Solution: No safe upgrade is available!
   = rsa v0.9.6
     └── sqlx-mysql v0.7.4
         ├── sqlx-macros-core v0.7.4
         │   └── sqlx-macros v0.7.4
         │       └── sqlx v0.7.4
         │           └── cargo-deny-dep-graph-issue v0.1.0
         └── sqlx v0.7.4 (*)

 advisories FAILED: 1 errors, 0 warnings, 0 notes
```

## Elaboration

The feature flags used of sqlx do not lead to the dependency `sqlx-mysql`, however `cargo deny advisories` seems to operate on this graph:

```raw
   = rsa v0.9.6
     └── sqlx-mysql v0.7.4
         ├── sqlx-macros-core v0.7.4
         │   └── sqlx-macros v0.7.4
         │       └── sqlx v0.7.4
         │           └── cargo-deny-dep-graph-issue v0.1.0
         └── sqlx v0.7.4 (*)
```

As if it would not consider the feature flags as `cargo tree` does.