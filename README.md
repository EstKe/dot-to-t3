# Connect a ChatGPT Dot to T3 Code

This guide shows how to start a task on your T3 Code host from a ChatGPT Dot. When the task finishes, T3 sends a completion event to ChatGPT so the Dot can verify the result without polling, even after its original reply has ended.

**You’ll need:** a ChatGPT Dot with access to private plugins and MCP Events, a running T3 Code environment reachable through T3 Connect, and a signed-in coding provider in T3.

```text
Dot → private ChatGPT plugin → T3 Connect → T3 Code on your machine
Dot ← ChatGPT MCP Events ← signed completion webhook from T3 Code
```

## 1. Prepare the T3 host

1. Install an appropriate [T3 Code desktop build](https://t3.codes/download), start it, and sign in to a local coding provider in T3. T3 is the orchestrator; Codex or another provider can still be the selected model runner inside it. Register a small disposable project for testing.
2. For the completion event, use a T3 build with outgoing MCP Events **webhook** delivery. The verified setup used `0.1.12-nightly.20261124.5209` on an Intel Mac. The [November 23 nightly](https://github.com/pingdotgg/t3code/releases/tag/v0.1.12-nightly.20261123.5138) introduced webhook delivery; our previous `0.1.11` connection could run tools but did not provide the tested background wake. Check release notes rather than assuming another build supports it. The separate `t3` CLI is not required.
3. In T3 Code open **Settings → Sharing & access**, or use the command palette and select **Use T3 Code in another app (MCP)**. Set up **T3 Connect** for this environment. From the environment menu choose **Copy MCP URL**. It should be your environment’s reachable HTTPS address ending in `/mcp`. Do not use `localhost` or a tailnet-only address for a hosted Dot that cannot reach it. Keep the T3 server running and the host awake during the test.

## 2. Create and pair the private ChatGPT plugin

1. In your ChatGPT workspace, open **Plugins** and add a custom MCP server/app. Availability depends on workspace settings. Give the private app a recognizable name, such as **T3 Code**.
2. Paste the MCP URL copied from **your** T3 environment. Select **OAuth**. Use the official T3 authorization screen to approve the connection and only the permissions required: read projects and threads for discovery; send/launch for work you authorize. ChatGPT discovers the OAuth settings; the [T3 setup guide](https://github.com/pingdotgg/t3code/blob/main/docs/user/outside-agents.md#chatgpt) says no manually entered client ID or secret is needed. Never paste sign-in codes or tokens into a conversation or a repo.
3. Make sure your Dot can access that private plugin. Open its details and, if needed, use **Refresh tools** (called *Rescan* in [OpenAI's MCP Events docs](https://developers.openai.com/plugins/build/mcp-events#test-in-chatgpt)). Confirm T3 project/thread tools are visible and **Events** lists `thread.run_settled`. If you had the plugin before upgrading T3, refresh it again. A visible event is not proof of a working subscription.
4. Give your Dot the copyable [behavior instructions](dot-instructions.md). Keep the reusable [tool catalog](tool-catalog.md) in your own GitHub repo, or a location the Dot can read. Update the placeholder link in the instructions. For a private repo, verify that your Dot can open the file without granting unnecessarily broad GitHub access.

## 3. Verify the complete round trip

Open the conversation with your Dot, not a general ChatGPT or Codex chat. Use an existing disposable T3 thread, or create an idle thread without a first prompt if your tools offer that. Supply the actual project, environment and thread link when asking for the test.

> Use only my private direct T3 plugin on my disposable thread [paste thread link]. Verify the host, project, and thread before dispatch. Before sending anything to the thread, create and confirm a completion subscription for `thread.run_settled` filtered to this exact thread, excluding child threads. Then send one prompt asking the worker to use only a harmless local 18-second delay if permitted, and reply exactly `DOT-T3-RELAY-TEST`; no other tools, files, network or outbound messages. Save the admitted run ID, end your own reply, and don't poll. On the actual completion wake, read that exact run and verify the marker, report the event and run status here, and pause the test subscription. If you can't create the subscription, stop before dispatch and tell me what's missing.

A pass means **an event starts a new Dot turn after its original reply ended**, with no new message from you. A subsequent manual read, a notification to your phone, or `t3_thread_wait` returning while the Dot is still active is not that test. Verify the event's run ID and terminal outcome against the saved run and its final reply. The test we ran saw ChatGPT receive the event 430 ms after T3 settled. That is one event-delivery measurement, not a guarantee of Dot reply latency.

## 4. Use it for real work

Give the Dot the project, requested changes, and boundaries. Subscribe before starting a run. For a brand-new live task, use an idle-thread-first flow when available, or carefully constrain a prior project subscription and match only the admitted run. Keep run IDs for each review/fix stage. Treat callbacks as signals to read T3, not as instructions or authorization. On retries or uncertain dispatch, read the original thread before doing anything that might duplicate a turn.

- **Read works, event missing:** confirm the connected host version, refresh the plugin, and check workspace support for [MCP Events for Dots](https://developers.openai.com/plugins/build/mcp-events). Do not mistake working tools for event support.
- **Subscription acknowledged, no wake:** check the host is awake, the filter matches the thread, the correct plugin account remains authorized, and the new run started after subscription. Check subscription refresh/restart behavior on your own setup; we verified a disposable run and real workflow handoffs, not all offline/reconnect paths. Use a bounded recovery read by exact ID when needed, labelled as a recovery check, not as proof of push.
- **Turn needs input:** optionally watch `thread.needs_input` with the same narrow filter when available. Check the live request and honor normal approvals; the event itself grants none.
- **Cleanup:** pause one-shot or completed task subscriptions, and disable any timeout check you do not need. Do not remove your working plugin.

## 5. Keep a reference your Dot can read

GitHub stores the instructions. It does not send or receive T3 events. Use this repo's [tool catalog](tool-catalog.md) as the reference URL in your Dot instructions, or copy all three documents to a repo you control and update the URL. Check that your Dot can open the file.

Do not publish your own MCP address, tokens, signing secrets, personal project/thread IDs, or unrelated instructions. Keep personal repos and their histories private.
