# Bolt: Open Source Plan

Status: Working plan. Companion to [HARNESS_PLAN.md](HARNESS_PLAN.md).

Updated: 2026-08-28

## 1. Why open source is load-bearing

For most products, open source is a distribution choice. For Bolt it is a structural requirement, because every promise in the product plan is a promise about trust:

- The adoption compiler converts other people's setups and asks them to run the result. Nobody runs converted code from a black box. The converter, the provenance format, and the verification pipeline must be inspectable or adoption is dead on arrival.
- The sandbox and permission model claim to be non-bypassable. A closed-source security boundary is a marketing claim; an open one is a checkable one.
- The learning loop edits instructions on the user's machine based on the user's behavior. That is either transparent or creepy, and the difference is source access.
- The thesis itself — "adapts to you instead of you adapting to it" — is an anti-lock-in position. It is incoherent to argue against lock-in from behind a proprietary license.

So openness is not a value statement appended to the project. It is the mechanism by which the product's claims become believable.

## 2. License

**Apache-2.0, everything.** The kernel, the clients, the adoption compiler, the extension API, the mobile apps, the cloud adapters. One license, no split, no protective carve-outs, no dual licensing.

Why Apache-2.0 over MIT:

- The explicit patent grant. Bolt sits in a space where large vendors are filing aggressively; contributors and adopters get a defensive floor MIT does not provide.
- It is the license serious infrastructure converges on when it expects corporate contributors, and Bolt's compatibility catalog (section 8) explicitly wants plugin vendors in its CI.
- Apache-2.0's explicit contribution terms (section 5 of the license) do the legal work a CLA would otherwise exist for, which is what lets us not have one (section 4).

Why no open-core split: a project whose pitch is "keep your stuff, no lock-in" cannot have a paywall in its own repository without the pitch curdling. Every feature in HARNESS_PLAN.md lands under Apache-2.0.

Third-party reuse obligations:

- Pi is MIT-licensed and Bolt reuses its agent loop, compaction, and session format by design (HARNESS_PLAN.md sections 14, 15.6). MIT folds into an Apache-2.0 project cleanly; original copyright notices and license text are preserved in the source tree and in a NOTICE file, with attribution beyond the legal minimum — named credit in the README. The same applies to `pi-web-access` and any other adopted implementation.
- Every vendored or adapted file records its origin in the file header: source project, commit, and what was changed. This is the same provenance discipline the adoption compiler applies to user packages (HARNESS_PLAN.md section 4.4), applied to ourselves. A project built on an adoption compiler does not get to be vague about what it adopted.

## 3. Governance: BDFL and maintainers

Bolt is an opinionated design — one voice, deliberate disagreements with DSH and Pi, a small kernel that is explicitly not a democracy. The governance model matches the design rather than pretending otherwise.

**The BDFL** owns architecture: the kernel guarantees, the event format, the child contract, the extension API surface, and final say when consensus fails. This is written down precisely so it is bounded — the BDFL role is a tiebreaker and a coherence-keeper, not a bottleneck through which all work flows.

**Area maintainers** own bounded surfaces with real authority inside them: the adoption compiler, the terminal client, the web client, the mobile clients, the protocol and SDKs, sandbox providers, docs. A maintainer merges within their area without BDFL sign-off. Cross-area changes need the affected maintainers; kernel changes need the BDFL.

**RFCs, lightweight.** Changes that alter a kernel guarantee, the event format, the extension API, or the protocol require a short written proposal in the repo — problem, design, alternatives, compatibility impact — open for comment before implementation. Everything else is a pull request. The RFC bar is deliberately placed at "things that are expensive to reverse" and nowhere lower; process weight is a tax on contributors and we keep it minimal.

**Becoming a maintainer** is earned in public: sustained quality contributions to an area, judgment visible in reviews, and a nomination by an existing maintainer with no maintainer objecting. Maintainership lapses after extended inactivity — quietly and without ceremony, reversible the same way.

**Succession**: if the BDFL steps away, the maintainers select a successor from among themselves. This sentence exists so the question has an answer before it is ever asked.

## 4. Contributions

**DCO, not CLA.** Contributors sign off commits (`Signed-off-by`) certifying they have the right to submit the work under Apache-2.0. No contributor license agreement, no copyright assignment, ever. CLAs are friction and an asymmetry — the project would hold rights contributors do not — and the projects Bolt most respects do without them. Apache-2.0's own contribution clause plus DCO covers what a CLA would.

**AI-assisted contributions are normal and stated plainly.** Bolt is built with agents (HARNESS_PLAN.md, "How is Bolt built?"), and it would be absurd to hold contributors to a purity standard the maintainers do not meet. The policy is about responsibility, not tooling:

- The contributor owns what they submit. "The agent wrote it" is not a defense for a broken patch, an unverified claim, or license-laundered code — the sign-off means a human stands behind the change.
- Agent-generated code meets the same bar as any code: it is reviewed by its submitter before it is reviewed by us.
- What is not welcome is unattended volume: auto-generated pull requests, drive-by agent output with no human who can answer questions about it, and issue spam. These are closed without ceremony.

**Review discipline scales with what the change touches.** Kernel and security-boundary changes get two reviewers, one of whom is the area maintainer or BDFL, plus the behavior-test suite. Everything else needs one maintainer. Tests accompany behavior changes; the Pi-fixture compatibility suites (HARNESS_PLAN.md section 14) are the regression floor and never go red on main.

**Good first contributions are curated, not scavenged.** The adoption compiler generates a natural stream of well-bounded work — each ecosystem quirk, each conversion gap, each catalog failure is an issue with a reproduction attached. Labeling that stream honestly is the onboarding program.

## 5. Relationship to the ecosystems Bolt adopts

Bolt's core feature is converting resources from Pi, OpenCode, DSH, and Claude Code. That relationship can be parasitic or symbiotic, and the difference is behavior:

- **Upstream first.** Bugs found in adopted code get reported upstream, with fixes offered where we have them. The adoption compiler's job is conversion, not forking; a fix that belongs upstream goes upstream.
- **Never scrape-and-strand.** Adoption preserves provenance, license, and notices for every converted package (HARNESS_PLAN.md section 4.4). The original author's name travels with the conversion. A plugin author should discover their work runs on Bolt and feel credited, not strip-mined.
- **The conformance suite is shared infrastructure.** Plugin authors can run Bolt's compatibility checks in their own CI (HARNESS_PLAN.md section 17.2) — the catalog is a service to the ecosystems, not a scorecard against them.
- **Respect the licenses, including the inconvenient ones.** A plugin whose license does not permit conversion is not converted. The compiler checks; the answer is sometimes no.
- **No adversarial framing.** Bolt's competitors are also its upstreams and its adoption targets. Public communication does not dunk on the projects whose users we hope to serve. The pitch is "keep everything, gain a runtime" — it only works if the projects being adopted from would grudgingly agree the behavior is fair.

## 6. Project structure

- **One monorepo**, mirroring the package layout that already works for Pi: kernel, protocol, clients, compiler, providers as packages with clear boundaries. The mobile apps live in the same repo — a protocol change and the client updates it forces belong in one review.
- **The plan documents live in the repo**, not in a wiki. HARNESS_PLAN.md and this document are versioned artifacts; changing the plan is a pull request with review, like changing code.
- **Issues are the coordination surface.** No private roadmap that contradicts the public one. Maintainer discussion happens in issues, RFCs, and a public chat channel; decisions made in private channels get written back into the public record or they did not happen.
- **Releases** follow semver with a changelog written for users, not generated from commit subjects. Pre-1.0, minor versions may break; the changelog says so loudly (fail-loudly applies to release notes too). The protocol and event format carry their own versioning per HARNESS_PLAN.md open decision 5, independent of the release train.

## 7. Security

- A `SECURITY.md` with a private disclosure channel and a committed response window. Sandbox escapes, permission bypasses, and credential leaks are treated as severity-one regardless of how theoretical the exploit looks.
- Security fixes may be developed in private and land with disclosure after release — the one sanctioned exception to develop-in-public.
- The threat model is documented: what the sandbox promises against which attacker, what it explicitly does not (HARNESS_PLAN.md sections 2.6, 10). Security claims that cannot be written down precisely are not made.
- Hardening claims invite verification: the seccomp profiles, the egress rules, and the extension isolation are all in-tree, and external audit findings — formal or drive-by — get the same triage as any severity-one report.

## 8. The compatibility catalog as a community program

The catalog (HARNESS_PLAN.md section 17.2) is Bolt's most communal artifact: a published, continuously re-verified record of which real-world plugins, skills, and configurations adopt cleanly. As a community program:

- Catalog runs are reproducible from the repo — anyone can re-run the conversion of any listed package and diff the result.
- A failing catalog entry is an open issue with the reproduction attached, which makes the ecosystem's rough edges the project's contributor funnel.
- Plugin authors can claim their entries: verify the conversion, mark caveats, wire the conformance suite into their CI. An author-verified entry outranks an automated one.
- The catalog never editorializes. It reports conversion status, not quality judgments about other people's work.

## 9. Community conduct

- **Contributor Covenant**, adopted as-is rather than hand-rolled, enforced by the maintainers with the BDFL as escalation point. Boring and standard is the point.
- The tone standard for maintainers is the tone of the plan documents: direct, technical, honest about tradeoffs, never contemptuous. How maintainers talk in reviews is the culture; no document overrides example.
- English is the project language for durable artifacts; nobody is penalized for imperfect English, and review feedback addresses the patch, not the prose.

## 10. Trademark and name

Apache-2.0 licenses the code, not the name. "Bolt" and its mark are held by the project with a short, permissive trademark policy published in-repo: unmodified redistribution and truthful compatibility claims ("works with Bolt", "adopted for Bolt") are always fine; forks are welcome and encouraged under any name that does not claim to be the canonical Bolt. This is standard hygiene, stated up front so it never surprises anyone later.

## 11. What this document refuses

Mirroring the product plan's non-goals (HARNESS_PLAN.md section 19):

- No CLA and no copyright assignment.
- No open-core split, no feature paywalls, no "enterprise edition" carve-out in this repository.
- No private roadmap that contradicts the public one.
- No governance theater: no boards, committees, or working groups until the contributor base is large enough that their absence hurts.
- No adversarial marketing against the ecosystems Bolt adopts from.
- No purity rules about AI-assisted contributions that the maintainers themselves could not pass.

## 12. Success criteria

The open-source posture is working when:

1. A contributor's first pull request lands within days, reviewed by a maintainer, without the BDFL involved.
2. A plugin author from an adopted ecosystem verifies their own catalog entry unprompted.
3. An upstream project accepts a fix that originated from Bolt's adoption pipeline.
4. A fork exists, is compliant with the trademark policy, and nobody considers it a crisis.
5. A kernel RFC is rejected — proof the process is real and not a rubber stamp.
6. A security researcher reports privately, the fix ships inside the committed window, and the disclosure is published without drama.
7. Someone becomes a maintainer whom none of the founding ten has ever met.
