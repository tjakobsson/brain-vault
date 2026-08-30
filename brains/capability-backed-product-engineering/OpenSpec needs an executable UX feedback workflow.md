---
type: permanent
status: developing
tags:
  - openspec
  - ux-ui
  - ios-development
  - contract-first
  - ai-assisted-development
created: 2026-08-30
---

OpenSpec needs a project-specific workflow for executable UX exploration when the design medium is a running app rather than a drawing tool. Its standard OPSX workflow can hold requirements, implementation tasks, and revisions. It does not itself build an Xcode project, drive Simulator, run API mocks, or turn a discovery in the interface into a proposed backend contract.

This is a real gap, but not a reason to put a conventional design phase in front of engineering. The useful addition is a feedback workflow in which a code agent designs with production UI code, contract-backed fakes, and a running simulator. Existing APIs inform what the agent can make real. Interaction work produces explicit proposals for APIs that do not exist yet.

This applies [[Executable capability contracts keep interface design honest|capability contracts]] to OpenSpec and supplies the agent loop needed by [[Build an iOS chat as a contract-backed vertical slice]].

## What OpenSpec already provides

OPSX describes itself as an iterative set of actions rather than fixed phases. Its default path is `propose`, `apply`, `update` or `sync`, then `archive`, with optional `explore` and `verify` actions. Two parts fit executable UX work well:

- A custom schema can define project-local artifacts, templates, and dependencies in `openspec/schemas/`.
- `/opsx:update` can revise existing planning artifacts when implementation reveals that the design or requirements were wrong.

OpenSpec also supports OpenCode by generating commands and skills under `.opencode/`. Its JSON status and instruction commands give an agent a machine-readable artifact graph and current task state.

These are planning and coordination mechanisms. OpenSpec marks an artifact complete mainly because its file exists. Apply guidance is advisory. `/opsx:verify` searches for evidence but does not enforce a successful build, a simulator interaction, or a screenshot. OpenSpec has no OpenAPI parser, mock server, contract diff, or Xcode executor.

The distinction matters. OpenSpec can tell a capable coding agent what loop to run and preserve what the loop learns. It is not the loop's runtime.

## Use the running app as the design artifact

For this kind of work, a static screen is weak evidence. It cannot establish navigation, keyboard behavior, waiting, cancellation, errors, accessibility focus, or whether an API has enough information to support the intended action.

The design artifact should therefore be an executable iOS build with named scenarios. The code agent uses natural language as input, edits the SwiftUI app, builds it, launches it, performs touches, and observes the result. Xcode 27 supports this through its MCP server. External agents connect through `xcrun mcpbridge`; Xcode 27 can boot simulators, install and launch apps, synthesize touches, capture screenshots, render previews, build, test, and inspect debugger state.

This does not make every rendered behavior a product promise. Each interaction still needs one of the dependency labels from [[Executable capability contracts keep interface design honest#A mock needs a source of truth]]:

| Label | Meaning in the executable prototype |
| --- | --- |
| `real` | The app uses an existing provider and integration evidence exists. |
| `contract-backed` | The app uses a fake generated or maintained from an agreed API contract. Provider work remains. |
| `speculative` | The behavior answers a UX question but no provider commitment exists. |

The simulator proves that the interaction works as software. It does not prove that a speculative backend capability exists.

## Add an `executable-ux` schema

A useful artifact graph is:

```text
proposal
   |
   v
capability inventory
   |
   v
interaction model
   |
   v
contract delta
   |
   v
prototype design
   |
   v
tasks -> apply, observe, update, apply again
```

The apparent sequence establishes a reliable starting point. OPSX updates make it a feedback loop once the app runs.

| Artifact | Required content |
| --- | --- |
| `proposal.md` | User outcome, scope, and the UX questions this change must answer. |
| `capabilities.md` | Existing operations and fields, source links, evidence, and `real`, `contract-backed`, or `speculative` status. |
| `interactions.md` | User paths, control-to-capability mapping, state transitions, accessibility behavior, and named scenarios. |
| `contract-delta.md` | Existing API facts plus proposed operations, fields, errors, timing, examples, and unresolved backend questions. |
| `prototype-design.md` | Swift dependency boundary, fake strategy, launch configuration, Xcode destination, and evidence locations. |
| `tasks.md` | Implementation and one simulator verification task for every named scenario. |

The custom schema can encode these dependencies and make `tasks` a prerequisite for apply:

```yaml
name: executable-ux
version: 1
description: Build and learn from a contract-backed executable interface

artifacts:
  - id: proposal
    generates: proposal.md
    template: proposal.md
    requires: []

  - id: capabilities
    generates: capabilities.md
    template: capabilities.md
    requires: [proposal]

  - id: interactions
    generates: interactions.md
    template: interactions.md
    requires: [capabilities]

  - id: contract-delta
    generates: contract-delta.md
    template: contract-delta.md
    requires: [interactions]

  - id: prototype-design
    generates: prototype-design.md
    template: prototype-design.md
    requires: [interactions, contract-delta]

  - id: tasks
    generates: tasks.md
    template: tasks.md
    requires: [prototype-design]

apply:
  requires: [tasks]
  tracks: tasks.md
  instruction: |
    Build and launch the iOS app through Xcode MCP.
    Exercise every named interaction scenario in Simulator.
    Do not silently invent an API operation, field, error, or timing rule.
    Record each unsupported assumption in the change artifacts, classify it,
    update the contract and examples, then continue implementation.
    Record build, test, interaction, and screenshot evidence.
```

Templates matter more than the artifact names. For example, `contract-delta.md` should separate observed facts from proposals. Otherwise the agent can write down its own invented mock and immediately cite that file as authority.

## Run a build-observe-reconcile loop

`/opsx:apply` should run this loop for each scenario:

1. Implement the narrow interaction against a typed client boundary.
2. Supply a named fake scenario from the current contract examples.
3. Build and launch through Xcode MCP.
4. Perform the interaction in Simulator rather than only inspecting a screenshot.
5. Compare the observed behavior with the interaction model.
6. Classify each discovery as a UI correction, a clarification of an existing capability, a proposed API change, or an unresolved product question.
7. Use `/opsx:update` when a discovery changes an artifact. Do not patch only the fake response.
8. Rebuild and repeat until the scenario has evidence.

This classification is the control point. Without it, natural-language-to-UI work is fast but untrustworthy. The same agent can invent a response field, add it to a fixture, and produce a convincing screen without ever exposing the new backend obligation.

## Practical example: running a saved search

Suppose an existing API returns saved searches:

```http
GET /v1/searches
```

The desired iOS behavior is: open a saved search, tap Run, see immediate waiting feedback, then see results or a recoverable error. The current provider can list searches but cannot run one.

The workflow produces this capability table:

| Control or state | Required capability | Initial status |
| --- | --- | --- |
| Saved-search list | `GET /v1/searches` | `real` |
| Run button | `POST /v1/searches/{id}/run` | `speculative` |
| Disabled Run explanation | `canRun` and `disabledReason` | `speculative` |
| Last-run context | `lastRunAt` | `speculative` |

The agent first builds the list from an existing API example. It then implements the Run interaction behind a `SearchClient` and uses contract examples for success, empty results, disabled, delayed, and failed scenarios. A tool such as Prism can serve those OpenAPI examples as an HTTP mock, while an in-process fake can provide deterministic SwiftUI previews and feature tests.

When simulator use shows that a disabled button without an explanation is confusing, the agent must not merely add `disabledReason` to a JSON fixture. It updates `interactions.md` with the explanatory behavior and `contract-delta.md` with the proposed field and examples. Backend work now has a reviewable obligation. The app remains executable against the revised mock, but the capability stays `contract-backed` until provider verification passes.

The evidence for one scenario could be:

```text
Scenario: delayed saved-search run
Build: passed on iPhone simulator with iOS 27
Interaction: launch -> Saved Searches -> "Weekly" -> Run
Observed: Run disables immediately; progress is announced; retry remains hidden
Contract example: run-delayed
Screenshot: artifacts/run-delayed.png
Automated check: SavedSearchRunTests.delayedRunShowsWaitingState
```

## Keep verification honest

Before archive, verification should require:

- Every control maps to a capability and a dependency label.
- Every named scenario has repeatable simulator or UI-test evidence.
- The production API client is exercised by consumer contract tests, not only by view tests.
- Existing API facts cite their source.
- Proposed API behavior appears in the contract delta and provider tasks.
- The agent did not add unclassified fixture fields or operations.
- `contract-backed` capabilities do not get relabeled `real` until provider or integration verification passes.

Because OpenSpec verification is advisory, CI or a repository check must enforce any non-negotiable items such as OpenAPI validation, test success, and required evidence files.

## Conclusion

The proposed workflow is needed. The default OpenSpec workflow is iterative enough to host it, but it lacks the artifacts and execution rules that make UX work both consume and produce API knowledge.

The clean division of responsibility is:

| Part | Responsibility |
| --- | --- |
| OpenSpec | Durable intent, interaction scenarios, discoveries, contract proposals, and task state |
| OpenAPI or AsyncAPI | Executable provider contract and examples |
| Contract mock and Swift fake | Repeatable behavior before the provider exists |
| Xcode 27 MCP and Simulator | Build, interaction, observation, and screenshots |
| Tests and CI | Enforced acceptance and regression evidence |

This is natural language to working UI, but with an audit trail for every capability the interface appears to promise.

## Sources

- [OpenSpec OPSX workflow](https://github.com/Fission-AI/OpenSpec/blob/v1.11.0/docs/opsx.md)
- [OpenSpec customization and custom schemas](https://github.com/Fission-AI/OpenSpec/blob/v1.11.0/docs/customization.md)
- [OpenSpec command reference](https://github.com/Fission-AI/OpenSpec/blob/v1.11.0/docs/commands.md)
- [OpenSpec agent contract](https://github.com/Fission-AI/OpenSpec/blob/v1.11.0/docs/agent-contract.md)
- [OpenSpec supported tools](https://github.com/Fission-AI/OpenSpec/blob/v1.11.0/docs/supported-tools.md)
- [Apple: Giving external agents access to Xcode](https://developer.apple.com/documentation/xcode/giving-external-agents-access-to-xcode)
- [Xcode 27 beta 6 release notes](https://developer.apple.com/documentation/xcode-release-notes/xcode-27-release-notes)
- [Prism API mocking and contract testing](https://github.com/stoplightio/prism)
- [OpenAPI Specification 3.2.0](https://spec.openapis.org/oas/latest.html)
