---
name: performance
description: Frontend performance optimization techniques
---

# Frontend Performance Optimization

## Core Web Vitals

| Metric | Good | Description |
|--------|------|-------------|
| **LCP** (Largest Contentful Paint) | < 2.5s | Time until largest content element is visible |
| **INP** (Interaction to Next Paint) | < 200ms | Responsiveness to user interactions |
| **CLS** (Cumulative Layout Shift) | < 0.1 | Visual stability (no unexpected shifts) |

## Loading Performance

### Code Splitting

```tsx
// Route-based splitting
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

// Component-based splitting
const HeavyChart = lazy(() => import('./components/HeavyChart'));

// Preload critical routes
const DashboardPreload = lazy(() =>
  import(/* webpackPreload: true */ './pages/Dashboard')
);
```

### Image Optimization

```tsx
// Modern image formats with fallback
<picture>
  <source srcSet="image.avif" type="image/avif" />
  <source srcSet="image.webp" type="image/webp" />
  <img src="image.jpg" alt="Description" loading="lazy" />
</picture>

// Responsive images
<img
  src="image-800.jpg"
  srcSet="image-400.jpg 400w, image-800.jpg 800w, image-1200.jpg 1200w"
  sizes="(max-width: 600px) 400px, (max-width: 1200px) 800px, 1200px"
  alt="Description"
  loading="lazy"
  decoding="async"
/>

// Next.js Image component
import Image from 'next/image';
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={600}
  priority // Above the fold
  placeholder="blur"
  blurDataURL={blurHash}
/>
```

### Resource Hints

```html
<!-- Preconnect to critical origins -->
<link rel="preconnect" href="https://api.example.com" />
<link rel="preconnect" href="https://fonts.googleapis.com" crossorigin />

<!-- Prefetch likely navigation -->
<link rel="prefetch" href="/dashboard" />

<!-- Preload critical resources -->
<link rel="preload" href="/fonts/main.woff2" as="font" type="font/woff2" crossorigin />
<link rel="preload" href="/critical.css" as="style" />
```

## Runtime Performance

### Memoization

```tsx
// Memoize expensive calculations
const expensiveResult = useMemo(() => {
  return items.reduce((acc, item) => {
    return acc + heavyComputation(item);
  }, 0);
}, [items]);

// Memoize callbacks to prevent child re-renders
const handleClick = useCallback((id: string) => {
  setSelected(id);
}, []);

// Memoize components
const MemoizedList = memo(function ItemList({ items }: Props) {
  return items.map(item => <Item key={item.id} {...item} />);
}, (prev, next) => {
  // Custom comparison
  return prev.items.length === next.items.length;
});
```

### Virtualization

```tsx
// For long lists, render only visible items
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
    overscan: 5,
  });

  return (
    <div ref={parentRef} style={{ height: 400, overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.key}
            style={{
              position: 'absolute',
              top: virtualRow.start,
              height: virtualRow.size,
            }}
          >
            {items[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Debouncing & Throttling

```tsx
// Debounce: Wait until user stops typing
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
const [search, setSearch] = useState('');
const debouncedSearch = useDebounce(search, 300);

useEffect(() => {
  fetchResults(debouncedSearch);
}, [debouncedSearch]);
```

### Web Workers

```tsx
// Offload heavy computation to worker
const worker = new Worker(new URL('./worker.ts', import.meta.url));

worker.postMessage({ data: largeDataset });
worker.onmessage = (e) => {
  setResult(e.data);
};

// worker.ts
self.onmessage = (e) => {
  const result = heavyComputation(e.data);
  self.postMessage(result);
};
```

## Bundle Optimization

### Tree Shaking

```tsx
// ❌ Imports entire library
import _ from 'lodash';
_.debounce(fn, 300);

// ✅ Import only what you need
import debounce from 'lodash/debounce';
debounce(fn, 300);

// ✅ Or use lodash-es for better tree shaking
import { debounce } from 'lodash-es';
```

### Analyze Bundle

```bash
# Next.js
ANALYZE=true npm run build

# Webpack
npx webpack-bundle-analyzer stats.json

# Vite
npx vite-bundle-visualizer
```

## Performance Monitoring

```tsx
// Measure component render time
import { Profiler } from 'react';

function onRender(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number
) {
  console.log(`${id} ${phase}: ${actualDuration}ms`);
}

<Profiler id="ExpensiveComponent" onRender={onRender}>
  <ExpensiveComponent />
</Profiler>

// Web Vitals
import { onCLS, onINP, onLCP } from 'web-vitals';

onCLS(console.log);
onINP(console.log);
onLCP(console.log);
```

## Performance Checklist

- [ ] Bundle size < 200KB (gzipped)
- [ ] LCP < 2.5s
- [ ] INP < 200ms
- [ ] CLS < 0.1
- [ ] Images optimized (WebP/AVIF, lazy loaded)
- [ ] Fonts preloaded, display: swap
- [ ] No layout shifts from dynamic content
- [ ] Long lists virtualized
- [ ] Heavy computations memoized or in workers
