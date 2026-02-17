# ⚡ Arcane Quest 01 — Arcane Dashboard

> *"Kalian udah belajar fetching data, state management, routing... Sekarang waktunya gabungin semua jadi satu dashboard analytics yang beneran berguna. Gua mau liat kalian bisa bikin app yang PRODUCTION-READY, bukan cuma tugas kampus."*

## 🎯 Misi

Bangun **Analytics Dashboard** yang nge-fetch data dari [JSONPlaceholder API](https://jsonplaceholder.typicode.com), tampilin di **sortable/filterable table**, pake **routing** buat navigasi antar halaman, dan **deploy ke Vercel**.

---

## 📋 Requirements

### Tech Stack (WAJIB)

- **Vite + TypeScript + React** (no CRA, no Next.js)
- **Tailwind CSS** untuk styling
- **TanStack Query** (`@tanstack/react-query`) untuk data fetching
- **TanStack Table** (`@tanstack/react-table`) untuk tabel
- **TanStack Router** atau **React Router v6** untuk routing
- **Deploy ke Vercel** — harus live, bukan localhost

### Fitur Wajib

#### 1. Dashboard Overview (`/`)
- Summary cards: Total Users, Total Posts, Total Comments, Total Albums
- Masing-masing card nge-fetch dari endpoint berbeda
- Loading skeleton saat data belum ready
- Error boundary kalo API gagal

#### 2. Users Table (`/users`)
- Fetch dari `https://jsonplaceholder.typicode.com/users`
- Kolom: Name, Email, Company, City, Phone
- **Sorting** — klik header kolom buat sort asc/desc
- **Filtering** — dropdown filter by City
- **Search** — input search dengan **debounce 300ms** (bukan on every keystroke!)
- **Pagination** — client-side, 5 rows per page

#### 3. Posts Table (`/posts`)
- Fetch dari `https://jsonplaceholder.typicode.com/posts`
- Kolom: ID, Title (truncated 50 chars), User (resolved dari userId)
- Sorting, search (debounce), pagination (10 rows per page)
- Klik row → navigasi ke `/posts/:id`

#### 4. Post Detail (`/posts/:id`)
- Fetch post + comments untuk post tersebut
- Tampilkan post title, body, dan list comments
- Back button ke `/posts`

#### 5. Responsive Design
- Desktop: sidebar navigation
- Mobile: hamburger menu / bottom nav
- Tabel harus scrollable horizontal di mobile

#### 6. Global Layout
- Sidebar/navbar dengan link ke semua halaman
- Active state pada current route
- Breadcrumb navigation

---

## 🏗️ Struktur Folder yang Diharapkan

```
src/
├── components/
│   ├── layout/
│   │   ├── Sidebar.tsx
│   │   ├── Header.tsx
│   │   └── Layout.tsx
│   ├── ui/
│   │   ├── Card.tsx
│   │   ├── Skeleton.tsx
│   │   ├── SearchInput.tsx
│   │   └── Pagination.tsx
│   └── tables/
│       ├── UsersTable.tsx
│       └── PostsTable.tsx
├── hooks/
│   ├── useDebounce.ts
│   └── useUsers.ts
├── pages/
│   ├── Dashboard.tsx
│   ├── Users.tsx
│   ├── Posts.tsx
│   └── PostDetail.tsx
├── lib/
│   ├── api.ts          // API client functions
│   └── queryClient.ts  // TanStack Query config
├── types/
│   └── index.ts        // TypeScript interfaces
├── routes/
│   └── index.tsx       // Route definitions
├── App.tsx
└── main.tsx
```

---

## 🔑 Kode yang HARUS Ada

### Custom Hook: `useDebounce`

```typescript
// hooks/useDebounce.ts
// Implement ini sendiri — JANGAN copy paste dari npm
// Harus pake useRef + useEffect
// Test: ketik "hello" cepet2 → cuma 1x re-render setelah delay
```

### TanStack Query Setup

```typescript
// lib/api.ts
const BASE_URL = 'https://jsonplaceholder.typicode.com';

export const fetchUsers = async (): Promise<User[]> => {
  const res = await fetch(`${BASE_URL}/users`);
  if (!res.ok) throw new Error('Failed to fetch users');
  return res.json();
};

// Buat fungsi serupa untuk posts, comments, albums
```

### TanStack Table Column Definition

```typescript
// components/tables/UsersTable.tsx
import { createColumnHelper } from '@tanstack/react-table';
import type { User } from '../../types';

const columnHelper = createColumnHelper<User>();

const columns = [
  columnHelper.accessor('name', {
    header: 'Name',
    cell: (info) => info.getValue(),
  }),
  // ... define semua kolom
];
```

---

## 📊 Grading

| Kriteria | Bobot | Detail |
|----------|-------|--------|
| **Functionality** | 30% | Semua fitur wajib berjalan, data fetch benar, routing works, search + sort + filter + pagination |
| **UI/UX** | 30% | Clean design, responsive, loading states, error handling, consistent spacing & typography |
| **Code Quality** | 20% | TypeScript strict, clean architecture, custom hooks, no `any`, proper error handling |
| **Bonus Features** | 20% | Lihat section bonus di bawah |

### Grading Breakdown

- **A (90-100):** Semua wajib + 3 bonus features, deployed, polished
- **B (75-89):** Semua wajib berjalan, 1-2 bonus, minor issues
- **C (60-74):** Most features work, beberapa bug, no bonus
- **D (<60):** Banyak fitur missing, banyak bug, not deployed

---

## ✨ Bonus Features (Poin Plus)

| Bonus | Poin | Detail |
|-------|------|--------|
| 🌙 **Dark Mode** | +5 | Toggle dark/light, persist ke localStorage, smooth transition |
| 📊 **Charts** | +5 | Pake **Recharts** — bar chart posts per user, pie chart users per city |
| 📥 **CSV Export** | +5 | Button "Export CSV" yang download current table data (filtered!) |
| 🔄 **Real-time Refresh** | +3 | Auto-refetch setiap 30 detik pake TanStack Query `refetchInterval` |
| 🔍 **Advanced Filters** | +2 | Multi-column filter, date range (kalo ada), combined filters |

---

## 📅 Submission

- **Submit:**
  1. GitHub repo link (public)
  2. Vercel deployment URL
  3. Screenshot/screen recording demo
- **Format repo name:** `arcane-dashboard-[nama-kalian]`

---

## ⚠️ Anti-Cheat

- Gua bakal review git history — harus ada minimal **10 commits** yang meaningful
- Copy-paste dari AI tanpa paham = langsung ketahuan pas live review
- Kalian harus bisa **explain setiap line** kalo gua tanya

---

## 💡 Tips dari Gua

1. **Mulai dari routing dulu** — setup semua routes, bikin placeholder pages
2. **Fetch data pake TanStack Query** — jangan pake useEffect + useState manual
3. **Bikin useDebounce sendiri** — ini interview question favorit, worth banget dipelajarin
4. **Table terakhir** — TanStack Table learning curve-nya lumayan, sisain waktu
5. **Deploy ASAP** — jangan tunggu selesai baru deploy. Deploy dari awal, update terus

> *"Dashboard ini bukan cuma tugas. Ini portfolio piece. Bikin yang kalian bangga tunjukin ke recruiter."*

---

**Selamat ngoding, Arcanists! 🔥**
