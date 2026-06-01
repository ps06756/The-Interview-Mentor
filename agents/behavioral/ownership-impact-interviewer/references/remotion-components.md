# Remotion Components -- Ownership & Impact Interview

These are React (Remotion) components for creating animated visualizations of the Ownership Growth
Ladder and the Impact Measurement Pyramid from the ownership-impact-interviewer SKILL.md. They are
reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## OwnershipGrowthLadderViz

Animates the 5-level Ownership Growth Ladder diagram from the SKILL.md Interactive Elements section.
At each frame, the active level (determined by `candidateLevel`) is highlighted and the levels below
it appear as context. Use this at scorecard-time to show the candidate which ownership level their
answer most closely demonstrated, and which level is the target for their next interview.

```tsx
import { AbsoluteFill, interpolate, useCurrentFrame } from 'remotion';

type OwnershipLevel = 0 | 1 | 2 | 3 | 4;

interface LevelConfig {
  level: OwnershipLevel;
  label: string;
  description: string;
  calibration: string;
  color: string;
}

const LEVELS: LevelConfig[] = [
  {
    level: 4,
    label: 'Level 4 -- Org-wide',
    description: 'Identified a pattern across multiple teams and built a shared solution.',
    calibration: 'Staff+',
    color: '#8e44ad',
  },
  {
    level: 3,
    label: 'Level 3 -- Cross-team',
    description: 'Drove an initiative requiring coordination across multiple teams.',
    calibration: 'Senior+',
    color: '#2980b9',
  },
  {
    level: 2,
    label: 'Level 2 -- Beyond assigned scope',
    description: 'Noticed a problem and fixed it without being asked.',
    calibration: 'Mid+',
    color: '#27ae60',
  },
  {
    level: 1,
    label: 'Level 1 -- Within assigned scope',
    description: 'Owned a project from estimation through launch and post-launch monitoring.',
    calibration: 'Junior',
    color: '#f39c12',
  },
  {
    level: 0,
    label: 'Level 0 -- Task-level',
    description: 'Delivered what was assigned.',
    calibration: 'Entry / Intern',
    color: '#7f8c8d',
  },
];

export const OwnershipGrowthLadderViz: React.FC<{ candidateLevel: OwnershipLevel }> = ({
  candidateLevel,
}) => {
  const frame = useCurrentFrame();

  // Each level fades in sequentially from bottom (Level 0) to top (Level 4)
  // so the visual builds upward over the first 60 frames
  const getOpacity = (level: OwnershipLevel): number => {
    const revealStart = (4 - level) * 10;
    return interpolate(frame, [revealStart, revealStart + 15], [0, 1], {
      extrapolateLeft: 'clamp',
      extrapolateRight: 'clamp',
    });
  };

  return (
    <AbsoluteFill
      style={{
        background: '#0d1117',
        fontFamily: 'system-ui, sans-serif',
        padding: 32,
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
      }}
    >
      <h2
        style={{
          color: '#e6edf3',
          fontSize: 22,
          marginBottom: 20,
          textAlign: 'center',
          letterSpacing: 0.5,
        }}
      >
        Ownership Growth Ladder
      </h2>

      {LEVELS.map((cfg) => {
        const isActive = cfg.level === candidateLevel;
        const isBelow = cfg.level < candidateLevel;
        const opacity = getOpacity(cfg.level);

        return (
          <div
            key={cfg.level}
            style={{
              opacity,
              marginBottom: 10,
              padding: '12px 18px',
              borderRadius: 8,
              border: isActive ? `2px solid ${cfg.color}` : '1px solid #30363d',
              background: isActive ? `${cfg.color}22` : isBelow ? '#161b22' : '#0d1117',
              display: 'flex',
              alignItems: 'center',
              gap: 16,
              transform: isActive
                ? interpolate(frame, [40, 55], [1.02, 1], {
                    extrapolateLeft: 'clamp',
                    extrapolateRight: 'clamp',
                  }).toString()
                : '1',
            }}
          >
            {/* Level badge */}
            <div
              style={{
                width: 36,
                height: 36,
                borderRadius: '50%',
                background: isActive ? cfg.color : '#21262d',
                color: isActive ? '#fff' : '#8b949e',
                display: 'flex',
                alignItems: 'center',
                justifyContent: 'center',
                fontWeight: 700,
                fontSize: 16,
                flexShrink: 0,
              }}
            >
              {cfg.level}
            </div>

            {/* Label and description */}
            <div style={{ flex: 1 }}>
              <div
                style={{
                  color: isActive ? cfg.color : '#c9d1d9',
                  fontWeight: isActive ? 700 : 400,
                  fontSize: 15,
                  marginBottom: 2,
                }}
              >
                {cfg.label}
              </div>
              <div style={{ color: '#8b949e', fontSize: 12 }}>{cfg.description}</div>
            </div>

            {/* Calibration chip */}
            <div
              style={{
                padding: '3px 10px',
                borderRadius: 12,
                background: isActive ? cfg.color : '#21262d',
                color: isActive ? '#fff' : '#8b949e',
                fontSize: 11,
                fontWeight: 600,
                flexShrink: 0,
              }}
            >
              {cfg.calibration}
            </div>
          </div>
        );
      })}

      {/* Calibration note at bottom */}
      <p
        style={{
          color: '#8b949e',
          fontSize: 12,
          textAlign: 'center',
          marginTop: 16,
        }}
      >
        A new-grad at Level 1 is strong. A senior at Level 1 is concerning.
      </p>
    </AbsoluteFill>
  );
};
```

**Usage**: Pass the level that most closely matches the candidate's answer -- typically determined
during scorecard generation based on whether the problem was assigned (`level: 1`) or proactively
identified (`level: 2+`) and whether the scope was single-team or cross-team. The animation builds
the ladder from the ground up over the first 60 frames, then highlights the candidate's level.

For the Impact Measurement Pyramid, extend this pattern with a `candidateTier` prop ranging from
`'activity' | 'qualitative' | 'quantified' | 'second-order'` and map each tier to the corresponding
pyramid band. The visual structure mirrors the pyramid diagram in SKILL.md Interactive Elements.
