# 🏰 Arcane Quest 04 — Arcane Guild Board (Group Project)

> *"Ngoding sendirian itu satu skill. Ngoding bareng tim itu skill yang COMPLETELY DIFFERENT. Di dunia kerja, kalian bakal selalu kerja di tim. Ini saatnya latihan."*

## 🎯 Misi

Dalam tim **3-4 orang**, bangun **Arcane Guild Board** — sebuah team productivity dashboard yang menggabungkan SEMUA konsep Week 3.

---

## 👥 Tim & Roles

Setiap tim WAJIB assign roles:

| Role | Tanggung Jawab |
|------|----------------|
| **Tech Lead** | Architecture decisions, code review, merge PRs, resolve conflicts |
| **Frontend Lead** | UI/UX design, component library, responsive layout |
| **Feature Dev 1** | Implement assigned features, write tests |
| **Feature Dev 2** | Implement assigned features, write tests |

> Kalo tim 3 orang, Tech Lead + Frontend Lead bisa di-merge.

---

## 📋 The App: Arcane Guild Board

Dashboard produktivitas tim yang punya fitur:

### Core Features (WAJIB)

#### 1. Task Board (Kanban)
- Columns: To Do → In Progress → Review → Done
- Drag & drop tasks antar kolom
- Create, edit, delete tasks
- Assign task ke team member
- Priority labels (Low, Medium, High, Urgent)
- Filter by assignee, priority, search

#### 2. Team Members
- List semua guild members
- Status: Online/Offline/Busy
- Role badge
- Activity feed (recent actions)

#### 3. Real-time Updates
- WebSocket ATAU polling (pilih salah satu)
- Saat satu orang move task → semua orang liat update
- Online presence indicators

#### 4. Analytics Dashboard
- Tasks per status (bar chart)
- Tasks per member (pie chart)
- Completion rate over time (line chart)
- Use **Recharts** atau **Chart.js**

#### 5. Auth & Profiles
- Login system (Clerk, Firebase Auth, atau custom)
- Protected routes
- User profile page

### Tech Requirements

- **Vite + React + TypeScript** (WAJIB)
- **Tailwind CSS** (WAJIB)
- **TanStack Query** untuk data fetching
- **Zustand** untuk client state
- **Git** dengan proper branching strategy

---

## 🏗️ Git Workflow (WAJIB)

```
main
├── develop
│   ├── feature/task-board        (Feature Dev 1)
│   ├── feature/team-members      (Feature Dev 2)
│   ├── feature/analytics         (Frontend Lead)
│   └── feature/auth              (Tech Lead)
```

### Rules:
1. **JANGAN push langsung ke `main`**
2. Semua changes lewat **Pull Request** ke `develop`
3. PR harus di-review minimal 1 orang lain sebelum merge
4. Commit messages harus descriptive: `feat: add drag-drop to task board`
5. **Minimal 10 commits per person**

---

## 🛤️ Phases

| Phase | Milestone |
|-------|-----------|
| **Planning** | Team formation, role assignment, project setup, architecture planning |
| **Setup** | Setup repo, CI/CD, basic routing + layout, component library |
| **Build** | Feature development (parallel) |
| **Integrate** | Integration, bug fixing, PR reviews |
| **Polish** | Polish UI, prepare demo, write documentation |
| **Demo** | **🎤 DEMO DAY** |

---

## 🎤 Demo Day Format

Setiap tim presentasi **10 menit**:

| Segment | Durasi | Detail |
|---------|--------|--------|
| **Live Demo** | 4 min | Showcase semua fitur, real-time demo |
| **Tech Talk** | 3 min | Architecture, tech decisions, challenges |
| **Q&A** | 3 min | Pertanyaan dari gua dan tim lain |

### Demo Day Rules:
- SEMUA anggota tim HARUS bicara
- Live demo — bukan recording/slides
- Kalo app crash pas demo, tetep jalan (show how you debug)
- Tim lain boleh kasih feedback

---

## 📊 Grading

| Kriteria | Bobot | Detail |
|----------|-------|--------|
| **Functionality** | 30% | Semua core features berjalan, no critical bugs |
| **Teamwork** | 20% | Git history, PR reviews, balanced contribution, communication |
| **UI/UX** | 20% | Consistent design, responsive, polished, good UX flow |
| **Code Quality** | 15% | TypeScript strict, clean architecture, reusable components |
| **Presentation** | 15% | Clear demo, good communication, handle Q&A well |

### Individual Adjustment

Gua cek git history per orang. Kalo ada yang free-ride:

- **Kontribusi merata:** Semua dapat grade yang sama
- **1 orang dominan:** Yang kerja dapat bonus +5, yang numpang grade -10
- **Bukti:** Git commits, PR history, code ownership

---

## ✨ Bonus Features

| Bonus | Poin | Detail |
|-------|------|--------|
| 🔄 **Real-time Sync** | +5 | WebSocket beneran (bukan polling) |
| 📱 **PWA** | +3 | Installable, works offline |
| 🔔 **Notifications** | +3 | In-app + browser notifications for task assignments |
| 📊 **Sprint Planning** | +3 | Sprint cycles, velocity tracking |
| 🎨 **Custom Themes** | +2 | Team bisa pilih theme/colors |
| 📝 **Activity Log** | +2 | Full audit trail of all actions |

---

## 📝 Submission

- **GitHub repo** (public, all members as collaborators)
- **Vercel deployment URL**
- **README.md** di repo:
  - Team members + roles
  - Tech stack
  - Setup instructions
  - Architecture diagram
  - Screenshots

---

## ⚠️ Anti-Free-Ride Policy

Gua bakal jalanin:

```bash
git shortlog -sn --all
```

Kalo ada anggota yang commit-nya < 5 atau semua commit cuma "fix typo", kita ngobrol.

---

## 💡 Tips

1. **Day 1 itu CRUCIAL** — spend full day planning. Architecture diagram, component list, API design, task breakdown
2. **Komunikasi > Code** — pake Discord/WA group, daily standup 10 menit
3. **Merge often** — jangan develop 3 hari baru merge, pasti conflict
4. **Help each other** — kalo teammate stuck, bantu. Grade tim = grade lu juga
5. **Demo prep** — practice demo minimal 2x sebelum Demo Day

> *"Tim yang planning-nya bagus di Day 1 SELALU menang. Tim yang langsung ngoding tanpa plan SELALU chaos. Percaya sama gua."*

---

---

## Demo Day di Magic Temple 🏛️

Setiap tim **WAJIB** presentasi project mereka di Discord voice channel **"Magic Temple"**.

### Format: 10 menit per tim

| Segment | Durasi | Detail |
|---------|--------|--------|
| **Live Demo** | 4 min | Showcase semua fitur, real-time demo, share screen di Discord |
| **Tech Talk** | 3 min | Architecture, tech decisions, challenges |
| **Q&A** | 3 min | Pertanyaan dari mentor dan tim lain |

### Rules

1. **SEMUA anggota tim HARUS bicara** — bukan cuma 1 orang yang present
2. **Share screen di Discord** untuk live demo
3. **App harus deployed** — demo dari deployed URL, bukan localhost
4. **Have backup recording** — record demo pakai OBS/screen recorder sebelumnya

### Tips buat Discord Demo

- [ ] Test screen share di Discord sebelumnya — pastiin work
- [ ] Test mic — pastiin suara jelas, gak ada echo
- [ ] Stable internet — pake kabel LAN kalau bisa
- [ ] Close unnecessary tabs — biar Discord gak lag
- [ ] Practice run minimal 1x — time it, pastiin 10 menit
- [ ] Join Magic Temple voice channel **5 menit sebelum jadwal**

---

**Assemble your guild, Arcanists! ⚔️🛡️**
