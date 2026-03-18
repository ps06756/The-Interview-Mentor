# Remotion Animation Components

These are React (Remotion) components for creating animated visualizations of interview concepts. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## Transformer Attention Demo Animation

```tsx
// Remotion component: TransformerAttentionDemo.tsx
import { useCurrentFrame, useVideoConfig } from 'remotion';

export const TransformerAttentionDemo = () => {
  const frame = useCurrentFrame();

  // Animation phases
  // 0-30: Tokens appear with embeddings
  // 30-60: Q, K, V projections animate
  // 60-90: Attention matrix fills in with weights
  // 90-120: Output tokens computed from weighted values

  const phase = Math.floor(frame / 30);
  const phaseProgress = (frame % 30) / 30;

  const tokens = ['The', 'cat', 'sat', 'on', 'mat'];

  // Simulated attention weights (row = query, col = key)
  const attentionWeights = [
    [0.1, 0.1, 0.1, 0.1, 0.6],
    [0.1, 0.3, 0.3, 0.1, 0.2],
    [0.1, 0.4, 0.2, 0.2, 0.1],
    [0.1, 0.1, 0.3, 0.3, 0.2],
    [0.5, 0.1, 0.1, 0.1, 0.2],
  ];

  return (
    <div style={{ width: 960, height: 540, background: '#0D1117', color: 'white', position: 'relative', fontFamily: 'monospace' }}>
      <h2 style={{ textAlign: 'center', padding: 16, fontSize: 24 }}>Self-Attention Mechanism</h2>

      {/* Input tokens */}
      <div style={{ display: 'flex', justifyContent: 'center', gap: 24, marginTop: 20 }}>
        {tokens.map((token, i) => (
          <div key={i} style={{
            padding: '8px 16px',
            background: phase >= 0 ? '#4A90D9' : '#333',
            borderRadius: 6,
            fontSize: 18,
            opacity: phase >= 0 ? Math.min(1, phaseProgress * 3 + i * 0.2) : 0.3,
          }}>
            {token}
          </div>
        ))}
      </div>

      {/* Q, K, V labels */}
      {phase >= 1 && (
        <div style={{ display: 'flex', justifyContent: 'center', gap: 80, marginTop: 30 }}>
          {['Q (Query)', 'K (Key)', 'V (Value)'].map((label, i) => (
            <div key={i} style={{
              padding: '6px 20px',
              background: ['#FF6B6B', '#50C878', '#7B68EE'][i],
              borderRadius: 6,
              fontSize: 14,
              opacity: phaseProgress,
            }}>
              {label}
            </div>
          ))}
        </div>
      )}

      {/* Attention matrix */}
      {phase >= 2 && (
        <div style={{ display: 'flex', justifyContent: 'center', marginTop: 30 }}>
          <div style={{ display: 'grid', gridTemplateColumns: `repeat(${tokens.length}, 48px)`, gap: 2 }}>
            {attentionWeights.flat().map((weight, i) => {
              const row = Math.floor(i / tokens.length);
              const col = i % tokens.length;
              const revealIndex = row * tokens.length + col;
              const revealed = phaseProgress > revealIndex / (tokens.length * tokens.length);
              return (
                <div key={i} style={{
                  width: 48, height: 48,
                  background: revealed
                    ? `rgba(74, 144, 217, ${weight})`
                    : '#1a1a2e',
                  display: 'flex', alignItems: 'center', justifyContent: 'center',
                  fontSize: 11,
                  borderRadius: 3,
                }}>
                  {revealed ? weight.toFixed(1) : ''}
                </div>
              );
            })}
          </div>
        </div>
      )}

      {/* Status text */}
      <div style={{ position: 'absolute', bottom: 40, width: '100%', textAlign: 'center', fontSize: 18 }}>
        {phase === 0 && 'Input tokens embedded with positional encoding...'}
        {phase === 1 && 'Computing Q, K, V linear projections...'}
        {phase === 2 && 'Computing attention weights: softmax(QK^T / sqrt(d_k))...'}
        {phase === 3 && 'Output = Attention weights x Values'}
      </div>
    </div>
  );
};
```
