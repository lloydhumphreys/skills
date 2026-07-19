---
name: react-performance-profiler
description: Profile and optimize React render performance using React DevTools Profiler JSON exports. Use when the user asks about slow renders, React profiling, commit duration analysis, re-render reduction, or provides a profiler JSON export for analysis.
metadata:
  version: "1.0.0"
  tags:
    - react
    - performance
    - profiling
---

# React Performance Profiling and Optimization

A systematic approach to profiling React applications and reducing unnecessary renders. Covers reading React Profiler JSON exports, identifying bottlenecks from the data, and applying targeted optimizations.

## When to Use

- The user provides a React Profiler JSON export and wants analysis
- The user reports slow renders, jank, or dropped frames in a React app
- Commit durations exceed 16ms (one frame budget at 60fps)
- The user asks how to find or fix unnecessary re-renders

**Important:** Always profile a production build (`npm run build` then serve locally) or at minimum disable StrictMode. Dev mode adds double-renders, extra warnings, and slower code paths that inflate commit durations and can point optimizations in the wrong direction.

## When Not to Use

- Server-side rendering or data-fetching performance (network/IO bound)
- CSS animation or layout thrashing not caused by React commits
- Bundle size optimization (unrelated to render performance)

## Required Inputs

For profiler analysis: a React DevTools Profiler JSON export (`.json` file exported via the Profiler tab gear icon > "Save profile").

For optimization work: access to the component source code where bottlenecks are identified.

## Reading React Profiler JSON Exports

### JSON Structure

```
{
  "version": 5,
  "dataForRoots": [
    {
      "rootID": 1,
      "commitData": [
        {
          "duration": 367.7,             // Total commit time in ms
          "effectDuration": 0.5,         // Time spent in useEffect callbacks
          "passiveEffectDuration": 1.2,  // Time in passive (useEffect) cleanup + run
          "priorityLevel": "Normal",     // React scheduler priority
          "timestamp": 1234.5,           // Relative time since profiling started (ms)
          "fiberActualDurations": [
            [fiberId, duration],          // Per-component render time
            ...
          ],
          "fiberSelfDurations": [
            [fiberId, duration],          // Self-time excluding children
            ...
          ],
          "updaters": [                  // What triggered this commit
            {
              "displayName": "MyComponent",
              "id": 474,
              "type": 8
            }
          ],
          "changeDescriptions": null     // Often null; when present: [[fiberId, { hooks, props }]]
        }
      ],
      "snapshots": [                     // Full fiber tree at time of profiling
        [fiberId, {
          "id": 1,
          "children": [2, 3],
          "displayName": "App",
          "type": 5
        }]
      ]
    }
  ]
}
```

**Notes on the structure:**
- `changeDescriptions` is frequently `null` in real exports. Do not depend on it — use `fiberActualDurations` and `updaters` as the primary analysis inputs.
- `effectDuration` and `passiveEffectDuration` help identify commits where useEffect/useLayoutEffect callbacks are the bottleneck rather than the render phase itself.
- `timestamp` is relative to profiling start, useful for correlating commits with user interactions (e.g., a cluster of commits starting at the same timestamp indicates a single interaction).

### Key Metrics to Extract

1. **Commit durations** — `commitData[].duration`. Sort to get max, p95, median.
2. **Fiber count per commit** — `commitData[].fiberActualDurations.length`. How many components re-rendered in each commit.
3. **Top self-time components** — Sort `fiberSelfDurations` descending within the slowest commits. These are your optimization targets.
4. **Total tree size** — `snapshots.length`. The total fiber count in the component tree.
5. **Commit triggers** — `commitData[].updaters[].displayName`. Which component's state change caused each commit.
6. **Effect cost** — `effectDuration` + `passiveEffectDuration`. If these are high relative to `duration`, the bottleneck is in effects rather than the render phase.
7. **Change reasons** — `changeDescriptions`, when non-null, tells you whether a component re-rendered due to changed hooks, props, or parent re-render. This field is frequently `null` in practice — don't depend on it.

### Analyzing with Coding Agent

For a quick summary:

> "Read this React Profiler JSON export. Extract all commit durations, then compute: max, p95, median, total commits, and the number of fibers rendered in the largest commit."

For deeper investigation:

> "For the 5 slowest commits, list the top 10 components by self-time, what triggered each commit (updaters), and the change descriptions."

For comparing before/after:

> "Compare these two profiler exports. Show a table of max, p95, median commit duration and total fiber tree size for each."

## Optimization Techniques

Six techniques, ordered by typical impact. Each targets a different layer of the problem.

### 1. Push State Down / Isolate State Consumers

**Problem:** `useState` or `useContext` in a component high in the tree causes the entire subtree to re-render on every state change.

**Pattern:** Move state reads into the smallest component that needs them. Extract state-dependent UI into leaf components. Parent components should be stable.

```tsx
// BEFORE: State in parent → entire tree re-renders on selection change
function Dashboard() {
  const [selected, setSelected] = useState(null)
  return (
    <div>
      <ExpensiveList onSelect={setSelected} /> {/* re-renders unnecessarily */}
      <Detail item={selected} />
    </div>
  )
}

// AFTER: Selection state co-located with the components that use it.
// ExpensiveList and SelectionDetail share state via a wrapper that
// doesn't render expensive siblings.
function Dashboard() {
  return (
    <div>
      <ExpensiveList />
      <SelectionPanel /> {/* owns selection state; siblings unaffected */}
    </div>
  )
}

function SelectionPanel() {
  const [selected, setSelected] = useState(null)
  return <Detail item={selected} onClear={() => setSelected(null)} />
}
```

The key constraint: don't break data flow. If two components need the same state, co-locate them under a shared wrapper that doesn't also contain expensive unrelated siblings. This applies equally to context consumers — if only one component needs a context value, only that component should subscribe to it.

### 2. Wrapper/Inner Memo Split

**Problem:** A component re-renders frequently (e.g., subscribed to a context or store) but most of its JSX is expensive and doesn't depend on the changing value.

**Pattern:** Split into a thin wrapper that handles the subscription and a `memo`'d inner component for the expensive content.

```tsx
function ItemWrapper({ item }) {
  // This re-renders frequently due to context subscription
  const { transform } = useSomeContextHook(item.id)
  const style = { transform }
  return (
    <div style={style}>
      <ItemInner item={item} />
    </div>
  )
}

const ItemInner = memo(function ItemInner({ item }) {
  // Expensive: store selectors, computed values, complex JSX
  return <div>{/* ... */}</div>
})
```

**Key:** The inner component only re-renders when its own props change by reference. The wrapper re-renders cheaply since it just computes a style object.

### 3. Reduce Fiber Tree Size

**Problem:** Large component trees are expensive even when individual components are cheap. Every fiber in a commit contributes to reconciliation cost.

**Common sources of unnecessary fibers:**
- Per-instance UI wrappers (context menus, tooltips, popovers) on repeated items — hoist to a shared singleton
- Icon library components that create full React component trees for each icon — use memoized inline SVGs on hot paths
- Deeply nested provider trees that could be flattened

```tsx
// BEFORE: 60 cards × ContextMenu wrapper = 180+ extra fibers
{cards.map(card => (
  <ContextMenu key={card.id}>
    <ContextMenuTrigger><Card card={card} /></ContextMenuTrigger>
    <ContextMenuContent>...</ContextMenuContent>
  </ContextMenu>
))}

// AFTER: One shared menu, target set on interaction
<ContextMenu>
  <ContextMenuTrigger asChild>
    <div>{cards.map(card => <Card key={card.id} card={card} />)}</div>
  </ContextMenuTrigger>
  <ContextMenuContent>
    {menuTarget && <MenuItems target={menuTarget} />}
  </ContextMenuContent>
</ContextMenu>
```

### 4. Stabilize Props and Dependencies

**Problem:** Components wrapped in `memo()` still re-render because their props change by reference every render (new objects, arrays, or callbacks).

**Pattern:** Memoize objects, arrays, and callbacks that are passed to `memo`'d children.

```tsx
// Unstable: new array every render → memo is useless
<SortableContext items={cards.map(c => c.id)}>

// Stable: only recomputes when cards changes
const cardIds = useMemo(() => cards.map(c => c.id), [cards])
<SortableContext items={cardIds}>
```

Also applies to: event handlers (use `useCallback`), style objects, config objects passed as props.

### 5. Conditional Subscriptions

**Problem:** Components subscribe to stores/contexts they only need in certain modes. During other modes, they re-render needlessly.

**Pattern:** Conditionally render the subscribed variant only when the subscription is needed. Otherwise render a static variant with no subscription.

```tsx
function Item({ data }) {
  const isActive = useStore((s) => s.activeMode)
  if (isActive) return <ItemInteractive data={data} />  // subscribes to updates
  return <ItemStatic data={data} />                       // no subscriptions
}
```

### 6. Reduce Per-Frame Work

**Problem:** Hot loops (animation frames, scroll handlers, pointer tracking) do O(n) DOM reads or writes per frame.

**Patterns:**
- Measure geometry once, cache it, invalidate selectively (dirty tracking)
- Use binary search instead of linear scans for positional lookups
- Batch DOM writes, avoid interleaving reads and writes (forced reflows)
- Throttle updates to animation frame boundaries with `requestAnimationFrame`

## Step-by-Step Procedure

1. **Capture baseline** — Profile a production build (or dev with StrictMode disabled) during the slow interaction. Export as JSON. Dev-mode double-renders and extra checks inflate numbers and can mislead optimization efforts.
2. **Extract metrics** — Feed JSON to Coding Agent. Get max/p95/median commit duration, fiber counts, and top self-time components.
3. **Identify the trigger** — Check `updaters` on the slowest commits. This tells you which component's state change is causing the expensive render.
4. **Check the blast radius** — Look at `fiberActualDurations.length` for the slow commits. If it's close to the total tree size, the state is too high in the tree.
5. **Apply techniques** — Work through the list above targeting the specific bottleneck.
6. **Verify** — Record a new profiler session. Compare metrics. Repeat until within budget.

## Verification Checklist

- [ ] Max commit duration is within budget
- [ ] P95 commit duration is within budget
- [ ] Largest commit re-renders fewer fibers than before
- [ ] Total fiber tree size decreased (if tree reduction was applied)
- [ ] No functional regressions from memoization or component splitting
- [ ] Interaction feels responsive (no perceptible delay)

## Common Failure Modes

- **Memo doesn't help** — Props aren't referentially stable. Check that objects/arrays passed to memo'd components are themselves memoized.
- **Median improves but max doesn't** — The max is usually a specific interaction (mount, mode switch, drag start). Profile that interaction specifically and focus on the trigger component.
- **Profiler numbers don't match perceived jank** — React commit duration excludes layout and paint. Use Chrome Performance tab alongside the React Profiler to catch forced reflows, expensive CSS (filters, shadows), or compositor-blocking work.
- **Tree size reduced but commits still slow** — The remaining fibers are individually expensive. Look at self-time, not just fiber count. Apply the wrapper/inner split to the top self-time components.
