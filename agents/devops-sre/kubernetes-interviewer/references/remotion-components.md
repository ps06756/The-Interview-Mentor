# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## KubernetesDeploymentDemo

```tsx
import { useCurrentFrame, interpolate } from 'remotion';

export const KubernetesDeploymentDemo = () => {
  const frame = useCurrentFrame();

  const pods = [
    { id: 0, version: frame < 30 ? 'v1' : frame < 60 ? 'terminating' : 'v2' },
    { id: 1, version: frame < 45 ? 'v1' : frame < 75 ? 'terminating' : 'v2' },
    { id: 2, version: frame < 60 ? 'v1' : frame < 90 ? 'terminating' : 'v2' },
    { id: 3, version: frame < 75 ? 'v1' : frame < 105 ? 'terminating' : 'v2' },
  ];

  const getColor = (version: string) => {
    if (version === 'v1') return '#4FC3F7';
    if (version === 'terminating') return '#FF7043';
    return '#66BB6A';
  };

  return (
    <div style={{ width: 800, height: 400, background: '#1e1e1e', color: 'white', padding: 30 }}>
      <h2 style={{ textAlign: 'center', marginBottom: 20 }}>Rolling Update: v1 to v2</h2>

      <div style={{ display: 'flex', justifyContent: 'center', gap: 20, marginTop: 40 }}>
        {pods.map((pod) => (
          <div
            key={pod.id}
            style={{
              width: 120,
              height: 120,
              background: getColor(pod.version),
              borderRadius: 12,
              display: 'flex',
              flexDirection: 'column',
              alignItems: 'center',
              justifyContent: 'center',
              opacity: pod.version === 'terminating' ? 0.5 : 1,
            }}
          >
            <div style={{ fontSize: 14, fontWeight: 'bold' }}>Pod {pod.id}</div>
            <div style={{ fontSize: 18, marginTop: 8 }}>{pod.version}</div>
          </div>
        ))}
      </div>

      <div style={{ display: 'flex', justifyContent: 'center', gap: 30, marginTop: 40 }}>
        <div style={{ display: 'flex', alignItems: 'center', gap: 8 }}>
          <div style={{ width: 16, height: 16, background: '#4FC3F7', borderRadius: 4 }} />
          <span>v1 (old)</span>
        </div>
        <div style={{ display: 'flex', alignItems: 'center', gap: 8 }}>
          <div style={{ width: 16, height: 16, background: '#FF7043', borderRadius: 4 }} />
          <span>Terminating</span>
        </div>
        <div style={{ display: 'flex', alignItems: 'center', gap: 8 }}>
          <div style={{ width: 16, height: 16, background: '#66BB6A', borderRadius: 4 }} />
          <span>v2 (new)</span>
        </div>
      </div>
    </div>
  );
};
```
