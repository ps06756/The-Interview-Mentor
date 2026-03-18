# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## ML Pipeline Demo Animation

```tsx
// Remotion component: MLPipelineDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const MLPipelineDemo = () => {
  const frame = useCurrentFrame();

  // Animation phases
  // 0-30: Data flows into feature engineering
  // 30-60: Features flow into training, model artifact produced
  // 60-90: Model deployed to serving, requests flow in
  // 90-120: Monitoring detects drift, triggers retraining

  const phase = Math.floor(frame / 30);
  const phaseProgress = (frame % 30) / 30;

  const stages = [
    { label: 'Raw Data', x: 50, color: '#4A90D9' },
    { label: 'Feature Store', x: 200, color: '#7B68EE' },
    { label: 'Training', x: 350, color: '#F5A623' },
    { label: 'Model Registry', x: 500, color: '#50C878' },
    { label: 'Serving', x: 650, color: '#FF6B6B' },
    { label: 'Monitoring', x: 800, color: '#FFD700' },
  ];

  return (
    <div style={{ width: 960, height: 540, background: '#0D1117', color: 'white', position: 'relative', fontFamily: 'monospace' }}>
      <h2 style={{ textAlign: 'center', padding: 20, fontSize: 28 }}>ML System Lifecycle</h2>

      {/* Pipeline stages */}
      {stages.map((stage, i) => (
        <div key={i} style={{
          position: 'absolute', top: 200, left: stage.x,
          width: 100, height: 60,
          background: phase >= i ? stage.color : '#333',
          borderRadius: 8,
          display: 'flex', alignItems: 'center', justifyContent: 'center',
          fontSize: 11, textAlign: 'center', padding: 4,
          transition: 'background 0.3s',
          opacity: phase >= i ? 1 : 0.4,
        }}>
          {stage.label}
        </div>
      ))}

      {/* Arrows between stages */}
      {stages.slice(0, -1).map((stage, i) => (
        <div key={`arrow-${i}`} style={{
          position: 'absolute', top: 225, left: stage.x + 105,
          width: 40, height: 2, background: phase > i ? '#fff' : '#333',
        }}>
          <div style={{
            position: 'absolute', right: -6, top: -4,
            width: 0, height: 0,
            borderLeft: '8px solid ' + (phase > i ? '#fff' : '#333'),
            borderTop: '5px solid transparent',
            borderBottom: '5px solid transparent',
          }} />
        </div>
      ))}

      {/* Drift feedback loop arrow (phase 3) */}
      {phase >= 3 && (
        <div style={{
          position: 'absolute', top: 280, left: 350, width: 500,
          height: 60, border: '2px dashed #FFD700', borderTop: 'none',
          borderRadius: '0 0 20px 20px', opacity: phaseProgress,
        }} />
      )}

      {/* Status text */}
      <div style={{ position: 'absolute', bottom: 60, width: '100%', textAlign: 'center', fontSize: 20 }}>
        {phase === 0 && 'Ingesting raw data into feature pipelines...'}
        {phase === 1 && 'Computing features, storing in feature store...'}
        {phase === 2 && 'Training model, registering artifact...'}
        {phase === 3 && 'Drift detected! Triggering retraining loop...'}
      </div>
    </div>
  );
};
```
