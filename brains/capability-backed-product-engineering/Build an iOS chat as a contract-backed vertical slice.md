---
type: permanent
status: developing
tags:
  - ios-development
  - vertical-slice
  - contract-testing
  - chat
created: 2026-08-30
---

Build one thin conversation path through the iOS app and backend before expanding either side. For chat, the first slice is small: enter text, send it, observe a real request, and display a real response. A fake model response is fine at first if the request crosses the actual client-server boundary.

This walking path reveals authentication, serialization, deployment, latency, and state-management problems that a complete mock interface hides. It applies [[@outcome-delivery/From function to flow|end-to-end flow]] to one product capability instead of optimizing the interface and backend separately.

## Start with one interaction

Write the user outcome first:

> When I send valid text, my message appears immediately, an assistant response begins, and the conversation eventually reaches a completed or recoverable failure state.

Then make the interaction concrete:

1. Define the request, response, error body, and authentication in OpenAPI.
2. Define streamed event shapes too if streaming is part of the first slice. OpenAPI 3.2 describes sequential streaming and server-sent events; AsyncAPI is another option when the conversation uses message channels.
3. Write a state table for the app. At minimum, cover `ready`, `sending`, `streaming`, `completed`, `failed`, and `cancelled` if cancellation exists.
4. Generate or hand-write a narrow Swift client boundary from that contract.
5. Implement a deterministic fake behind the same boundary.
6. Build the SwiftUI feature against the boundary, not against fixture files or `URLSession` directly.
7. Verify the real provider against the contract, then run one end-to-end test through the deployed test service.

Do not begin with conversation history, attachments, tools, regeneration, rich Markdown, and offline persistence. Add the next capability only after the current slice works across the whole path.

## Keep the app boundary narrow

The feature needs an operation-shaped dependency, conceptually:

```swift
protocol ChatClient {
    func sendMessage(
        conversationID: Conversation.ID,
        text: String,
        clientMessageID: UUID
    ) -> AsyncThrowingStream<ChatEvent, Error>
}
```

Both `LiveChatClient` and `FakeChatClient` implement it. The exact Swift shape depends on the chosen transport, but the view should not know whether events came from a fixture, server-sent events, or a WebSocket.

The fake is not a parallel design. Its event types, error cases, and timing scenarios come from [[Executable capability contracts keep interface design honest|the capability contract]].

## Build scenario fixtures, not one happy mock

Give previews and feature tests named scenarios:

| Scenario | What it tests in the interface |
| --- | --- |
| Empty conversation | Initial guidance and focus |
| Sending | Optimistic message and disabled duplicate submit |
| Slow first event | Waiting feedback without false progress |
| Streaming | Incremental text and scrolling behavior |
| Completed | Final actions and accessibility announcement |
| Offline | Recovery without losing the draft |
| Unauthorized | Reauthentication path |
| Rate limited | Clear wait or retry guidance |
| Stream interrupted | Partial content and retry behavior |
| Cancelled | Stable transcript after Stop |

Apple recommends passing views only the data they need and using separate sample data in previews. Follow that separation here. A view preview receives a display state; a feature preview receives a fake client scenario. Neither should perform an unplanned network call.

## Turn a desired mock into backend instructions

When design needs a capability that is not built yet, use this sequence:

1. Mark the interaction `speculative` in the control-to-capability table.
2. Prototype only enough to answer the UX question.
3. If the design survives, define the operation and state transition in the contract.
4. Add representative success and failure examples.
5. Run consumer tests against the real client and a contract mock server. Generate the in-app fake scenarios from the same examples.
6. Create provider work whose acceptance condition is passing the same contract.
7. Change the label to `contract-backed`; change it to `real` only after integration passes.

The backend task is therefore not "make the mock work." It is a testable obligation such as:

> Given an authenticated conversation that accepts messages, when the client submits a unique message ID and non-empty text, the provider acknowledges the user message once and emits an ordered sequence ending in `completed` or a documented terminal error.

That wording gives backend implementation freedom while preserving what the interface relies on.

## Give AI agents hard boundaries

Include these artifacts in design and coding prompts:

- The current OpenAPI or AsyncAPI file.
- The UI state table and allowed transitions.
- Named fixtures for success, delay, partial response, and failures.
- The control-to-capability table with `real`, `contract-backed`, and `speculative` labels.
- The rule that a new control needs an existing capability or a proposed contract change.

Ask an agent to report unsupported assumptions separately. Do not let it silently create endpoints, response fields, or retry behavior. The contract and tests, rather than the prompt alone, keep generated code aligned with the system.

## Definition of done for a slice

A chat slice is done when:

- The control maps to a named capability.
- The iOS client and provider agree on requests, events, and terminal errors.
- Previews cover the meaningful visual states.
- Consumer tests exercise the real client implementation.
- Provider verification passes.
- One automated or repeatable test crosses the deployed boundary.
- The interface remains usable under the agreed delay and failure scenarios.

Only then widen the slice.

## Sources

- [Apple: Adding previews to your interface files](https://developer.apple.com/documentation/xcode/adding-previews-to-your-interface-files)
- [OpenAPI Specification 3.2.0](https://spec.openapis.org/oas/latest.html)
- [AsyncAPI document structure](https://www.asyncapi.com/docs/concepts/asyncapi-document/structure)
- [Pact: Writing consumer tests](https://docs.pact.io/consumer)
- [Pact: Using Pact to support UI testing](https://docs.pact.io/consumer/using_pact_to_support_ui_testing)
