# Remotion Components -- Failure & Learning Interview

These are React (Remotion) components for creating animated visualizations of interview concepts from the failure-learning-interviewer skill. They are reference implementations for building educational content -- not rendered in CLI or chat interfaces.

To use these, set up a Remotion project: https://www.remotion.dev/

---

## FailureLearningLoopViz

Animates the Failure to Learning Loop diagram from SKILL.md, highlighting each step in sequence as the candidate works through their answer. Each step lights up as the candidate reaches that stage of their story: failure description, detection, recovery action, retrospective ownership, and the concrete behavioral change. Useful for scorecard segments to visually map where the candidate's answer fell short.

```tsx
// Remotion component: FailureLearningLoopViz.tsx
import {
  AbsoluteFill,
  useCurrentFrame,
  interpolate,
  Sequence,
} from 'remotion';
import React from 'react';

type Step = {
  id: number;
  label: string;
  sublabel: string;
  color: string;
};

const STEPS: Step[] = [
  { id: 1, label: '1. Failure occurs', sublabel: 'What happened?', color: '#e74c3c' },
  { id: 2, label: '2. Detection', sublabel: 'When did YOU notice?', color: '#e67e22' },
  { id: 3, label: '3. Recovery action', sublabel: 'What did YOU do next?', color: '#f1c40f' },
  { id: 4, label: '4. Retrospective', sublabel: 'What did YOU own?', color: '#3498db' },
  { id: 5, label: '5. Specific change adopted', sublabel: 'Habit / checklist / metric / review', color: '#2ecc71' },
];

export const FailureLearningLoopViz: React.FC<{ activeStep: 1 | 2 | 3 | 4 | 5 }> = ({
  activeStep,
}) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#0a0a0a',
        fontFamily: 'monospace',
        padding: 40,
        display: 'flex',
        flexDirection: 'column',
        alignItems: 'center',
        justifyContent: 'center',
      }}
    >
      <div
        style={{
          color: '#ecf0f1',
          fontSize: 20,
          fontWeight: 'bold',
          marginBottom: 32,
          letterSpacing: 1,
        }}
      >
        Failure to Learning Loop
      </div>

      <div style={{ display: 'flex', flexDirection: 'column', gap: 12, width: 520 }}>
        {STEPS.map((step) => {
          const isActive = step.id === activeStep;
          const isCompleted = step.id < activeStep;
          const entryOpacity = interpolate(frame, [0, 20], [0, 1], {
            extrapolateLeft: 'clamp',
            extrapolateRight: 'clamp',
          });

          return (
            <div
              key={step.id}
              style={{
                display: 'flex',
                alignItems: 'center',
                gap: 16,
                opacity: entryOpacity,
                padding: '12px 16px',
                borderRadius: 8,
                backgroundColor: isActive
                  ? `${step.color}22`
                  : isCompleted
                  ? '#1a1a1a'
                  : '#111111',
                border: `2px solid ${isActive ? step.color : isCompleted ? '#333' : '#222'}`,
                transition: 'all 0.3s ease',
              }}
            >
              <div
                style={{
                  width: 36,
                  height: 36,
                  borderRadius: '50%',
                  backgroundColor: isActive ? step.color : isCompleted ? '#2a2a2a' : '#1a1a1a',
                  display: 'flex',
                  alignItems: 'center',
                  justifyContent: 'center',
                  color: isActive ? '#fff' : isCompleted ? '#555' : '#333',
                  fontWeight: 'bold',
                  fontSize: 14,
                  flexShrink: 0,
                }}
              >
                {step.id}
              </div>
              <div>
                <div
                  style={{
                    color: isActive ? step.color : isCompleted ? '#555' : '#333',
                    fontWeight: isActive ? 'bold' : 'normal',
                    fontSize: 15,
                  }}
                >
                  {step.label}
                </div>
                <div
                  style={{
                    color: isActive ? '#aaa' : '#3a3a3a',
                    fontSize: 12,
                    marginTop: 2,
                  }}
                >
                  {step.sublabel}
                </div>
              </div>
              {isActive && (
                <div
                  style={{
                    marginLeft: 'auto',
                    color: step.color,
                    fontSize: 12,
                    fontStyle: 'italic',
                  }}
                >
                  current
                </div>
              )}
            </div>
          );
        })}
      </div>

      {activeStep === 5 && (
        <Sequence from={20}>
          <div
            style={{
              marginTop: 24,
              padding: '12px 20px',
              backgroundColor: '#1a2e1a',
              border: '2px solid #2ecc71',
              borderRadius: 8,
              color: '#2ecc71',
              fontSize: 13,
              textAlign: 'center',
              width: 520,
              opacity: interpolate(frame, [0, 15], [0, 1], {
                extrapolateLeft: 'clamp',
                extrapolateRight: 'clamp',
              }),
            }}
          >
            Pattern recognition feeds future detection (Senior+)
          </div>
        </Sequence>
      )}
    </AbsoluteFill>
  );
};
```

Renders during the scorecard segment to visualize the candidate's progression through the learning loop. Pass `activeStep` as the highest step the candidate's answer clearly reached: a candidate who described the failure and recovery but gave a vague lesson would receive `activeStep={3}`, showing visually that step 4 (retrospective ownership) and step 5 (concrete change) were not reached.
