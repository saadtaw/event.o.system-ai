# Technical Specification — EventOps AI

**Smart Operations Room for the 7th Health Innovation Hackathon**

| Item | Value |
| --- | --- |
| Source document | `BRD.md` v1.0 (approved) |
| Release | Prototype (MVP), demo data only |
| Spec version | 1.0 |
| Date | 2026-10-22 |
| UI language | Arabic, right-to-left (RTL) |

---

## 1. Summary

EventOps AI is a single-page web app that Protocol Committee staff use to run guest reception at the hackathon. In this first version:

- One device is enough. All roles are tested on the same device, using a **role switcher** at the top of the page instead of a login.
- All data is fake (40 guests, 42 staff) and is stored in the browser on that device.
- The daily report is **generated from a fixed template filled with the recorded numbers**. It is not written by an AI model in this version.
- The app is published as a **public link** on free static hosting.
- The look follows the **Health Innovation Hackathon visual identity**: deep indigo and violet, white, neon glow, hexagon shapes, and the official logo.

### 1.1 Decisions made with the business owner

| # | Question | Decision |
| --- | --- | --- |
| D-1 | Test on several phones at once, or on one device? | **One device**, using the role switcher |
| D-2 | Real AI for the daily report? | **No.** Template-based report built from the numbers |
| D-3 | Where does the app run? | **Public internet link** |
| D-4 | Place names | **Real venue names** from the committee meeting deck |
| D-5 | Gate locations | Proposed mapping accepted (§6.2) |
| D-6 | Destinations per track | Proposed mapping accepted (§6.2) |
| D-7 | Event days and shifts | Default: 3 days, two shifts per day (§6.3) |
| D-8 | Official hackathon logo | **Yes.** The owner supplies the logo file |
| D-9 | Priority levels | **Three**: Urgent, Medium, Normal |
| D-10 | Parking input | Guest **types the parking number** (free text, e.g. `P-12`) |
| D-11 | Event date in demo data | **The day the app is opened.** Day 1 = today |

### 1.2 Differences from the BRD

These changes follow from the decisions above. Everything else in the BRD applies as written.

| BRD item | Change in this version | Reason |
| --- | --- | --- |
| Page 6 "AI Reports", F-6.3 | The daily report text is built from a fixed Arabic template filled with numbers from the event log. No AI service is called. The page keeps its name and is designed so an AI writer can be plugged in later (§10.3). | D-2 |
| Acceptance criterion 15 (daily report) | Pass condition: every number in the report matches the event log, and the report can be edited and approved before sharing. | D-2 |
| Real-time updates | "Real-time" means instant updates **on the same device**. There is no syncing between devices. | D-1 |
| Constraint "AI only summarizes" (BR-7) | Satisfied: there is no AI in this version. | D-2 |

---

## 2. Architecture

```
┌──────────────────────────── Browser (one device) ────────────────────────────┐
│                                                                               │
│  Role switcher ─► Current actor (role + person)                               │
│        │                                                                      │
│        ▼                                                                      │
│  Pages (6)  ──read──►  Selectors (filtered by actor permissions)              │
│     │                        ▲                                                │
│     │ dispatch(command)      │ derived state                                  │
│     ▼                        │                                                │
│  Command handlers ──► Business rules (routing, escalation, guards)            │
│     │                                                                         │
│     ▼                                                                         │
│  Event log (append-only)  ──►  Store (Zustand)  ──►  localStorage             │
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘
          ▲
          │ static files (HTML/JS/CSS/logo)
   Cloudflare Pages (free)
```

**Key design choice: the event log is the source of truth.** Every action (guest is near, gate changed, request received, and so on) is recorded as an event with a time and the person who did it. Dashboard counts and report numbers are **calculated from that log**. This is what guarantees the BRD's "100% match between dashboard and event log" criterion.

There is no server, no database service and no API in this version.

---

## 3. Technology choices

| Layer | Choice | Why |
| --- | --- | --- |
| Language | TypeScript | Catches data mistakes early. AI coding tools handle it well |
| UI framework | React 18 | Most widely supported by vibe-coding tools |
| Build tool | Vite 5 | Outputs a plain static folder (`dist/`) that any free host serves |
| Styling | Tailwind CSS 3 with design tokens (§9) | Fast to iterate. RTL supported through logical properties |
| State | Zustand + `persist` middleware (localStorage) | Small and simple. Survives page refresh |
| Routing | React Router 6 (hash router) | Works on static hosting with no server rewrite rules |
| Dates and time | `Intl.DateTimeFormat` with `timeZone: 'Asia/Riyadh'` | Saudi time (UTC+3) with no extra library |
| IDs | `crypto.randomUUID()` | Built into the browser |
| Icons | `lucide-react` | Free, consistent line icons |
| Font | **IBM Plex Sans Arabic** (Google Fonts), weights 400/500/700 | Clean geometric Arabic that suits the hackathon identity |
| Tests | Vitest | Unit tests for the business rules (§12) |

All of the above are free and open source.

---

## 4. External services

The only external service needed is **hosting**, which gives the app a public link (D-3). Because there is no AI (D-2) and no shared database (D-1), nothing else is required.

### 4.1 Hosting comparison

| Option | Cost | Limits that matter here | Ease for a non-technical owner | Verdict |
| --- | --- | --- | --- | --- |
| **Cloudflare Pages** | Free | Static file traffic is free and unlimited, 500 builds per month | Upload the `dist` folder from the dashboard (Direct Upload). No Git needed | ✅ **Recommended** |
| Netlify | Free tier, credit-based | About 300 credits per month. The site **pauses** if credits run out | Drag-and-drop deploy | Good alternative |
| GitHub Pages | Free | Soft caps. Needs a GitHub repository. Terms limit some uses | Needs Git knowledge | Not recommended |

**Recommendation:** Cloudflare Pages on the free plan, **0 SAR per month**. Its free static traffic is unlimited, so the demo link will not stop working halfway through a test day.

### 4.2 Deployment steps (owner-friendly)

1. Run `npm run build`. This creates a `dist` folder.
2. Sign in to Cloudflare, then go to **Workers & Pages → Create → Pages → Upload assets**.
3. Name the project (for example `eventops-hih`) and upload the `dist` folder.
4. Share the link (`https://eventops-hih.pages.dev`).

> The demo link is public. That is acceptable here only because all data is fake (BRD §7). Never put real names or numbers in this version.

---

## 5. Roles, actors and the role switcher

### 5.1 Role switcher

A fixed bar at the top of every page has two drop-down lists:

1. **Role**: Guest, Reception Member, Gate Officer, Track Leader, Committee Rep, Protocol Leadership.
2. **Person**: everyone who holds that role, from the seed data. For example, pick "Reception Member" and then choose which of the 27 receptionists.

When the person changes:
- The app goes to that role's home page (§7).
- Pages and data are re-filtered to that person's permissions (§5.3).
- A badge shows that person's unread notifications.

The chosen actor is saved in localStorage, so a page refresh keeps it.

The bar also has a **"Reset demo data"** button, shown to Protocol Leadership only. It restores the seed data after a confirmation dialog.

### 5.2 Role keys

```ts
type Role =
  | 'guest'
  | 'receptionist'
  | 'gate_officer'
  | 'track_leader'
  | 'committee_rep'
  | 'leadership';
```

### 5.3 Permission matrix (enforced in selectors and command handlers)

| Page / route | guest | receptionist | gate_officer | track_leader | committee_rep | leadership |
| --- | --- | --- | --- | --- | --- | --- |
| `/guest` Guest Portal | own record | — | — | — | — | — |
| `/reception` Reception Ops | — | own guests | — | — | — | — |
| `/dashboard` Live Ops | — | — | read, own gate | read, own track | — | full |
| `/support` Support Ops | — | create + own requests | create + own requests | create + track requests | requests for own committee, update status | full |
| `/tasks` Task Management | — | read own tasks | read own tasks | read track tasks | read own tasks | **full edit** (only role allowed) |
| `/reports` Reports | — | — | — | — | — | full |

Rules:
- The navigation bar shows only the pages the current role can open. If a role opens a route it cannot access, the app shows a "not available for your role" message.
- **Every command handler checks permission again** before acting. Hiding a button is not enough. A rejected command adds nothing to the log and shows an error toast. This is what acceptance criterion 13 tests.

---

## 6. Data

### 6.1 Entities (TypeScript)

```ts
type TrackId = 'investors' | 'intl_judges' | 'mentors' | 'expo_judges';
type GateId = 'G1' | 'G2' | 'G3' | 'G4';
type CommitteeId = 'registration' | 'parking' | 'transport' | 'hospitality' | 'security_safety';
type Priority = 'urgent' | 'medium' | 'normal';

type GuestStatus = 'scheduled' | 'near' | 'arrived' | 'received' | 'escorted';

interface Guest {
  id: string;               // 'GST-01' … 'GST-40'
  name: string;             // fictional Arabic name
  avatar: string;           // initials-based generated avatar, no real photos
  title: string;            // e.g. 'Investment Director'
  expertise: string;
  track: TrackId;
  phone: string;            // fake: '05X-XXX-XXXX' pattern, clearly marked 'demo'
  email: string;            // fake: '@demo.local'
  status: GuestStatus;
  gate: GateId;             // defaults to the track's gate
  parking: string | null;   // free text, e.g. 'P-12' (D-10)
  destination: string;      // from track (§6.2)
  receptionistId: string;
  trackLeaderId: string;    // derived from track, stored for clarity
}

interface Staff {
  id: string;               // 'STF-01' … 'STF-42'
  name: string;
  phone: string;            // fake
  role: Exclude<Role, 'guest'>;
  track: TrackId | null;
  zone: string | null;
  gate: GateId | null;      // gate officers only
  committee: CommitteeId | null; // committee reps only
  shift: ShiftId;
  managerId: string | null;
}

interface Task {
  id: string;
  title: string;
  description: string;
  assigneeId: string;
  priority: Priority;
  dueAt: string;            // ISO time
  status: 'new' | 'in_progress' | 'done';
  updatedAt: string;
}

interface HelpRequest {
  id: string;               // 'REQ-0001' running number
  committee: CommitteeId;   // request type = committee
  description: string;
  location: string;
  requesterId: string;      // guest or staff id
  relatedGuestId: string | null;
  track: TrackId | null;    // used for escalation
  priority: Priority;
  status: 'new' | 'in_progress' | 'done';
  createdAt: string;
  receivedAt: string | null;   // moved to in_progress
  closedAt: string | null;
  overdue: boolean;
  escalatedTo: string | null;  // track leader id
}

interface Notification {
  id: string;
  recipientId: string;
  eventId: string;          // the event that caused it
  text: string;             // Arabic message
  createdAt: string;
  read: boolean;
}

interface DailyReport {
  id: string;
  date: string;             // event day
  scope: { track: TrackId | 'all'; from: string; to: string };
  body: string;             // generated, then edited
  status: 'draft' | 'approved';
  approvedBy: string | null;
  approvedAt: string | null;
}
```

### 6.2 Venue reference data (real names, D-4 to D-6)

| Track | Arabic name | Gate | Gate location | Destination |
| --- | --- | --- | --- | --- |
| `investors` | مسار المستثمرين | G1 | المدخل الرئيسي لمبنى الدراسات العليا | معرض رواد الأعمال – الدور الثاني |
| `intl_judges` | مسار المحكمين الدوليين | G2 | مدخل البرج | المسرح الرئيسي |
| `mentors` | مسار المرشدين | G3 | الممر الخارجي (بين الدراسات العليا والبرج) | قاعات الهاكاثون A1 وA2 |
| `expo_judges` | مسار محكمي معرض الابتكار | G4 | مدخل الدور الثاني (معرض رواد الأعمال) | معرض رواد الأعمال – الدور الثاني |

Other named places, used for help-request locations: منطقة التسجيل، منطقة الضيافة.

### 6.3 Event days and shifts (D-7, D-11)

- Day 1 = the date the app is first opened (or reset). Days 2 and 3 follow it.
- Shifts: `AM` 08:00–14:00 and `PM` 14:00–20:00, Riyadh time. `ShiftId` = `D1-AM` … `D3-PM`.
- All of this lives in one config file (`src/config/event.ts`) so it is easy to change.

### 6.4 Seed data (`src/data/seed.ts`)

| Group | Count | Details |
| --- | --- | --- |
| Leadership | 2 | Committee leader + deputy |
| Investors track | 7 | 1 track leader + 6 receptionists |
| International judges track | 5 | 1 track leader + 4 receptionists |
| Mentors track | 13 | 1 track leader + 12 receptionists |
| Expo judges track | 6 | 1 track leader + 5 receptionists |
| Gate officers | 4 | One per gate (G1–G4) |
| Committee reps | 5 | Registration, Parking, Transport, Hospitality, Security & Safety |
| **Staff total** | **42** | |
| Guests | 40 | 10 per track, all VIP. Spread evenly over the track's receptionists |

Seed validation runs at startup in development and in tests. It fails loudly if the counts above are wrong, or if any guest is missing a receptionist, track leader or gate officer (acceptance criterion 4).

### 6.5 Event log

```ts
interface OpsEvent {
  id: string;
  type: EventType;
  at: string;          // ISO time, Riyadh shown in UI
  actorId: string;     // who did it
  entityId: string;    // guest / request / task id
  data: Record<string, unknown>; // before/after values
}

type EventType =
  | 'GUEST_NEAR' | 'GUEST_ARRIVED' | 'GUEST_RECEIVED' | 'GUEST_ESCORTED'
  | 'GATE_CHANGED' | 'PARKING_SET'
  | 'RECEPTION_DELAY_REPORTED'
  | 'REQUEST_CREATED' | 'REQUEST_RECEIVED' | 'REQUEST_CLOSED' | 'REQUEST_ESCALATED'
  | 'TASK_CREATED' | 'TASK_REASSIGNED' | 'TASK_STATUS_CHANGED'
  | 'GUEST_REASSIGNED' | 'SHIFT_CHANGED'
  | 'REPORT_GENERATED' | 'REPORT_EDITED' | 'REPORT_APPROVED';
```

The log is append-only. Nothing is edited or deleted, except by "Reset demo data", which clears everything back to seed.

### 6.6 Storage

- One localStorage key, `eventops:v1`, holds `{ guests, staff, tasks, requests, notifications, reports, events, actor, eventStartDate }`.
- The expected size is well under 1 MB.
- A `schemaVersion` field allows a safe reset if the data shape changes.

---

## 7. Pages

All pages are RTL, mobile-first, with buttons at least 48 px tall (§9.4).

### 7.1 Guest Portal — `/guest` (home of `guest`)

| Element | Behavior |
| --- | --- |
| Visit card | Name, title, track, destination, assigned receptionist name (no phone shown to the guest) |
| Gate selector | 4 large buttons (G1–G4) with location names. Defaults to the track's gate |
| Parking field | Text input with placeholder `P-12`. Saved on blur or "Save" (`PARKING_SET`) |
| **"أنا قريب"** button | Enabled only when status = `scheduled` → `GUEST_NEAR` |
| **"وصلت"** button | Enabled when status ∈ {`scheduled`, `near`} → `GUEST_ARRIVED` |
| Status timeline | Scheduled → Near → Arrived → Received → Escorted, with times |
| Help request | Pick a type (5 committees), add a short description, submit. Below it: the guest's own requests and their status |

### 7.2 Reception Operations — `/reception` (home of `receptionist`)

| Element | Behavior |
| --- | --- |
| My guests list | Cards sorted by status (arrived first, then near), showing status chip, gate, parking and destination |
| Guest detail | Gate, parking, destination, demo phone (authorized for this role) |
| **"تم الاستقبال"** | Enabled when status = `arrived` → `GUEST_RECEIVED` |
| **"تمت المرافقة"** | Enabled when status = `received` → `GUEST_ESCORTED` |
| "Report delay" | Short reason → `RECEPTION_DELAY_REPORTED` (notifies the track leader) |
| "Request support" | Opens the help-request form, prefilled with the guest and location |
| Notifications | Bell with a list of near/arrived alerts for my guests |

### 7.3 Live Operations Dashboard — `/dashboard`

- **Leadership:** everything. **Track leader:** filtered to their track. **Gate officer:** filtered to guests whose current gate is theirs.
- KPI tiles, all derived from the event log:
  - Guests per status: scheduled, near, arrived, received, escorted.
  - Open requests, overdue requests, overdue tasks.
  - Average arrived→received time.
  - Average request created→received time.
- A 4-column track board (one column per track) with guest chips colored by status.
- Team readiness: receptionists on the current shift, and how many are "busy" (they have a guest in `received` that is not yet escorted).
- Live feed: the latest 20 events in plain Arabic sentences.
- Updates come from a store subscription, so changes appear **immediately** (well under the 3-second target).

### 7.4 Support Operations — `/support`

- Request form: committee (type), priority (3 levels), location (venue list or free text), description, optional related guest.
- Lists: **New**, **In progress** and **Done** columns. Overdue requests get a red "متأخر" badge.
- Committee rep: "استلام" (new → in_progress) and "إغلاق" (in_progress → done) buttons.
- Every row shows its created, received and closed times.

### 7.5 Task Management — `/tasks`

- **Leadership (edit):**
  - Staff table (42 rows) with filters for role, track and shift.
  - Guest assignment table (40 rows) with a drop-down to change the receptionist.
  - Tasks: create, edit, reassign, and change status.
  - Edit a person's shift, zone or gate.
- **Other roles (read-only):** a list of their own tasks (track leaders see their track's tasks).
- Gate change history per guest, taken from `GATE_CHANGED` events.

### 7.6 Reports — `/reports` (leadership only)

- Filters: track (all or one), and period (day 1/2/3 or a custom time range).
- Statistics panel: attendance, reception times, task completion and requests by committee.
- **"إنشاء التقرير"** builds the Arabic report from the template (§10).
- The report opens in an editable text area. **"اعتماد"** locks it (`REPORT_APPROVED`). After approval, **"نسخ"** copies the text to share.

---

## 8. Business rules (implementation)

### 8.1 Guest state machine

```
scheduled ──NEAR──► near ──ARRIVED──► arrived ──RECEIVED──► received ──ESCORTED──► escorted
    └────────────ARRIVED─────────────────┘
```

- A transition is allowed **only forward**, and only from the listed source state.
- **Duplicate-press guard (BR-5):** If a command asks for a state the guest is already in, or has already passed, it is a no-op. There is no event, no notification and no error, only a quiet "already recorded" toast. The button also turns disabled the moment it is pressed. Both guards together keep the log free of duplicates (acceptance criterion 17).

### 8.2 Notification routing (BR-2)

| Event | Recipients |
| --- | --- |
| `GUEST_NEAR`, `GUEST_ARRIVED` | Assigned receptionist, **officer of the guest's current gate**, track leader |
| `GATE_CHANGED` | New gate officer (with the guest's current status), the receptionist, the track leader. The old gate officer gets a "moved to G?" notice |
| `RECEPTION_DELAY_REPORTED` | Track leader |
| `REQUEST_CREATED` | Rep of the chosen committee, plus the assigned receptionist when the requester is a guest |
| `REQUEST_ESCALATED` | Track leader (§8.4), and the request also shows on the leadership dashboard |
| `REQUEST_RECEIVED`, `REQUEST_CLOSED` | The requester |
| `TASK_CREATED`, `TASK_REASSIGNED` | New assignee (and the old one on reassignment) |

Routing lives in one pure function, `routeNotifications(event, state): Notification[]`, which is unit-tested.

### 8.3 Gate change (BR-3)

When a guest changes gate:
1. The guest's `gate` field is updated.
2. A `GATE_CHANGED` event is written with `{ from, to }`.
3. The responsible gate officer is now the officer of the new gate. This is looked up, not stored, so it can never be stale.
4. If the guest is already `near` or `arrived`, the new gate officer gets the current status notification again.

### 8.4 Escalation (BR-4)

- A request is **overdue** when `status = 'new'` and `now − createdAt ≥ 3 minutes`. This applies to every priority.
- A checker runs every **15 seconds**, and also once when the app opens. For each overdue request that has not been escalated yet:
  - set `overdue = true` and `escalatedTo = trackLeaderId`;
  - write a `REQUEST_ESCALATED` event and notify the track leader.
- Which track leader is used:
  1. the track of the related guest, if there is one; otherwise
  2. the requester's track (receptionist or track leader); otherwise
  3. the track of the gate the gate officer covers.
- If no track can be found (for example, a request raised by a committee rep or by leadership), the request is shown to leadership as overdue with no track leader.
- Each request is escalated only once.

### 8.5 Tasks

- A task is overdue when `status ≠ 'done'` and `now > dueAt`. This is calculated, not stored.
- Only `leadership` can create, edit or reassign tasks (BR-8). The assignee may move their own task's status (new → in progress → done). This is the "update status" action in BRD page 5 and acceptance criterion 12.

### 8.6 Time

- All stored times are ISO UTC strings.
- All displayed times use `Asia/Riyadh` with Arabic digits off (Latin digits are easier to read on small screens), for example `09:42`.

---

## 9. Visual design — Hackathon identity

Taken from the 7th Health Innovation Hackathon materials: the cover slide, the registration desk, the wayfinding stands and the meeting deck.

### 9.1 Color tokens

| Token | Hex | Use |
| --- | --- | --- |
| `--bg-deep` | `#0A0638` | App background (the deep midnight indigo of the slides) |
| `--bg-surface` | `#14108A` | Cards and panels |
| `--brand-indigo` | `#2A14E0` | Primary actions and the active nav item (the vivid blue-violet slide panels) |
| `--brand-violet` | `#6B3BFF` | Secondary accents and focus rings |
| `--brand-lavender` | `#B8A4FF` | Headings and highlights (the "Beyond Visions" title color) |
| `--glow` | `#E4DBFF` | Neon edge glow on key cards and buttons |
| `--text` | `#FFFFFF` | Main text |
| `--text-muted` | `#C9C2F2` | Secondary text |
| `--status-scheduled` | `#8A85B8` | Guest status chips |
| `--status-near` | `#FFC24B` | |
| `--status-arrived` | `#4FC3FF` | |
| `--status-received` | `#B8A4FF` | |
| `--status-escorted` | `#3DDC97` | |
| `--danger` | `#FF5470` | Overdue and escalated items |

- The background is a soft radial gradient from `--brand-violet` (top corner) into `--bg-deep`, echoing the light beam in the cover slide.
- Text contrast must meet WCAG AA (4.5:1) on `--bg-deep` and `--bg-surface`.
- Status is never shown by color alone. Each chip also has an icon and a word.

### 9.2 Shapes and motifs

- **Hexagon:** used for avatars, KPI tile icons and the empty-state illustration (from the logo and the QR frame).
- **Neon edge:** 1 px `--glow` border plus `box-shadow: 0 0 16px rgba(180,160,255,.35)` on primary cards and active buttons, echoing the lit arches and the registration desk.
- **Angled panel:** page headers use a diagonal violet panel on the right (RTL start), like the meeting-deck slides.
- **"7" accent:** an optional subtle shape on the dashboard header, as on the wayfinding stands.
- Corner radius: 14 px for cards, 12 px for buttons.

### 9.3 Logo

- The owner places the official logo file at `public/brand/hih-logo.svg` (PNG accepted: `hih-logo.png`).
- It is shown in the top bar (32 px tall) next to the "EventOps AI" title.
- Until the file is added, a plain hexagon placeholder is shown so the build never breaks.

### 9.4 Layout and usability

- `<html lang="ar" dir="rtl">`. Use only logical CSS (`ms-`, `me-`, `ps-`, `pe-` in Tailwind), never `left`/`right`.
- Mobile first. It must work at **360 px** wide with **no horizontal scrolling** (acceptance criterion 16).
- Action buttons are at least **48 px** tall, full width on phones, 18 px bold text.
- Navigation: a bottom tab bar on phones and a right-side rail on desktop. Both show only the pages allowed for the current role.
- Toasts appear top-center. Notification badges use `--danger`.
- The top role switcher collapses into one compact button on phones.

---

## 10. Daily report (template-based)

### 10.1 Inputs
The event log is filtered by the chosen track and period.

### 10.2 Template (Arabic output)

```
التقرير التشغيلي اليومي — {اليوم} {التاريخ}
النطاق: {المسار أو جميع المسارات} | الفترة: {من}–{إلى} (توقيت الرياض)

أولًا: الضيوف
- إجمالي الضيوف: {n}، وصل منهم {arrived+}، استُقبل {received+}، رُوفق إلى وجهته {escorted}.
- متوسط زمن الاستقبال بعد الوصول: {avg} دقيقة.
- ضيوف لم يصلوا بعد: {list or "لا يوجد"}.

ثانيًا: طلبات المساعدة
- أُنشئ {n} طلبًا: {by committee}. أُغلق {done}، وما زال {open} مفتوحًا.
- طلبات تجاوزت 3 دقائق وصُعّدت: {escalated}.
- متوسط زمن الاستلام: {avg} دقيقة.

ثالثًا: المهام
- {done} من {total} مكتملة. المهام المتأخرة: {list or "لا يوجد"}.

رابعًا: المشكلات المتكررة
- {top 3 committees / locations by request count, or "لا توجد مشكلات متكررة"}

خامسًا: تغييرات البوابات
- {n} تغيير بوابة خلال الفترة.
```

- Each `{…}` value is calculated by a pure function, `buildReportData(events, filter)`, which is unit-tested.
- **Repeated problems** = a committee or location with 2 or more requests in the period.

### 10.3 Future AI slot
`generateReportText(data): Promise<string>` is the single place that turns numbers into text. Today it fills the template. A later version can swap in an AI writer here without changing the page. This requires a small server function so that no API key is ever placed in the browser.

---

## 11. Project structure

```
eventops/
├─ public/brand/hih-logo.svg        # supplied by owner
├─ src/
│  ├─ config/event.ts               # tracks, gates, destinations, shifts, committees, 3-min rule
│  ├─ data/seed.ts                  # 40 guests, 42 staff, sample tasks
│  ├─ domain/
│  │  ├─ types.ts
│  │  ├─ guestMachine.ts            # transitions + duplicate guard
│  │  ├─ routing.ts                 # routeNotifications()
│  │  ├─ escalation.ts              # findOverdue(), escalate()
│  │  ├─ permissions.ts             # can(actor, action, entity)
│  │  ├─ selectors.ts               # visibility-filtered views, KPIs from events
│  │  └─ report.ts                  # buildReportData(), generateReportText()
│  ├─ store/useOps.ts               # Zustand store, commands, persist
│  ├─ components/                   # RoleSwitcher, Nav, StatusChip, KpiTile, HexAvatar, Toasts…
│  ├─ pages/                        # Guest, Reception, Dashboard, Support, Tasks, Reports
│  ├─ styles/tokens.css
│  └─ main.tsx
├─ tests/                           # Vitest
└─ package.json
```

---

## 12. Testing and acceptance mapping

### 12.1 Automated (Vitest), run with `npm test`

| Test | Covers BRD criterion |
| --- | --- |
| Seed counts: 40 guests (10 per track), 42 staff by role | 2, 3 |
| Every guest has a receptionist, track leader and gate officer | 4 |
| Near/arrived routes to exactly the 3 correct recipients (10 cases) | 5, 6 |
| Full journey scheduled → escorted (10 guests) | 8 |
| Gate change re-routes to the new gate officer and writes history (5 cases) | 9 |
| Request create → receive → close, with times recorded (5 cases) | 10 |
| Overdue after 3 minutes escalates once to the correct track leader (5 cases, fake timers) | 11 |
| Task assign / reassign / status change (5 cases) | 12 |
| Non-leadership task edits are rejected and nothing is logged (5 cases) | 13 |
| Dashboard KPIs equal counts recomputed from the raw event log | 14 |
| Report numbers equal `buildReportData` from the raw log | 15 |
| Double-press, missing parking, and gate change mid-journey create no duplicates (5 cases) | 17 |

### 12.2 Manual checks on the published link
- Criterion 1: all 4 tracks appear on the dashboard with correct data.
- Criterion 7: the dashboard updates within 3 s. Do an action, then switch to leadership; the count has already changed.
- Criterion 16: on a real phone (or a 360 px browser window), complete the guest → reception journey with no horizontal scroll.
- The two end-to-end scenarios in BRD §8.2, timed with a stopwatch and compared with the manual process.

---

## 13. Non-functional requirements

| Area | Requirement |
| --- | --- |
| Performance | First load under 3 s on 4G. JS bundle under 300 KB gzipped |
| Responsiveness | 360 px to 1920 px wide |
| Browsers | Latest Chrome, Safari (iOS) and Edge |
| Accessibility | WCAG AA contrast, visible focus rings, buttons with labels, status shown with text and icon |
| Privacy | Fake data only. No analytics, no cookies, no third-party calls except Google Fonts |
| Reliability | Data survives a page refresh (localStorage). "Reset demo data" restores seed |

---

## 14. Out of scope for this version

Everything in BRD §9, plus:
- Syncing between devices (D-1).
- Any AI service (D-2).
- Server, database or API.
- Push notifications outside the app.

## 15. Path to the next version (for reference only)

Each of these is a separate decision for later. None is built now.

| Need | Likely change |
| --- | --- |
| Several phones live at once | Add a shared database with live updates. A free tier is enough for a pilot |
| AI-written report | Add one server function that calls an AI model and keeps the key secret. Plug it in at `generateReportText` |
| Real login and permissions | Use the shared database's login and enforce the §5.3 matrix as database rules |
