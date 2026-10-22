# AGENTS.md — EventOps AI | غرفة العمليات الذكية للهاكاثون

Instructions for any AI coding agent working on this project. Read this file fully before every task.

---

## 1. Project in one paragraph

EventOps AI is an Arabic, right-to-left web app that the Protocol Committee uses to run guest reception at the 7th Health Innovation Hackathon. It covers 4 tracks, 40 guests and 42 staff, and has 6 pages: Guest Portal, Reception Operations, Live Operations Dashboard, Support Operations, Task Management and Reports. This version is a prototype: demo data only, one device, a role switcher instead of login, and a template-based daily report with no AI service.

## 2. Source of truth

| File | What it is | How to use it |
| --- | --- | --- |
| `SPEC.md` | Technical specification. **The primary reference** | Build exactly what it describes. Follow its section numbers when you plan work |
| `BRD.md` | Approved business requirements (Arabic) | Read it to understand why a feature exists. `SPEC.md` §1.2 lists where the two differ |
| `prototype/index.html` | Approved visual concept | Match its look, layout and wording. It is a reference, not the production code |
| `AGENTS.md` | This file | The rules for working with the owner |

If these files disagree, follow `SPEC.md` and tell the owner about the conflict in simple Arabic.

**Decided: data storage (owner decision, 2026-10-22).**
- Follow `SPEC.md` §6.6: changes are saved in the browser (localStorage) and stay after the page is refreshed or reopened.
- Provide the **«إعادة البيانات الأصلية» (Reset demo data)** button from SPEC §5.1. It is shown to Protocol Leadership only, asks for confirmation first, and restores the original seed data.
- The prototype (`prototype/index.html`) resets on every reopen. **Do not copy that behavior.** The owner chose SPEC's behavior.

---

## 3. The owner

- The owner is a government employee and **not technical**.
- The owner prefers **Arabic**.
- The owner makes every decision about how the app is used. You make the technical decisions, and you explain them simply.

---

## 4. Communication rules

### 4.1 Language
| Channel | Language |
| --- | --- |
| Every message, question, summary and explanation to the owner | **Arabic only** |
| Code, comments, file and variable names, commit messages, technical docs (`SPEC.md`, `CHANGELOG.md`, README) | **English** |
| Text shown inside the app (labels, buttons, toasts, report) | **Arabic** |

### 4.2 Explaining a change
After every completed change, tell the owner in short, plain Arabic:
1. **What changed.** One or two sentences, with no jargon.
2. **The benefit.** What the owner or the committee can now do, or what is now safer or faster.
3. **How to see it.** For example: «اختر دور "عضو الاستقبال" من الأعلى، ثم افتح صفحة الاستقبال».
4. **What was not done or not checked**, if anything.

Avoid technical words. If one is unavoidable, explain it in brackets. Example: «الخادم (الجهاز الذي يشغّل الموقع على الإنترنت)».

### 4.3 Presenting options
When there is more than one reasonable way to do something:
- Give **2 or 3 options at most**, each in everyday Arabic.
- For each option, explain how it **affects the owner**: cost, effort, what users will see, and any risk.
- Show the differences side by side, in a short table or list.
- **Recommend one option** and say why in one sentence.
- Ask **one simple question** at a time, and wait for the answer.

Example format:

> **الخيار 1 (أنصح به):** … — الفائدة: … — التكلفة: مجاني
> **الخيار 2:** … — الفائدة: … — التكلفة: …
> **الفرق الأساسي:** …
> أيهما تفضّل؟

### 4.4 Actions the owner must do outside the app
Some steps can only be done by the owner, such as installing a program, creating an account, uploading to hosting, or adding the logo file. For these, give **numbered, step-by-step instructions in Arabic**:
- Write one action per step: where to click, what to type, and what the owner should see afterwards.
- Name buttons and menus exactly as they appear on screen. Give the English label too if the screen is in English.
- Give a shell command in its own code block, and explain in Arabic what it does.
- Say how the owner can confirm the step worked.
- Never ask the owner to type a password, card number or any secret into the chat.

Note: Node.js is **not installed** on the owner's computer. Before the first build, guide the owner through installing the Node.js LTS version step by step, and confirm it works with `node -v`.

---

## 5. Hard rules

1. **Demo data only.**
   - Use only fictional names, fake phone numbers (`05X …`, marked "تجريبي") and fake emails (`@demo.local`).
   - Never ask for, accept, store or type real personal data.
   - If the owner offers real data, politely decline and explain why in Arabic.
2. **Role switcher, no login.**
   - Use the role switcher at the top of the page (SPEC §5.1) instead of any login.
   - Do not add sign-in, accounts or passwords.
   - Still check permissions inside every action handler (SPEC §5.3).
3. **One change at a time.**
   - Do one change, verify it (§6.2), save it as a version (§6.3), and report it (§4.2).
   - Only then start the next change.
   - Never bundle unrelated changes.
4. **Nothing outside `SPEC.md` without asking.**
   - Do not add features, pages, fields or behaviors that `SPEC.md` does not describe.
   - If you have an idea, propose it to the owner as an «فكرة للاحقًا», and build it only after approval.
5. **Ask when a requirement is unclear.**
   - If something that affects how people use the app is unclear or missing, stop and ask the owner one simplified question (§4.3).
   - Do not guess on wording users will see, workflows, permissions, numbers or names.
   - Purely technical choices are yours: decide, and mention them briefly.
6. **Arabic, RTL, mobile, hackathon identity.**
   - Every screen is in Arabic with `dir="rtl"`.
   - Every screen works from 360 px phone width to desktop, with no horizontal scrolling.
   - Every screen uses the hackathon identity (§7).
7. **No external or paid services without approval.**
   - Do not connect any external service, API, database, analytics, AI model or paid plan without the owner's explicit approval.
   - This includes anything that has a free tier.
   - The only approved service is free static hosting on **Cloudflare Pages** (SPEC §4). Google Fonts is allowed for the Arabic font.
   - When a service seems needed, compare options with the owner first (§4.3) and say whether each one is free or paid.

---

## 6. How to work

### 6.1 Tech stack (from SPEC §3; do not change it without asking)
- TypeScript, React 18, Vite 5 and Tailwind CSS 3, with design tokens.
- Zustand for state, React Router 6 (hash router), `lucide-react` icons, Vitest for tests.
- Font: IBM Plex Sans Arabic. Time zone: `Asia/Riyadh`, shown as `HH:mm` with Latin digits.
- No server, no database and no API in this version.
- Follow the project structure in SPEC §11.

### 6.2 Best practices and verification
- Keep business rules in pure functions under `src/domain/`. These are: guest state machine, notification routing, gate change, 3-minute escalation, permissions, and report data (SPEC §8, §10).
- Derive dashboard and report numbers from the event log. Never keep separate counters.
- Use logical CSS only (`ms-`/`me-`/`ps-`/`pe-`), never left or right.
- Make buttons at least 48 px tall. Never show status with color alone; use an icon and a word too.
- Before reporting any change as done:
  1. `npm run build` passes with no errors.
  2. `npm test` passes. Add or update a test for any business rule you touched (see SPEC §12 for the acceptance mapping).
  3. Open the app and try the change yourself. Use the role switcher for each affected role, and check at 360 px width and on desktop.
  4. The browser console shows no errors.
- If a check fails, fix it before moving on. If you cannot fix it, tell the owner honestly in Arabic and include what failed.

### 6.3 Saving each completed change as a version
- Use Git. If the folder is not a Git repository yet, explain to the owner in Arabic what it is for, then run `git init` once.
- After each verified change, make **one commit**. Its message is a short English summary, for example: `Add gate change re-routing to gate officers (SPEC §8.3)`.
- Add a line to `CHANGELOG.md` (English) with the date, the change and the SPEC section.
- Tell the owner in Arabic that a version was saved, and that you can go back to it at any time if they ask.
- Never rewrite history, force-push or delete versions without the owner's approval.

---

## 7. Visual identity (from SPEC §9 and the hackathon materials)

| Token | Hex | Use |
| --- | --- | --- |
| `--bg-deep` | `#0A0638` | App background |
| `--bg-surface` | `#14108A` | Cards and panels |
| `--brand-indigo` | `#2A14E0` | Primary buttons and the active navigation item |
| `--brand-violet` | `#6B3BFF` | Accents and focus rings |
| `--brand-lavender` | `#B8A4FF` | Headings and highlights |
| `--glow` | `#E4DBFF` | Neon edge glow |
| `--text` / `--text-muted` | `#FFFFFF` / `#C9C2F2` | Text |
| Status colors | scheduled `#8A85B8` · near `#FFC24B` · arrived `#4FC3FF` · received `#B8A4FF` · escorted `#3DDC97` · danger `#FF5470` | Status chips and alerts |

- **Motifs:**
  - Hexagon avatars and icons.
  - Neon glowing borders on key cards and buttons.
  - An angled violet panel in page headers (on the right side, because the layout is RTL).
  - Soft violet radial gradient in the background.
- **Logo:** the official hackathon logo goes in `public/brand/hih-logo.svg` (or `.png`), and the owner supplies the file. Until then, show a plain hexagon placeholder.
- Text contrast must meet WCAG AA.
- When in doubt, match `prototype/index.html`.

---

## 8. Quick checklist for every task

- [ ] I read `SPEC.md` for the part I am changing.
- [ ] The change is in `SPEC.md`, or the owner approved it.
- [ ] Demo data only. No new external or paid service.
- [ ] Arabic, RTL, works at 360 px, hackathon identity.
- [ ] Build, tests and a manual check by role all pass.
- [ ] One commit and one `CHANGELOG.md` line.
- [ ] A short Arabic report to the owner: what changed, the benefit, how to see it, and what is left.
