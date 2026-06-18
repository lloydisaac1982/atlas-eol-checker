# 🤖 Atlas — EOL Device Checker Agent

Atlas is an AI agent (a [Claude Code](https://claude.com/claude-code) subagent) that turns a raw
device/CMDB inventory export into an actionable **End-of-Life (EOL) / End-of-Support (EOS) risk
assessment** — across **two independent dimensions**:

- **Hardware lifecycle** — End of Sale, End of SW Maintenance, End of Security Support, Last Day of Support (LDoS)
- **OS / software lifecycle** — Windows Server, RHEL/Ubuntu/SLES, VMware ESXi, Cisco IOS/NX-OS, Junos, NetApp ONTAP, Brocade FOS, and more (including ESU / extended-support windows)

A device with in-support hardware but an EOL operating system is still a critical risk, so Atlas
scores **Hardware**, **OS**, and a **Combined** risk (worst-of-the-two) for every device.

## What it produces

1. A structured **EOL Infrastructure Assessment Report** (executive summary, per-category inventory,
   OS consolidation view, critical/high-risk detail, full recommendations, strategic notes).
2. An optional single-file, self-contained **interactive HTML dashboard** (Chart.js + Font Awesome
   via CDN) — KPI cards, combined-risk donut, hardware-vs-OS comparison, searchable/filterable
   device inventory, OS-platform breakdown, expandable per-device cards, and Export-to-PDF.

## Repository contents

```
.claude/agents/Atlas/
  atlas.md                       # the agent definition + system prompt
  atlas-dashboard-template.html  # canonical dashboard template (logo embedded inline)
  atlas-logo.svg                 # standalone Atlas logo (design reference)
.claude/commands/
  atlas.md                       # /atlas slash command (interactive onboarding)
```

The dashboard template is fully self-contained — the Atlas logo is embedded inline as SVG, so no
external image file is required. The only runtime dependencies are Chart.js and Font Awesome from
CDN, and the page degrades gracefully offline (charts simply don't render).

## Install / use

This is a Claude Code subagent with a matching slash command. To use it:

1. Copy the `.claude/` folder into your Claude Code workspace (or clone this repo and work
   from inside it). This brings both the `agents/Atlas/` definition and the `commands/atlas.md`
   slash command.
2. Atlas registers automatically. Start it with the **`/atlas`** slash command, by asking
   *"Use the atlas agent to assess this inventory,"* or let Claude Code auto-select it when your
   request matches an EOL assessment.
3. Atlas runs a short onboarding — it asks for the **client/organization name**, then the **device
   inventory file** (`.csv`, `.xlsx`/`.xls`, `.txt`, or `.json`). OS/version fields are especially
   valuable because they drive the software-layer risk independently of hardware.
4. After the text report, Atlas **always** generates the interactive HTML dashboard automatically.

## Running without approval prompts (optional)

Atlas writes the dashboard + KB and runs Python to parse spreadsheets, so by default Claude Code
asks permission for those steps. To let it run end-to-end without prompts, copy the
`permissions.allow` block from [`plugins/atlas-eol-checker/settings.example.json`](plugins/atlas-eol-checker/settings.example.json)
into **your own** `.claude/settings.json` (project) or `~/.claude/settings.json` (global).

> ⚠️ This is **opt-in and per-user** — Claude Code never auto-applies it on install, because
> pre-authorizing tools is a trust decision only you should make. The example is deliberately
> broad (it auto-approves all writes/edits and any `python`/`cmd`/`powershell` command for the
> whole session, not just Atlas). **Trim it to the narrowest scope you're comfortable with.**

## Risk model

| Risk | Hardware criteria | OS criteria |
|---|---|---|
| 🔴 Critical | Already past EoS / LDoS | OS EOL, no ESU available |
| 🟠 High | EoS within 12 months | OS in paid ESU / extended support only |
| 🟡 Medium | EoS within 13–24 months | Mainstream ended, extended support active |
| 🟢 Low | EoS beyond 24 months | Fully supported, current release |
| ⚪ Unknown | EOL data unavailable | OS version not provided / unrecognized |

**Combined risk = the worst of the two dimensions.**

## Behavioral guarantees

- Never fabricates EOL dates — verifies uncertain dates against vendor lifecycle pages /
  [endoflife.date](https://endoflife.date), and clearly flags anything it can't confirm.
- Always provides a recommendation, even for unknown devices.
- Handles partial data gracefully and lists assumptions in a Data Quality section.

---

*Built as part of an internal agent suite (alongside Orion and Isilon-Perf). Atlas shares the same
Unisys/Flowserve dashboard house style.*
