# Collaboration workflow setup

Workflow: .github/workflows/collaboration-handoff.yml

## Events
- Push changing .collaboration/inbox/chatgpt-to-glm.md emits task_ready to GLM relay if configured.
- Push changing .collaboration/inbox/glm-to-chatgpt.md, sglang-video/g2.1/review/**, or G2.1-review-report.md creates/reuses a GitHub Issue tied to the commit and emits review_requested to ChatGPT relay if configured.
- If relay secrets are absent, the workflow logs that no Agent was awakened. The Issue remains the durable review inbox.

## Repository Actions Secrets
- GLM_TASK_WEBHOOK_URL: HTTPS endpoint dispatching task_ready to GLM/ZCode runner.
- CHATGPT_REVIEW_WEBHOOK_URL: HTTPS endpoint starting an external review-capable Agent/relay. This does not directly invoke the current ChatGPT conversation.
- COLLABORATION_WEBHOOK_SECRET: optional shared secret sent as x-collaboration-secret.

Payload fields: event_id, event_type, repository, ref, commit_sha, task_id, changed_files, delivery_url.

## Security
Use HTTPS. Store URLs and credentials only in Actions Secrets. Receiver must validate secret, repository allowlist, event ID/idempotency and replay window. Never execute arbitrary shell commands from repository text. Restrict GLM runner to the intended repository/task directory. Do not configure an endpoint until its owner and authentication behavior are verified.

## Limitation
GitHub Actions can send an HTTP request on Push. The current ChatGPT GitHub connection is not a persistent webhook listener and cannot be directly awakened by this workflow. A separate ChatGPT-side relay or Agent service must be deployed and configured.
