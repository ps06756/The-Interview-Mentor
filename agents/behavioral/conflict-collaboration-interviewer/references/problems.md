# Problem Bank -- Conflict & Collaboration Behavioral Interview

---

## How to Use This Bank

Each scenario below includes the question, what the interviewer is evaluating, example hints,
a worked STAR answer at the Senior Engineer level, and follow-up questions to probe deeper.
Adapt the scope and specificity to the candidate's target level.

---

### Scenario: Peer Technical Disagreement

**Question**: "Tell me about a time you and a peer disagreed on a technical approach and you
had to find a path forward."

**Evaluates**: Intellectual humility, ability to articulate disagreement without blame,
persuasion through evidence rather than authority, and repair of the working relationship
after the disagreement resolves.

**Hints**:
- **Level 1**: "Think about a time you weren't fully convinced your peer's approach was right.
  What made the situation feel difficult -- the technical question, the relationship, or both?"
- **Level 2**: "Structure your answer using STAR. For the Action, focus on HOW you raised the
  disagreement -- what channel did you choose (1:1, Slack, design doc), and how did you frame
  your concern so it didn't come across as personal?"
- **Level 3**: "Strong answers include: (a) what each of you wanted and the reasoning behind
  each position -- state your peer's view as clearly as your own, (b) the channel and framing
  you chose, (c) what new information or evidence you introduced, (d) the decision outcome,
  and (e) what changed in the working relationship afterward."
- **Level 4**: "Example structure: 'My peer proposed [X]. I had concerns about [specific
  concern]. I asked for a 30-minute 1:1 rather than raising it in standup because [reasoning].
  We agreed to spike both approaches for two days. The benchmark showed [outcome] and we went
  with [choice]. My peer later told me [specific relationship outcome].'"

**Example STAR Answer**:

**Situation**: In Q2 2024, my peer proposed using an event-sourcing architecture for a new
user-settings service. I had concerns: the team had no prior experience with event sourcing,
the query patterns were simple reads and writes with no need for a full event log, and the
operational overhead would fall on two engineers who were already handling two on-call rotations.

**Task**: As the other senior engineer on the project, I needed to voice my concerns without
dismissing the idea outright. My peer had already done a proof of concept and invested two weeks
in it. I knew if I raised objections in the architecture review without warning, he would feel
ambushed and I would lose the room.

**Action**: I asked for a 30-minute 1:1 before the architecture review. He explained his
reasoning: we had a compliance requirement for a full history of settings changes, and event
sourcing gave us that for free. I had underweighted that requirement. I acknowledged it was
a real constraint I had not thought through, then proposed an alternative -- a standard
PostgreSQL table with an audit-log side table that captured every write -- and sketched the
schema on a shared doc during our call. I estimated two days of work versus event sourcing's
estimated six. We agreed to bring both proposals to the review and let the team weigh in.

**Result**: The team chose the audit-log approach. We shipped the settings service in three
weeks. Three months later, when a compliance audit required a full change history for a specific
user, the PM pulled it with two SQL queries in under five minutes. My peer told me afterward
that the 1:1 was the right call -- he felt I had engaged with his actual problem rather than
blocking his idea. We now run a pre-review sync on any architectural decision where we think
we might differ.

**Follow-up Questions**:
- "What would you do differently if the same disagreement happened today?"
- "Describe the same situation from your peer's perspective -- what do you think they were
  most worried about?"
- "How did this experience change how you approach future technical disagreements with this peer?"

---

### Scenario: Cross-Functional Stakeholder Pushback

**Question**: "Describe a time a product manager or designer wanted something you believed was
the wrong technical call. What did you do?"

**Evaluates**: Ability to translate technical concerns into business language, navigating
cross-functional power dynamics, choosing the right channel and timing for a difficult
conversation, and executing gracefully on a decision you disagreed with.

**Hints**:
- **Level 1**: "Think about a time someone outside engineering asked for something that worried
  you technically. What made it hard to push back directly?"
- **Level 2**: "Focus on the translation problem: how did you express a technical concern in
  terms the stakeholder could understand and act on? What channel did you choose, and why that
  setting rather than a larger group meeting?"
- **Level 3**: "Strong answers include: (a) the specific technical risk you were worried about,
  (b) how you reframed that risk in terms the stakeholder cared about (timeline, user experience,
  cost), (c) the options you proposed, (d) who had the authority to decide, and (e) what you
  did once the decision was made -- especially if it did not go your way."
- **Level 4**: "Example structure: 'The PM wanted [X] by [date], which required [shortcut]. I
  set up a 30-minute 1:1 with a one-pager showing two options: ship fast with [specific risks]
  vs. two more weeks for a cleaner solution. I framed the risk in terms of [customer impact /
  future cost]. The PM chose [option] because [their reasoning]. I disagreed but executed. Here
  is what happened: [specific outcome].'"

**Example STAR Answer**:

**Situation**: In late 2023, our PM proposed adding a magic-link login flow for a B2B enterprise
client pilot, with a two-week deadline tied to a sales close. I was concerned: we had no
email-sending infrastructure, the magic-link flow required rate limiting, token expiry, and
secure storage -- none of which existed -- and we were mid-sprint on a payment integration
that could not slip.

**Task**: I needed to communicate this clearly without simply blocking her request in a sprint
planning meeting in front of six people, which would have put her in a difficult position with
her sales stakeholder and made me look like I was prioritizing engineering preferences over
business needs.

**Action**: I sent a Slack message asking for a 20-minute 1:1 before the sprint kick-off. I
came with a written one-pager: three options, each with an effort estimate, a risk summary, and
what the sales team would actually see by the deadline. Option A was the magic link as requested
(eight engineering days, high risk of last-minute security gaps). Option B was a temporary
password-based flow for the pilot with the magic link on the next sprint (three days, low risk,
fully demo-ready). Option C was a shared staging credential for the pilot only (two hours, no
security guarantees, only viable if the client would sign a limited pilot agreement). I
recommended Option B and explained my reasoning.

**Result**: She chose Option C for the demo and Option B for the live pilot two weeks later --
she had stakeholder context I did not have, and Option C bought her the time she needed. The
pilot launched with the proper magic-link flow. The sales deal closed. She told me afterward
that the one-pager was the most useful thing anyone had handed her in a sprint discussion that
quarter. I now use the same format whenever I need to push back on a product request: three
options, clear estimates, risks stated in terms the stakeholder cares about.

**Follow-up Questions**:
- "How did you handle the moment when the PM chose an option different from what you recommended?"
- "If the decision had led to a customer-facing security incident, how would you have handled
  the post-mortem conversation with the PM?"
- "What did this experience change about how you present technical constraints to non-engineers?"

---

### Scenario: Manager Pushback

**Question**: "Tell me about a time you disagreed with your manager about a priority or
technical decision. How did you handle it?"

**Evaluates**: Calibrated assertiveness without insubordination, knowing when to escalate
versus defer, executing on decisions you disagree with without undermining them, and learning
from outcomes when the other party had organizational context you lacked.

**Hints**:
- **Level 1**: "Think about a time you believed your manager was wrong about something
  important. What made it hard to say so directly -- the stakes, the relationship, or
  uncertainty about whether you were right?"
- **Level 2**: "Focus on two things: (1) how you raised the concern -- what you said, when,
  and in what setting -- and (2) what you did after the decision was made, regardless of
  whether it went your way."
- **Level 3**: "Strong answers demonstrate calibrated assertiveness: you raised the concern
  clearly and once (not repeatedly or passive-aggressively), you made sure your manager heard
  the full picture, and then you accepted the decision or escalated appropriately. Describe
  what you learned about the line between advocating for your view and undermining a decision."
- **Level 4**: "Example structure: 'My manager decided [X]. I believed [Y] was better because
  [specific reasoning]. I asked for a 20-minute conversation to present my case. My manager
  heard me and proceeded with X because [their reasoning, stated fairly]. I executed on X by
  doing [Z]. The outcome was [specific result]. What I learned: [specific insight about timing,
  framing, or knowing when to let go].'"

**Example STAR Answer**:

**Situation**: In early 2024, my manager decided to pause the migration of our logging
infrastructure from a self-hosted ELK stack to a managed observability platform. He cited the
cost ($14K per month) and said it was not the right quarter to add spend. I had spent two months
building the internal case for the migration and believed the maintenance cost was being
significantly underestimated.

**Task**: I needed to make sure my manager had the full picture before the decision was
finalized -- without lobbying repeatedly or creating the impression I was going around him.

**Action**: I asked for one more conversation, framing it as "I want to make sure you have the
full cost numbers before this closes." I came with a single page of data: the engineering-time
cost of ELK maintenance from the past quarter (110 engineer-hours, or roughly $16K in loaded
cost at our internal rate -- more than the managed service's monthly fee), and a summary of
three incidents in the past six months where the ELK cluster went down during production,
costing us observability exactly when we needed it. I stated my recommendation once. My manager
thanked me for the data and decided to pause the migration anyway -- there was an org-wide
budget freeze he could not share with the team at the time.

**Result**: The migration was approved the following quarter when the freeze lifted. My manager
told me later that the one-page summary I had prepared was what he used to reopen the
conversation with his VP. The migration shipped on time and reduced incident mean-time-to-
detection by 35%. What I learned: my manager often has organizational context he cannot share
in the moment. Raising a concern clearly once, with data, is the right move. Raising it
repeatedly signals I do not trust him to weigh it correctly, which erodes the relationship
faster than the decision itself.

**Follow-up Questions**:
- "How did you manage executing on the decision while privately believing the pause was wrong?"
- "If the pause had caused a serious incident, how would you have handled the post-mortem
  conversation with your manager?"
- "What would you do differently if you faced the same situation today?"

---

## Quick Reference: Scenario-to-Competency Mapping

| Scenario | Primary Competency | Power Dynamic | Best For Level |
|----------|--------------------|---------------|----------------|
| Peer Technical Disagreement | Conflict Articulation / Intellectual Humility | Lateral (same level) | All Levels |
| Cross-Functional Stakeholder Pushback | Communication Choice / Translation | Lateral (different function) | Mid+ |
| Manager Pushback | Calibrated Assertiveness / Deference | Upward | All Levels |
