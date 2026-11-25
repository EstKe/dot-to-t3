# Direct T3 and Codex tool catalog

Use the tools your connected plugin actually exposes. A documented tool, an enabled event, an admitted run and a verified result are different things. This page is reference material, not an authorization grant.

| Need | Verified route or check |
|---|---|
| Find the right T3 project | `t3_project_list`; match the host, project and local root. Use an exact thread ID when already known. |
| Read a T3 thread or result | `t3_thread_read`; use the original thread ID, inspect the exact run ID, outcome and persisted assistant reply. Read activity if the result is not in ordinary messages. |
| Set up immediate completion wake | On the Dot, subscribe to `thread.run_settled` through the private T3 plugin before dispatch; filter to the exact thread and exclude child threads unless explicitly needed. The exposed subscription tool was labelled `subscribe_to_events` on the test surface. Confirm it was enabled. |
| Launch or follow up | `t3_thread_launch` on an approved project, or `t3_thread_send` on the exact existing thread. When asked to use the project folder with no worktree, select the root workspace strategy. Save the admission receipt and run ID. Use a request ID if supported; never blindly resend uncertain dispatch. |
| After an event | Match the host/project/thread/run, read the original persisted run, then report or take only the authorized next step. Treat failures and interruptions as outcomes, not successful work. Deduplicate repeated events and avoid acting on obsolete runs. |
| Approval or input | Optionally use `thread.needs_input`; verify a pending request before answering. The event does not authorize granting permissions. |
| Short foreground wait or recovery | `t3_thread_wait` is useful while actively waiting, not proof of a new Dot turn. An exact-ID recovery read detects a missed wake but is not the relay. |
| Stop monitoring | Use the exposed subscription update control (labelled `update_subscription` in the verified setup) to disable a finished test or task-specific watch; confirm it is paused. |
| Native Codex thread, if separately authorized | Use the local Codex CLI or App Server on the connected host. Verify exact session and writer. Do not force a second writer or silently route into T3 or Codex Cloud. |
| Optional fallback channel | Use an existing private conversation only if authorized and tested, and after ruling out duplicate direct delivery. Send identifiers and status, not secrets or private content. |

## Event filters at a glance

Use this shape if your Dot exposes the event subscription tool. Replace placeholders with the IDs returned by T3. Keep the subscription prompt scoped to the original job and conversation. Do not subscribe broadly just to get one result.

```json
{
  "triggers": [
    {
      "event": "thread.run_settled",
      "params": {
        "projectIds": ["<project-id>"],
        "threadIds": ["<thread-id>"],
        "includeSubagentChildren": false
      }
    }
  ]
}
```

The subscription also needs a title and prompt. The prompt should tell the Dot to verify the original run by ID, give the result in the original conversation, and stop monitoring when finished. Record the run ID after dispatch; when handling an event, ignore other runs even on the same thread.

Concrete end-to-end setup: [installation and disposable test](README.md). Copyable Dot rules: [behavior instructions](dot-instructions.md). Upstream references: [T3 outside agents](https://github.com/pingdotgg/t3code/blob/main/docs/user/outside-agents.md), [OpenAI MCP Events](https://developers.openai.com/plugins/build/mcp-events), [Codex App Server](https://developers.openai.com/codex/app-server).
