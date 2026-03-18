# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## CICDPipelineDemo

```tsx
import { useCurrentFrame, interpolate } from 'remotion';

export const CICDPipelineDemo = () => {
  const frame = useCurrentFrame();

  const stages = [
    { name: 'Build', duration: 30, color: '#42A5F5' },
    { name: 'Unit Tests', duration: 20, color: '#66BB6A' },
    { name: 'Integration', duration: 25, color: '#FFA726' },
    { name: 'Staging', duration: 15, color: '#AB47BC' },
    { name: 'Canary', duration: 30, color: '#EF5350' },
    { name: 'Production', duration: 10, color: '#26A69A' },
  ];

  let cumulativeStart = 0;
  const stagesWithTiming = stages.map((stage) => {
    const start = cumulativeStart;
    cumulativeStart += stage.duration;
    return { ...stage, start, end: cumulativeStart };
  });

  const getStageStatus = (stage: typeof stagesWithTiming[0]) => {
    if (frame < stage.start) return 'pending';
    if (frame < stage.end) return 'running';
    return 'complete';
  };

  return (
    <div style={{ width: 800, height: 400, background: '#1e1e1e', color: 'white', padding: 30 }}>
      <h2 style={{ textAlign: 'center', marginBottom: 30 }}>CI/CD Pipeline: Push to Production</h2>

      <div style={{ display: 'flex', flexDirection: 'column', gap: 12 }}>
        {stagesWithTiming.map((stage) => {
          const status = getStageStatus(stage);
          const progress = status === 'running'
            ? interpolate(frame, [stage.start, stage.end], [0, 100], { extrapolateRight: 'clamp' })
            : status === 'complete' ? 100 : 0;

          return (
            <div key={stage.name} style={{ display: 'flex', alignItems: 'center', gap: 16 }}>
              <div style={{ width: 120, textAlign: 'right', fontSize: 14 }}>{stage.name}</div>
              <div style={{ flex: 1, height: 28, background: '#333', borderRadius: 6, overflow: 'hidden' }}>
                <div
                  style={{
                    width: `${progress}%`,
                    height: '100%',
                    background: stage.color,
                    borderRadius: 6,
                    transition: 'width 0.1s',
                  }}
                />
              </div>
              <div style={{ width: 80, fontSize: 12, color: status === 'complete' ? '#66BB6A' : '#999' }}>
                {status === 'pending' ? 'Waiting' : status === 'running' ? 'Running...' : 'Passed'}
              </div>
            </div>
          );
        })}
      </div>
    </div>
  );
};
```
