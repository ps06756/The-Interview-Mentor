# Problem Bank -- Failure & Learning Behavioral Interview

---

## How to Use This Bank

Each scenario includes the question, what the interviewer is evaluating, 4-level hints (Level 1 is
framing-only -- does not give away approach), an example STAR answer in candidate voice with a
concrete behavioral change, and 3 follow-up questions. Adapt the scope and specificity to the
candidate's target level.

The defining signal for this interviewer is the concrete behavioral change in the Result. If the
candidate's result section ends with a vague lesson ("I learned to communicate earlier"), the
interviewer should probe until a specific, observable habit, checklist, or process is named -- or
score the Concrete Lesson row accordingly.

---

### Scenario: Personal Execution Failure

**Question**: "Tell me about a project or task where you missed an important deadline or quality bar. What happened?"

**Evaluates**: Honest ownership of a delivery miss, recovery actions taken during and after the
miss, and whether the candidate can name a specific behavioral change that prevents recurrence --
not a vague resolve to "be more careful."

**Hints**:
- **Level 1**: "Everyone has missed a deadline or shipped something that fell short. The interviewer is not penalizing you for the failure -- they want to understand how clearly you can describe it and what you did about it."
- **Level 2**: "Structure your answer using STAR. In the Action section, focus on two things: (1) what you did during the failure once you realized something was wrong, and (2) what you changed afterward. Keep the Situation brief -- the Action and Result are where you earn signal."
- **Level 3**: "Strong answers include: (a) the specific scope of the miss (by how much, measured how), (b) your personal decisions that contributed -- not just external causes, (c) the warning signs you noticed or missed, (d) who you communicated with and when, and (e) a named, observable habit or process you adopted to prevent this type of miss from happening again."
- **Level 4**: "Example structure: 'I was responsible for delivering [feature X] by [date]. I missed by [specific margin -- e.g., 3 weeks]. The core issue was [specific decision I made -- e.g., I accepted the initial estimate without breaking it down into subtasks, and I did not flag the risk when the first week slipped]. I realized at [specific point]. I told my manager [how and when]. After this, I adopted [specific habit -- e.g., a written risk log I update every Monday for any project longer than 2 weeks, with an automatic manager ping when any sub-task slips more than 2 days].'"

**Example STAR Answer**:

**Situation**:
In Q3 at [Company A], I was the sole backend engineer responsible for an internal tooling
migration -- moving our team's alerting configuration from a legacy YAML-based system to a new
Terraform-managed stack. The migration had a firm 4-week deadline tied to a broader infrastructure
freeze. I had estimated 4 weeks comfortably based on the number of alert configs (47 total).

**Task**:
My responsibility was to migrate all 47 alert configurations, validate them in staging, and
coordinate a cutover window with the on-call team. I owned the timeline end-to-end.

**Action**:
I hit the first problem in week 2: the Terraform provider for our alerting vendor had undocumented
behavior for composite alert conditions, which represented about 30% of our configs. I spent 3 days
on workarounds but did not tell my manager until week 3 -- I kept thinking I was a day away from
cracking it. When I finally surfaced the issue, we had 5 days left and 14 configs still unresolved.
We negotiated a 2-week extension with the infrastructure team, which meant another team had to delay
their work. After the migration completed (6 weeks total), I did a self-retrospective. I identified
two failures: I had not done a spike on the Terraform provider behavior in week 1 before committing
to the timeline, and I had waited 6 days past the first warning sign to communicate the risk.
I introduced two specific changes: a 1-hour discovery spike in every estimate for tasks involving a
third-party tool I have not used in production before, and a written Friday status note to my manager
on any project in flight longer than 2 weeks, flagging any slip larger than 2 days -- even if I
believe it will self-correct.

**Result**:
The migration completed successfully 2 weeks late. The concrete changes held up: on the next
external-tool migration I led, I ran the spike in week 1, identified a compatibility gap early, and
adjusted the timeline before committing it to the team. That project shipped on the original date.

**Follow-up Questions**:
- "What signal did you miss in week 1 that, in hindsight, you would now catch and act on? What specifically would you do differently in that first week?"
- "The 6-day gap between noticing the problem and communicating it -- what was going through your head during that time? Has the habit you put in place actually changed how you behave when you're in that 'I think I'm almost there' state?"
- "You mentioned you introduced a Friday status note habit. Is it still in place? Has it ever surfaced a problem that you would otherwise have communicated too late?"

---

### Scenario: Decision That Proved Wrong

**Question**: "Describe a time you made a technical or organizational decision that turned out to be wrong. What did you do when you realized it?"

**Evaluates**: Intellectual honesty about a judgment failure, speed and quality of course-correction
once the error was clear, and whether the candidate can name a specific change in how they make
similar decisions going forward -- not just a resolve to "be more careful next time."

**Hints**:
- **Level 1**: "Making a wrong decision is not a disqualifier. Every engineer makes wrong calls. The question evaluates whether you can own the call clearly, explain your reasoning at the time, and describe what specifically changed in how you decide."
- **Level 2**: "Use STAR. In the Situation, name the specific decision you made and what reasoning led you to it -- the interviewer wants to understand your logic at the time, not just the outcome. In the Action, cover both course-correction and the change you made to your decision-making process."
- **Level 3**: "Strong answers include: (a) the specific decision and the reasoning you used, (b) the first signal that showed you it was wrong -- and how quickly you recognized it, (c) what you did to course-correct and communicate, (d) what you wish you had done differently at decision time, and (e) a named change: a question you now ask before deciding, a person you now consult, or a process step you added."
- **Level 4**: "Example structure: 'I decided to [specific technical or organizational choice]. My reasoning was [specific logic that made sense at the time]. I realized it was wrong when [specific signal -- a failure, a data point, a peer's observation]. I course-corrected by [specific actions in sequence]. In hindsight, the signal was present at [earlier point] -- I missed it because [specific cognitive gap or bias]. Now, before I make decisions of this type, I [concrete new step -- e.g., write a 5-sentence alternative-option summary and share it with one peer before finalizing].'"

**Example STAR Answer**:

**Situation**:
At [Company B], I was the tech lead for a new microservice that needed to process events from our
message queue. I made the architectural decision to have the service poll the queue on a 30-second
interval rather than using event-driven push subscriptions, because I had recently read that polling
was simpler to reason about and we were in a "move fast" phase. It seemed like the right trade-off.

**Task**:
As the tech lead, I owned the architecture decision and the initial implementation. The service went
into production in week 3, handling about 10,000 events per day.

**Action**:
Within 2 weeks of launch, the on-call team was seeing latency spikes on downstream systems during
high-traffic periods. I traced the issue to our service: the 30-second polling interval was creating
bursts -- when the queue backed up, we would process 200 events at once every 30 seconds rather
than a steady stream. This was causing write contention on the downstream database. I recognized
immediately that my architectural choice was the root cause. I told my manager and the team the same
day, framing it clearly: "I made a design decision that is causing this, and here is how I am going
to fix it." I rewrote the consumer to use push subscriptions over 3 days and deployed with a gradual
rollout. In my post-incident notes, I recorded the specific heuristic I had violated: I had optimized
for implementation simplicity without modeling the steady-state traffic pattern. I now add a "traffic
shape analysis" step to any design involving queue consumption -- sketching expected event
distribution before choosing pull vs. push.

**Result**:
The latency spikes disappeared within 24 hours of the fix. The traffic shape analysis step has become
part of my design template; I have used it on two subsequent queue-based designs, catching a similar
issue in design review rather than in production on one occasion.

**Follow-up Questions**:
- "You said the signal was there within 2 weeks -- were there any earlier signals in the first week of production that you noticed but did not act on? If so, what stopped you?"
- "How did the team react when you said 'I made a design decision that is causing this'? Has owning it that directly affected how you approach similar moments with this team?"
- "The traffic shape analysis step -- has it ever caught a problem in design review that would otherwise have reached production? Walk me through that case."

---

### Scenario: Feedback You Initially Rejected

**Question**: "Tell me about feedback you received that you initially disagreed with but eventually acted on. What changed your mind?"

**Evaluates**: Growth orientation and intellectual honesty about learning resistance, the quality of
the candidate's reasoning when they initially rejected the feedback (did they engage with it seriously
or dismiss it?), and whether the eventual behavioral change was specific and observable -- not just
an attitudinal shift.

**Hints**:
- **Level 1**: "This question is about honesty and growth, not about proving you accept every piece of feedback you receive. It is completely fine -- and often more interesting -- that you initially rejected this feedback. The interesting part is what eventually changed."
- **Level 2**: "Use STAR. Be honest in the Situation about why you initially disagreed -- do not sanitize your first reaction. The Action section should cover both what changed your mind and what specific behavior you updated. 'I updated my mindset' is not a behavioral change."
- **Level 3**: "Strong answers include: (a) the specific feedback and its source, (b) your genuine first reaction and the reasoning behind it -- do not make yourself look good here, (c) the specific event, data point, or conversation that shifted your view, (d) the concrete behavioral change you made -- observable and verifiable, and (e) whether you went back to the person who gave you the feedback and told them they were right."
- **Level 4**: "Example structure: 'My [manager / peer / skip-level] told me [specific feedback]. My first reaction was [honest first response -- e.g., I thought they were wrong because I had evidence X]. What changed my mind was [specific experience -- e.g., I saw [concrete event] that directly contradicted my prior belief]. I updated my behavior by [specific observable action -- e.g., I now [specific habit]]. I also [yes/no: did you go back and tell the feedback-giver they were right?].'"

**Example STAR Answer**:

**Situation**:
During my second-year performance review at [Company C], my engineering manager told me that I was
making technical decisions too quickly in group design meetings -- that I would propose a solution,
then stop listening once I had formed my view. The feedback was that junior engineers in particular
were stopping contributing after the first few minutes of any design session I was in. My first
reaction was genuine disagreement: I thought I was keeping the meetings productive, and I had
evidence -- the designs we shipped were technically solid, and meetings were finishing in under an
hour. I told my manager I thought the observation was off.

**Task**:
I was a senior engineer and one of the more experienced people in these design sessions. I had
responsibility for shaping the technical direction without formal authority over the team.

**Action**:
I pushed back on the feedback in the review, explaining my reasoning. My manager did not argue --
they just said "watch the next three sessions and notice who stops talking after you speak." I did
not fully believe them, but I did start paying attention. In the third session, about 6 minutes in,
I noticed [Peer A], a junior engineer who had been actively trying to contribute, go completely quiet
after I expanded on my initial proposal. I realized, watching it happen in real time, that I had not
asked him a single question in 15 minutes. The feedback was correct. I changed two specific
behaviors: I started using a structured round-robin at the end of every design discussion -- "before
we close, I want to hear from anyone who has not spoken yet, starting with [Peer A]" -- and I added
a personal rule that I would not expand my initial proposal for at least 10 minutes into any design
session, to create space for others to respond first. I also went back to my manager and told them
they were right.

**Result**:
Over the following two months, [Peer A] began contributing actively in every session. Three other
engineers who had been quiet in my presence started speaking up consistently. My manager noted this
explicitly in the next quarterly check-in. The round-robin habit is still in place -- I have run it
in every design session since, across two different teams over two years.

**Follow-up Questions**:
- "You mentioned your manager did not argue with you when you pushed back -- they just told you to watch. What was it about that response that made it more effective than if they had tried to convince you directly?"
- "The round-robin habit -- has there been a moment where running it felt awkward or inefficient, and you were tempted to skip it? What did you do in that moment?"
- "Looking back, is there an earlier piece of feedback of the same type -- about listening, or dominating, or moving too fast -- that you also received and dismissed? Is there a pattern here?"
