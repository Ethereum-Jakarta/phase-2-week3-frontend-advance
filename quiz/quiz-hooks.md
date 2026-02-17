# 🪝 Quiz — Advanced Hooks

> *"Hooks bukan cuma useState sama useEffect. Kalian HARUS master useRef, useContext, useCallback, useMemo, useReducer. Ini 8 soal HARD. Good luck."*

**Difficulty: 🔴 HARD**

---

## Challenge 1: useRef — Stopwatch (Focus: Mutable Ref)

Bikin stopwatch yang bisa start, stop, reset. **TANPA menyebabkan re-render saat timer berjalan.**

### Starter Code

```tsx
import { useState } from 'react';

export function Stopwatch() {
  const [time, setTime] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  // ❌ Problem: setInterval causes re-render every 10ms
  // ❌ Clearing interval on stop doesn't work reliably

  const start = () => {
    setIsRunning(true);
    setInterval(() => {
      setTime((t) => t + 10);
    }, 10);
  };

  const stop = () => {
    setIsRunning(false);
    // How to clear the interval? 🤔
  };

  const reset = () => {
    setTime(0);
    setIsRunning(false);
  };

  return (
    <div>
      <p>{formatTime(time)}</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

function formatTime(ms: number): string {
  const minutes = Math.floor(ms / 60000);
  const seconds = Math.floor((ms % 60000) / 1000);
  const centiseconds = Math.floor((ms % 1000) / 10);
  return `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}.${String(centiseconds).padStart(2, '0')}`;
}
```

### Requirements
- Use `useRef` to store interval ID
- Start/stop/reset work correctly
- No memory leaks — cleanup interval on unmount
- Display updates every 10ms

### Test Cases
```
1. Click Start → timer increments
2. Click Stop → timer stops at current value
3. Click Start again → timer resumes from stopped value
4. Click Reset → timer goes to 00:00.00
5. Multiple Start clicks → only one interval running (no speed-up bug)
6. Unmount component → interval cleared (no console errors)
```

### Expected Output
```
Display: 00:05.30
Start → runs → Stop at 00:05.30 → Start → continues from 00:05.30
Reset → 00:00.00
```

---

## Challenge 2: useRef — Previous Value Tracker

Bikin hook `usePrevious<T>` yang return nilai sebelumnya dari sebuah value.

### Starter Code

```tsx
// Implement this:
function usePrevious<T>(value: T): T | undefined {
  // YOUR CODE — must use useRef
  // Must NOT cause extra re-render
}

// Usage:
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount ?? 'N/A'}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
```

### Test Cases
```
1. Initial render: current=0, previous=undefined
2. After first increment: current=1, previous=0
3. After second increment: current=2, previous=1
4. Works with any type T (string, object, etc.)
5. Does NOT cause extra re-render
```

---

## Challenge 3: useContext — Multi-language System

Bikin i18n system pake Context yang support multiple languages dan nested translations.

### Starter Code

```tsx
// translations.ts
const translations = {
  en: {
    greeting: 'Hello, {{name}}!',
    nav: {
      home: 'Home',
      about: 'About',
      contact: 'Contact',
    },
    form: {
      submit: 'Submit',
      cancel: 'Cancel',
      errors: {
        required: '{{field}} is required',
        minLength: '{{field}} must be at least {{min}} characters',
      },
    },
  },
  id: {
    greeting: 'Halo, {{name}}!',
    nav: {
      home: 'Beranda',
      about: 'Tentang',
      contact: 'Kontak',
    },
    form: {
      submit: 'Kirim',
      cancel: 'Batal',
      errors: {
        required: '{{field}} wajib diisi',
        minLength: '{{field}} minimal {{min}} karakter',
      },
    },
  },
};

// Implement:
// 1. I18nProvider — wraps app, provides current language + setter
// 2. useTranslation() hook — returns { t, language, setLanguage }
// 3. t('nav.home') → 'Home' or 'Beranda'
// 4. t('greeting', { name: 'Budi' }) → 'Halo, Budi!'
// 5. t('form.errors.required', { field: 'Email' }) → 'Email wajib diisi'
```

### Test Cases
```
1. t('nav.home') with lang='en' → 'Home'
2. t('nav.home') with lang='id' → 'Beranda'
3. t('greeting', { name: 'Budi' }) → 'Halo, Budi!'
4. t('form.errors.minLength', { field: 'Password', min: '8' }) → 'Password minimal 8 karakter'
5. t('nonexistent.key') → returns the key itself as fallback
6. setLanguage('id') → all components using t() re-render with Indonesian
7. TypeScript: t() autocompletes valid keys (bonus)
```

---

## Challenge 4: useCallback — Stable Callbacks in List

1000 item list. Parent re-render TIDAK boleh cause semua items re-render.

### Starter Code

```tsx
// ❌ Every item re-renders when parent state changes
function TodoList() {
  const [todos, setTodos] = useState<Todo[]>(generateTodos(1000));
  const [filter, setFilter] = useState('');

  const toggleTodo = (id: string) => {
    setTodos(prev => prev.map(t =>
      t.id === id ? { ...t, done: !t.done } : t
    ));
  };

  const deleteTodo = (id: string) => {
    setTodos(prev => prev.filter(t => t.id !== id));
  };

  const filtered = todos.filter(t =>
    t.text.toLowerCase().includes(filter.toLowerCase())
  );

  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      {filtered.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={toggleTodo}     // ❌ new function every render
          onDelete={deleteTodo}     // ❌ new function every render
        />
      ))}
    </div>
  );
}

function TodoItem({ todo, onToggle, onDelete }: TodoItemProps) {
  console.log(`Rendering: ${todo.id}`); // This should NOT log for every item
  return (
    <div>
      <input type="checkbox" checked={todo.done} onChange={() => onToggle(todo.id)} />
      <span>{todo.text}</span>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </div>
  );
}
```

### Requirements
- Fix dengan `useCallback` + `React.memo`
- Toggling 1 todo → only THAT todo re-renders
- Typing in filter → items re-render (expected, filtered list changes)
- Delete → only removed item unmounts, others don't re-render

### Test Cases
```
1. Initial render: 1000 TodoItems render
2. Toggle todo #500 → only todo #500 re-renders (check console.log count)
3. Type in filter → filtered items render (expected)
4. Parent re-render from unrelated state → 0 TodoItems re-render
5. Performance: toggling a todo takes <16ms
```

---

## Challenge 5: useMemo — Expensive Computation

Kalkulator statistik yang compute mean, median, mode, stddev dari dataset besar. HARUS di-memoize.

### Starter Code

```tsx
function StatsDashboard() {
  const [data, setData] = useState<number[]>(generateRandomData(10000));
  const [highlightThreshold, setHighlightThreshold] = useState(50);

  // ❌ These recalculate on EVERY render (even when just changing threshold)
  const mean = data.reduce((a, b) => a + b, 0) / data.length;
  const sorted = [...data].sort((a, b) => a - b);
  const median = sorted[Math.floor(sorted.length / 2)];
  const mode = findMode(data); // expensive
  const stddev = Math.sqrt(
    data.reduce((sum, val) => sum + Math.pow(val - mean, 2), 0) / data.length
  );

  // This also recalculates everything
  const aboveThreshold = data.filter(d => d > highlightThreshold);

  return (
    <div>
      <input
        type="range" min={0} max={100}
        value={highlightThreshold}
        onChange={e => setHighlightThreshold(Number(e.target.value))}
      />
      <p>Mean: {mean.toFixed(2)}</p>
      <p>Median: {median}</p>
      <p>Mode: {mode}</p>
      <p>StdDev: {stddev.toFixed(2)}</p>
      <p>Above {highlightThreshold}: {aboveThreshold.length}</p>
      <button onClick={() => setData(generateRandomData(10000))}>
        New Dataset
      </button>
    </div>
  );
}
```

### Requirements
- Stats (mean, median, mode, stddev) ONLY recalculate when `data` changes
- `aboveThreshold` recalculates when `data` OR `highlightThreshold` changes
- Moving the slider should feel instant (no lag from stats recalc)
- Implement `findMode` that handles multi-modal data

### Test Cases
```
1. Initial render: all stats calculated correctly
2. Move threshold slider → stats DO NOT recalculate (console.log proof)
3. Move threshold slider → aboveThreshold DOES recalculate
4. Click "New Dataset" → all stats recalculate
5. Performance: slider movement <16ms per frame
6. findMode([1,2,2,3,3]) → [2, 3] (multi-modal)
```

---

## Challenge 6: useReducer — Shopping Cart with Discounts

Cart system yang handle add, remove, update quantity, apply discount codes, dan calculate totals.

### Starter Code

```tsx
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartState {
  items: CartItem[];
  discountCode: string | null;
  discountPercent: number;
}

type CartAction =
  | { type: 'ADD_ITEM'; item: Omit<CartItem, 'quantity'> }
  | { type: 'REMOVE_ITEM'; id: string }
  | { type: 'UPDATE_QUANTITY'; id: string; quantity: number }
  | { type: 'APPLY_DISCOUNT'; code: string }
  | { type: 'REMOVE_DISCOUNT' }
  | { type: 'CLEAR_CART' };

const DISCOUNT_CODES: Record<string, number> = {
  'SAVE10': 10,
  'SAVE20': 20,
  'HALF': 50,
};

// Implement:
// 1. cartReducer function
// 2. Cart component using useReducer
// 3. Computed values: subtotal, discount amount, total
// 4. ADD_ITEM: if item already exists, increment quantity (don't duplicate)
// 5. UPDATE_QUANTITY: if quantity = 0, remove item
// 6. APPLY_DISCOUNT: validate code, reject invalid
```

### Test Cases
```
1. ADD_ITEM (new) → item added with quantity 1
2. ADD_ITEM (existing) → quantity incremented
3. REMOVE_ITEM → item removed
4. UPDATE_QUANTITY to 3 → quantity = 3
5. UPDATE_QUANTITY to 0 → item removed
6. APPLY_DISCOUNT "SAVE20" → discountPercent = 20
7. APPLY_DISCOUNT "INVALID" → no change (code rejected)
8. Subtotal = sum(price * quantity)
9. Total = subtotal * (1 - discountPercent/100)
10. CLEAR_CART → empty items, remove discount
```

---

## Challenge 7: useContext + useReducer — Notification System

Global notification system (toast-style) yang bisa dipanggil dari mana aja.

### Expected API

```tsx
// Anywhere in the app:
function SomeComponent() {
  const { notify } = useNotifications();

  const handleSave = async () => {
    try {
      await saveData();
      notify({ type: 'success', message: 'Data saved!' });
    } catch {
      notify({ type: 'error', message: 'Failed to save', duration: 5000 });
    }
  };
}

// Auto-dismiss after duration (default 3000ms)
// Max 5 notifications visible at once (oldest removed)
// Can manually dismiss
// Types: success (green), error (red), warning (yellow), info (blue)
```

### Requirements
- `NotificationProvider` wraps the app
- `useNotifications()` hook returns `{ notify, dismiss, notifications }`
- useReducer for state management
- Auto-dismiss with configurable duration
- Animation on enter/exit (CSS transition OK)
- Stack from top-right corner

### Test Cases
```
1. notify({ type: 'success', message: 'Done' }) → notification appears
2. After 3000ms → notification auto-dismissed
3. notify with duration: 5000 → dismissed after 5s
4. 6 notifications → oldest one removed (max 5)
5. Click dismiss → notification removed immediately
6. Multiple rapid notifications → all appear, auto-dismiss independently
7. Unmount component that called notify → no errors (timer cleanup)
```

---

## Challenge 8: Combined — useRef + useCallback + useMemo — Virtual List

Bikin virtualized list yang cuma render items yang visible di viewport. 10,000 items tapi cuma ~20 DOM nodes.

### Starter Code

```tsx
// Implement useVirtualList hook
function useVirtualList<T>({
  items,
  itemHeight,
  containerHeight,
  overscan = 3,
}: {
  items: T[];
  itemHeight: number;
  containerHeight: number;
  overscan?: number;
}): {
  virtualItems: { index: number; item: T; offsetTop: number }[];
  totalHeight: number;
  containerRef: React.RefObject<HTMLDivElement>;
} {
  // YOUR CODE
  // Use useRef for scroll position (no re-render on scroll)
  // Use useMemo for computing visible items
  // Use useCallback for scroll handler
  // requestAnimationFrame for smooth scrolling
}

// Usage:
function VirtualList() {
  const items = Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    text: `Item ${i}`,
  }));

  const { virtualItems, totalHeight, containerRef } = useVirtualList({
    items,
    itemHeight: 40,
    containerHeight: 600,
    overscan: 5,
  });

  return (
    <div ref={containerRef} style={{ height: 600, overflow: 'auto' }}>
      <div style={{ height: totalHeight, position: 'relative' }}>
        {virtualItems.map(({ index, item, offsetTop }) => (
          <div
            key={index}
            style={{
              position: 'absolute',
              top: offsetTop,
              height: 40,
              width: '100%',
            }}
          >
            {item.text}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Test Cases
```
1. 10,000 items → only ~25 DOM nodes rendered (visible + overscan)
2. Scroll down → items update, smooth scrolling
3. Scroll to bottom → last items visible
4. No re-render on scroll (useRef for scroll position)
5. virtualItems recalculates only when scroll position changes significantly
6. Performance: 60fps while scrolling
7. Overscan: items slightly outside viewport are pre-rendered
8. Resize container → visible items update correctly
```

---

## 📊 Scoring

| Challenge | Poin | Hooks Tested |
|-----------|------|-------------|
| 1. Stopwatch | 10 | useRef |
| 2. usePrevious | 10 | useRef |
| 3. i18n System | 15 | useContext |
| 4. Stable Callbacks | 12 | useCallback, React.memo |
| 5. Stats Dashboard | 12 | useMemo |
| 6. Shopping Cart | 13 | useReducer |
| 7. Notifications | 13 | useContext + useReducer |
| 8. Virtual List | 15 | useRef + useCallback + useMemo |
| **TOTAL** | **100** | |

---

**Buktiin kalian master hooks, Arcanists. 🪝⚡**
