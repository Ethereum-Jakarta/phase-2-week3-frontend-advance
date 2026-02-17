# ⚡ useCallback & useMemo: Performance Optimization yang Sering Disalahgunakan

## Cerita Dulu: Senior Developer yang Salah Kaprah

Di kantor lama gue, ada senior dev — sebut aja Bang Rudi. Bang Rudi wrap **SEMUA function** pakai `useCallback` dan **SEMUA value** pakai `useMemo`. Alasannya: "Biar performant bro."

Code review jadi nightmare:

```typescript
// Bang Rudi's code 💀
const handleClick = useCallback(() => {
  console.log('clicked')
}, [])

const title = useMemo(() => 'Hello World', [])

const className = useMemo(() => `text-lg ${isActive ? 'text-blue-500' : 'text-gray-500'}`, [isActive])
```

Gue tanya: "Bang, ini perlu di-memo gak sih?"

"Perlu lah, biar gak re-create tiap render."

**WRONG.** `useCallback` dan `useMemo` itu **bukan gratis**. Mereka punya cost. Dan kalau dipake tanpa alasan, malah bikin app **lebih lambat**.

Hari ini gue bakal jelasin kapan PERLU dan kapan JANGAN pakai dua hook ini.

---

## Prerequisite: Referential Equality

Sebelum masuk ke hooks, kalian HARUS paham konsep ini dulu.

```typescript
// Primitives: compared by VALUE
'hello' === 'hello' // true
42 === 42 // true

// Objects/Arrays/Functions: compared by REFERENCE
{ name: 'Budi' } === { name: 'Budi' } // FALSE! Different objects in memory
[1, 2, 3] === [1, 2, 3] // FALSE!
(() => {}) === (() => {}) // FALSE!
```

Setiap kali component re-render, **semua function dan object di dalamnya di-create ulang**. Mereka identical secara value, tapi **beda reference**.

```typescript
function MyComponent() {
  // Setiap render, function baru dibuat di memory
  const handleClick = () => console.log('click') // Render 1: ref A
  // Re-render → handleClick dibuat lagi              Render 2: ref B
  // Re-render → handleClick dibuat lagi              Render 3: ref C
  // A !== B !== C (beda reference!)
}
```

**Kenapa ini masalah?** Karena React pakai referential equality buat decide apakah props berubah.

---

## React.memo: The Missing Piece

`useCallback` dan `useMemo` **gak berguna** tanpa `React.memo`. Ini kuncinya.

`React.memo` wrap component supaya dia **SKIP re-render kalau props-nya sama**.

```typescript
import { memo, useState } from 'react'

// Tanpa memo: re-render setiap kali parent re-render
function ExpensiveList({ items }: { items: string[] }) {
  console.log('ExpensiveList rendered!') // Log setiap render
  return (
    <ul>
      {items.map((item, i) => (
        <li key={i} className="p-2 border-b">{item}</li>
      ))}
    </ul>
  )
}

// Dengan memo: skip re-render kalau `items` reference sama
const MemoizedList = memo(ExpensiveList)

function Parent() {
  const [count, setCount] = useState(0)
  const items = ['React', 'Vue', 'Angular'] // ⚠️ NEW array setiap render!
  
  return (
    <div className="p-4">
      <button onClick={() => setCount(c => c + 1)} className="bg-blue-500 text-white p-2 rounded">
        Count: {count}
      </button>
      <MemoizedList items={items} /> {/* Still re-renders! items is new array each time */}
    </div>
  )
}
```

**Expected console output saat klik button:**
```
ExpensiveList rendered!  // masih re-render karena `items` baru tiap render
```

`React.memo` gak membantu karena `items` itu **array baru setiap render** (referential equality fail). Di sinilah `useMemo` masuk.

---

## useMemo: Memoize Values

```typescript
import { memo, useState, useMemo } from 'react'

const MemoizedList = memo(function ExpensiveList({ items }: { items: string[] }) {
  console.log('ExpensiveList rendered!')
  return (
    <ul>
      {items.map((item, i) => <li key={i} className="p-2 border-b">{item}</li>)}
    </ul>
  )
})

function Parent() {
  const [count, setCount] = useState(0)
  
  // ✅ useMemo: same reference kalau deps gak berubah
  const items = useMemo(() => ['React', 'Vue', 'Angular'], [])
  
  return (
    <div className="p-4">
      <button onClick={() => setCount(c => c + 1)} className="bg-blue-500 text-white p-2 rounded">
        Count: {count}
      </button>
      <MemoizedList items={items} /> {/* ✅ Skips re-render! */}
    </div>
  )
}
```

**Expected console output:** `ExpensiveList rendered!` cuma muncul **SEKALI** saat mount. Klik button berkali-kali? Gak ada log tambahan.

### useMemo untuk Expensive Computation

```typescript
import { useState, useMemo } from 'react'

function FilteredProducts() {
  const [search, setSearch] = useState('')
  const [sortBy, setSortBy] = useState<'name' | 'price'>('name')
  
  // Simulate 10,000 products
  const allProducts = useMemo(() => 
    Array.from({ length: 10000 }, (_, i) => ({
      id: i,
      name: `Product ${i}`,
      price: Math.floor(Math.random() * 1000000),
      category: ['Electronics', 'Fashion', 'Food'][i % 3],
    }))
  , [])
  
  // ✅ Expensive computation: filter + sort 10K items
  const filteredProducts = useMemo(() => {
    console.log('Filtering and sorting...') // See when this runs
    
    const filtered = allProducts.filter(p =>
      p.name.toLowerCase().includes(search.toLowerCase())
    )
    
    return filtered.sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name)
      return a.price - b.price
    })
  }, [allProducts, search, sortBy]) // Only recompute when these change
  
  return (
    <div className="p-4">
      <input
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search products..."
        className="border p-2 rounded w-full mb-4"
      />
      <select
        value={sortBy}
        onChange={(e) => setSortBy(e.target.value as 'name' | 'price')}
        className="border p-2 rounded mb-4"
      >
        <option value="name">Sort by Name</option>
        <option value="price">Sort by Price</option>
      </select>
      <p className="mb-2">Showing {filteredProducts.length} of {allProducts.length}</p>
      <ul className="max-h-96 overflow-auto">
        {filteredProducts.slice(0, 50).map(p => (
          <li key={p.id} className="p-2 border-b flex justify-between">
            <span>{p.name}</span>
            <span>Rp {p.price.toLocaleString('id-ID')}</span>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

---

## useCallback: Memoize Functions

`useCallback` itu `useMemo` tapi khusus buat functions.

```typescript
// Ini identical:
const fn = useCallback(() => doSomething(a, b), [a, b])
const fn = useMemo(() => () => doSomething(a, b), [a, b])
```

### Real Use Case: Preventing Child Re-renders

```typescript
import { memo, useState, useCallback } from 'react'

interface SearchBarProps {
  onSearch: (query: string) => void
}

const SearchBar = memo(function SearchBar({ onSearch }: SearchBarProps) {
  console.log('SearchBar rendered!')
  return (
    <input
      onChange={(e) => onSearch(e.target.value)}
      placeholder="Search..."
      className="border p-2 rounded w-full"
    />
  )
})

function ProductPage() {
  const [search, setSearch] = useState('')
  const [cart, setCart] = useState<string[]>([])
  
  // ❌ Without useCallback: SearchBar re-renders when cart changes
  // const handleSearch = (query: string) => setSearch(query)
  
  // ✅ With useCallback: SearchBar stays memoized
  const handleSearch = useCallback((query: string) => {
    setSearch(query)
  }, [])
  
  return (
    <div className="p-4">
      <SearchBar onSearch={handleSearch} />
      <p>Searching: {search}</p>
      <button 
        onClick={() => setCart(c => [...c, 'item'])}
        className="bg-green-500 text-white p-2 rounded mt-2"
      >
        Add to Cart ({cart.length})
      </button>
    </div>
  )
}
```

**Tanpa useCallback:** Klik "Add to Cart" → `handleSearch` dibuat ulang → `SearchBar` re-render (sia-sia!)

**Dengan useCallback:** Klik "Add to Cart" → `handleSearch` reference sama → `SearchBar` skip re-render ✅

---

## ❌ Kapan JANGAN Pakai

### 1. Memoize Primitives

```typescript
// ❌ USELESS — string comparison is already fast
const title = useMemo(() => 'Hello', [])

// ✅ Just use it directly
const title = 'Hello'
```

### 2. Function yang Gak di-pass ke Memoized Child

```typescript
function MyComponent() {
  // ❌ USELESS — no memo'd child receives this
  const handleClick = useCallback(() => {
    console.log('clicked')
  }, [])
  
  // ✅ Just use normal function
  const handleClick = () => console.log('clicked')
  
  return <button onClick={handleClick}>Click</button>
  // <button> is a DOM element, not a memo'd component
}
```

### 3. Cheap Computations

```typescript
// ❌ USELESS — this is already instant
const fullName = useMemo(() => `${firstName} ${lastName}`, [firstName, lastName])

// ✅ Just compute it
const fullName = `${firstName} ${lastName}`
```

### The Rule

> **useMemo/useCallback berguna HANYA kalau:**
> 1. Value/function di-pass ke `React.memo` child, ATAU
> 2. Value di-pass ke dependency array hook lain, ATAU
> 3. Computation genuinely expensive (loop 1000+ items, complex math)

---

## Measuring with React DevTools Profiler

Jangan guess. **Measure.**

1. Install React DevTools browser extension
2. Open DevTools → Profiler tab
3. Click Record → interact with app → Stop
4. Look at flame chart

```
How to read the Profiler:
┌─────────────────────────────────────────────┐
│ Component          │ Render Time │ Why?      │
├────────────────────┼─────────────┼───────────┤
│ App                │ 2.3ms       │ State     │
│ ├─ Header          │ 0.1ms       │ Props     │
│ ├─ ProductList     │ 15.2ms ⚠️   │ Props     │ ← Masalah!
│ │  ├─ ProductCard  │ 0.5ms       │ Props     │
│ │  ├─ ProductCard  │ 0.5ms       │ Props     │
│ │  └─ ... (100x)   │             │           │
│ └─ Footer          │ 0.1ms       │ -         │
└─────────────────────────────────────────────┘
```

**Steps:**
1. Lihat component mana yang paling lama render
2. Check "Why did this render?" di Profiler settings
3. Kalau "Props changed" tapi seharusnya gak berubah → pakai `useMemo/useCallback`
4. Kalau render time < 1ms → **DON'T bother memoizing**

### Highlight Updates

Di React DevTools → Settings → ✅ "Highlight updates when components render"

Sekarang setiap re-render bakal ada **green flash** di UI. Kalau lo liat component yang gak seharusnya re-render tapi flash, itu target optimization lo.

---

## Complete Example: Optimized Todo App

```typescript
import { memo, useState, useCallback, useMemo } from 'react'

interface Todo {
  id: number
  text: string
  completed: boolean
}

// Memoized child component
const TodoItem = memo(function TodoItem({
  todo,
  onToggle,
  onDelete,
}: {
  todo: Todo
  onToggle: (id: number) => void
  onDelete: (id: number) => void
}) {
  console.log(`TodoItem ${todo.id} rendered`)
  
  return (
    <li className="flex items-center gap-3 p-3 border-b">
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
        className="w-5 h-5"
      />
      <span className={`flex-1 ${todo.completed ? 'line-through text-gray-400' : ''}`}>
        {todo.text}
      </span>
      <button
        onClick={() => onDelete(todo.id)}
        className="text-red-500 hover:text-red-700"
      >
        🗑️
      </button>
    </li>
  )
})

// Stats component — only re-renders when todos actually change
const TodoStats = memo(function TodoStats({ todos }: { todos: Todo[] }) {
  console.log('TodoStats rendered')
  
  const stats = useMemo(() => ({
    total: todos.length,
    completed: todos.filter(t => t.completed).length,
    pending: todos.filter(t => !t.completed).length,
  }), [todos])
  
  return (
    <div className="flex gap-4 p-4 bg-gray-100 rounded mb-4">
      <span>Total: {stats.total}</span>
      <span>✅ Done: {stats.completed}</span>
      <span>⏳ Pending: {stats.pending}</span>
    </div>
  )
})

function TodoApp() {
  const [todos, setTodos] = useState<Todo[]>([
    { id: 1, text: 'Belajar useCallback', completed: false },
    { id: 2, text: 'Belajar useMemo', completed: false },
    { id: 3, text: 'Build project Week 3', completed: false },
  ])
  const [input, setInput] = useState('')
  
  // ✅ Stable function references for memo'd children
  const handleToggle = useCallback((id: number) => {
    setTodos(prev => prev.map(t => 
      t.id === id ? { ...t, completed: !t.completed } : t
    ))
  }, [])
  
  const handleDelete = useCallback((id: number) => {
    setTodos(prev => prev.filter(t => t.id !== id))
  }, [])
  
  const handleAdd = () => {
    if (!input.trim()) return
    setTodos(prev => [...prev, {
      id: Date.now(),
      text: input.trim(),
      completed: false,
    }])
    setInput('')
  }
  
  return (
    <div className="max-w-md mx-auto p-4">
      <h1 className="text-2xl font-bold mb-4">📝 Todo App (Optimized)</h1>
      
      <TodoStats todos={todos} />
      
      <div className="flex gap-2 mb-4">
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyDown={(e) => e.key === 'Enter' && handleAdd()}
          placeholder="Add todo..."
          className="border p-2 rounded flex-1"
        />
        <button onClick={handleAdd} className="bg-blue-500 text-white px-4 rounded">
          Add
        </button>
      </div>
      
      <ul>
        {todos.map(todo => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={handleToggle}
            onDelete={handleDelete}
          />
        ))}
      </ul>
    </div>
  )
}

export default TodoApp
```

**Expected behavior:** Ketik di input field → cuma `TodoApp` re-render (karena `input` state). `TodoItem` dan `TodoStats` **SKIP** re-render. Check console — gak ada log dari child components saat typing.

---

## Cheat Sheet

| Scenario | Solution |
|----------|----------|
| Expensive array/object passed as prop | `useMemo` + `React.memo` on child |
| Callback passed to memo'd child | `useCallback` + `React.memo` on child |
| Heavy computation (sort 10K items) | `useMemo` |
| Simple string concatenation | ❌ Don't memoize |
| Inline function on `<button>` | ❌ Don't memoize |
| Everything, just in case | ❌ STOP IT BANG RUDI |

---

## Latihan

1. Buat app dengan parent + 5 child components. Measure renders with/without memo
2. Create a component that filters + sorts 10K items. Optimize with useMemo
3. Use React DevTools Profiler to find the slowest component in your app
4. Refactor Bang Rudi's code: remove unnecessary memoization

---

**Next Part -> [05 - useReducer](https://github.com/Ethereum-Jakarta/phase-2-week3-frontend-advance/blob/main/study-material/05-usereducer.md)**
