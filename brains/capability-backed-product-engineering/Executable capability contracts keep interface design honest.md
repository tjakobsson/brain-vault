---
type: permanent
status: developing
tags:
  - contract-first
  - product-engineering
  - ux-ui
  - ai-assisted-development
created: 2026-08-30T09:24:04.912472889+02:00
updated: 2026-08-30T10:45:26.008568790+02:00
---

An interface should only promise actions that the product can perform. In AI-assisted development, that requires more than asking a designer or agent to read backend code. The team needs a small, executable description of each user capability that both the interface and the implementation must obey.

Call this a capability contract. It connects a user intention such as "send this message" to the operation, data, states, and failure behavior that make the intention real.

## What belongs in the contract

For each user action, record:

| Concern | Example for chat |
| --- | --- |
| User intention | Send a message |
| Trigger | Tap Send or submit from the keyboard |
| Preconditions | Non-empty text, authenticated session, conversation accepts messages |
| Operation | `sendMessage(conversationID:text:)` |
| Inputs and outputs | Client message ID, text, accepted message, assistant events |
| Observable states | Ready, sending, streaming, completed, failed, cancelled |
| Failure semantics | Offline, unauthorized, rate limited, server failure |
| Timing semantics | First response target, stream timeout, retry policy |
| Available follow-up actions | Cancel, retry, copy, regenerate |
| Evidence | Contract test, integration test, or production implementation |

OpenAPI can describe HTTP operations, schemas, examples, authentication, and errors. AsyncAPI can describe message channels and events. Neither automatically captures every UX fact, so keep a short state table and acceptance scenarios beside the protocol description.

This is deliberately narrower than a product requirements document. It describes what a specific interaction can rely on. It is also broader than an endpoint schema because waiting, cancellation, authorization, and partial results change the interface.

## Make capability visible in the interface

Every interactive control should trace to one capability contract. A simple review table catches attractive but impossible designs:

| Control | Capability | Current evidence | UI decision |
| --- | --- | --- | --- |
| Send button | Send message | Real API passes integration test | Enabled when preconditions hold |
| Stop button | Cancel response | Contract exists, provider unfinished | Build against contract-backed fake |
| Regenerate button | Regenerate response | No contract | Omit or label as speculative work |

At runtime, the product may also need the server to report which actions are currently allowed. Hypermedia controls are one established form: a response advertises valid next actions instead of forcing the client to guess. A mobile app will usually map those responses into a stable, typed set of known actions rather than render arbitrary server-provided controls. Include a reason when an action is unavailable so the interface can explain a restriction when explanation helps.

## A mock needs a source of truth

A mock is useful when it implements the same contract as the future provider. The app can then switch between fake and live implementations without changing the view or feature logic. Shared examples can drive previews and UI tests, while provider verification proves that the backend returns compatible responses.

Classify dependencies honestly:

- **Real** means the app exercises a deployed implementation.
- **Contract-backed** means an executable fake satisfies an agreed contract that the provider must verify.
- **Speculative** means the team is exploring behavior that nobody has committed to implement.

Speculative prototypes are still useful. Their job is to answer a design question, not to masquerade as completed product work. If the team chooses the design, convert its assumptions into a capability contract and provider work before treating it as part of the application.

Contract tests should exercise the actual API client, not the UI. UI previews and tests can reuse fixtures produced from the contract. This keeps protocol assertions precise without coupling every layout test to HTTP details.

## Why this matters more with AI

An AI agent can create a polished control and invent the service behind it in one pass. Visual completeness therefore says even less about product completeness than it used to.

Give design and coding agents the capability contracts, state tables, and verified examples as context. Require a proposed contract change when an interaction needs an operation or state that does not exist. This turns "the backend cannot do that" into an early, reviewable change instead of a late integration surprise.

The practical consequence is [[Build an iOS chat as a contract-backed vertical slice]]. [[OpenSpec needs an executable UX feedback workflow]] describes how a code agent can preserve these distinctions while designing in Xcode and Simulator. This also applies the end-to-end reasoning in [[@outcome-delivery/Local acceleration cannot fix end-to-end flow]]: faster interface generation does not improve delivery if integration remains the constraint.

## Sources

- [OpenAPI Specification 3.2.0](https://spec.openapis.org/oas/latest.html)
- [AsyncAPI document structure](https://www.asyncapi.com/docs/concepts/asyncapi-document/structure)
- [Consumer-Driven Contracts: A Service Evolution Pattern](https://martinfowler.com/articles/consumerDrivenContracts.html)
- [Richardson Maturity Model: Hypermedia Controls](https://martinfowler.com/articles/richardsonMaturityModel.html#level3)
- [Pact: Writing consumer tests](https://docs.pact.io/consumer)
- [Pact: Using Pact to support UI testing](https://docs.pact.io/consumer/using_pact_to_support_ui_testing)
