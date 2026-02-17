# 🧪 Arcane Quest 05 — Week 3 UJIAN

> *"Ini bukan latihan lagi. Ini ujian. 8 soal coding. Ada starter code, ada test cases, ada expected output. Kalian punya 3 jam. Good luck."*

## 📋 Rules

- **Durasi:** 3 jam
- **Open book:** Boleh buka dokumentasi resmi (React docs, MDN, TanStack docs)
- **NO ChatGPT/Copilot** — gua bisa bedain code AI vs code manusia
- **TypeScript WAJIB** — kalo pake `any` tanpa alasan, poin dikurangi
- **Setiap soal ada test cases** — code kalian harus pass semua tests

---

## Soal 1: Optimize Re-renders (15 poin)

Component ini re-render **47 kali** saat user ketik di search box. **Optimize sampai cuma 2 re-renders** (1 untuk debounce update, 1 untuk filtered list).

### Starter Code

```tsx
// ❌ BROKEN — 47 re-renders per keystroke
import { useState } from 'react';

interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
}

const products: Product[] = [
  { id: 1, name: 'Laptop', price: 999, category: 'Electronics' },
  { id: 2, name: 'Phone', price: 699, category: 'Electronics' },
  { id: 3, name: 'Shirt', price: 29, category: 'Clothing' },
  { id: 4, name: 'Pants', price: 49, category: 'Clothing' },
  { id: 5, name: 'Book', price: 15, category: 'Books' },
  // ... imagine 100+ items
];

function ExpensiveProductCard({ product }: { product: Product }) {
  console.log(`Rendering product: ${product.name}`); // This logs 47 times!
  // Simulate expensive render
  const start = performance.now();
  while (performance.now() - start < 2) {} // artificial 2ms delay
  return (
    <div className="p-4 border rounded">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <p>{product.category}</p>
    </div>
  );
}

function CategoryFilter({ onFilter }: { onFilter: (cat: string) => void }) {
  console.log('CategoryFilter re-rendered');
  return (
    <select onChange={(e) => onFilter(e.target.value)}>
      <option value="">All</option>
      <option value="Electronics">Electronics</option>
      <option value="Clothing">Clothing</option>
      <option value="Books">Books</option>
    </select>
  );
}

export function ProductSearch() {
  const [search, setSearch] = useState('');
  const [category, setCategory] = useState('');

  const filtered = products.filter((p) => {
    const matchSearch = p.name.toLowerCase().includes(search.toLowerCase());
    const matchCategory = category === '' || p.category === category;
    return matchSearch && matchCategory;
  });

  return (
    <div>
      <input
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search products..."
      />
      <CategoryFilter onFilter={setCategory} />
      <div className="grid grid-cols-3 gap-4">
        {filtered.map((p) => (
          <ExpensiveProductCard key={p.id} product={p} />
        ))}
      </div>
    </div>
  );
}
```

### Expected Result
- `ExpensiveProductCard` hanya re-render kalo datanya BERUBAH
- `CategoryFilter` TIDAK re-render saat search berubah
- Search di-debounce 300ms
- Total re-renders per keystroke: **2** (bukan 47)

### Test Cases
```
1. Type "lap" quickly → ExpensiveProductCard renders only ONCE after debounce
2. Change category → only matching products re-render
3. CategoryFilter does NOT re-render when typing in search
4. Performance: 100 products, typing should feel instant (<16ms per frame)
```

---

## Soal 2: Compound Components — Tabs API (15 poin)

Bikin compound component `<Tabs>` yang punya API seperti ini:

### Expected Usage

```tsx
function App() {
  return (
    <Tabs defaultTab="tab1">
      <Tabs.List>
        <Tabs.Tab id="tab1">Profile</Tabs.Tab>
        <Tabs.Tab id="tab2">Settings</Tabs.Tab>
        <Tabs.Tab id="tab3">Notifications</Tabs.Tab>
      </Tabs.List>
      <Tabs.Panels>
        <Tabs.Panel id="tab1">
          <p>Profile content here</p>
        </Tabs.Panel>
        <Tabs.Panel id="tab2">
          <p>Settings content here</p>
        </Tabs.Panel>
        <Tabs.Panel id="tab3">
          <p>Notifications content here</p>
        </Tabs.Panel>
      </Tabs.Panels>
    </Tabs>
  );
}
```

### Requirements
- Active tab highlighted (CSS class `active`)
- Only active panel visible
- Keyboard navigation: Arrow Left/Right to switch tabs
- `onChange` callback prop on `<Tabs>`
- TypeScript: semua props properly typed, no `any`

### Test Cases
```
1. Default tab renders correct panel
2. Clicking tab switches panel
3. Arrow Right from tab1 → activates tab2
4. Arrow Left from tab1 → wraps to tab3 (circular)
5. onChange fires with correct tab id
6. Only one panel visible at a time
7. TypeScript: passing invalid id shows type error
```

---

## Soal 3: useInfiniteScroll Hook (12 poin)

Bikin custom hook `useInfiniteScroll` yang automatically fetch next page saat user scroll ke bawah.

### Starter Code

```tsx
// Implement this hook
function useInfiniteScroll<T>(
  fetchFn: (page: number) => Promise<{ data: T[]; hasMore: boolean }>,
  options?: { threshold?: number }
): {
  data: T[];
  isLoading: boolean;
  isFetchingMore: boolean;
  error: Error | null;
  sentinelRef: React.RefObject<HTMLDivElement>;
} {
  // YOUR CODE HERE
}

// Usage:
function PostList() {
  const { data, isLoading, isFetchingMore, error, sentinelRef } = useInfiniteScroll(
    async (page) => {
      const res = await fetch(`https://jsonplaceholder.typicode.com/posts?_page=${page}&_limit=10`);
      const posts = await res.json();
      return { data: posts, hasMore: posts.length === 10 };
    },
    { threshold: 200 }
  );

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <div>
      {data.map((post: any) => (
        <div key={post.id}>{post.title}</div>
      ))}
      <div ref={sentinelRef} />
      {isFetchingMore && <p>Loading more...</p>}
    </div>
  );
}
```

### Requirements
- Use `IntersectionObserver` (bukan scroll event listener)
- Prevent duplicate fetches
- Handle loading, error, and empty states
- `threshold` option: pixels before bottom to trigger fetch
- Cleanup observer on unmount

### Test Cases
```
1. Initial load fetches page 1
2. Scrolling to sentinel triggers page 2 fetch
3. Does NOT fetch page 2 twice (race condition safe)
4. Stops fetching when hasMore === false
5. Error on fetch → shows error, allows retry
6. Unmount → no memory leak (observer disconnected)
```

---

## Soal 4: Fix the Race Condition (12 poin)

Kode ini punya **race condition**. User ketik "react" tapi hasil yang muncul buat "rea" (request sebelumnya yang balik duluan). **Fix it.**

### Buggy Code

```tsx
// ❌ RACE CONDITION
import { useState, useEffect } from 'react';

interface SearchResult {
  id: number;
  title: string;
}

async function searchAPI(query: string): Promise<SearchResult[]> {
  // Simulate variable latency
  const delay = Math.random() * 2000;
  await new Promise((r) => setTimeout(r, delay));
  const res = await fetch(
    `https://jsonplaceholder.typicode.com/posts?title_like=${query}`
  );
  return res.json();
}

export function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<SearchResult[]>([]);
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    if (!query) {
      setResults([]);
      return;
    }
    setIsLoading(true);
    searchAPI(query).then((data) => {
      setResults(data);  // 🐛 This might set stale results!
      setIsLoading(false);
    });
  }, [query]);

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      {isLoading && <p>Searching...</p>}
      {results.map((r) => <div key={r.id}>{r.title}</div>)}
    </div>
  );
}
```

### Fix Requirements
- Results ALWAYS match the current query (no stale data)
- Provide **2 different solutions:**
  1. Using `useEffect` cleanup (AbortController)
  2. Using `useRef` to track latest request
- Bonus: add debounce

### Test Cases
```
1. Type "r" → "re" → "rea" → "reac" → "react" quickly
   → Results shown are ALWAYS for "react" (the latest query)
2. Slow response for "re" arrives after fast response for "react"
   → "re" results are DISCARDED
3. AbortController version: previous requests are actually cancelled
4. No "set state on unmounted component" warnings
```

---

## Soal 5: Debounced Search with TanStack Query (12 poin)

Implement debounced search yang pake TanStack Query properly.

### Starter Code

```tsx
// Implement this component
// Requirements:
// 1. Search input with 500ms debounce
// 2. TanStack Query for fetching (useQuery)
// 3. Show loading, error, empty states
// 4. Cancel previous query when new one starts
// 5. Cache results — searching same term again should be instant
// 6. Stale time: 30 seconds

import { useQuery } from '@tanstack/react-query';

// YOUR IMPLEMENTATION:
// - useDebounce hook (custom, bukan library)
// - SearchComponent that uses useQuery with debounced value
// - Proper queryKey strategy
// - enabled: only when debouncedQuery.length >= 2
```

### Expected Behavior

```tsx
// User types "react hooks"
// Timeline:
// t=0ms:    user types "r"       → no query (length < 2)
// t=100ms:  user types "re"      → no query yet (debouncing)
// t=200ms:  user types "rea"     → no query yet (debouncing)
// t=700ms:  debounce fires       → query for "rea" starts
// t=800ms:  user types "reac"    → debounce resets
// t=1300ms: debounce fires       → query for "reac" starts, "rea" query cancelled
// t=1500ms: user types "react"   → debounce resets
// t=2000ms: debounce fires       → query for "react" starts
// t=2500ms: results for "react" displayed
// t=5000ms: user clears and types "react" again → INSTANT (from cache!)
```

### Test Cases
```
1. Typing fast → only final debounced value triggers fetch
2. Query not enabled for strings shorter than 2 chars
3. Same search term → served from cache (no network request)
4. Loading state shown while fetching
5. Error state with retry button
6. Cache invalidates after 30 seconds
```

---

## Soal 6: useReducer — Complex Form State (10 poin)

Bikin multi-step form pake `useReducer` yang handle complex state transitions.

### Starter Code

```tsx
// Form with 3 steps:
// Step 1: Personal Info (name, email, phone)
// Step 2: Address (street, city, zip, country)
// Step 3: Review & Submit

// State shape:
interface FormState {
  step: 1 | 2 | 3;
  data: {
    name: string;
    email: string;
    phone: string;
    street: string;
    city: string;
    zip: string;
    country: string;
  };
  errors: Record<string, string>;
  isSubmitting: boolean;
  isComplete: boolean;
}

// Actions — define these:
type FormAction =
  | { type: 'UPDATE_FIELD'; field: string; value: string }
  | { type: 'NEXT_STEP' }
  | { type: 'PREV_STEP' }
  | { type: 'SET_ERRORS'; errors: Record<string, string> }
  | { type: 'SUBMIT_START' }
  | { type: 'SUBMIT_SUCCESS' }
  | { type: 'SUBMIT_ERROR'; error: string }
  | { type: 'RESET' };

// Implement the reducer and the MultiStepForm component
// Validation:
// - Step 1: name required, email must be valid, phone must be digits
// - Step 2: street required, city required, zip must be 5 digits
// - Cannot go to next step if current step has errors
```

### Test Cases
```
1. Initial state: step 1, all fields empty, no errors
2. NEXT_STEP with empty name → errors.name = "Name is required"
3. NEXT_STEP with valid step 1 → moves to step 2
4. PREV_STEP from step 2 → back to step 1, data preserved
5. SUBMIT_START → isSubmitting = true
6. SUBMIT_SUCCESS → isComplete = true, step stays at 3
7. RESET → back to initial state
8. UPDATE_FIELD clears error for that field
```

---

## Soal 7: Context + Performance — Theme System (12 poin)

Bikin theme system yang TIDAK cause unnecessary re-renders.

### The Problem

```tsx
// ❌ This causes ALL consumers to re-render when ANY value changes
const ThemeContext = createContext({
  theme: 'light',
  fontSize: 16,
  toggleTheme: () => {},
  setFontSize: (size: number) => {},
});
```

### Your Task

Implement a theme system where:
1. Changing theme does NOT re-render components that only use fontSize
2. Changing fontSize does NOT re-render components that only use theme
3. Use **context splitting** or **selector pattern**

### Requirements

```tsx
// These components should ONLY re-render when their specific value changes:

function ThemeDisplay() {
  const theme = useTheme(); // only re-renders when theme changes
  console.log('ThemeDisplay rendered');
  return <div>Current theme: {theme}</div>;
}

function FontDisplay() {
  const fontSize = useFontSize(); // only re-renders when fontSize changes
  console.log('FontDisplay rendered');
  return <div>Font size: {fontSize}px</div>;
}

function ThemeControls() {
  const toggleTheme = useToggleTheme(); // stable reference, never re-renders
  const setFontSize = useSetFontSize(); // stable reference, never re-renders
  console.log('ThemeControls rendered');
  return (
    <div>
      <button onClick={toggleTheme}>Toggle Theme</button>
      <button onClick={() => setFontSize(20)}>Big Font</button>
    </div>
  );
}
```

### Test Cases
```
1. Toggle theme → ThemeDisplay re-renders, FontDisplay does NOT
2. Change fontSize → FontDisplay re-renders, ThemeDisplay does NOT
3. ThemeControls NEVER re-renders (stable function references)
4. Initial render: all 3 components render once
5. Theme persists to localStorage
6. TypeScript: useTheme() outside provider throws helpful error
```

---

## Soal 8: Build a Data Table Hook (12 poin)

Bikin `useDataTable` hook yang encapsulate sorting, filtering, pagination, dan selection.

### Expected API

```tsx
function UserTable() {
  const {
    data,           // current page data (sorted + filtered)
    columns,        // column definitions with sort state
    sorting,        // { column: string, direction: 'asc' | 'desc' } | null
    filters,        // active filters
    pagination,     // { page, pageSize, totalPages, totalItems }
    selection,      // Set<string> of selected row ids
    setSorting,     // (column: string) => void — toggles asc/desc/none
    setFilter,      // (column: string, value: string) => void
    setPage,        // (page: number) => void
    setPageSize,    // (size: number) => void
    toggleSelect,   // (id: string) => void
    selectAll,      // () => void
    clearSelection, // () => void
    isAllSelected,  // boolean
  } = useDataTable({
    data: users,
    columns: [
      { key: 'name', header: 'Name', sortable: true, filterable: true },
      { key: 'email', header: 'Email', sortable: true, filterable: true },
      { key: 'role', header: 'Role', sortable: true, filterable: true },
      { key: 'status', header: 'Status', sortable: false, filterable: true },
    ],
    pageSize: 10,
    getId: (row) => row.id,
  });

  return (
    <table>
      <thead>
        <tr>
          <th>
            <input
              type="checkbox"
              checked={isAllSelected}
              onChange={selectAll}
            />
          </th>
          {columns.map((col) => (
            <th key={col.key} onClick={() => col.sortable && setSorting(col.key)}>
              {col.header} {sorting?.column === col.key && (sorting.direction === 'asc' ? '↑' : '↓')}
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((row) => (
          <tr key={row.id}>
            <td>
              <input
                type="checkbox"
                checked={selection.has(row.id)}
                onChange={() => toggleSelect(row.id)}
              />
            </td>
            {columns.map((col) => (
              <td key={col.key}>{row[col.key]}</td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

### Test Cases
```
1. Initial render: shows first page, no sorting, no filters
2. setSorting('name') → data sorted by name asc
3. setSorting('name') again → sorted by name desc
4. setSorting('name') third time → no sorting
5. setFilter('role', 'admin') → only admin rows shown
6. Multiple filters stack: filter role + filter status
7. setPage(2) → shows page 2 data
8. Pagination updates when filter reduces total items
9. selectAll → selects all items on CURRENT PAGE (not all data)
10. Selection persists across page changes
11. TypeScript: column keys must match data object keys
```

---

## 📊 Scoring Summary

| Soal | Poin | Topic |
|------|------|-------|
| 1. Optimize Re-renders | 15 | useMemo, useCallback, React.memo |
| 2. Compound Tabs | 15 | Compound components, Context, keyboard nav |
| 3. useInfiniteScroll | 12 | Custom hooks, IntersectionObserver |
| 4. Race Condition | 12 | useEffect cleanup, AbortController |
| 5. Debounced TanStack Query | 12 | TanStack Query, debounce, caching |
| 6. useReducer Form | 10 | useReducer, complex state |
| 7. Theme Context | 12 | Context splitting, performance |
| 8. Data Table Hook | 12 | Complex custom hook |
| **TOTAL** | **100** | |

### Grade Scale
- **S (95-100):** Legendary. Semua soal perfect.
- **A (85-94):** Excellent. Minor issues only.
- **B (70-84):** Good. Beberapa soal incomplete tapi approach benar.
- **C (60-69):** Passing. Core concepts understood, execution rough.
- **F (<60):** Need more practice.

---

## ⏱️ Time Management Tips

- Soal 1, 4, 5 → paling cepet kalo udah paham konsep (masing-masing ~15-20 min)
- Soal 2, 7 → medium (masing-masing ~25 min)
- Soal 3, 6, 8 → paling lama (masing-masing ~30 min)
- Sisain **15 menit** buat review dan cleanup

> *"Ini ujian paling hardcore di Phase 2. Tapi kalo kalian beneran belajar selama 3 minggu ini, kalian pasti bisa. Trust the process. 💪"*

---

**Show me what you've learned, Arcanists. ⚡🔥**
