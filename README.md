# EventOps AI | غرفة العمليات الذكية للهاكاثون

> نموذج أولي لمنصة تدير عمليات لجنة البروتوكول في هاكاثون الابتكار الصحي السابع: وصول الضيوف، والاستقبال، وطلبات المساعدة، وتوزيع المهام، والتقارير. جميع البيانات تجريبية.

An Arabic (RTL) operations-room prototype for the Protocol Committee of the 7th Health Innovation Hackathon. It covers guest arrival, reception, support requests to five committees, task assignment, a live dashboard and a template-based daily report.

## Try it

- **Live demo:** open the GitHub Pages link of this repository (`https://<username>.github.io/<repo>/`), or open `prototype/index.html` in any browser.
- There is no login. Use the **role switcher** at the top of the page to act as a guest, receptionist, gate officer, track leader, committee rep or protocol leadership.
- The demo data resets every time the page is reopened.

### Suggested 3-minute walkthrough
1. **Role: Guest.** Pick a gate, type a parking number, then press «أنا قريب» and «وصلت».
2. **Role: Receptionist** (auto-selects that guest's receptionist). Read the alert, then press «تأكيد الاستقبال» and «تأكيد المرافقة».
3. **Role: Protocol Leadership.** Open the live dashboard, then go to Reports and press «إنشاء التقرير», edit the text and approve it.
4. Within about a minute of opening the page, a seeded help request passes the 3-minute limit and is escalated to its track leader automatically.

## Repository contents

| Path | What it is |
| --- | --- |
| `BRD.md` | Approved business requirements (Arabic) |
| `SPEC.md` | Technical specification for the production build |
| `AGENTS.md` | Working rules for AI coding agents on this project |
| `prototype/index.html` | Interactive prototype: a single HTML file with no build step |
| `index.html` | Redirects to the prototype (for GitHub Pages) |
| `EventOps_AI_Guests_and_Team_Arabic.xlsx` | Synthetic guest and team roster (40 guests, 42 staff) |

## Scope of this version

- Demo data only: fictional names, non-working phone numbers, `@demo.local` / `example.com` emails.
- One device. Roles are switched in the page; there is no sync between devices.
- No external services. The daily report is filled from a fixed template using the event log, with no AI call.
- The official hackathon logo is not included. A hexagon placeholder is shown instead.

The production build described in `SPEC.md` (React + TypeScript, saved data, tests, Cloudflare Pages hosting) is the next step.
