---
id: "c7d3f5a1-2b4e-4c6f-8d9e-0a1b2c3d4e5f"
name: notify
version: "0.1.0"
enabled: true
display-name: Notify
task: Fires a "Done." desktop notification. Composable completion primitive for skill chains.
icon: bell
llm-hint: "Composable completion primitive. Invoke via `cwidget skill call official/utilities/notify` (no input required). Returns {sent: true}. No permissions, no sidecars, no network. Use as the terminal step of any chain to signal completion to the user. Pairs with official/productivity/clipboard-summary as its JIT-resolved dependency."
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
