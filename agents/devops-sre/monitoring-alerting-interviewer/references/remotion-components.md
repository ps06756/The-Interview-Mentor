# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## MonitoringDashboardDemo

```tsx
import { useCurrentFrame, interpolate } from 'remotion';

export const MonitoringDashboardDemo = () => {
  const frame = useCurrentFrame();

  // Simulate a metric that degrades over time then recovers after remediation
  const errorRate = frame < 60
    ? interpolate(frame, [0, 60], [0.1, 5.2], { extrapolateRight: 'clamp' })
    : interpolate(frame, [60, 90], [5.2, 0.3], { extrapolateRight: 'clamp' });

  const latencyP99 = frame < 60
    ? interpolate(frame, [0, 60], [200, 8500], { extrapolateRight: 'clamp' })
    : interpolate(frame, [60, 90], [8500, 250], { extrapolateRight: 'clamp' });

  const getStatusColor = (value: number, threshold: number) => {
    if (value > threshold * 2) return '#EF5350';
    if (value > threshold) return '#FFA726';
    return '#66BB6A';
  };

  const phase = frame < 30 ? 'Normal' : frame < 60 ? 'Incident' : frame < 75 ? 'Mitigating' : 'Recovered';

  return (
    <div style={{ width: 800, height: 450, background: '#1e1e1e', color: 'white', padding: 30, fontFamily: 'monospace' }}>
      <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', marginBottom: 20 }}>
        <h2>Payment Service Dashboard</h2>
        <div style={{
          padding: '4px 12px',
          borderRadius: 4,
          background: phase === 'Normal' || phase === 'Recovered' ? '#66BB6A' : '#EF5350',
          fontSize: 14,
        }}>
          {phase}
        </div>
      </div>

      <div style={{ display: 'grid', gridTemplateColumns: '1fr 1fr', gap: 20 }}>
        {/* Error Rate Panel */}
        <div style={{ background: '#2d2d2d', borderRadius: 8, padding: 16 }}>
          <div style={{ fontSize: 12, color: '#999', marginBottom: 8 }}>Error Rate</div>
          <div style={{
            fontSize: 36,
            fontWeight: 'bold',
            color: getStatusColor(errorRate, 1),
          }}>
            {errorRate.toFixed(1)}%
          </div>
          <div style={{ fontSize: 11, color: '#666', marginTop: 4 }}>SLO: 0.1%</div>
        </div>

        {/* Latency Panel */}
        <div style={{ background: '#2d2d2d', borderRadius: 8, padding: 16 }}>
          <div style={{ fontSize: 12, color: '#999', marginBottom: 8 }}>Latency p99</div>
          <div style={{
            fontSize: 36,
            fontWeight: 'bold',
            color: getStatusColor(latencyP99, 500),
          }}>
            {latencyP99 > 1000 ? `${(latencyP99 / 1000).toFixed(1)}s` : `${Math.round(latencyP99)}ms`}
          </div>
          <div style={{ fontSize: 11, color: '#666', marginTop: 4 }}>SLO: 500ms</div>
        </div>

        {/* Request Rate Panel */}
        <div style={{ background: '#2d2d2d', borderRadius: 8, padding: 16 }}>
          <div style={{ fontSize: 12, color: '#999', marginBottom: 8 }}>Request Rate</div>
          <div style={{ fontSize: 36, fontWeight: 'bold', color: '#66BB6A' }}>
            5,012/s
          </div>
          <div style={{ fontSize: 11, color: '#666', marginTop: 4 }}>Expected: 5,000/s</div>
        </div>

        {/* Error Budget Panel */}
        <div style={{ background: '#2d2d2d', borderRadius: 8, padding: 16 }}>
          <div style={{ fontSize: 12, color: '#999', marginBottom: 8 }}>Error Budget Remaining</div>
          <div style={{
            fontSize: 36,
            fontWeight: 'bold',
            color: frame < 60 ? '#EF5350' : '#FFA726',
          }}>
            {frame < 30 ? '72%' : frame < 60 ? '31%' : frame < 75 ? '29%' : '28%'}
          </div>
          <div style={{ fontSize: 11, color: '#666', marginTop: 4 }}>30-day rolling window</div>
        </div>
      </div>

      {/* Timeline */}
      <div style={{ marginTop: 20, fontSize: 12, color: '#999' }}>
        {frame >= 30 && <div style={{ color: '#EF5350' }}>02:00 - Alert: Error rate &gt; 5%</div>}
        {frame >= 45 && <div style={{ color: '#FFA726' }}>02:02 - Root cause: Stripe API degraded</div>}
        {frame >= 60 && <div style={{ color: '#66BB6A' }}>02:03 - Mitigation: Circuit breaker enabled</div>}
        {frame >= 75 && <div style={{ color: '#66BB6A' }}>02:05 - Recovered: Error rate nominal</div>}
      </div>
    </div>
  );
};
```
