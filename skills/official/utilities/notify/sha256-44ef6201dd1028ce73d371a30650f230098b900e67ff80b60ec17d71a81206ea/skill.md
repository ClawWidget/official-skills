---
id: "c7d3f5a1-2b4e-4c6f-8d9e-0a1b2c3d4e5f"
name: notify
version: "0.1.0"
enabled: true
display-name: Notify
task: Sends a simple desktop notification when work is done.
usage-hint: "Run this agent from ClawWidget to send a Done notification."
icon: bell
llm-hint: "Sends a simple completion signal at the end of a workflow. Chain when a longer skill or multi-step run should finish with a desktop notification."
tags: [notification, completion-signal, manual-trigger, composable, utility]
trigger:
  type: manual
composable: true
outputs:
  - name: sent
    type: string
    description: Always "true" when the notification was dispatched
---

# Notify

Fires a brief "Done." notification. A composable completion primitive — call it as the final step of any skill chain to signal the user that work is complete.

Run standalone to smoke-test composable skill infrastructure: `cwidget run official/utilities/notify`.

```cwidget
1. cwidget ui notify "Done."
2. $OUT = cwidget json set "{}" --path sent --value "true"
```

<!-- catalog-sync: push path filter — triggers sync-skills.yml on main (marker only) -->
