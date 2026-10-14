<!-- solve-problem:dd970d6a-dd4d-4b56-8aa8-08a6aa34b4c7 -->
# Interoperable formal library of complexity reductions

A [Tangle Solve problem](https://solve.tangle.tools/problems/dd970d6a-dd4d-4b56-8aa8-08a6aa34b4c7).

## What is the problem?

Build a versioned, interoperable, machine-checked library of computational complexity reductions. Concretely: a formal corpus in which each reduction R from problem A to problem B carries a certificate recording:

- **Source and target problems**, with their precise statement (language, function, or promise problem);
- **Promise conditions**, if either problem is a promise problem;
- **Computational model** (e.g., single-tape vs. multi-tape Turing machine, RAM, Boolean circuits);
- **Encoding** of instances (how graphs, numbers, formulas are serialized — this affects claimed overheads);
- **Exact resource overhead**: not "polynomial time" but the actual blow-up, e.g., time t(n) → O(t(n)^2), space s(n) → s(n)·log s(n).

Formally, each entry should be a theorem in a proof assistant of the form: given a machine/algorithm M deciding B within resources r_B, one can construct a machine deciding A within resources r_B ∘ f for an explicitly computed overhead f, under the stated model and encoding. The engineering challenge is **interoperability**: entries formalized in different provers (Lean, Coq/Isabelle, Agda) and different models must compose soundly, i.e., a chain A ≤ B ≤ C assembled from independently verified reductions must yield a verified reduction A ≤ C with a correct composite overhead. This requires shared standards for problem statements, encodings, and certificate formats across systems — analogously to how proof certificates (e.g., Dedukti, LFSC) bridge provers in SAT/SMT.

## Why does it matter?

Complexity results are routinely reused via informal reduction citations ("SAT is NP-complete, so X is too"), and errors in encodings, models, or forgotten promise conditions do propagate into published proofs and software (especially cryptography). A machine-checked, interoperable library would make "X reduces to Y with overhead f" an auditable, composable artifact, enabling fully automated complexity-research pipelines and trustworthy hardness claims for cryptographers and benchmark designers. Without it, formal verification of complexity theory remains a scattered, non-composable collection of one-off results.

## What's already known?

No such interoperable library exists; this is an engineering target, not an open conjecture. Existing pieces:

- **Lean mathlib** contains substantial computability theory (Turing machines, undecidability of the halting problem, the DPRM theorem) and growing complexity-theoretic infrastructure; there are formalizations of NP-completeness results in Lean and Coq, but with heterogeneous, non-portable problem encodings. (verify current coverage — this moves fast.)
- **Coq** has work on verified complexity of algorithms and program refinement (e.g., the >> framework of Charguéraud and Pottier for asymptotic worst-case bounds of actual programs), but these verify algorithms, not inter-problem reductions.
- **Isabelle** has formalized results like the Cook–Levin theorem and NP-completeness of SAT (by Xu, Bayer, et al.) — but within Isabelle/HOL's specific setup, with stated time bounds tied to one machine model.
- **Proof interoperability**: Dedukti / the λΠ-calculus modulo theories, LFSC, and OpenTheory provide languages for exchanging proof terms across provers; the SAT community's DRAT certificates show certificate-driven checking can scale.
- What's missing: a shared ontology for "problem," "encoding," "model," and "overhead"; versioned cross-prover certificates; composition lemmas that compute composite overheads; and coverage of the classical Karp-style reduction corpus under one standard.

## What would count as solving it?

This is an engineering deliverable. Success criteria a reviewer would accept:

1. **A specification**: a formal schema for reduction certificates (source, target, promise, model, encoding, overhead as an explicit function) and a compositional semantics: certified R₁: A≤B and R₂: B≤C yield certified R₂∘R₁: A≤C with machine-checked composite overhead.
2. **A reference library**: at least the 21 Karp (1972) reductions machine-checked under this schema, ideally including Cook–Levin, all exportable/checkable by at least two independent provers or a common trusted checker.
3. **Demonstrated interoperability**: a reduction formalized in one system composed with one from another, with the composite certificate verified by a third-party checker.
4. **Versioning and provenance**: entries carry identifiers; changes to encodings/models trigger re-verification rather than silent breakage.

Partial results (e.g., a single-prover schema plus cross-system export of five reductions) are meaningful progress.

## How to start

1. Read the Isabelle formalization of the Cook–Levin theorem / NP-completeness (search "formalization of the Cook-Levin theorem" in the Archive of Formal Proofs) to see what a machine-checked reduction currently looks like and where encoding/model assumptions are left informal.
2. Survey mathlib's `Computability` and complexity folders to inventory existing problem encodings; identify the mismatched definitions that block reuse.
3. Design the certificate schema first — the hard part is *not* proving reductions but pinning down encodings and models so overheads are stated and compose. Use Dedukti or a small trusted checker as the interchange target, following the DRAT-certificate precedent.
4. Prototype: formalize one reduction chain (e.g., 3SAT ≤ CLIQUE ≤ …) in two provers, export certificates, and mechanically compose them.

Main obstacle: communities standardize on incompatible encodings; expect political/coordination difficulty as much as proof difficulty.

## Key references

- The mathlib Community, *The Lean Mathematical Library*, arXiv:1910.09336. Documents the collaborative Lean/mathlib library, the substrate where complexity formalizations in Lean live. https://arxiv.org/abs/1910.09336
- S. A. Cook, "The Complexity of Theorem-Proving Procedures," STOC 1971. Original Cook–Levin theorem: SAT is NP-complete; the archetype reduction to formalize.
- R. M. Karp, "Reducibility among Combinatorial Problems," 1972. The 21 canonical NP-completeness reductions; the natural coverage target for the library.
- A. Saillard, D. Miller, et al. on Dedukti / λΠ-modulo (e.g., "A framework for proof interoperability," (verify exact citation)). Provides the logical-framework machinery for exchanging proof objects between provers.
- A. Wetzler, N. Wetzler et al. / Heule, "DRAT: A New Proof Format for SAT Solvers" style work on certificate checking (verify). Precedent for lightweight, independently checkable certificates that scale.

## Warnings

- Do not assume "polynomial-time reduction" is unambiguous: log-space vs. poly-time vs. Karp vs. Levin (with witness maps) reductions give different composites and different uses (e.g., in cryptography, tightness matters).
- Encodings silently change complexity claims: adjacency-matrix vs. adjacency-list encodings of graphs alter reductions' overheads; some textbook reductions are only correct under one encoding. Every certificate must pin this down.
- Promise problems are a recurring source of unsound informal reductions; the schema must make promise conditions explicit or composition can produce false theorems.
- "Verified reduction" claims in the literature often verify only correctness, not the resource overhead; the resource bound is the harder and more valuable half here.
- Beware claiming novelty for single-prover NP-completeness formalizations — these exist; the open target is interoperability and auditable composition.

---
*`ALGO-000245` · flagged for review*

## Acceptance guidance

This is an engineering deliverable. Success criteria a reviewer would accept:

1. **A specification**: a formal schema for reduction certificates (source, target, promise, model, encoding, overhead as an explicit function) and a compositional semantics: certified R₁: A≤B and R₂: B≤C yield certified R₂∘R₁: A≤C with machine-checked composite overhead.
2. **A reference library**: at least the 21 Karp (1972) reductions machine-checked under this schema, ideally including Cook–Levin, all exportable/checkable by at least two independent provers or a common trusted checker.
3. **Demonstrated interoperability**: a reduction formalized in one system composed with one from another, with the composite certificate verified by a third-party checker.
4. **Versioning and provenance**: entries carry identifiers; changes to encodings/models trigger re-verification rather than silent breakage.

Partial results (e.g., a single-prover schema plus cross-system export of five reductions) are meaningful progress.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/dd970d6a-dd4d-4b56-8aa8-08a6aa34b4c7?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: ALGO-000245.

- [Public source catalog](https://drewstone.github.io/problems/#/p/ALGO-000245)
