# Bolt: Universal Agent Harness Plan

Status: Working product plan

Updated: 2026-08-24

## 1. Product thesis

Build a universal coding-agent harness that adopts existing agent ecosystems instead of forcing users to migrate manually.

The product is named Bolt. The logo is a screw bolt drawn in a lightning-bolt silhouette: fast, and able to bolt any ecosystem's features onto one runtime.

The product promise is:

> Keep your extensions, skills, hooks, configuration, and sessions. Adopt them into one runtime, then run locally, through a web interface, or in an OCI environment.

The harness is not another generic Claude Code clone. Its differentiators are:

1. One-command adoption of Pi extensions, OpenCode plugins, and DeepSeek Harness plugins.
2. A shared local and remote session runtime with terminal, web, and mobile clients.
3. Explicit, non-bypassable sandbox profiles using operating-system or OCI isolation.
4. Minimal prompts and no default model-driven subagent delegation.
5. Pi-compatible compaction behavior.
6. An optional learning loop, grounded in an insights engine over real session history, that maintains concise user instructions and can draft skills, hooks, and experimental self-extensions.
7. Model-agnostic by construction: the frontier model is the magic, the harness is the enabler, and every role that calls a model is provider-swappable.

## 2. Product principles

### 2.1 Adopt instead of replace

Existing installations remain untouched. Adoption creates a native converted copy with provenance, tests, and a lockfile.

### 2.2 Fail loudly

There is no silent compatibility fallback. Unsupported APIs, unavailable sandbox providers, incomplete cloud cleanup, invalid generated extensions, and failed conversions must produce explicit errors.

### 2.3 Keep the kernel small

The kernel owns only:

- Session event log
- Agent loop
- Tool execution protocol
- Cancellation and operation ownership
- Permission and sandbox policy
- Extension-host lifecycle
- Client attachment
- Local and remote worker lifecycle

Everything else should be an extension or provider.

### 2.4 Model-visible means logged

Every value that can affect a model request must be reconstructable from the session log, including:

- User and injected messages
- System prompt sections
- AGENTS.md instructions
- Tool schemas
- Model and reasoning selection
- Compaction summaries
- Provider request configuration
- Extension-provided context

Logged does not mean leaked: secrets are redacted at log-write time. Provider request configuration is recorded with credential fields masked, and the redaction field list is itself versioned so a session log can never contain a live token. This is what reconciles model-visible logging with the credential rules in section 12.4.

### 2.5 No default subagents

The normal agent does not receive a subagent tool or subagent instructions by default. Delegation is user-invoked (section 6.4), never something the model decides.

Explicit system operations such as `/adopt` may use bounded internal conversion workers. These workers are not exposed to the normal model as general delegation tools.

### 2.6 Security is enforced below extensions

Permission hooks alone are not a security boundary. Foreign extensions, MCP servers, hooks, tools, and commands must not be able to bypass required isolation by directly using host APIs.

### 2.7 Model-agnostic by construction

Most of what makes a great agent is the underlying model; the harness is what lets the model act. Bolt's value must survive a model swap:

- Pi's provider API is the model abstraction; no second abstraction is layered on top (section 18).
- Every model-calling role is independently selectable: the main agent loop, adoption conversion workers, the permission classifier, compaction summarization, facet extraction, and insight generation.
- Reasoning effort is first-class alongside model choice: every role and every subagent child carries an effort level (low through max), switchable mid-session and logged as a config change.
- The kernel makes no vendor-specific prompt assumptions. Provider quirks live in provider adapters.
- A model lacking a capability a role requires (tool calling, structured output) fails loudly at selection time instead of degrading silently.
- Local models are first-class providers, subject to the same capability checks.

### 2.8 Zero-config, discovered in use

Bolt has many capabilities and therefore many settings — and that is exactly why none of them may be required. Every option ships with a default good enough that a user who never opens a config file gets an excellent agent. Features are discovered while using the product (section 3.6), not chosen up front: there is no setup wizard, no mandatory profile selection, no decision gauntlet on first launch. The adoption scan is the entire onboarding. Configuration exists for the users who go looking for it.

## 3. User experience

### 3.1 First launch

The harness scans known configuration locations without executing discovered code.

Example:

```text
Existing setups found

Pi
  7 extensions
  12 skills
  4 prompts

OpenCode
  5 plugins
  3 agents
  6 MCP servers

DeepSeek Harness
  2 profiles
  9 plugins

Claude Code
  8 skills
  2 MCP servers

Run /adopt to review and adopt selected resources.
```

### 3.2 Adoption command

Interactive:

```text
/adopt
```

Direct installation:

```bash
bolt install pi @juicesharp/pi-web-tools
bolt install opencode opencode-wakatime
bolt install dsh @vendor/dsh-plugin
```

Optional passthrough form:

```bash
bolt adopt -- pi install @juicesharp/pi-web-tools
```

### 3.3 Adoption result

```text
Adoption complete: @juicesharp/pi-web-tools

Converted
  4 tools
  2 lifecycle hooks
  1 command

Verification
  Typecheck passed
  Compatibility checks passed
  Sandbox smoke passed

Permissions requested
  Workspace read
  Network: api.example.com

Source retained at:
  ~/.bolt/adopted/pi/@juicesharp/pi-web-tools/1.4.2/source
```

### 3.4 Plan mode

An optional propose, approve, execute flow: the agent produces a plan before touching anything, and execution starts only after approval.

Plan review is not a yes/no prompt. Plans open in a Plannotator-style annotation surface — terminal summary plus a local browser UI — where the user comments on exact text, marks steps for removal, edits the plan directly, or attaches a general note, then approves or sends the annotations back as structured feedback for revision. The same annotation surface reviews diffs from the review inbox (section 17.3).

### 3.5 Project directory

Bolt reads a `.bolt/` directory in the project alongside the global `~/.bolt/`:

- Project-scoped skills, hooks, extensions, and prompt templates
- The team profile lockfile (section 17.4)
- Project settings and sandbox policy defaults
- Project rules stay project-scoped: nothing in `.bolt/` is promoted into global learning (section 8.2)

Settings resolve project over global; capabilities and sandbox restrictions can only narrow from global to project, never widen.

### 3.6 Progressive disclosure

Features are taught where the user's eyes already are: the working indicator. While the agent runs, the status line carries an occasional one-line tip, video-game loading-screen style:

```text
⚡ Bolting… (12s · 3.1k tokens)
Tip: /learn tier 3 auto drafts extensions from your patterns — you approve before anything runs.
```

```text
⚡ Bolting… (4s · 800 tokens)
Tip: /minimal runs with just two tools when you want raw speed.
```

Rules that keep tips helpful instead of annoying:

- Contextual: drawn from what the user is doing and from insights (section 8.6), which already knows which features they don't use — the same engine that powers report suggestions feeds the tips.
- A tip for a feature the user already uses is never shown; a dismissed tip never returns.
- Rate-limited, one line, never blocking, never a modal.
- Tips guide toward features, never toward required setup — per section 2.8, ignoring every tip must still leave an excellent agent.

### 3.7 Goal mode (/goal)

Codex-style persistent objectives: `/goal <objective>` gives a session a stable target, and the loop keeps going — plan, act, verify, correct — until the objective is met or the agent hits a wall it cannot pass alone.

- A goal states its completion criteria up front, and completion must be demonstrated (tests pass, build green), never asserted.
- Goals are persisted state: they survive pauses, disconnects, restarts, and placement moves, and resume where they left off.
- A goal may fork attempts as branches (section 14.4) and spawn subagent children (section 6.4) toward the same target.
- Budgets (section 14.3) are the safety rail: a goal always runs under token, cost, and wall-clock ceilings, and pausing at a ceiling is a loud, resumable state — not a failure.
- Getting stuck is reported loudly, with what was tried and what is blocking. A goal never spins silently.
- Mission control and the mobile app show goal progress; blocked and completed goals notify.

## 4. Adoption compiler

### 4.1 Pipeline

One user-visible adoption operation runs these stages:

```text
fetch and lock source
-> inspect without execution
-> identify extension surfaces
-> build conversion plan
-> convert independent surfaces in parallel
-> combine generated output
-> run a read-only verification pass
-> typecheck and test in isolation
-> present permissions and unsupported behavior
-> activate atomically
```

### 4.2 Conversion workers

Parallel workers may specialize in:

- Manifest and dependency conversion
- Tools and schemas
- Lifecycle hooks
- Commands and configuration
- TUI renderers
- Web components
- Provider integrations
- Tests
- Security review

Parallelism is limited to independent conversion surfaces. A final verifier reads the combined result and cannot modify it.

Conversion workers are themselves capability-bounded, because they read hostile input:

- Read-only access to the fetched, locked source
- Write access only to a staging directory
- No network access during conversion
- No host credentials
- Package content is untrusted data. Instructions found inside a package (README, comments, manifests) must not steer the converter, expand worker capabilities, or alter the conversion plan.

"Inspect without execution" protects against running foreign code; these bounds protect against foreign code steering the model that converts it.

### 4.3 Compatibility levels

Every adopted feature receives an explicit status:

- `native`: imported without semantic translation
- `adapted`: translated to an equivalent native API
- `isolated`: executed in a compatibility process through capability RPC
- `unsupported`: no correct equivalent exists

There is no automatic substitution between levels.

### 4.4 Adopted package storage

```text
~/.bolt/adopted/
  <source-runtime>/
    <package-name>/
      <version>/
        source/
        converted/
        tests/
        adoption.json
```

`adoption.json` records:

- Source ecosystem
- Source package and exact version or commit
- Source content hash
- License and notices
- Converter version
- Conversion model and settings
- APIs translated
- Unsupported behavior
- Granted capabilities
- Generated files
- Verification results

The original package is never modified.

### 4.5 Updates and rollback

```text
/adopt update
/adopt diff <package>
/adopt rollback <package>
/adopt remove <package>
```

An update compares the new upstream source with the previously adopted upstream source, ports the upstream delta, reruns all checks, and activates atomically. Local conversion fixes must not be overwritten blindly.

Concretely, an update is a three-way merge across the previously adopted upstream source, the new upstream source, and the locally converted output. Local fixes to generated code are stored as explicit overlay patches rather than in-place edits, so the converter can regenerate from the new upstream and reapply them deterministically. A patch that no longer applies is a loud conflict that drops the update into manual review. It is never silently dropped and never blindly overwritten.

### 4.6 Partial adoption

A package with a mix of supported and `unsupported` surfaces is the common case, not the edge case.

- A package may activate partially, but only after its unsupported surfaces are listed and explicitly acknowledged by the user.
- The adoption result names every unsupported surface; `adoption.json` records them and `/doctor` reports them.
- No stubs or silent no-ops are generated for unsupported behavior. Calling an unsupported surface fails loudly.
- If the package's primary entry surface is unsupported, adoption fails as a whole rather than producing a hollow package.

### 4.7 Conversion cost and reproducibility

Adoption is a model-driven compile, so its cost and repeatability are user-facing properties:

- Adoption shows an estimated duration and model cost before conversion starts, and the result reports actual elapsed time and tokens spent.
- Conversion output is cached by source content hash plus converter version. Re-adopting identical input reuses the cached result instead of regenerating.
- Model output is not bit-reproducible, but `adoption.json` records the model, settings, and converter version needed to explain any difference between two conversions of the same source.
- Verification, unlike conversion, must be deterministic: the same converted output always passes or fails the same checks.

## 5. Native extension runtime

The native extension API should remain small and stable:

```text
registerTool
registerCommand
registerProvider
registerSkill
registerHook
registerRenderer
registerWebPanel
on
```

Every registration returns a disposer. Extensions declare capabilities before activation.

Example manifest:

```json
{
  "permissions": {
    "filesystem": ["workspace"],
    "process": true,
    "network": ["api.example.com"],
    "credentials": ["github"],
    "ui": ["terminal", "web"]
  }
}
```

The runtime must not expose arbitrary kernel internals as public extension APIs.

## 6. Subagents

### 6.1 Default policy

```text
Model-visible subagent tools: disabled
Automatic task delegation: disabled
Internal /adopt conversion workers: enabled for explicit adoption only
```

The system prompt contains no subagent guidance when model-visible delegation is disabled.

### 6.2 Supported execution forms

The runtime may support:

- One-shot child
- Fresh-context child
- Forked-history child
- Persistent background child
- External harness child
- Local subprocess child
- OCI child
- Remote cloud child
- Workflow-managed child

All forms use a common lifecycle:

```text
start
send
interrupt
status
wait
snapshot
resume
dispose
```

Each backend advertises capabilities such as continuation, history fork, structured output, tool restriction, resume, remote attachment, and hard termination. Unsupported capability requests fail explicitly.

### 6.3 Imported subagent plugins

An adopted plugin may register subagent capabilities, but activation must show that it enables model-visible delegation. Installing a plugin must not silently change the default single-agent posture.

### 6.4 Explicit subagents (/subagents)

Delegation is something the user invokes, not something the model decides. `/subagents` spawns child sessions over the daemon RPC — each with its own chosen model, effort level, tools, and placement — for jobs like a cross-model review or a cheap-model test sweep. Children appear in mission control and in the session tree like any other session. There is no subagent tool in the model's hands by default and no delegation prose in the prompt, ever (section 2.5).

### 6.5 Side conversations (/btw)

`/btw` opens a threaded side conversation: a side-channel branch of the session tree that sees everything the main agent has done, answers immediately while the main agent keeps working, and is never seen by the main agent.

- Threads are first-class: `/btw <question>` starts one, `/btw continue [thread]` resumes it, and multiple named threads coexist.
- A side thread can use tools under the session's normal permission and sandbox policy.
- Side-channel branches are excluded from main-branch compaction (section 13).
- A thread's conclusions can be injected back into the main conversation as an explicit event, never silently.

### 6.6 Workflows

Deterministic orchestration in the style of Claude Code workflows and Codex automations: a workflow is a plain script over the SDK that spawns subagent children — fan-out, pipelines, verify passes — with ordinary code deciding control flow instead of the model. Workflows are user-invoked like everything else in this section, their children appear in the tree and mission control, and each child carries its own model and effort level.

This does not reopen the non-goal (section 18): there is no bespoke workflow language. A workflow is code over the same RPC surface every client uses — nothing to learn beyond the SDK. Existing Claude Code workflow scripts and Codex automations are adoption targets for the compiler like any other ecosystem resource.

## 7. Prompt design

### 7.1 System prompt

Follow Pi's minimal approach.

The stable base contains only:

- Agent identity
- Working directory
- Active tools
- Applicable AGENTS.md instructions
- Essential operating constraints

Rules:

- No subagent section unless enabled.
- No full skill body until selected.
- No instructions for inactive features.
- No complete plugin catalog in the prompt.
- Dynamic information should not invalidate the stable prompt-cache prefix unnecessarily. The current user request and other per-turn content are appended after the stable prefix, never inside it.
- If a capability is unavailable for a request, it contributes zero prompt tokens.

Cache discipline is the enforcement mechanism behind these rules, and Pi's hit rate is the target because Pi follows it strictly: provider prompt caches match on the longest unchanged prefix, so the prompt is built append-only. The system prompt and tool schemas freeze at session start; skills, context, and injected instructions arrive as appended messages, never as edits to earlier content; nothing already sent is reordered, rewritten, or timestamped. The only deliberate cache break is compaction. Any feature that would mutate the prefix mid-session must find an append-only design instead.

### 7.2 Global AGENTS.md

The global file should remain intentionally small. Automated learning may edit only a managed block:

```md
<!-- bolt:auto:start -->
- Prefer the smallest relevant verification command first.
- Report commands run and their outcomes.
<!-- bolt:auto:end -->
```

Content outside this block remains user-owned.

### 7.3 Minimal profile

A DSH-style minimal profile ships alongside the standard one: two tools (shell and file editing), the base prompt, nothing else. It is the smallest thing that is still Bolt — for cheap models, benchmarking, and users who want the model to do the thinking rather than the harness.

### 7.4 Tool surface

The model's tool surface is a profile decision, and the default roster stays deliberately small — every added tool costs prompt tokens and decision quality:

```text
chat      (none)
minimal   shell, edit
standard  shell, read, edit, write, web_search, fetch_content
```

- The browser (section 10.5) is an opt-in addition to any profile, not a standard tool.
- There is no subagent tool, task list tool, or planning tool in the roster: delegation is user-invoked (section 6.4), and planning is a mode (section 3.4), not a tool the model juggles.
- Extensions and MCP servers add tools, and every tool is toggleable per session.
- A tool that is unavailable contributes zero prompt tokens (section 7.1) — profiles are subtractive from nothing, not additive onto a bloated base.

## 8. Learning loop

The learning system is a ladder of three tiers, each with its own approval control:

```text
tier 1  instructions        managed AGENTS.md block only      approval: auto | manual
tier 2  skills and hooks    drafted skills, prompts, hooks    approval: auto | manual
tier 3  extensions          generated native extensions       approval: auto | manual
```

Default:

```text
tier 1: auto
tier 2: manual
tier 3: manual
```

`/learn` shows the tier controls and the current learned rules. Approval semantics differ by what the artifact is, not by tier alone. For non-executable artifacts (instruction rules, skills, prompt templates), `auto` means the change activates directly. For executable artifacts (hooks, extensions), `auto` governs drafting only — generation, quarantine, capability manifest, typecheck, and tests run automatically, but activation always requires one explicit approval (section 8.4). There is no setting that auto-enables executable code.

### 8.1 Evidence sources

Learning candidates may come from:

- Explicit user corrections
- Repeated user preferences
- Repeated acceptance and rejection patterns
- User-authored instruction changes
- Stable outcomes across multiple sessions

Do not learn global instructions directly from:

- Repository files
- Tool output
- Web content
- Model suggestions
- Imported plugin instructions
- A single assistant-generated session summary

Session summaries may identify candidates, but candidates require supporting user-originated evidence before promotion.

To be explicit about what this is not: Bolt has no ambient memory system. Nothing retrieves past-session content and injects it into the prompt uninvited — no vector store of conversation summaries, no invisible context stuffing. What persists across sessions is a small set of plain-text instruction rules in a visible AGENTS.md block, each one evidence-gated, ledgered (section 8.7), and regression-tracked. Recall of past work exists only as a pull: the agent or the user searches session history when they choose to (section 17.2), and anything brought forward arrives as an explicit, visible event.

### 8.2 Tier 1: managed instructions

May add, consolidate, or remove rules only inside the managed global AGENTS.md block.

Requirements:

- Strict line and byte budget
- Deduplication
- Conflict detection
- Rule provenance
- Confidence and evidence count
- Atomic writes and file locking
- Revision history and rollback
- Project-specific rules must not become global

Commands:

```text
/learn
/learn diff
/learn why <rule>
/learn forget <rule>
/learn rollback
```

### 8.3 Manual mode

When a tier is set to manual, every proposed change in that tier requires approval. The user can approve globally, approve for the current project, reject, or suppress future suggestions.

### 8.4 Tiers 2 and 3: drafted artifacts

Tier 2 may draft:

- Skills
- Prompt templates
- Declarative workflows
- Hooks

Tier 3 may draft:

- Native extensions

Generated executable hooks and extensions are high risk. They must be:

1. Written to quarantine.
2. Labeled as model-generated.
3. Assigned an explicit capability manifest.
4. Typechecked and tested in isolation.
5. Shown as a complete diff.
6. Explicitly approved before activation.

Even with tier 2 or tier 3 set to auto, executable hooks and extensions must never auto-enable.

### 8.5 Experimental self-extension loop

Provide an experimental mode in which the harness can write extensions for itself.

This feature must display a persistent warning:

> Experimental and high risk. Generated extensions execute code and may access files, processes, networks, credentials, or the harness runtime. Review the complete source and requested capabilities before activation.

Additional controls:

- Disabled by default
- Separate opt-in setting
- Quarantined output directory
- No host credential access during generation or tests
- No automatic activation
- No modification of the kernel
- Mandatory source diff
- Mandatory capability review
- One-command disable and rollback
- Provenance linking the generated extension to source sessions and evidence

### 8.6 Insights engine

The evidence layer for the learning loop is an adopted implementation of `observal/pi-insights` (the Observal Pi extension), promoted from extension to a built-in `/insights` command.

```text
/insights
/insights --refresh
/insights --since 7d
/insights --md
```

The pipeline is kept as implemented there:

1. Scan all session logs.
2. Extract deterministic per-session stats (tool counts, tokens, cost, languages, git activity, response times), cached permanently.
3. LLM facet extraction per session (goals, outcomes, satisfaction, friction), cached until `--refresh`.
4. Aggregate with decay weighting (10-day half-life), compute week-over-week diffs, detect anomalies and trajectories, distinguish resolved from ongoing friction, and gather user context (AGENTS.md, installed skills, extensions, packages) so suggestions never propose what already exists.
5. Generate report sections with parallel prompts plus a synthesis pass, rendered as a self-contained HTML or Markdown report.

Properties Bolt depends on:

- Temporal awareness: diffs and trajectories, not a static portrait of usage.
- Negative suggestions: a "stop doing" section with concrete alternatives, not just additions.
- Model spend analysis: overspend and underspend detection with estimated savings, which feeds per-role model selection (section 2.7).
- Deterministic stats kept separate from model-generated facets, each cached independently.

Relationship to learning (section 8.1): the insights report is a candidate generator, not an evidence source. A suggestion appearing in a report still requires user-originated evidence before promotion into the managed AGENTS.md block; the user accepting a report suggestion is that evidence.

### 8.7 Learning ledger and regression tracking

Every change the learning system makes, in any tier, is recorded in its own ledger under `~/.bolt/learn/`, separate from the artifacts it modifies:

- Each entry records the change itself, its tier, provenance, supporting evidence, confidence, and when it took effect.
- The ledger is append-only and is what powers `/learn diff`, `/learn why`, and rollback.
- Outcome tracking: insights metrics (friction, tool errors, cost, satisfaction, outcome rates) are compared between sessions before and after each change took effect, so the system can see whether its own changes helped.
- A change whose after-metrics regress is flagged in the insights report with the evidence. A tier 1 auto change that regresses is automatically reverted, and the reversion is itself a ledger entry; changes in manual tiers are proposed for removal, never silently removed.
- Attribution stays honest about confounders: metrics move for many reasons, so regression flags carry confidence, and changes that took effect together are evaluated together rather than blamed individually.

The learning loop thereby gets the same treatment Bolt gives everything else: its actions are logged, reversible, and judged by evidence.

Licensing: pi-insights is AGPL-3.0-only. Bundling it requires either relicensing by its copyright holders or keeping Bolt's insights module under a compatible license; this folds into open decision 9.

## 9. Permission model

### 9.1 Default philosophy

Follow Pi's core insight: repeated manual permission dialogs create habituation and do not provide a strong security boundary.

The primary safety mechanism is sandbox policy, not constant confirmation prompts.

### 9.2 Permission profiles

Provide at least:

- `direct`: Pi-style operation within the selected sandbox, without routine per-command prompts
- `auto`: classifier-mediated approval similar to Claude Code auto mode
- `manual`: explicit approval for gated actions
- `deny`: deny the capability

The initial default between `direct` and `auto` remains a product decision. Regardless of UI mode, sandbox restrictions remain authoritative.

### 9.3 Auto mode

Auto mode is the recommended midpoint for users who want more intervention than Pi without approving every command.

The classifier receives structured action data and policy, not arbitrary untrusted prose. It may approve only within administrator and user policy ceilings. It cannot widen filesystem, network, credential, or cloud permissions.

All automatic decisions are logged with their reason and policy version.

### 9.4 Manual mode

Manual approval remains available but should present concise consequences rather than raw commands alone:

```text
This action will:
  Write outside the workspace: ~/.config/example
  Contact: api.example.com
  Execute: npm install

[Allow once] [Allow for session] [Deny]
```

## 10. Sandbox architecture

### 10.1 Isolation levels

Support the same broad classes as Claude Code:

1. Per-command sandbox
2. Whole-runtime sandbox
3. Dev container
4. Custom OCI container
5. Virtual machine provider
6. Managed or self-hosted cloud worker

### 10.2 Per-command controls

- Filesystem read/write allowlists
- Explicit read/write denials
- Protected harness and extension configuration
- Canonical path and symlink enforcement
- Network domain allowlist and denylist
- Strict network allowlist mode
- Local port binding policy
- Unix socket policy
- HTTP and SOCKS proxy support
- Credential file blocking or masking
- Secret environment-variable blocking or masking
- Optional approved unsandboxed execution
- Violation reporting
- `failIfUnavailable`

If confinement is required and no provider is available, execution must fail. It must never silently run unsandboxed.

### 10.3 Platform providers

- Linux: Bubblewrap and Landlock
- macOS: Seatbelt
- Windows: restricted token and ACL, with WSL2 as an additional option
- Portable: OCI

### 10.4 Extension isolation

Whole-runtime or extension-process isolation is required for untrusted adopted extensions. Sandboxing only shell commands does not constrain an extension that directly uses filesystem or process APIs.

### 10.5 Sandboxed browser

Browser and computer use are tools that live inside the sandbox, not beside it: the browser runs under the session's sandbox profile, its egress under the same network allowlists, its downloads inside the workspace mounts. The agent can drive the app it is building, take screenshots, and do web research — with no side door around network policy.

### 10.6 Web access

Web search and content fetching are core capabilities, not an afterthought — an agent that cannot read the web re-derives what a search would have told it. The base is an adoption of `pi-web-access` (MIT), which already has the right product shape:

- Two tools — `web_search` and `fetch_content` — with everything else as routing beneath them.
- Zero-config default: keyless search works out of the box; API keys, reused subscription auth, and self-hosted endpoints unlock more providers.
- Provider fallback chains with private-first routing: a self-hosted or local search endpoint is tried before any hosted provider.
- Content extraction modes: readable markdown, exact raw bodies, or a grounded answer produced by a cheap summary model — the main model never burns context on pages it didn't ask to read in full.
- GitHub URLs are cloned locally, not scraped: the agent gets real files and a path to explore, not rendered HTML.
- Video understanding: transcripts, visual descriptions, and frame extraction from video links and local screen recordings.
- SSRF protection and content sanitization built in; hosted third-party page fetchers are explicit opt-in.

Under Bolt, provider calls are ordinary egress: they obey the session's network policy and appear in the event log like any other tool traffic. Provider keys live in the secrets layer (section 12.4), never in prompts or logs.

## 11. OCI runtime

OCI is the common execution substrate for local containers and cloud workers.

The provider contract should cover:

```text
create
uploadWorkspace
start
attach
snapshot
stop
terminate
verifyTerminated
```

Required features:

- Image selection and digest pinning
- CPU, memory, disk, time, and cost limits
- Read-only base filesystem
- Explicit workspace mounts
- Egress policy
- Secret injection
- Durable external session log
- Artifact upload
- Graceful shutdown followed by forced termination
- Idempotent cleanup
- Resource lease and expiry

## 12. Cloud runtime

### 12.1 Provider architecture

Implement trusted cloud provider adapters rather than relying on model-authored CLI commands:

- AWS ECS/Fargate
- Azure Container Apps Jobs or Azure Container Instances
- GCP Cloud Run Jobs
- Kubernetes
- Generic SSH worker

Cloud skills may explain setup, diagnose configuration, and prepare manifests. They must not be the sole owner of provisioning or cleanup.

### 12.2 Lifecycle

```text
requested
-> provisioning
-> starting
-> running
-> draining
-> terminating
-> terminated
```

Failure is recorded separately without pretending cleanup completed.

Termination sequence:

```text
stop admission
-> signal agent
-> flush event log
-> upload artifacts
-> request graceful process exit
-> wait bounded grace period
-> force kill
-> delete compute resources
-> revoke temporary credentials
-> verify resources are absent
```

### 12.3 Leak prevention

Every cloud resource is tagged with:

- Session ID
- Owner ID
- Creation time
- Expiry time
- Provider adapter version

An external reconciler periodically destroys expired or orphaned resources. Cleanup cannot depend on the agent running inside the container.

### 12.4 Cloud credentials

Prefer:

- AWS IAM roles
- Azure managed identities
- GCP workload identity
- Short-lived Git credentials
- External secret brokers

Never persist cloud credentials in prompts, session logs, generated extensions, or adoption metadata.

## 13. Compaction

Adopt Pi's compaction behavior and defaults as the compatibility target. Reuse Pi's MIT-licensed implementation where practical, preserving required notices and attribution.

Required behavior:

- Proactive threshold compaction
- Overflow recovery and retry
- Manual compaction
- Configurable reserved output budget
- Configurable recent-context budget
- Turn-boundary cut selection
- Split-turn handling
- Tool call/result integrity
- Previous-summary iteration
- Structured continuation summary
- Cumulative read-file and modified-file tracking
- Branch summarization
- Side-channel exclusion: /btw threads never enter main-branch compaction
- Full original history retained outside the compacted model surface
- Extension interception for custom compaction
- Compaction usage included in cost and token totals

Compatibility should be enforced with behavior tests against Pi fixtures rather than only copied implementation details.

## 14. Session daemon and clients

One authoritative daemon owns the session. Clients are projections over the same event protocol:

- Terminal UI
- Web UI
- Mobile app
- IDE extension (VS Code, JetBrains)
- Headless client
- SDK
- Remote control client
- Cloud worker connection

Capabilities:

- Simultaneous terminal and web attachment
- Detach without stopping work
- Reconnect after transport loss
- Permission requests from any authorized client
- Tree-structured sessions with first-class branches
- Checkpoint and rewind
- Model and effort switching
- Context and cost visibility
- Sandbox status
- Background operation status
- Cloud transfer
- Push notifications for permission requests, blockers, and completion

The web, terminal, and mobile clients must not implement separate agent loops.

### 14.1 Session placement

Mirroring the Claude app's mode structure (chat, cowork, code — with code split into remote control, local, and cloud), Bolt organizes its clients around one Code surface with three session placements:

- `local`: the agent loop runs on the user's machine, driven from the terminal, desktop, or local web client.
- `cloud`: the session runs on a cloud worker in an OCI sandbox (sections 11 and 12); any client can spawn one.
- `attach`: remote control of an existing session from phone or web — a projection with steering rights, never a second loop (section 15.3).

Placement is only where the loop runs; every placement speaks the same daemon protocol and event log, and a session can move between placements via cloud transfer. `bolt web` serves the web client on localhost with two modes, DSH-style: **code**, the full session UI, and **chat**, a zero-tool chat profile for using the harness as a local chat app — same daemon, same event log, one toggle apart. A general cowork-style assistant surface is out of scope initially (section 18).

Parallel sessions on the same repository isolate their working state in git worktrees, so mission control (section 17.3) can run several agents against one repo without them fighting over files. Worktrees are created on demand when a session opens an already-busy repo and cleaned up automatically when a session ends without changes.

### 14.2 Checkpoint and rewind

The event-sourced log already makes conversation state replayable; checkpointing extends that to the workspace. Every turn that modifies files records a workspace snapshot, and a session can rewind to any turn — conversation and files together, or either alone. Snapshots use copy-on-write where the filesystem supports it, with git-based shadow commits as the fallback. Rewind never rewrites the user's own git history.

### 14.3 Session budgets

A session can carry token, cost, and wall-clock budgets. Crossing a budget pauses the loop loudly at the next safe boundary rather than killing work mid-write, and budget state is visible in every attached client. Cloud sessions inherit the resource leases in section 11 on top of this.

### 14.4 Tree sessions

A session is a tree of turns, not a list. Every event carries an id and a parent id; branching, rewind (section 14.2), and forked-history children (section 6.2) are all the same operation — start a new branch from an existing node. Nothing is ever destroyed by branching: the old branch remains addressable, compaction summarizes per branch (section 13), and clients render the tree rather than pretending the session is linear.

### 14.5 Protocol and SDK

The SDK is deliberately thin because the daemon owns all behavior; a client is transport plus rendering. The surface is:

- **RPC** for requests with answers: session lifecycle (create, branch, transfer, dispose), send, interrupt, permission responses, configuration, placement moves.
- **Event stream** for the session tree: subscribe from any node, tail live turns. This is the JSONL log over the wire, nothing more.
- **Snapshot plus tail** so attachment is cheap: a client joining a long session fetches a state snapshot and streams from that point, instead of replaying thousands of events.
- **Resumable cursors**: every event has an id; reconnection resumes from the last acknowledged id with at-least-once delivery, so a phone losing signal misses nothing.
- **Idempotency keys on sends**: a client retrying over a flaky network must never double-send a message or a permission grant.
- **A blob channel** separate from the event stream for large payloads — artifacts, images, file uploads. The event log carries references, never megabytes.
- **Auth and pairing**: revocable per-device credentials with capability scopes; an observer credential can stream but not steer (section 15.3).
- **Capability negotiation**: client and daemon exchange protocol version and feature sets on connect, and mismatches degrade loudly (open decision 5 covers the versioning scheme).
- **Presence**: who else is attached to the session, so simultaneous terminal, web, and mobile clients can indicate each other.

All client SDKs are generated from one protocol schema — the TypeScript, Swift, Kotlin, and Rust clients are codegen over the same definitions, not four hand-written libraries. Transports are pluggable beneath the same surface: Unix socket locally, WebSocket through the relay remotely. MCP remains the protocol for external tool servers; Bolt is an MCP client, never a replacement (section 18).

Commands are RPCs, and that closes a gap other harnesses have: in Pi, slash commands live inside the TUI process, so the model has no path to them — the user can never say "resume yesterday's session about the auth bug" and have the agent do it. In Bolt, every slash command is a thin client wrapper over a daemon RPC, and a scoped harness-control tool exposes that same RPC surface to the model on request. Anything the user can do with a command, the agent can do when asked in natural language — find and resume a session, branch, compact, switch model — under the session's normal permission policy, with every harness action it takes logged like any other tool call.

### 14.6 Session storage

Storage is split into a canonical log and a derived index, each in the format that suits its job:

- **JSONL is the source of truth.** One append-only event log per session; the tree lives in the parent pointers. Appends are crash-safe (a torn write corrupts at most the final line, and recovery is truncation), the format is human-readable and greppable, it streams and tails naturally, it diffs and syncs cleanly from cloud workers, it needs no library to parse, and it is the lingua franca of every ecosystem Bolt adopts — session import is largely a JSONL-to-JSONL translation.
- **Per-session JSON sidecar caches are the default index.** The pi-insights pattern (section 8.6): deterministic stats extracted once per session and cached as small JSON files. This covers the session picker, insights aggregation, and most viewer queries with no database at all.
- **SQLite is the escalation for search, never authoritative.** The one workload sidecar caches cannot serve is full-text search over transcript content across thousands of sessions; grep over gigabytes of logs is seconds, FTS5 is milliseconds with ranking. When viewer search demands it, the index is built from the logs already on disk. Because it is derived, it carries no migration burden on history — a schema change means deleting the database and rebuilding, never rewriting a log.
- The write path is: append to JSONL first, then update whatever index exists. An index may lag; the log may not. If they disagree, the log wins and the index is rebuilt.
- Cloud workers durably append and upload only JSONL (section 11); each client maintains its own local caches and index. Index files never cross machines.

## 15. Interface direction

### 15.1 Terminal

Use Pi's rendering principles:

- Differential rendering
- Synchronized output
- Main-screen scrollback preservation
- Optional alternate-screen mode
- Overlay support
- Inline diffs and images
- IME-safe cursor placement
- Responsive layouts

### 15.2 Web

Use DSH's plugin-oriented presentation principles:

- One conversation event projection
- Tool-specific render intents
- Diff, terminal, read, search, web, workflow, and generic cards
- Extension-provided panels and conversation nodes
- Live permission and sandbox state
- Detachable and reconnectable sessions

### 15.3 Mobile

The mobile app is a remote-control client over the same daemon protocol, in the way Claude Code and Codex expose sessions on a phone. It runs no agent loop of its own and is a projection like every other client.

Capabilities:

- List, open, and start sessions on cloud workers or reachable daemons
- Live conversation view with tool, diff, and terminal cards
- Reply to and steer a running session
- Answer permission requests from the phone
- Push notifications for permission requests, blockers, task completion, and PR events
- Review diffs and approve before a push
- Detach and reconnect; a phone losing signal never kills the session

Requirements:

- Connections go through an authenticated relay or direct daemon pairing; the daemon, not the client, remains authoritative
- Device pairing with revocable per-device credentials
- Read-only observer mode for watching a session without steering rights
- Notification payloads exclude secrets and full file contents

The apps are native per platform — SwiftUI on iOS, Jetpack Compose on Android — not Flutter or React Native. Because the client is a projection with almost no business logic, a cross-platform framework saves little while blocking the platform features this app lives on: iOS Live Activities showing a running session's status on the lock screen, Android foreground services with approve/deny actions in the notification shade, widgets, share sheets, and native streaming-text performance for live transcripts. The protocol client, event sync, reconnection, and auth live in the shared core generated from the protocol schema (section 14.5), so the per-platform code is UI only.

### 15.4 Headless and automation

The same daemon serves non-interactive callers:

- `bolt run -p "<prompt>"` executes a session headlessly and can emit structured JSON output for scripting.
- CI integration: run Bolt as a pipeline step or a GitHub Action, with the event log uploaded as the run artifact.
- Event subscriptions: a session can be woken by external events — a PR comment, a CI failure, a webhook — and act on them under its normal permission and sandbox policy.
- Schedules: recurring triggers that start a fresh session or wake a persistent one.
- Automation runs under the same daemon, log, budgets, and sandbox rules as interactive sessions; there is no separate headless code path.

### 15.5 Session viewer

A DSH-style session viewer is the observability surface over the event log and its index:

- Tree visualization of sessions and branches with jump-to-node
- Event timeline with per-turn tokens, cost, latency, and model used
- Tool call inspection: inputs, outputs, duration, sandbox profile applied
- Permission decisions and classifier reasons (section 9.3), as logged
- Compaction events with before and after context sizes
- Cross-session search and filtering, backed by the caches and search index (section 14.6)
- Live tail of running sessions, local and cloud, in the same view
- Export of any subtree as a plain JSONL slice

The viewer is a read-only projection over the same protocol as every other client; it introduces no second data path.

## 16. Compatibility and diagnostics

### 16.1 Doctor

```text
/doctor
```

Reports:

- Detected harness installations
- Compatible and incompatible resources
- Upstream API changes
- Conversion drift
- Dependency conflicts
- Missing binaries
- Sandbox availability
- Cloud provider readiness
- Credential references requiring setup
- Extensions with elevated permissions
- Generated extensions awaiting approval
- Leaked or orphaned cloud resources

### 16.2 Compatibility catalog

Publish tested compatibility status for real plugins. Plugin authors should be able to run the same conformance suite in their CI.

Seed the catalog with what people demonstrably install across ecosystems: documentation retrieval, memory extensions (adoptable, though never part of core — section 8.1), planning workflows, side-conversation channels, browser control, git workflows, config sync, and usage analytics. The proven winners get adopted, not rebuilt.

### 16.3 Deterministic replay

Because sessions are event-sourced, a recorded session is a test fixture for free. Replaying logged sessions against a new build — with model responses stubbed from the log — catches regressions in tool execution, compaction, permission decisions, and rendering without spending a token. The same mechanism, with live models substituted for the recorded ones, is the personal model bench (section 19): the user's own workflows become the benchmark suite.

## 17. Feature directions

Beyond the core, these are the features Bolt's own primitives make uniquely possible.

### 17.1 From the session tree

- **What-if branches**: re-run any node's branch with a different model, prompt, or approach; compare the branches side by side and keep the winner.
- **Second opinion**: one command to have a different model review the current branch's diff. Model-agnosticism makes cross-model review nearly free, and it catches the blind spots a model has about its own work.
- **Shareable replays**: export any subtree as a scrubbable replay link — for bug reports, teaching, and showing what the agent did.

### 17.2 From insights and learning

- **Cost autopilot**: close the loop on spend analysis — route mechanical tasks to cheaper models, escalate on failure, and log every routing decision with its reason.
- **Guardrails from your own mistakes**: a repeated failure pattern detected by insights becomes a drafted tier 2 hook that blocks or warns. The agent stops making the user's recurring mistakes, specifically.
- **Cross-session recall**: on request, search past session facets to reapply an old fix instead of re-deriving it. Strictly pull-based — nothing from past sessions enters the prompt uninvited (section 8.1).
- **Daily digest**: an end-of-day summary of what every session did, what it cost, and what is waiting on the user.

### 17.3 From the daemon and placements

- **Mission control**: one view of every running session across repos and placements — status, cost, current blocker — with the mobile app as the pocket version.
- **Live app preview**: a cloud session tunnels its dev server to the phone, so the user watches the running app change while steering the agent from the same screen.
- **Review inbox**: agents deliver finished diffs into an inbox that is reviewed and approved in batches, from any client.

### 17.4 From adoption and sandboxing

- **Team profile in the repo**: a checked-in profile lockfile gives a new teammate the team's exact skills, hooks, adopted packages, and sandbox policy in one run.
- **Personal config sync**: the user's own skills, hooks, and settings follow them across their machines — distinct from the team profile.
- **Privacy switch**: per-session zero-egress mode with a local model, enforced by sandbox network policy rather than promises.
- **Blind secrets**: the model reads placeholders; real values are injected only at execution time inside the sandbox. The agent uses credentials it never sees.

## 18. Explicit non-goals

Bolt should not add:

- Another skill format
- Another MCP replacement
- Another provider API abstraction when Pi's is sufficient
- A plugin marketplace before adoption works reliably
- Default model-driven subagents
- A large built-in workflow language
- Automatic activation of generated executable code
- Silent emulation of unsupported foreign APIs
- A general cowork-style assistant surface before the Code surface is excellent
- An ambient memory system that injects past-session content into the prompt uninvited

## 19. Delivery phases

### Phase 1: foundation and adoption proof

- Minimal event-sourced kernel
- Pi-style agent loop and compaction
- Pi-quality terminal client
- Read-only installation discovery
- Native extension API
- Pi, OpenCode, and DSH conversion proof for tools, hooks, and skills
- Adoption manifest, checks, activation, and rollback
- No default subagents

### Phase 2: daily-driver runtime

- Session daemon
- Web client
- Simultaneous TUI and web attachment
- Local sandbox providers
- Whole-runtime OCI execution
- Compatibility doctor
- Session viewer
- Profile lockfiles
- Session import
- Tier 1 learning (managed instructions)
- Built-in insights report (adopted pi-insights pipeline)
- Checkpoint and rewind

### Phase 3: remote execution

- Cloud worker protocol
- AWS adapter
- Azure adapter
- GCP adapter
- Kubernetes and SSH adapters
- Detach and reconnect
- Mobile remote-control app
- Push notification relay
- Artifact synchronization
- Resource reconciler
- Cost and lease enforcement

### Phase 4: tiers 2 and 3 learning

- Tier 2 skill and hook drafting
- Tier 3 experimental self-extension generation
- Quarantine and capability review
- Compatibility-aware extension updates
- Public compatibility catalog
- Personal model bench: replay real workflows from insights data to compare models on the user's own tasks

## 20. Open decisions

1. Product name: Bolt (screw-bolt-as-lightning-bolt logo). Decided as the working name; the package namespace and npm scope remain open.
2. Whether the default permission profile is `direct` or `auto`. Current lean: `auto`, since section 9.3 already positions it as the recommended midpoint; this must be settled before Phase 1 ships because the agent loop needs a permission posture from day one.
3. Exact compatibility surface promised for the first Pi, OpenCode, and DSH release.
4. Whether adopted extensions are always isolated or may be promoted to trusted in-process execution.
5. Protocol versioning for the client wire format. The at-rest format is decided: JSONL event log as source of truth with derived caches and a search index (section 14.6).
6. First cloud provider to support before generalizing all three.
7. Global and project learning budgets.
8. Whether cloud transfer moves a session or creates a fork.
9. Licensing and attribution policy for reused Pi code, and license compatibility or relicensing for the AGPL-3.0 pi-insights implementation (section 8.6).
10. Whether the mobile app connects through a hosted relay service or direct daemon pairing first.

## 21. Success criteria

The initial product thesis is proven when a user can:

1. Install the harness with one command.
2. Detect existing Pi, OpenCode, and DSH resources.
3. Select a real third-party plugin.
4. Adopt it in one operation using isolated conversion workers.
5. Inspect generated code, permissions, provenance, and test results.
6. Activate it without modifying the original installation.
7. Use it in the terminal and web clients against one session.
8. Run its tools inside a required sandbox.
9. Detach and reconnect without losing work.
10. Update or roll back the adopted plugin safely.
11. Watch and steer a cloud session from the mobile app, including answering a permission request from the phone.
