## [01] Description
The project depends on a number transitive dependencies that are either unsound, deprecated or unmaintained. Use of these crates may pose risks due to their outdated nature. By using [`cargo-audit`](https://crates.io/crates/cargo-audit), the following obsolete dependencies were identified

**Crate:**     [`ansi_term`](https://crates.io/crates/ansi_term)
**Version:**   0.12.1
**Warning:**   unmaintained
**Title:**     ansi_term is Unmaintained
**ID:**        [RUSTSEC-2021-0139](https://rustsec.org/advisories/RUSTSEC-2021-0139)

**Crate:**     [`parity-wasm`](https://crates.io/crates/parity-wasm)
**Version:**   0.45.0
**Warning:**   unmaintained
**Title:**     Crate parity-wasm deprecated by the author
**ID:**        [RUSTSEC-2022-0061](https://rustsec.org/advisories/RUSTSEC-2022-0061)

**Crate:**     [`mach`](https://crates.io/crates/mach)
**Version:**   0.3.2
**Warning:**   unmaintained
**Title:**     mach is unmaintained
**ID:**        [RUSTSEC-2020-0168](https://rustsec.org/advisories/RUSTSEC-2020-0168)

**Crate:**     [`iana-time-zone`](https://crates.io/crates/iana-time-zone)
**Version:**   0.1.59
**Warning:**   yanked
**Title:**     unsound, memory exposure patched for >=0.1.45
**ID:**        [RUSTSEC-2022-0049](https://rustsec.org/advisories/RUSTSEC-2022-0049.html)

**Recommendations**
Use safer alternatives & dependency specific migrations for the aforementioned outdated crates that are mentioned on [RUSTSEC](https://rustsec.org/) Advisory Database. Run [cargo-audit]("https://crates.io/crates/cargo-audit") as a part of the CI/CD pipeline.

## [02] Description
In `src/orml/rewards/README.md`, [Prove](https://github.com/code-423n4/2024-03-acala/tree/main/src/orml/rewards#prove) has the incorrect final expression of $\delta_{R}$. Even though the implementation of the relevant function [`add_share()`](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L143) in the file `src/orml/rewards/src/lib.rs` is [correct](https://github.com/code-423n4/2024-03-acala/blob/9c71c05cf2d9f0a2603984c50f76fc8a315d4d65/src/orml/rewards/src/lib.rs#L161), this inaccuracy may lead to unnecessary confusion and logical errors. 
**Correction**
The correct expression should be:
$$ \delta_R = R_n * ({s_{m+1}}/{\sum_{i=1}^{m} s_i}) $$