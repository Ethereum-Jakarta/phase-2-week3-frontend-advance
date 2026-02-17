# ⚡ Quiz — Performance Optimization

> *"Gua kasih kalian code yang LAMBAT. Tugas kalian: bikin CEPAT. Measure before/after. Buktiin pake angka, bukan feeling."*

**Difficulty: 🔴 HARD**

---

## Challenge 1: Unnecessary Re-renders — User Dashboard

### Buggy Code (SLOW)

Component ini re-render **semua child components** setiap kali clock ticks (setiap detik). **Fix it.**

```tsx
// ❌ SLOW — everything re-renders every second
import { useState, useEffect, createContext, useContext } from 'react';

interface AppState {
  user: { name: string; email: string };
  theme: 'light' | 'dark';
  notifications: number;
  currentTime: Date;
}

const AppContext = createContext<AppState | null>(null);

function AppProvider({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState<AppState>({
    user: { name: 'Budi', email: 'budi@mail.com' },
    theme: 'light',
    notifications: 3,
    currentTime: new Date(),
  });

  // Updates every second → EVERYTHING re-renders
  useEffect(() => {
    const interval = setInterval(() => {
      setState(prev => ({ ...prev, currentTime: new Date() }));
    }, 1000);
    return () => clearInterval(interval);
  }, []);

  return <AppContext.Provider value={state}>{children}</AppContext.Provider>;
}

function useAppState() {
  const ctx = useContext(AppContext);
  if (!ctx) throw new Error('Missing AppProvider');
  return ctx;
}

// All these re-render EVERY SECOND even though they don't use currentTime:
function UserProfile() {
  const { user } = useAppState();
  console.log('UserProfile rendered'); // logs every second! 🐛
  return <div>{user.name} - {user.email}</div>;
}

function ThemeToggle() {
  const { theme } = useAppState();
  console.log('ThemeToggle rendered'); // logs every second! 🐛
  return <button>{theme}</button>;
}

function NotificationBadge() {
  const { notifications } = useAppState();
  console.log('NotificationBadge rendered'); // logs every second! 🐛
  return <span>🔔 {notifications}</span>;
}

function Clock() {
  const { currentTime } = useAppState();
  console.log('Clock rendered'); // This SHOULD render every second ✅
  return <span>{currentTime.toLocaleTimeString()}</span>;
}

function Dashboard() {
  return (
    <AppProvider>
      <UserProfile />
      <ThemeToggle />
      <NotificationBadge />
      <Clock />
    </AppProvider>
  );
}
```

### Expected After Fix
```
Before: 4 components re-render every second = 240 renders/minute
After:  1 component (Clock) re-renders every second = 60 renders/minute

UserProfile: renders 1x (initial)
ThemeToggle: renders 1x (initial)  
NotificationBadge: renders 1x (initial)
Clock: renders 60x per minute (correct)
```

### Requirements
- Split context OR use selectors
- UserProfile, ThemeToggle, NotificationBadge MUST NOT re-render on clock tick
- Clock SHOULD re-render every second
- No external libraries (no zustand, no jotai)

### Test Cases
```
1. Clock updates every second ✓
2. UserProfile console.log fires only once on mount
3. ThemeToggle console.log fires only once on mount
4. After 10 seconds: Clock rendered 10x, others rendered 1x
5. Changing theme → only ThemeToggle re-renders (not others)
6. Changing notifications → only NotificationBadge re-renders
```

---

## Challenge 2: Expensive List — Comment Thread

### Buggy Code (SLOW)

1000 comments. Typing a reply causes ALL comments to re-render. Page freezes for 2 seconds.

```tsx
// ❌ SLOW — typing causes full list re-render
import { useState } from 'react';

interface Comment {
  id: string;
  author: string;
  text: string;
  timestamp: Date;
  likes: number;
}

function CommentThread() {
  const [comments, setComments] = useState<Comment[]>(generateComments(1000));
  const [replyText, setReplyText] = useState('');
  const [sortBy, setSortBy] = useState<'newest' | 'likes'>('newest');

  const sortedComments = comments.sort((a, b) => {   // 🐛 mutates original!
    if (sortBy === 'newest') return b.timestamp.getTime() - a.timestamp.getTime();
    return b.likes - a.likes;
  });

  const handleLike = (id: string) => {
    setComments(comments.map(c =>                      // 🐛 creates new array every time
      c.id === id ? { ...c, likes: c.likes + 1 } : c
    ));
  };

  const handleReply = () => {
    const newComment: Comment = {
      id: crypto.randomUUID(),
      author: 'You',
      text: replyText,
      timestamp: new Date(),
      likes: 0,
    };
    setComments([newComment, ...comments]);
    setReplyText('');
  };

  return (
    <div>
      <div>
        <textarea
          value={replyText}
          onChange={(e) => setReplyText(e.target.value)}  // 🐛 re-renders entire list
          placeholder="Write a reply..."
        />
        <button onClick={handleReply}>Reply</button>
      </div>

      <select value={sortBy} onChange={(e) => setSortBy(e.target.value as any)}>
        <option value="newest">Newest</option>
        <option value="likes">Most Liked</option>
      </select>

      {sortedComments.map((comment) => (
        <CommentItem
          key={comment.id}
          comment={comment}
          onLike={() => handleLike(comment.id)}      // 🐛 new function every render
        />
      ))}
    </div>
  );
}

function CommentItem({ comment, onLike }: { comment: Comment; onLike: () => void }) {
  console.log(`Rendering comment: ${comment.id}`);
  // Expensive render simulation
  const start = performance.now();
  while (performance.now() - start < 1) {}

  return (
    <div className="p-4 border-b">
      <div className="font-bold">{comment.author}</div>
      <p>{comment.text}</p>
      <div className="flex gap-2">
        <button onClick={onLike}>👍 {comment.likes}</button>
        <span>{comment.timestamp.toLocaleDateString()}</span>
      </div>
    </div>
  );
}
```

### Bugs to Fix
1. `.sort()` mutates state directly
2. Typing reply re-renders ALL 1000 comments
3. `onLike` creates new function reference every render
4. No memoization on sorted list
5. No virtualization (1000 DOM nodes)

### Expected After Fix
```
Before: Type 1 char → 1000 CommentItems re-render → 1+ second freeze
After:  Type 1 char → 0 CommentItems re-render → instant

Before: Like 1 comment → 1000 re-renders
After:  Like 1 comment → 1 re-render (only that comment)
```

### Test Cases
```
1. Typing in reply textarea → 0 CommentItem re-renders
2. Like comment #500 → only comment #500 re-renders
3. Change sort → list re-sorts (re-renders expected here)
4. Add reply → new comment appears at top, others don't re-render
5. .sort() does NOT mutate original comments array
6. Performance: typing feels instant (<16ms)
```

---

## Challenge 3: Unnecessary API Calls — Search with Cache

### Buggy Code (SLOW)

Search component that fires API call on EVERY keystroke, doesn't cache, and has race conditions.

```tsx
// ❌ SLOW — API call every keystroke, no cache, race condition
import { useState, useEffect } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<any[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [suggestions, setSuggestions] = useState<string[]>([]);

  // 🐛 Fires on EVERY keystroke
  useEffect(() => {
    if (!query) return;
    setIsLoading(true);

    fetch(`https://jsonplaceholder.typicode.com/posts?title_like=${query}`)
      .then(res => res.json())
      .then(data => {
        setResults(data);        // 🐛 Race condition — might be stale
        setIsLoading(false);
      });
  }, [query]);

  // 🐛 Also fires every keystroke, separate from main search
  useEffect(() => {
    if (!query) return;
    fetch(`https://jsonplaceholder.typicode.com/posts?title_like=${query}&_limit=5`)
      .then(res => res.json())
      .then(data => {
        setSuggestions(data.map((d: any) => d.title));
      });
  }, [query]);

  // 🐛 Renders results AND suggestions on every state change
  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      {isLoading && <p>Loading...</p>}
      <div className="suggestions">
        {suggestions.map((s, i) => (
          <div key={i} onClick={() => setQuery(s)}>{s}</div>
        ))}
      </div>
      <div className="results">
        {results.map((r: any) => (
          <ResultCard key={r.id} result={r} />
        ))}
      </div>
    </div>
  );
}

function ResultCard({ result }: { result: any }) {
  console.log('ResultCard rendered');
  return (
    <div className="p-4 border">
      <h3>{result.title}</h3>
      <p>{result.body?.substring(0, 100)}</p>
    </div>
  );
}
```

### Requirements
- Debounce search (300ms)
- Cancel previous requests (AbortController)
- Cache results (same query = no re-fetch for 60 seconds)
- Suggestions and results share the same fetch (not 2 separate calls)
- Race condition fixed
- ResultCard memoized

### Expected After Fix
```
Before: Type "react" (5 chars) → 5 API calls for search + 5 for suggestions = 10 API calls
After:  Type "react" (5 chars) → 1 API call (debounced, combined)

Before: Search "react" again → new API call
After:  Search "react" again → instant from cache
```

### Test Cases
```
1. Type "hello" fast → 1 API call (after debounce)
2. Type "hello" → wait → type "hello" again → 0 API calls (cached)
3. Type "hel" → "hello" → results are for "hello" not "hel" (no race condition)
4. Only 1 fetch call per debounce (not 2 separate for suggestions/results)
5. Cache expires after 60s → new API call
6. AbortController cancels in-flight requests
7. ResultCard only re-renders when result data changes
```

---

## Challenge 4: Heavy Computation — Data Processing Pipeline

### Buggy Code (SLOW)

Dashboard that processes 50,000 data points on every render. UI freezes for 3 seconds when changing any filter.

```tsx
// ❌ SLOW — recalculates everything on every render
import { useState } from 'react';

interface DataPoint {
  id: string;
  value: number;
  category: string;
  date: string;
  region: string;
}

function AnalyticsDashboard() {
  const [data] = useState<DataPoint[]>(generateData(50000));
  const [categoryFilter, setCategoryFilter] = useState<string>('all');
  const [regionFilter, setRegionFilter] = useState<string>('all');
  const [dateRange, setDateRange] = useState({ start: '2024-01-01', end: '2024-12-31' });
  const [chartType, setChartType] = useState<'bar' | 'line'>('bar');

  // 🐛 ALL of these recalculate on ANY state change (including chartType!)

  const filtered = data.filter(d => {
    const matchCategory = categoryFilter === 'all' || d.category === categoryFilter;
    const matchRegion = regionFilter === 'all' || d.region === regionFilter;
    const matchDate = d.date >= dateRange.start && d.date <= dateRange.end;
    return matchCategory && matchRegion && matchDate;
  });

  const totalValue = filtered.reduce((sum, d) => sum + d.value, 0);
  const average = totalValue / filtered.length || 0;

  const byCategory = filtered.reduce((acc, d) => {
    acc[d.category] = (acc[d.category] || 0) + d.value;
    return acc;
  }, {} as Record<string, number>);

  const byRegion = filtered.reduce((acc, d) => {
    acc[d.region] = (acc[d.region] || 0) + d.value;
    return acc;
  }, {} as Record<string, number>);

  const byMonth = filtered.reduce((acc, d) => {
    const month = d.date.substring(0, 7);
    acc[month] = (acc[month] || 0) + d.value;
    return acc;
  }, {} as Record<string, number>);

  const percentiles = calculatePercentiles(filtered.map(d => d.value)); // Very expensive

  return (
    <div>
      {/* Filter controls */}
      <select value={categoryFilter} onChange={e => setCategoryFilter(e.target.value)}>
        {/* categories */}
      </select>
      <select value={regionFilter} onChange={e => setRegionFilter(e.target.value)}>
        {/* regions */}
      </select>
      <select value={chartType} onChange={e => setChartType(e.target.value as any)}>
        <option value="bar">Bar</option>
        <option value="line">Line</option>
      </select>

      {/* Stats cards */}
      <div>Total: {totalValue}</div>
      <div>Average: {average.toFixed(2)}</div>
      <div>Count: {filtered.length}</div>

      {/* Charts — re-render on chartType change should be CHEAP */}
      <Chart type={chartType} data={byCategory} title="By Category" />
      <Chart type={chartType} data={byRegion} title="By Region" />
      <Chart type={chartType} data={byMonth} title="By Month" />

      {/* Percentiles */}
      <div>P50: {percentiles.p50} | P90: {percentiles.p90} | P99: {percentiles.p99}</div>
    </div>
  );
}
```

### Requirements
- `useMemo` with proper dependency arrays
- Changing `chartType` should NOT recalculate any data
- Changing `categoryFilter` should recalculate filtered + aggregates
- Consider `useDeferredValue` for non-urgent updates
- Split into smaller components to isolate re-renders

### Expected After Fix
```
Before: Change chartType → 3 second freeze (recalculates 50k data points)
After:  Change chartType → instant (0 recalculations)

Before: Change category filter → 3 seconds
After:  Change category filter → <100ms (memoized pipeline)
```

### Test Cases
```
1. Change chartType → 0 data recalculations (check with console.log)
2. Change categoryFilter → filtered + aggregates recalculate, percentiles recalculate
3. Change regionFilter → same as above
4. No filter change + parent re-render → 0 recalculations
5. Performance: filter change <100ms for 50k items
6. Percentile calculation only runs when filtered data changes
```

---

## Challenge 5: Memory Leak — Real-time Dashboard

### Buggy Code (MEMORY LEAK)

Dashboard that subscribes to WebSocket events. After 10 minutes, browser tab uses 2GB RAM and crashes.

```tsx
// ❌ MEMORY LEAK — grows unbounded
import { useState, useEffect } from 'react';

interface Event {
  id: string;
  type: string;
  data: any;
  timestamp: number;
}

function RealtimeDashboard() {
  const [events, setEvents] = useState<Event[]>([]);
  const [filter, setFilter] = useState<string>('all');
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    const ws = new WebSocket('wss://example.com/events');

    ws.onopen = () => setIsConnected(true);
    ws.onclose = () => setIsConnected(false);

    ws.onmessage = (msg) => {
      const event: Event = JSON.parse(msg.data);
      setEvents(prev => [...prev, event]);  // 🐛 Array grows FOREVER
    };

    // 🐛 Missing cleanup!
  }, []);

  // 🐛 Creates new filtered array on EVERY event (every 100ms)
  const filteredEvents = events.filter(e =>
    filter === 'all' || e.type === filter
  );

  // 🐛 Creates new chart data on EVERY event
  const chartData = events.reduce((acc, e) => {
    const minute = Math.floor(e.timestamp / 60000) * 60000;
    acc[minute] = (acc[minute] || 0) + 1;
    return acc;
  }, {} as Record<number, number>);

  // 🐛 Renders ALL events (no virtualization, no limit)
  return (
    <div>
      <div>Status: {isConnected ? '🟢' : '🔴'}</div>
      <div>Total events: {events.length}</div>

      <select value={filter} onChange={e => setFilter(e.target.value)}>
        <option value="all">All</option>
        <option value="error">Errors</option>
        <option value="warning">Warnings</option>
        <option value="info">Info</option>
      </select>

      <div style={{ height: 300 }}>
        {/* Chart */}
      </div>

      <div className="event-list">
        {filteredEvents.map(event => (
          <div key={event.id} className="p-2 border-b">
            <span className={`badge ${event.type}`}>{event.type}</span>
            <span>{JSON.stringify(event.data)}</span>
            <span>{new Date(event.timestamp).toLocaleTimeString()}</span>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Bugs to Fix
1. **Events array grows forever** — cap at 1000, use ring buffer
2. **No WebSocket cleanup** — close on unmount
3. **No reconnection logic** — handle disconnects
4. **Filter + chartData recalculates on every event** — memoize
5. **Renders ALL events** — virtualize or paginate
6. **JSON.stringify on every render** — memoize

### Requirements
- Ring buffer: keep last 1000 events max
- WebSocket cleanup on unmount
- Reconnect with exponential backoff
- `useMemo` for filtered events and chart data
- Virtual list or "show last 50" with "load more"
- `useRef` for WebSocket instance

### Expected After Fix
```
Before: 10 min @ 10 events/sec = 6000 events in memory, growing
After:  Always ≤ 1000 events, stable memory

Before: Every event → full re-render of all events
After:  Every event → only new event rendered (or batched updates)
```

### Test Cases
```
1. After 2000 events received → events.length === 1000 (ring buffer)
2. Unmount → WebSocket closed (check ws.readyState)
3. Server disconnect → auto-reconnect with backoff
4. Filter change → filtered list updates, chart doesn't recalculate
5. New event → chart recalculates, previous filtered array reused if filter unchanged
6. Memory stable after 10 minutes (no growth)
7. No "set state on unmounted component" warnings
```

---

## 📊 Scoring

| Challenge | Poin | Focus |
|-----------|------|-------|
| 1. Context Re-renders | 20 | Context splitting |
| 2. List Re-renders | 20 | memo, useCallback, useMemo |
| 3. API Call Optimization | 20 | Debounce, cache, abort |
| 4. Heavy Computation | 20 | useMemo, component splitting |
| 5. Memory Leak | 20 | Cleanup, ring buffer, reconnect |
| **TOTAL** | **100** | |

### Measurement Required

Untuk setiap challenge, tunjukkan:
```
BEFORE: X re-renders / Y ms per interaction / Z MB memory
AFTER:  X re-renders / Y ms per interaction / Z MB memory
```

Pake React DevTools Profiler atau `console.log` count.

---

**Optimize like your app depends on it. Because it does. ⚡🔧**
