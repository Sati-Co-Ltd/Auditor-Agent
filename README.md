# Auditor-Agent

SATI's Auditor Agent, built from scratch with **AI-DLC** (AI-Driven Development Life Cycle), the
open-source method from AWS Labs: <https://github.com/awslabs/aidlc-workflows>.

**Status: Ideation is done. Start at Inception.** The first intent, the **2026 NHSO DENY-list checker**,
has been through AI-DLC's Initialization and Ideation phases (rehearsal run, 22 Oct 2026). Its record is
committed in `aidlc/spaces/default/intents/261022-deny-list-checker/`. You don't redo it: install AI-DLC,
open the repo, and **resume** (section 6).

> ⚠️ **This repository is PUBLIC.** Everything you commit, including AI-DLC's records in `aidlc/`, can be
> read by anyone. **Never commit** the ICD-10-TM coding guideline, the 3-fund master data, NHSO/SSO/CGD
> internal documents, or any patient data. Keep them in `private-inputs/` (git-ignored). Use fictional
> cases only. · ⚠️ repo นี้เป็นสาธารณะ ห้าม commit ข้อมูลภายในหรือข้อมูลผู้ป่วย

---

## What Ideation decided (summary)

| | Decision | Source in the record |
|---|---|---|
| **Problem** | Medical-record coders find 2026 NHSO DENY risks only **after** NHSO rejects a claim, because the 2026 DENY list is not in the Auditor's knowledge base | `ideation/intent-capture/intent-statement.md` |
| **Users** | Coders (primary) · Medical Statistics auditors (secondary) · doctors (indirect) | intent statement, `stakeholder-map.md` |
| **Success** | 100% of sample cases give the expected result · 100% of non-UC funds return "not applicable" · every finding cites its DENY rule and source row · 0 model calls per case · DENY risks caught before submission measured after go-live | intent statement |
| **DENY data** | A **seed list** (13 rows, 10 distinct DENY codes) extracted from the ICD-10-TM coding guideline's `DENY_CODE_AI_IP_Audit` column, until SATI's official 2026 list arrives. **Internal: not in this repo.** Put it at `private-inputs/deny-list-seed-2569.csv` (ask the facilitator) | intent statement Q3, Q6, Q7 |
| **Decision-makers** | K'Pop (product owner) decides scope and priority; SATI's clinical coders own the ground truth | `stakeholder-map.md` |
| **Feasibility** | **Feasible within the time box, low technical risk**: deterministic lookup, no model, no database, no outside system | `ideation/feasibility/feasibility-assessment.md` |
| **Constraints** | 13 in total, including PDPA (fictional data only), the UC-only rule, cite every finding, Python 3.12 / FastAPI / pytest, no DB, no LLM, no pandas | `constraint-register.md` |
| **Risks** | Time box · ICD code-range edge cases (`A40-A41`, `E86,E87.7`) · official list format · thin ground truth (3 cases) · fund labels ("UC" vs "UC (NHSO)") | `raid-log.md` |
| **In scope** | Load the DENY list with source rows · check one case (fund + ICD-10-TM codes) · UC-only rule · code and range matching · tests from sample cases · minimal HTTP API | `ideation/scope-definition/scope-document.md` |
| **Out of scope** | Real patient data / HIS · UI · LLM agent · documentation-completeness rules · other funds' rulebooks · deployment (this run is local) | scope document |
| **Build order** | **Walking skeleton first**: IB-1 CR1 sepsis through load → check → API → test, then IB-2…IB-6; IB-7 batch check (Could); IB-8 Excel input and IB-9 criteria text (Won't, this time) | `intent-backlog.md` |

### Open decisions for Inception
1. **One finding or several** when a code maps to several DENY codes or rows (sepsis `A40-A41` carries **CR1 and CR37**; test case GT-001 expects only CR1).
2. Whether a DENY finding should also show the guideline's **stricter-criteria text**.
3. The **fund-name mapping** ("UC" = "UC (NHSO)").
4. Extending the **ground truth** from 3 to 15–25 fictional cases (Pod B).

### What AI-DLC caught during Ideation
No DENY list file in the project · a data source it couldn't see · CR1 vs CR37 · a miscount ("12 codes" → 10, caught by the reviewer agent) · mismatched fund labels · thin ground truth. All of this before any code.

## Contents

1. [What you will do next](#1-what-you-will-do-next)
2. [Install AI-DLC](#2-install-ai-dlc-once-per-laptop)
3. [Set up your copy of this repo](#3-set-up-your-copy-of-this-repo)
4. [Input notes (already written)](#4-input-notes-already-written)
5. [Open your AI tool](#5-open-your-ai-tool)
6. [Resume into Inception](#6-resume-into-inception)
7. [What each stage asks and produces](#7-what-each-stage-asks-and-produces)
8. [Approving or requesting changes at a gate](#8-approving-or-requesting-changes-at-a-gate)
9. [Stop at the end of Inception](#9-stop-at-the-end-of-inception)
10. [Working as a team](#10-working-as-a-team)
11. [When you are stuck](#11-when-you-are-stuck)
12. [Cheat sheet](#12-cheat-sheet)

---

## 1. What you will do next

The intent uses the **`mvp`** profile at minimal depth. Done and remaining:

```
0 Initialization      ✓ done
1 Ideation            ✓ done   Intent capture · Feasibility · Scope definition (Rough mockups skipped: no UI)
2 Inception           ▶ NEXT   Practices discovery · Requirements · User stories · (Refined mockups) ·
                               Domain design · Units of work · Contracts · Delivery plan
3 Construction                 Functional design … Code generation · Build and test
```

Inception has **started**: Practices Discovery (2.2) has written its questions (branching, testing, CI,
deployment, code style) and is waiting for answers. At the end of Inception you will have requirements,
user stories, a domain model, **units of work**, API contracts and a delivery plan.

---

## 2. Install AI-DLC (once per laptop)

**macOS / Linux** (Terminal):
```bash
curl -fsSL https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.sh | sh
```

**Windows** (a normal PowerShell window, *not* "Run as administrator"):
```powershell
irm https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.ps1 | iex
```

Open a **new** terminal and check:
```bash
aidlc version
```
✅ It prints a version (2.11.x or later). If it says *command not found*, follow the PATH line the
installer printed, or open a new terminal.

⚠️ **Kiro users:** if Kiro was open while you installed, **quit Kiro completely and reopen it**.
*Developer: Reload Window* is not enough.

**You also need:** Kiro IDE 1.1.70+ (signed in) **or** Claude Code (signed in), plus Git.
AI-DLC recommends a strong reasoning model: in Kiro choose **Claude Opus 4.8**; in Claude Code an
Opus-class model.

---

## 3. Set up your copy of this repo

```bash
git clone https://github.com/Sati-Co-Ltd/Auditor-Agent.git
cd Auditor-Agent
git checkout -b inception/<your-name-or-pod>      # e.g. inception/pod-a
```

The repo is already configured for **Kiro IDE** (`.kiro/`) and **Claude Code** (`.claude/`). Re-run the
config for your tool once on your laptop (it is safe to repeat):
```bash
aidlc config --harness kiro-ide     # Kiro IDE
aidlc config --harness claude       # Claude Code
```
Get the internal seed list from your facilitator and put it at `private-inputs/deny-list-seed-2569.csv`
(git-ignored). The record refers to it as `inputs/deny-list-seed-2569.csv`; tell AI-DLC the new location
when it asks, or copy it locally to `inputs/` and **never commit it**.
Then check the setup:
```bash
aidlc doctor
```
✅ The last line says **0 problems**. *"AIDLC hooks have not run in this project yet"* is normal
before your first chat. If it asks to turn off Kiro's **Workflows** feature, answer **yes**.

> `aidlc config` adds tool folders such as `.kiro/` or `.claude/` and an `aidlc/` workspace. Commit
> them with your work so teammates get the same setup.

---

## 4. Input notes (already written)

| File | What it is |
|---|---|
| `vision.md` | What we build, for whom, in/out of scope, open questions |
| `aidlc/spaces/default/knowledge/aidlc-shared/technical-environment.md` | Stack, prohibited items, security basics: every stage reads it |
| `inputs/sample-cases.csv` | 3 **fictional** ground-truth cases (GT-001 sepsis/CR1, GT-002 SSO, GT-003 no rule) |
| `private-inputs/deny-list-seed-2569.csv` | **Internal, git-ignored**: the DENY seed list (ask the facilitator) |

For a **new** intent later, copy `vision.md` as a template, and add fictional cases to `inputs/` and
internal data to `private-inputs/`.

---

## 5. Open your AI tool

### Kiro IDE
1. **File → Open Folder** → `Auditor-Agent`.
2. If a **Restricted Mode** banner shows: **Manage → Trust** (or *Trust Folder & Continue*).
3. **Cmd+Shift+P** / **Ctrl+Shift+P** → **Developer: Reload Window**.
4. In the chat panel's **agent picker**, choose **aidlc**. Do this in **every new chat**, because
   Kiro starts each chat on its *Default* agent.
5. Model: **Claude Opus 4.8**. Optional: switch **Autopilot** on (bottom right of the chat box). AI-DLC
   still stops at every question and gate.
6. In the chat type: `/aidlc --doctor`

### Claude Code
```bash
cd Auditor-Agent
claude
```
Choose **Yes, I trust this folder**, then type `/aidlc --doctor`.

---

## 6. Resume into Inception

In the chat (Kiro: **aidlc** agent selected; Claude Code: in the repo folder):

```text
/aidlc --resume
```

AI-DLC reads `aidlc-state.md`, sees Ideation is verified, and continues at **Practices discovery (2.2)**.
Check first if you like: `/aidlc --status`.

- It asks how to answer: **Guide me** (chat, numbered options), **I'll edit the file** (fill the
  `[Answer]:` lines, then type `done`), or **Chat**.
- Practices discovery questions are already waiting in
  `aidlc/spaces/default/intents/261022-deny-list-checker/inception/practices-discovery/practices-discovery-questions.md`
  (branching, how to prove the thin slice works, testing/coverage/CI, deployment, code style).
- Each stage ends with **confirm the summary** → documents written → reviewer findings → **approval gate**.
- **Requirements Analysis (2.3)** is where the open decisions above get settled. Bring K'Pop and a coder.

> **Starting a different intent from scratch instead?** Use
> `/aidlc mvp --depth minimal Read ./<your-vision>.md and build what it describes. This is a new project.`
> It runs Initialization and Ideation first.

---

## 7. What each stage asks and produces

| # | Stage | It asks about | It writes (under the record folder) |
|---|---|---|---|
| 1.1 | Intent capture | problem, users, success, decision-makers, what's out | `ideation/intent-capture/intent-statement.md`, `stakeholder-map.md` |
| 1.3 | Feasibility | time box, outside inputs, data, compliance | `feasibility-assessment.md`, `constraint-register.md`, `raid-log.md` |
| 1.4 | Scope definition | must / could / won't, build order | `scope-document.md`, `intent-backlog.md` |
| 1.6 | Rough mockups | screens (skipped automatically if there is no UI) | mockups |
| 2.2 | Practices discovery | branching, testing, CI, deployment, code style | team practices |
| 2.3 | Requirements analysis | functional + non-functional requirements | `requirements.md` |
| 2.4 | User stories | personas, stories, acceptance criteria | user stories |
| 2.6 | Domain design | the domain model (entities, rules) | domain model |
| 2.7 | Units generation | how to split the work into buildable units | units + dependency graph |
| 2.8 | Contract design | APIs between units | contracts |
| 2.9 | Delivery planning | order of units, walking skeleton first | delivery plan |

Exact stage lists and file names can vary by AI-DLC version; `/aidlc --status` always shows where you are.

---

## 8. Approving or requesting changes at a gate

At every gate AI-DLC shows a summary, the reviewer's findings (each with an ID such as `R-01`), and:

- **Approve**: move on. Open findings are recorded as *accepted risks*; that is normal.
- **Request Changes**: say exactly what to change, one line per finding. AI-DLC revises, re-reviews,
  and shows each finding as *Resolved* under the same ID.

A good Request Changes is specific:
```text
Request Changes. R-01: the seed list has 10 distinct DENY codes, not 12; correct it. R-02: a case passes when the expected code is contained in the findings.
```

**Who approves:** the person who owns the decision. For clinical or coding correctness that is a domain
expert (coder or clinician), not the person typing.

---

## 9. Stop at the end of Inception

After **Delivery planning (2.9)** is approved, AI-DLC moves into **Construction** (Functional design,
then Code generation). Stop there for today:

- Don't answer the first Construction question. Just close the chat; nothing is lost.
- Check where you are: `/aidlc --status`. You should see Initialization and Ideation *Verified*,
  Inception *Verified* or complete, and Construction pending.
- **Commit your record** (it holds no internal data if you followed section 4.3):
  ```bash
  git status                      # confirm nothing from private-inputs/ is listed
  git add -A
  git commit -m "Inception: <intent name>"
  git push -u origin inception/<your-name-or-pod>
  ```
- Next time, open a new chat (Kiro: choose the **aidlc** agent) and type `/aidlc --resume` to continue
  into Construction.

---

## 10. Working as a team

- **One run, one screen (recommended for Inception).** One **Driver** types; a **domain expert**
  approves the gates; a **Navigator** reads the documents aloud; a **Scribe** notes decisions and open
  questions. AWS calls this *Mob Elaboration*.
- **One run per branch.** Each pod uses its own branch (`inception/pod-a`, `inception/pod-c`, …), so
  records don't collide.
- **Practice alone** afterwards with a small throwaway intent:
  `/aidlc express <a small change>` in a scratch folder.

---

## 11. When you are stuck

| What you see | What to do |
|---|---|
| It stopped and won't continue | It is waiting for you: answer the question or the gate |
| Too many questions | `/aidlc --depth minimal` |
| Where am I? | `/aidlc --status` |
| Chat is long, slow or confused | New chat (Kiro: choose **aidlc** again) → `/aidlc --resume` |
| Kiro: your reply isn't seen, or commands show exit code -1 | **Manage → Trust**, then **Developer: Reload Window**, then type `carry on` |
| Kiro: "Approval needed in agent-aidlc…" panels | `aidlc config trust --kiro-workflows off --yes`, then Reload Window |
| Claude Code: approvals refused | Restart `claude` in the folder and choose *Yes, I trust this folder* |
| AI goes off scope | `Stop. That is out of scope. Add it to a "next steps" list and continue with the approved requirements only.` |
| Same error twice | Stop the agent (stop button / Esc), run `aidlc doctor --export` **in your own terminal**, and share the report |

More help: [AI-DLC troubleshooting](https://github.com/awslabs/aidlc-workflows/blob/main/docs/guide/15-troubleshooting.md) ·
[onboarding guide](https://github.com/awslabs/aidlc-workflows/blob/main/docs/guide/onboarding.md).

---

## 12. Cheat sheet

| I want to… | Type | Where |
|---|---|---|
| Install AI-DLC (Mac) | `curl -fsSL https://github.com/awslabs/aidlc-workflows/releases/latest/download/install.sh \| sh` | Terminal |
| Set up the repo | `aidlc config --harness kiro-ide` (or `claude`) then `aidlc doctor` | Terminal |
| Start Ideation + Inception | `/aidlc mvp --depth minimal Read ./vision.md and build what it describes. This is a new project.` | Chat |
| Where am I? | `/aidlc --status` | Chat |
| Fewer questions | `/aidlc --depth minimal` | Chat |
| Continue later | `/aidlc --resume` | Chat |
| Check the setup | `/aidlc --doctor` (chat) · `aidlc doctor --verbose` (terminal) | |
| Diagnostic report | `aidlc doctor --export` | **Your own** terminal |

Codex CLI users type `$aidlc` instead of `/aidlc`.
