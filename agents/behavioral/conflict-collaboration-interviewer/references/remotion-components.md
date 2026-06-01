# Remotion Components -- Conflict & Collaboration Interview

These are React (Remotion) components for creating animated visualizations of conflict-resolution
concepts. They are reference implementations for building educational content -- not rendered
in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## ConflictResolutionDecisionTreeViz

Animates the conflict-resolution decision tree from SKILL.md. The component walks through each
decision node in sequence -- surfacing the disagreement, checking whether the candidate can
state the other side's view, identifying the stakes, and choosing the right response channel.
Each node lights up in order, highlighting the candidate's path through the tree.

```tsx
// Remotion component: ConflictResolutionDecisionTreeViz.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

type CandidateChoice = 'listen-first' | 'stakes-low' | 'stakes-high';

export const ConflictResolutionDecisionTreeViz: React.FC<{
  candidateChoice: CandidateChoice;
}> = ({ candidateChoice }) => {
  const frame = useCurrentFrame();

  // Animation phases: each decision node appears in sequence
  // 0-20:  "Disagreement surfaces" node
  // 20-45: "Can you state the other side?" diamond
  // 45-70: Branch outcome (listen vs. stakes)
  // 70-95: Stakes assessment (if applicable)
  // 95-120: Final channel choice

  const nodeOpacity = (start: number) =>
    interpolate(frame, [start, start + 15], [0, 1], {
      extrapolateLeft: 'clamp',
      extrapolateRight: 'clamp',
    });

  const nodeSlide = (start: number) =>
    interpolate(frame, [start, start + 15], [20, 0], {
      extrapolateLeft: 'clamp',
      extrapolateRight: 'clamp',
    });

  const isListenPath = candidateChoice === 'listen-first';
  const isHighStakes = candidateChoice === 'stakes-high';

  return (
    <AbsoluteFill
      style={{
        background: '#0d1117',
        fontFamily: 'monospace',
        color: '#e6edf3',
        padding: 40,
        flexDirection: 'column',
        alignItems: 'center',
      }}
    >
      <h2 style={{ fontSize: 20, marginBottom: 32, color: '#58a6ff' }}>
        Conflict Resolution Decision Tree
      </h2>

      {/* Node 1: Disagreement surfaces */}
      <div
        style={{
          opacity: nodeOpacity(0),
          transform: `translateY(${nodeSlide(0)}px)`,
          background: '#21262d',
          border: '1px solid #30363d',
          borderRadius: 8,
          padding: '12px 24px',
          marginBottom: 16,
          fontSize: 14,
          textAlign: 'center',
        }}
      >
        Disagreement surfaces
      </div>

      {/* Node 2: Can you state the other side? */}
      <div
        style={{
          opacity: nodeOpacity(20),
          transform: `translateY(${nodeSlide(20)}px)`,
          background: '#1f3a5f',
          border: '2px solid #388bfd',
          borderRadius: 8,
          padding: '12px 24px',
          marginBottom: 16,
          fontSize: 13,
          textAlign: 'center',
          maxWidth: 300,
        }}
      >
        Can you state the OTHER side&apos;s view fairly?
      </div>

      {/* Node 3: Branch result */}
      {frame >= 45 && (
        <div
          style={{
            opacity: nodeOpacity(45),
            transform: `translateY(${nodeSlide(45)}px)`,
            display: 'flex',
            gap: 40,
            marginBottom: 16,
          }}
        >
          <div
            style={{
              background: isListenPath ? '#3d1f1f' : '#1a3320',
              border: `2px solid ${isListenPath ? '#f85149' : '#3fb950'}`,
              borderRadius: 8,
              padding: '10px 18px',
              fontSize: 12,
              textAlign: 'center',
              maxWidth: 160,
            }}
          >
            {isListenPath ? 'No -- Listen again before responding' : 'Yes -- Identify the stakes'}
          </div>
        </div>
      )}

      {/* Node 4: Channel choice */}
      {frame >= 95 && !isListenPath && (
        <div
          style={{
            opacity: nodeOpacity(95),
            transform: `translateY(${nodeSlide(95)}px)`,
            background: isHighStakes ? '#1f3a5f' : '#1a3320',
            border: `2px solid ${isHighStakes ? '#388bfd' : '#3fb950'}`,
            borderRadius: 8,
            padding: '12px 20px',
            fontSize: 13,
            textAlign: 'center',
            maxWidth: 280,
          }}
        >
          {isHighStakes
            ? 'Slow down: 1:1, write it up, bring data'
            : 'Decide quickly in conversation'}
        </div>
      )}

      {/* Tip */}
      {frame > 110 && (
        <div
          style={{
            position: 'absolute',
            bottom: 30,
            left: 30,
            right: 30,
            padding: 14,
            background: '#161b22',
            borderRadius: 8,
            borderLeft: '4px solid #d29922',
            fontSize: 12,
            color: '#8b949e',
          }}
        >
          Tip: Choosing the right channel (1:1 vs. doc vs. meeting) is as important as the
          argument itself. Strong candidates explain WHY they chose the channel they did.
        </div>
      )}
    </AbsoluteFill>
  );
};
```

Renders during the scorecard summary segment. The `candidateChoice` prop is set based on
transcript analysis: `listen-first` if the candidate admitted they needed to hear more before
responding, `stakes-high` if they identified significant business or technical risk and chose
a structured channel, or `stakes-low` if they resolved the conflict quickly in conversation.
The highlighted path shows reviewers which branch the candidate naturally followed.
