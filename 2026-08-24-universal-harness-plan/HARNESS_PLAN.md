# Universal Harness Plan

Status: Working product plan

Updated: 2026-08-24

## 1. Product thesis

Build a universal coding-agent harness that adopts existing agent ecosystems instead of forcing users to migrate manually.

The product promise is:

> Keep your extensions, skills, hooks, configuration, and sessions. Adopt them into one runtime, then run locally, through a web interface, or in an OCI environment.

The harness is not another generic Claude Code clone. Its differentiators are:

1. One-command adoption of Pi extensions, OpenCode plugins, and DeepSeek Harness plugins.
2. A shared local and remote session runtime with terminal, web, and mobile clients.
3. Explicit, non-bypassable sandbox profiles using operating-system or OCI isolation.
4. Minimal prompts and no default model-driven subagent delegation.
5. Pi-compatible compaction behavior.
6. An optional learning loop that maintains concise user instructions and can draft skills, hooks, and experimental self-extensions.

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

The normal agent does not receive a subagent tool or subagent instructions by default. Subagents are opt-in capabilities.

Explicit system operations such as `/adopt` may use bounded internal conversion workers. These workers are not exposed to the normal model as general delegation tools.

### 2.6 Security is enforced below extensions

Permission hooks alone are not a security boundary. Foreign extensions, MCP servers, hooks, tools, and commands must not be able to bypass required isolation by directly using host APIs.

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
harness install pi @juicesharp/pi-web-tools
harness install opencode opencode-wakatime
harness install dsh @vendor/dsh-plugin
```

Optional passthrough form:

```bash
harness adopt -- pi install @juicesharp/pi-web-tools
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
  ~/.harness/adopted/pi/@juicesharp/pi-web-tools/1.4.2/source
```

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
~/.harness/adopted/
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

### 7.2 Global AGENTS.md

The global file should remain intentionally small. Automated learning may edit only a managed block:

```md
<!-- harness:auto:start -->
- Prefer the smallest relevant verification command first.
- Report commands run and their outcomes.
<!-- harness:auto:end -->
```

Content outside this block remains user-owned.

## 8. Learning loop

The learning system has two independent controls:

```text
approval: auto | manual
scope: minimal | extensive
```

Default:

```text
approval: auto
scope: minimal
```

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

### 8.2 Auto plus minimal

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
/memory
/memory diff
/memory why <rule>
/memory forget <rule>
/memory rollback
```

### 8.3 Manual mode

Every proposed change requires approval. The user can approve globally, approve for the current project, reject, or suppress future suggestions.

### 8.4 Extensive learning

Extensive scope may draft:

- Skills
- Prompt templates
- Declarative workflows
- Hooks
- Native extensions

Generated executable hooks and extensions are high risk. They must be:

1. Written to quarantine.
2. Labeled as model-generated.
3. Assigned an explicit capability manifest.
4. Typechecked and tested in isolation.
5. Shown as a complete diff.
6. Explicitly approved before activation.

Even in automatic extensive mode, executable hooks and extensions must never auto-enable.

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
- Full original history retained outside the compacted model surface
- Extension interception for custom compaction
- Compaction usage included in cost and token totals

Compatibility should be enforced with behavior tests against Pi fixtures rather than only copied implementation details.

## 14. Session daemon and clients

One authoritative daemon owns the session. Clients are projections over the same event protocol:

- Terminal UI
- Web UI
- Mobile app
- Headless client
- SDK
- Remote control client
- Cloud worker connection

Capabilities:

- Simultaneous terminal and web attachment
- Detach without stopping work
- Reconnect after transport loss
- Permission requests from any authorized client
- Session branching
- Model switching
- Context and cost visibility
- Sandbox status
- Background operation status
- Cloud transfer
- Push notifications for permission requests, blockers, and completion

The web, terminal, and mobile clients must not implement separate agent loops.

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

## 17. Explicit non-goals

Initial versions should not add:

- Another skill format
- Another MCP replacement
- Another provider API abstraction when Pi's is sufficient
- A plugin marketplace before adoption works reliably
- Default model-driven subagents
- A large built-in workflow language
- Automatic activation of generated executable code
- Silent emulation of unsupported foreign APIs

## 18. Delivery phases

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
- Profile lockfiles
- Session import
- Auto/manual minimal learning

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

### Phase 4: extensive learning

- Skill and hook drafting
- Experimental self-extension generation
- Quarantine and capability review
- Compatibility-aware extension updates
- Public compatibility catalog

## 19. Open decisions

1. Product name and package namespace.
2. Whether the default permission profile is `direct` or `auto`. Current lean: `auto`, since section 9.3 already positions it as the recommended midpoint; this must be settled before Phase 1 ships because the agent loop needs a permission posture from day one.
3. Exact compatibility surface promised for the first Pi, OpenCode, and DSH release.
4. Whether adopted extensions are always isolated or may be promoted to trusted in-process execution.
5. Canonical session wire format and protocol versioning.
6. First cloud provider to support before generalizing all three.
7. Global and project learning budgets.
8. Whether cloud transfer moves a session or creates a fork.
9. Licensing and attribution policy for reused Pi code.
10. Whether the mobile app connects through a hosted relay service or direct daemon pairing first.

## 20. Success criteria

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
