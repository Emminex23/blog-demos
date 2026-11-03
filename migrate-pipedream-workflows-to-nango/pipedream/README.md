# Pipedream source baseline

These are the source files for the purpose-built tutorial workflow tested on November 1, 2026. They are **not an official export or an importable workflow JSON**. The test workspace's free plan did not include File Store, required by Pipedream's [official export process](https://pipedream.com/docs/workflows/export-workflows). The article explicitly documents this limitation.

## Build the workflow

1. Create an HTTP-triggered workflow. Select **Raw request** event data and **Return a custom response from your workflow**. Raw bytes are required for GitHub HMAC verification.
2. Add a Node.js step named `verify_issue`, using `verify-issue.mjs`. Refresh fields and configure the allowed numeric repository ID and a dedicated signing secret. The sample customer ID is `demo_customer_support`; replace it consistently for your test.
3. Add `update_support_record` from its matching file. Select a new **Migration demo support records** Data Store and set the approved Slack channel ID. The published source makes this a field; the live run used the same ID directly in code.
4. Create a dedicated Slack app using `slack-app-manifest.json`, install it in your test workspace, and add it only to the test channel. Its sole bot scope is `chat:write`. Save its Bot User OAuth Token through Pipedream's **Bot for Slack** connector.
5. Add `send_message` from `send-message.mjs`; refresh fields and select that connection. The step sends one `chat.postMessage` request and returns only `ok`, `channel`, and `ts`.
6. Add `record_receipt` from its matching file, selecting the same Data Store. It verifies and records the Slack receipt before returning HTTP 200.
7. Test the steps with a uniquely identified, signed setup payload, then deploy. In the GitHub test repository's Webhooks settings, add the Pipedream HTTP URL, select JSON, enter the same signing secret, keep SSL verification enabled, and subscribe only to **Issues**. This is a repository webhook, not a GitHub OAuth connection.
8. Close/reopen a real test issue and inspect the execution, support record, and Slack receipt. Pause Nango forwarding during baseline writes. Do not run both notification paths for the same customer.

## What the baseline stores

- `issue:<customer>:<repository>:<issue>`: the latest issue snapshot.
- `delivery:<GitHub delivery GUID>`: pending processing, then a confirmed Slack receipt.

This is a sequential demonstration. Pipedream [Data Stores are not atomic or transactional](https://pipedream.com/docs/workflows/data-management/data-stores). A delivery record alone is not a production outbox or concurrency guarantee. Pending deliveries need manual reconciliation after failure. Testing a step independently can repeat its external side effects, bypassing earlier deduplication steps.

The prebuilt Bot for Slack Send Message v0.0.5 required conversation-read scopes. During setup it reported a scope error after a message had already arrived. Re-testing with the custom step produced a second labeled setup message. Those two messages are documented in VALIDATION.md and excluded from real-event counts. Never interpret a failed step as proof that its external write did not happen.

Keep automated workflow retries disabled for this demo. A timeout or failed receipt step may follow a successful message write. Check Slack history before replaying or resetting a pending delivery. No Slack token is migrated to Nango; accounts are connected separately.
