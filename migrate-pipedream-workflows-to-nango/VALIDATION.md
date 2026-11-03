# Validation: November 1, 2026

Author and test owner: Emmanuel Oyibo. This is a purpose-built tutorial demonstration with connected GitHub and Slack accounts, not a migration of an existing customer's workflow.

## Local verification

- Node 24.10.0; pinned `nango` and `@nangohq/node` 0.71.14; Zod 4.6.7.
- The core's 24 tests passed before live execution: HMAC/schema validation, customer isolation, concurrent delivery/database workers, atomic rollback, duplicate/conflicting deliveries, stale/equal timestamps, leases, rate limits, auth rejection, and ambiguous writes. The pinned backend SDK was exercised against a local mock HTTP server.
- Three additional checkpoint-import tests cover no resend, same-ID replay suppression, route/destination validation, duplicate-import rejection, and transactional rollback. These test the reusable importer added after the live rehearsal. The rehearsal used an equivalent maintainer script with explicitly verified source values.
- Compile passed for the owned `migration-send-message` action before live deployment. The prepared source package uses generic integration ID `slack`; the live environment used its existing Slack integration ID. Final clean-checkout results are recorded below.
- Synthetic demo and failure tests make no provider requests. Rate-limit, expired-auth, unknown-route, and out-of-order cases were simulated locally; do not describe them as authenticated failure tests.

## Authenticated observations

Test repository: [Emminex23/blog-demos issue #1](https://github.com/Emminex23/blog-demos/issues/1). This fork was used for disposable issue events; the intended source destination is NangoHQ/blog-demos.

The GitHub App had Issues read-only and Metadata read-only, installed only on this repository. Nango attributed its events to the selected connection; provider event/delivery headers and the Nango HMAC reached the receiver. The owned Slack action was deployed to dev without changing other functions. The connected bot was a member of the approved test channel.

| Phase | Event | Slack receipt |
| --- | --- | --- |
| Initial Nango run | opened | `1793532707.697889` |
| Initial Nango run | closed | `1793533045.530499` |
| Initial Nango run | reopened | `1793533068.088259` |
| Pipedream baseline | closed | `1793535165.938499` |
| Cutover to Nango | reopened | `1793535482.370509` |
| Rollback to Pipedream | closed | `1793535564.772789` |
| Restore Nango | reopened | `1793535693.784099` |

GitHub redelivery of the initial opened event retained GUID `9adfe1ac-d5e7-11f1-8761-3911c22de4ee`, returned HTTP 200, and created no additional outbox entry or Slack notification.

The first Nango worker attempt lacked network access and became uncertain. A read-only probe failed with ENOTFOUND, Nango had no action execution, and Slack history had no matching message. Only after that reconciliation was the single job reset and successfully delivered. Its two attempts are visible in the dashboard screenshot. This was manual recovery, not automatic retry of an ambiguous write.

The Pipedream baseline used a signed repository HTTP webhook, a Data Store, and a dedicated Slack bot with only `chat:write`. No broad GitHub or Slack OAuth grant was used. The deployed v37 workflow completed the baseline closed event in 4,231 ms and the rollback closed event in 5,646 ms. These are individual observations, not performance benchmarks.

Before each writer switch, the previous trigger/forwarding was paused and its work was drained. Confirmed source issue snapshots and receipts were imported without sending notifications. The next live change then ran on the selected writer. Independent Slack history checks found **seven real-event notifications for seven real issue changes**. This controlled test covered one customer/repository, not concurrent production cutover or automatic backfill.

The final application database has seven sent records: five Nango sends and two imported Pipedream receipts. Imported receipts have zero target attempts and `imported_pipedream_receipt` provenance. One non-issue setup event remains quarantined. It produced no notification.

## Setup duplicate and export limitation

The prebuilt Pipedream Bot for Slack Send Message action v0.0.5 reported missing read scopes, but Slack history showed a message had already arrived (`1793534806.488169`). Testing the replacement custom step sent a second labeled setup message (`1793534896.480799`). Both say “Synthetic setup event - not live evidence.” They are retained and excluded from the seven real-event results. A failed action can follow a successful external write; reconcile before replaying.

The free Pipedream workspace did not include File Store, so the official project-export archive was not generated. The author explicitly chose to document this limitation and provide the tested source files. `pipedream/` is source code plus setup instructions, not an official export or a Nango importer.

## Screenshots and cleanup

The author captured the genuine white Nango success dialog, light support dashboard, Slack notification, limited baseline-bot authorization, successful Pipedream workflow, cutover/rollback Slack sequence, and GitHub connection Settings page. The seventh capture documents the saved per-connection webhook override after the temporary tunnel was stopped; it is not evidence of a running receiver.

All seven original screenshots are retained unchanged. Separate article copies remove automation cursors and their shadows over blank backgrounds and outline the webhook override in red. At Emmanuel's request, his personal account details, including his name and email, remain visible; earlier redactions were restored from the original pixels. Every image retains its original pixel dimensions, without cropping or resizing. No success state, provider result, message, or configuration value was changed. Source/output hashes and edit rectangles are recorded in the article's `screenshot-provenance.json`; pixel comparisons verified that edits stay within those rectangles.

The architecture diagram is supplied as rendered PNG plus editable SVG and Mermaid sources. The article was checked in a local browser preview and converted to the CMS's Portable Text format offline, without uploading assets or creating a CMS draft. The final body is 2,520 words including code, excluding metadata and editorial comments.

After live testing, the repository's Pipedream webhook was left inactive and forwarding on the dedicated Nango GitHub integration was paused. The temporary public tunnel and signing-key copy helper were stopped. Credentials, raw local databases, operational scripts, and private keys are excluded from this repository.

## Final clean-checkout verification

- `npm ci --ignore-scripts --no-audit --no-fund`: passed with the supplied lockfile.
- `npm test`: 27 passed, zero failed.
- `NANGO_CLI_UPGRADE_MODE=ignore npm run compile`: type-check, build, and generated artifacts passed with generic integration ID `slack`.
- `npm run demo`: passed; zero real provider requests, with duplicate suppression, quarantine, and uncertain-write outcomes in the synthetic report.

The final revision changed documentation and image assets only; the implementation covered by the 27 passing tests is unchanged. The pinned CLI's `pull --help` confirms the documented arguments and catalog flag. The optional coding-agent skill installation and catalog pull were documented, not executed during this revision.

At the close of the November 1 validation, the package was local pending article review; no source commit, push, PR, or CMS upload had been made. The article should link to NangoHQ/blog-demos only after the reviewed source is available there. No production deployment or exactly-once external-write guarantee is implied.


## Official-documentation audit, November 1, 2026

Rechecked current official Nango, Pipedream, GitHub, and Slack documentation and Nango’s public catalog/routing source. Clarified latest-deployed export semantics, bounded forwarding retries, Slack-only automatic rate-limit recovery, and CLI dependency preservation in the article/README. Corrected the export-exclusions link fragment. The pinned CLI confirmed `--no-dependency-update`; compilation passed again with upgrades disabled, and package/lockfile bytes are unchanged. No integration implementation changed, deployment ran, or authenticated provider test was repeated. The existing 27-test and live-run results above retain their stated scope. The article’s 25 distinct public source pages return HTTP 200; all article fragments/local assets resolve, and offline Portable Text conversion passed. The complete evidence table is in the article research folder’s `CON-337-technical-audit-2026-11-01.md`.


## Pre-push verification, November 3, 2026

Reinstalled the locked dependencies and reran all 27 tests, the pinned action compilation, and the synthetic demo successfully. The demo made zero real provider requests. The publication copy contains 42 source, documentation, fixture, and diagram files, including `.env.example` and the required `.nango/` configuration. Local credentials, customer routes, databases, build output, and dependencies are excluded. The copied files were verified against the reviewed local package; integration code is unchanged. No new authenticated run or deployment was performed.
