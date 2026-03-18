# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## DPTableFillingDemo

```tsx
// Remotion component: DPTableFillingDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const DPTableFillingDemo = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const coins = [1, 3, 4];
  const amount = 6;
  const dp = [0, 1, 2, 1, 1, 2, 2];
  const visibleCells = Math.min(Math.floor(frame / fps) + 1, dp.length);

  return (
    <div style={{ fontFamily: 'monospace', fontSize: 36, padding: 50 }}>
      <div>Coins: [{coins.join(', ')}] | Amount: {amount}</div>
      <div style={{ marginTop: 30, display: 'flex', gap: 4 }}>
        {dp.map((val, i) => (
          <div key={i} style={{ textAlign: 'center' }}>
            <div style={{ fontSize: 20, marginBottom: 8, color: '#666' }}>{i}</div>
            <div style={{
              width: 64, height: 64,
              border: '3px solid #333',
              display: 'flex', alignItems: 'center', justifyContent: 'center',
              background: i < visibleCells ? '#90EE90' : '#f0f0f0',
            }}>
              {i < visibleCells ? val : ''}
            </div>
          </div>
        ))}
      </div>
    </div>
  );
};
```
