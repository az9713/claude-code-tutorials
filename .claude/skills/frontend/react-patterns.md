---
name: react-patterns
description: Modern React patterns, hooks, and best practices
---

# React Patterns & Best Practices

## Component Patterns

### Functional Components with TypeScript

```tsx
interface UserCardProps {
  user: User;
  onEdit?: (user: User) => void;
  className?: string;
}

export function UserCard({ user, onEdit, className }: UserCardProps) {
  return (
    <div className={cn('user-card', className)}>
      <h2>{user.name}</h2>
      {onEdit && <button onClick={() => onEdit(user)}>Edit</button>}
    </div>
  );
}
```

### Compound Components

```tsx
// Parent provides context
const TabsContext = createContext<TabsContextValue | null>(null);

function Tabs({ children, defaultTab }: TabsProps) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  );
}

// Children consume context
function TabList({ children }: { children: ReactNode }) {
  return <div role="tablist">{children}</div>;
}

function Tab({ id, children }: TabProps) {
  const { activeTab, setActiveTab } = useTabsContext();
  return (
    <button
      role="tab"
      aria-selected={activeTab === id}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
}

// Usage
<Tabs defaultTab="profile">
  <TabList>
    <Tab id="profile">Profile</Tab>
    <Tab id="settings">Settings</Tab>
  </TabList>
  <TabPanel id="profile">Profile content</TabPanel>
  <TabPanel id="settings">Settings content</TabPanel>
</Tabs>
```

### Render Props (When Needed)

```tsx
interface MouseTrackerProps {
  children: (position: { x: number; y: number }) => ReactNode;
}

function MouseTracker({ children }: MouseTrackerProps) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handler = (e: MouseEvent) => setPosition({ x: e.clientX, y: e.clientY });
    window.addEventListener('mousemove', handler);
    return () => window.removeEventListener('mousemove', handler);
  }, []);

  return <>{children(position)}</>;
}
```

## Hook Patterns

### Custom Hooks

```tsx
// Data fetching hook
function useQuery<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const controller = new AbortController();

    fetch(url, { signal: controller.signal })
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));

    return () => controller.abort();
  }, [url]);

  return { data, error, loading };
}

// Debounced value hook
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Local storage hook
function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}
```

## State Management

### useReducer for Complex State

```tsx
type State = { count: number; step: number };
type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'setStep'; step: number };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment': return { ...state, count: state.count + state.step };
    case 'decrement': return { ...state, count: state.count - state.step };
    case 'setStep': return { ...state, step: action.step };
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0, step: 1 });
  // ...
}
```

### Context + useReducer Pattern

```tsx
const StateContext = createContext<State | null>(null);
const DispatchContext = createContext<Dispatch<Action> | null>(null);

function AppProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}

// Separate hooks for state and dispatch
function useAppState() {
  const context = useContext(StateContext);
  if (!context) throw new Error('useAppState must be used within AppProvider');
  return context;
}

function useAppDispatch() {
  const context = useContext(DispatchContext);
  if (!context) throw new Error('useAppDispatch must be used within AppProvider');
  return context;
}
```

## Performance Patterns

### Memoization

```tsx
// Memoize expensive computations
const sortedItems = useMemo(
  () => items.sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);

// Memoize callbacks
const handleClick = useCallback(
  (id: string) => dispatch({ type: 'select', id }),
  [dispatch]
);

// Memoize components
const MemoizedList = memo(function List({ items }: ListProps) {
  return items.map(item => <Item key={item.id} {...item} />);
});
```

### Code Splitting

```tsx
// Lazy load components
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

## Error Boundaries

```tsx
class ErrorBoundary extends Component<Props, State> {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? <DefaultErrorUI error={this.state.error} />;
    }
    return this.props.children;
  }
}
```
