# 🧩 Quiz — React Patterns

> *"Patterns itu yang bedain junior dev sama senior dev. Kalian harus tau kapan pake compound components, kapan render props, kapan HOC. 5 soal, semuanya HARD."*

**Difficulty: 🔴 HARD**

---

## Challenge 1: Compound Components — Accordion

Bikin `<Accordion>` compound component yang support single dan multiple expand modes.

### Expected API

```tsx
// Single mode — only one item open at a time
<Accordion mode="single" defaultOpen="item-1">
  <Accordion.Item id="item-1">
    <Accordion.Trigger>What is React?</Accordion.Trigger>
    <Accordion.Content>
      React is a JavaScript library for building user interfaces.
    </Accordion.Content>
  </Accordion.Item>
  <Accordion.Item id="item-2">
    <Accordion.Trigger>What is TypeScript?</Accordion.Trigger>
    <Accordion.Content>
      TypeScript is a typed superset of JavaScript.
    </Accordion.Content>
  </Accordion.Item>
</Accordion>

// Multiple mode — multiple items can be open
<Accordion mode="multiple" defaultOpen={["item-1", "item-3"]}>
  {/* ... */}
</Accordion>
```

### Requirements
- `mode="single"`: opening one closes others
- `mode="multiple"`: independent open/close
- `defaultOpen`: initial open item(s)
- `onChange`: callback with open item id(s)
- Keyboard: Enter/Space to toggle, Arrow Up/Down to navigate
- Animated expand/collapse (CSS transition on height)
- TypeScript: proper discriminated union for single vs multiple mode

### Test Cases
```
1. Single mode: open item-2 → item-1 closes
2. Multiple mode: open item-2 → item-1 stays open
3. defaultOpen="item-1" → item-1 starts open
4. Click trigger → toggles content visibility
5. Enter key on trigger → toggles
6. Arrow Down from trigger-1 → focuses trigger-2
7. onChange fires with correct open id(s)
8. Content animates open/close (not instant show/hide)
9. TypeScript: mode="single" + defaultOpen={["a","b"]} → type error
```

---

## Challenge 2: Render Props — Data Fetcher

Bikin `<DataFetcher>` component pake render props pattern yang reusable untuk any API.

### Expected API

```tsx
// Basic usage
<DataFetcher<User[]> url="/api/users">
  {({ data, isLoading, error, refetch }) => {
    if (isLoading) return <Spinner />;
    if (error) return <Error message={error.message} onRetry={refetch} />;
    return <UserList users={data} />;
  }}
</DataFetcher>

// With transform
<DataFetcher<Post[], { title: string; id: number }[]>
  url="/api/posts"
  transform={(posts) => posts.map(p => ({ title: p.title, id: p.id }))}
>
  {({ data }) => <SimpleList items={data} />}
</DataFetcher>

// With polling
<DataFetcher<Stats> url="/api/stats" pollInterval={5000}>
  {({ data, lastUpdated }) => (
    <div>
      <Stats data={data} />
      <p>Updated: {lastUpdated?.toLocaleTimeString()}</p>
    </div>
  )}
</DataFetcher>

// With dependent fetching
<DataFetcher<User> url={userId ? `/api/users/${userId}` : null}>
  {({ data }) => <UserProfile user={data} />}
</DataFetcher>
```

### Requirements
- Generic types: `DataFetcher<TData, TTransformed>`
- Props: `url`, `transform`, `pollInterval`, `headers`, `onSuccess`, `onError`
- Children as render prop with `{ data, isLoading, error, refetch, lastUpdated }`
- `url=null` → skip fetch (dependent fetching)
- Abort previous request on url change
- Cleanup polling on unmount

### Test Cases
```
1. Renders loading state initially
2. Renders data after successful fetch
3. Renders error on failed fetch
4. refetch() triggers new fetch
5. transform: data is transformed before passed to children
6. pollInterval: re-fetches every N ms
7. url change: aborts previous, fetches new
8. url=null: no fetch, data=undefined
9. Unmount during fetch: no state update (no warning)
10. TypeScript: data type matches generic parameter
```

---

## Challenge 3: Higher-Order Component — withAuth

Bikin HOC `withAuth` yang wraps any component dan handle authentication logic.

### Expected API

```tsx
// Basic protected component
const ProtectedDashboard = withAuth(Dashboard);

// With role requirement
const AdminPanel = withAuth(AdminPanelBase, { roles: ['admin'] });

// With custom redirect
const ProfilePage = withAuth(ProfileBase, {
  redirectTo: '/login',
  loadingComponent: <CustomSpinner />,
});

// Usage in routes
<Route path="/dashboard" element={<ProtectedDashboard />} />
<Route path="/admin" element={<AdminPanel />} />
```

### Implementation Requirements

```tsx
interface AuthOptions {
  roles?: string[];
  redirectTo?: string;
  loadingComponent?: React.ReactNode;
  fallbackComponent?: React.ReactNode;
}

function withAuth<P extends object>(
  WrappedComponent: React.ComponentType<P & { user: User }>,
  options?: AuthOptions
): React.FC<P> {
  // YOUR CODE
  // 1. Check if user is authenticated (use a useAuth hook)
  // 2. If loading → show loading component
  // 3. If not authenticated → redirect to login (or show fallback)
  // 4. If authenticated but wrong role → show "Access Denied"
  // 5. If authenticated + correct role → render WrappedComponent with user prop injected
  // 6. Display name for DevTools: `withAuth(ComponentName)`
}
```

### Also implement `useAuth` hook:

```tsx
function useAuth(): {
  user: User | null;
  isLoading: boolean;
  isAuthenticated: boolean;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => Promise<void>;
} {
  // Mock implementation OK for quiz
}
```

### Test Cases
```
1. Unauthenticated → redirects to /login
2. Authenticated → renders wrapped component
3. Authenticated + wrong role → shows "Access Denied"
4. Authenticated + correct role → renders component
5. Loading state → shows loading component
6. Wrapped component receives `user` prop
7. All original props of WrappedComponent are preserved
8. DevTools shows `withAuth(Dashboard)` not `withAuth(Anonymous)`
9. TypeScript: WrappedComponent props are correctly inferred minus `user`
```

---

## Challenge 4: Custom Hook — useForm

Bikin `useForm` hook yang handle validation, dirty tracking, submission, dan field-level errors.

### Expected API

```tsx
function RegistrationForm() {
  const {
    values,
    errors,
    touched,
    isDirty,
    isValid,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    setFieldValue,
    setFieldError,
    reset,
  } = useForm({
    initialValues: {
      username: '',
      email: '',
      password: '',
      confirmPassword: '',
    },
    validate: (values) => {
      const errors: Record<string, string> = {};
      if (!values.username) errors.username = 'Username required';
      if (values.username.length < 3) errors.username = 'Min 3 characters';
      if (!values.email.includes('@')) errors.email = 'Invalid email';
      if (values.password.length < 8) errors.password = 'Min 8 characters';
      if (values.password !== values.confirmPassword) {
        errors.confirmPassword = 'Passwords do not match';
      }
      return errors;
    },
    onSubmit: async (values) => {
      await api.register(values);
    },
  });

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="username"
        value={values.username}
        onChange={handleChange}
        onBlur={handleBlur}
      />
      {touched.username && errors.username && <span>{errors.username}</span>}
      {/* ... more fields */}
      <button type="submit" disabled={!isValid || isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Register'}
      </button>
      <button type="button" onClick={reset}>Reset</button>
    </form>
  );
}
```

### Requirements
- Type-safe: field names inferred from initialValues
- `validate` runs on blur and on submit
- `touched`: tracks which fields user has interacted with
- `isDirty`: true if any value differs from initialValues
- `handleChange`: works with native input onChange
- `handleBlur`: marks field as touched, triggers validation
- `handleSubmit`: prevents default, validates all, calls onSubmit if valid
- `setFieldValue`: programmatic value set
- `setFieldError`: programmatic error set (e.g., server validation)
- `reset`: back to initial state
- `isSubmitting`: true during async onSubmit

### Test Cases
```
1. Initial state: values=initialValues, no errors, not dirty, not touched
2. Type in username → values.username updates, isDirty=true
3. Blur username (empty) → touched.username=true, errors.username shown
4. Valid username → errors.username cleared
5. Submit with errors → onSubmit NOT called, all fields marked touched
6. Submit with valid data → isSubmitting=true, onSubmit called
7. After successful submit → isSubmitting=false
8. setFieldError('email', 'Already taken') → errors.email set
9. reset() → back to initial state, errors cleared
10. TypeScript: values.nonexistent → type error
```

---

## Challenge 5: Custom Hook + Compound — Modal System

Bikin reusable modal system yang combines custom hook + compound components.

### Expected API

```tsx
// Hook for controlling modals
function App() {
  const confirmDelete = useModal();
  const editProfile = useModal();

  return (
    <div>
      <button onClick={confirmDelete.open}>Delete Account</button>
      <button onClick={() => editProfile.open({ userId: 123 })}>Edit Profile</button>

      <Modal modal={confirmDelete}>
        <Modal.Header>
          <Modal.Title>Confirm Delete</Modal.Title>
          <Modal.Close />
        </Modal.Header>
        <Modal.Body>
          Are you sure? This cannot be undone.
        </Modal.Body>
        <Modal.Footer>
          <button onClick={confirmDelete.close}>Cancel</button>
          <button onClick={handleDelete}>Delete</button>
        </Modal.Footer>
      </Modal>

      <Modal modal={editProfile}>
        <Modal.Header>
          <Modal.Title>Edit Profile</Modal.Title>
          <Modal.Close />
        </Modal.Header>
        <Modal.Body>
          {/* Access data passed to open() */}
          <EditForm userId={editProfile.data?.userId} />
        </Modal.Body>
      </Modal>
    </div>
  );
}
```

### Requirements for `useModal`

```tsx
interface UseModalReturn<T = unknown> {
  isOpen: boolean;
  data: T | null;
  open: (data?: T) => void;
  close: () => void;
  toggle: () => void;
}
```

### Requirements for `<Modal>`
- Portal rendering (`createPortal` to document.body)
- Backdrop click to close
- Escape key to close
- Focus trap (Tab cycles within modal)
- Scroll lock on body when open
- Animated enter/exit
- Compound: Header, Body, Footer, Title, Close are sub-components
- Multiple modals can be stacked (z-index management)

### Test Cases
```
1. open() → modal visible, backdrop shown
2. close() → modal hidden
3. open({ userId: 123 }) → data accessible as modal.data
4. Click backdrop → closes
5. Press Escape → closes
6. Tab key → focus stays within modal (focus trap)
7. Body scroll locked when modal open
8. Two modals open → second one on top (higher z-index)
9. Close second → first still visible
10. Renders in portal (not in parent DOM tree)
11. Animated entrance and exit
```

---

## 📊 Scoring

| Challenge | Poin | Pattern |
|-----------|------|---------|
| 1. Accordion | 20 | Compound Components |
| 2. DataFetcher | 20 | Render Props |
| 3. withAuth | 20 | Higher-Order Component |
| 4. useForm | 20 | Custom Hook |
| 5. Modal System | 20 | Custom Hook + Compound |
| **TOTAL** | **100** | |

---

**Master the patterns, master React. 🧩⚡**
