# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## BFS Traversal Demo

```tsx
// Remotion component: BFSTraversalDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const BFSTraversalDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const adjList: Record<number, number[]> = {
    0: [1, 2],
    1: [0, 3, 4],
    2: [0, 3],
    3: [1, 2],
    4: [1],
  };

  const positions: Record<number, { x: number; y: number }> = {
    0: { x: 100, y: 50 },
    1: { x: 250, y: 50 },
    2: { x: 100, y: 200 },
    3: { x: 250, y: 200 },
    4: { x: 400, y: 50 },
  };

  const bfsOrder = [0, 1, 2, 3, 4];
  const visitedCount = Math.min(Math.floor(frame / fps) + 1, bfsOrder.length);
  const visited = new Set(bfsOrder.slice(0, visitedCount));

  return (
    <div style={{ fontFamily: 'monospace', fontSize: 18, padding: 40, position: 'relative', width: 500, height: 300 }}>
      <div style={{ marginBottom: 10 }}>BFS from node 0 (level by level)</div>
      {Object.entries(positions).map(([id, pos]) => (
        <div
          key={id}
          style={{
            position: 'absolute',
            left: pos.x,
            top: pos.y + 40,
            width: 50,
            height: 50,
            borderRadius: '50%',
            border: '3px solid #333',
            display: 'flex',
            alignItems: 'center',
            justifyContent: 'center',
            background: visited.has(Number(id)) ? '#90EE90' : 'white',
            transition: 'background 0.3s',
          }}
        >
          {id}
        </div>
      ))}
    </div>
  );
};
```
