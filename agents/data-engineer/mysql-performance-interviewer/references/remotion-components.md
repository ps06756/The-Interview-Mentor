# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of the interview concepts. They are reference implementations for building educational content — not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## ESR Rule Visualizer

```tsx
// Remotion component: ESRRuleVisualizer.tsx
import { useCurrentFrame, useVideoConfig, interpolate, spring } from 'remotion';

const COLORS = {
  equality: '#4EC9B0',
  sort: '#DCDCAA',
  range: '#CE9178',
  bg: '#1e1e1e',
  text: '#d4d4d4',
  dim: '#555',
};

export const ESRRuleVisualizer = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const phases = [
    {
      label: 'WRONG Order: Range → Equality → Sort',
      index: ['created_at', 'customer_id', 'amount'],
      colors: [COLORS.range, COLORS.equality, COLORS.sort],
      usable: [true, false, false],
      note: 'Index stops after first range column. customer_id and amount unreachable.',
    },
    {
      label: 'CORRECT Order: Equality → Sort → Range (ESR)',
      index: ['customer_id', 'amount', 'created_at'],
      colors: [COLORS.equality, COLORS.sort, COLORS.range],
      usable: [true, true, true],
      note: 'All three columns used: filter → sort → range filter. No filesort!',
    },
  ];

  const activePhase = Math.floor(frame / (4 * fps)) % phases.length;
  const phase = phases[activePhase];

  return (
    <div style={{ fontFamily: 'monospace', padding: 40, background: COLORS.bg, color: COLORS.text }}>
      <h2 style={{ color: COLORS.equality, marginBottom: 10 }}>ESR Rule: Equality → Sort → Range</h2>
      <p style={{ color: COLORS.dim, fontSize: 14, marginBottom: 30 }}>
        Query: WHERE customer_id = 123 AND created_at {'>'} '2024-01-01' ORDER BY amount DESC
      </p>

      <h3 style={{ color: phase.usable.every(Boolean) ? COLORS.equality : COLORS.range }}>
        {phase.label}
      </h3>

      <div style={{ display: 'flex', gap: 10, margin: '20px 0' }}>
        {phase.index.map((col, idx) => {
          const s = spring({ frame: frame - activePhase * 4 * fps - idx * 10, fps, config: { damping: 12 } });
          return (
            <div
              key={col}
              style={{
                padding: '15px 25px',
                border: `2px solid ${phase.colors[idx]}`,
                background: phase.usable[idx] ? `${phase.colors[idx]}22` : '#2d2d2d',
                opacity: interpolate(s, [0, 1], [0, 1]),
                transform: `translateY(${interpolate(s, [0, 1], [20, 0])}px)`,
                textAlign: 'center',
              }}
            >
              <div style={{ fontWeight: 'bold', color: phase.colors[idx] }}>{col}</div>
              <div style={{ fontSize: 12, marginTop: 5, color: phase.usable[idx] ? COLORS.text : COLORS.dim }}>
                {phase.usable[idx] ? 'USED' : 'WASTED'}
              </div>
            </div>
          );
        })}
      </div>

      <div style={{
        padding: 15,
        background: '#264f4f',
        borderLeft: `4px solid ${COLORS.equality}`,
        marginTop: 20,
        fontSize: 14,
      }}>
        {phase.note}
      </div>
    </div>
  );
};
```

---

## InnoDB Locking Animation

```tsx
// Remotion component: InnoDBLockingVisualizer.tsx
import { useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

export const InnoDBLockingVisualizer = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const rows = [
    { id: 1, customerId: 100, status: 'completed' },
    { id: 2, customerId: 200, status: 'pending' },
    { id: 3, customerId: 123, status: 'pending' },
    { id: 4, customerId: 300, status: 'completed' },
    { id: 5, customerId: 123, status: 'shipped' },
    { id: 6, customerId: 400, status: 'pending' },
  ];

  const phase = Math.floor(frame / (3 * fps)) % 2; // 0 = without index, 1 = with index

  const isLocked = (row: typeof rows[0]) => {
    if (phase === 0) return true; // Without index: ALL rows locked (table scan)
    return row.customerId === 123; // With index: only matching rows locked
  };

  return (
    <div style={{ fontFamily: 'monospace', padding: 40, background: '#1e1e1e', color: '#d4d4d4' }}>
      <h2 style={{ color: '#4EC9B0' }}>
        InnoDB Locking: {phase === 0 ? 'WITHOUT Index (Table Scan)' : 'WITH Index (Index Scan)'}
      </h2>
      <p style={{ color: '#888', fontSize: 14, margin: '10px 0 20px' }}>
        UPDATE orders SET status = 'shipped' WHERE customer_id = 123
      </p>

      <table style={{ borderCollapse: 'collapse', width: '100%' }}>
        <thead>
          <tr>
            {['id', 'customer_id', 'status', 'Lock State'].map(h => (
              <th key={h} style={{ padding: 10, borderBottom: '2px solid #555', textAlign: 'left', color: '#9CDCFE' }}>
                {h}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {rows.map((row, idx) => {
            const locked = isLocked(row);
            const matched = row.customerId === 123;
            const delay = idx * 5;
            const opacity = interpolate(frame - delay, [0, 10], [0, 1], { extrapolateRight: 'clamp' });

            return (
              <tr key={row.id} style={{ opacity }}>
                <td style={{ padding: 10, borderBottom: '1px solid #333' }}>{row.id}</td>
                <td style={{
                  padding: 10,
                  borderBottom: '1px solid #333',
                  color: matched ? '#4EC9B0' : '#d4d4d4',
                  fontWeight: matched ? 'bold' : 'normal',
                }}>
                  {row.customerId}
                </td>
                <td style={{ padding: 10, borderBottom: '1px solid #333' }}>{row.status}</td>
                <td style={{
                  padding: 10,
                  borderBottom: '1px solid #333',
                  color: locked ? '#F44747' : '#4EC9B0',
                  fontWeight: 'bold',
                }}>
                  {locked ? 'LOCKED' : 'FREE'}
                </td>
              </tr>
            );
          })}
        </tbody>
      </table>

      <div style={{
        marginTop: 20,
        padding: 15,
        background: phase === 0 ? '#4f2626' : '#264f4f',
        borderLeft: `4px solid ${phase === 0 ? '#F44747' : '#4EC9B0'}`,
        fontSize: 14,
      }}>
        {phase === 0
          ? 'Without index: InnoDB scans ALL rows and locks ALL scanned rows. Effectively a table lock!'
          : 'With index: InnoDB scans only matching rows via index. Only 2 rows locked. Other writes proceed freely.'}
      </div>
    </div>
  );
};
```

---

## Connection Pool Exhaustion Animation

```tsx
// Remotion component: ConnectionPoolVisualizer.tsx
import { useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

export const ConnectionPoolVisualizer = () => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const poolSize = 10;
  const scenarios = [
    {
      label: 'Sequential: 1 connection per request',
      connectionsPerRequest: 1,
      rps: 500,
      needed: 500,
      note: 'Each request uses 1 connection for ~5ms. Pool of 10 handles 2000 RPS easily.',
    },
    {
      label: 'Parallel: 3 connections per request',
      connectionsPerRequest: 3,
      rps: 500,
      needed: 1500,
      note: '3 parallel DB calls × 500 RPS = 1500 connections needed. Pool of 10 is overwhelmed.',
    },
  ];

  const activeScenario = Math.floor(frame / (4 * fps)) % scenarios.length;
  const scenario = scenarios[activeScenario];

  const poolSlots = Array.from({ length: poolSize }, (_, i) => {
    const inUse = i < Math.min(poolSize, scenario.connectionsPerRequest);
    return { id: i, inUse };
  });

  const waitingRequests = Math.max(0, scenario.needed - poolSize);

  return (
    <div style={{ fontFamily: 'monospace', padding: 40, background: '#1e1e1e', color: '#d4d4d4' }}>
      <h2 style={{ color: '#4EC9B0' }}>Connection Pool: {scenario.label}</h2>

      <div style={{ display: 'flex', gap: 5, margin: '20px 0' }}>
        {poolSlots.map(slot => (
          <div
            key={slot.id}
            style={{
              width: 40,
              height: 40,
              border: '2px solid',
              borderColor: slot.inUse ? '#F44747' : '#4EC9B0',
              background: slot.inUse ? '#4f2626' : '#264f4f',
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'center',
              fontSize: 10,
            }}
          >
            {slot.inUse ? 'BUSY' : 'FREE'}
          </div>
        ))}
      </div>

      <div style={{ fontSize: 14, marginTop: 15 }}>
        <div>Pool size: <strong>{poolSize}</strong></div>
        <div>Connections needed: <strong style={{ color: scenario.needed > poolSize ? '#F44747' : '#4EC9B0' }}>
          {scenario.connectionsPerRequest} × {scenario.rps} RPS = {scenario.needed}
        </strong></div>
        {waitingRequests > 0 && (
          <div style={{ color: '#F44747', marginTop: 5 }}>
            {waitingRequests} requests waiting for a connection...
          </div>
        )}
      </div>

      <div style={{
        marginTop: 20,
        padding: 15,
        background: activeScenario === 1 ? '#4f2626' : '#264f4f',
        borderLeft: `4px solid ${activeScenario === 1 ? '#F44747' : '#4EC9B0'}`,
        fontSize: 14,
      }}>
        {scenario.note}
      </div>
    </div>
  );
};
```
