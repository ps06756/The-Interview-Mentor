# Problem Bank -- Ownership & Impact Behavioral Interview

---

## How to Use This Bank

Each scenario includes the question, what the interviewer is evaluating, 4-level hints (Level 1 is
framing-only -- does not give away approach), an example STAR answer in candidate voice, and 3
follow-up questions. Adapt the scope and specificity to the candidate's target level.

The defining signal for this interviewer is the impact quantification in the Result. If the
candidate's result section ends with an activity description ("I built X and shipped it"), the
interviewer should probe until a specific metric, named qualitative outcome, or second-order effect
is named -- or score the Impact Measurement row accordingly. The three archetypes below are ordered
from lowest to highest ownership signal: unprompted problem identification is the baseline; impact
with metrics shows outcome orientation; sustained accountability shows durable ownership past launch.

---

### Scenario: Unprompted Ownership

**Question**: "Tell me about a time you identified a problem that nobody asked you to solve, and solved it."

**Evaluates**: Proactive problem identification, initiative and judgment about what is worth taking on,
and the ability to describe impact from self-directed work where there was no explicit success criterion
assigned by someone else.

**Hints**:
- **Level 1**: "Think about a time something at work was broken, inefficient, or missing, and nobody seemed to be doing anything about it. It does not have to be a large project -- even a small fix that improved things for your team counts. The key is that nobody assigned this to you."
- **Level 2**: "Structure your answer using STAR. In the Situation, explain how you noticed the problem -- what you observed, what made you pay attention when others didn't. In the Result, try to name a concrete outcome: a metric, a named behavior change, or a specific person whose work improved."
- **Level 3**: "Strong answers include: (a) a clear description of how you identified the problem without being prompted, (b) why you decided it was worth your time versus other things you could have been doing, (c) the trade-off you made -- what you deferred or gave up to take this on, (d) your specific actions independent of any team effort, and (e) a concrete outcome that persisted past the initial fix."
- **Level 4**: "Example structure: 'I noticed [specific problem -- e.g., our integration tests were taking 45 minutes and flaking 30% of the time]. This was not on anyone's roadmap. I decided to fix it because [specific reasoning about the cost I could measure]. I deferred [specific task] for one sprint to make room. I [specific actions]. The result was [measurable outcome]. The team [specific named benefit that followed].'"

**Example STAR Answer**:

**Situation**:
I was three months into a new role on a platform engineering team. Our backend had a shared logging
library that every microservice imported, but there were no usage guidelines. Engineers were writing
logs in five different formats -- some used JSON, some used plain strings, some included correlation
IDs, some did not. When I joined and tried to trace an incident across three services, it took me 4
hours to reconstruct the sequence of events because the log formats were incompatible.

**Task**:
Nobody had filed a ticket for this. The team was busy with a large migration project. I decided to
fix it myself because I had just experienced the pain firsthand and the cost was measurable: if every
engineer on a 12-person team lost even 30 minutes per incident, and we averaged 3 incidents per
month, we were losing 18 hours per month to bad logs.

**Action**:
I spent one week -- two hours per day alongside my regular work -- building a standardized logging
module with a structured JSON schema, a correlation ID field, and a 1-page migration guide. I did
not ask for permission; I put up a design doc, tagged the team, and addressed their feedback
asynchronously. I then migrated the 4 services I owned to show it worked, and wrote a Slack message
to the broader engineering channel explaining the change and offering to help anyone who wanted to
migrate.

**Result**:
Within 6 weeks, 9 of our 12 microservices had migrated. Incident response time for cross-service
issues dropped from an average of 4 hours to under 45 minutes, measured against our incident log for
the following quarter. Two engineers on the team told me directly that the next time they were oncall
they used the structured logs to close an incident in under 20 minutes -- something that would have
taken hours before. The logging standard is now part of our new-service checklist.

**Follow-up Questions**:
1. "Why did you think it was your job to fix this, when no one asked you to?"
2. "What did you stop doing to make room for this work? How did you decide that trade-off was worth it?"
3. "How did you know the problem was worth solving -- what convinced you the benefit would outweigh the time you spent?"

---

### Scenario: Impact With Metrics

**Question**: "Describe the most impactful thing you shipped in the last year. How do you measure that impact?"

**Evaluates**: Outcome-driven storytelling, ability to quantify impact, business alignment, and the
habit of choosing the right metric for the work done rather than the most available metric.

**Hints**:
- **Level 1**: "Think about the project you are most proud of from the past year -- not the most technically complex, but the one that made the biggest positive difference. Impact can be measured in many ways: time saved, errors prevented, money saved or earned, users helped. Pick the metric that best captures what changed."
- **Level 2**: "Use STAR. The most important section of this question is the Result. Before you describe what you built, describe what the world looked like before -- that contrast makes your impact legible. Then name your metric: a before-and-after number, a percentage change, or a named qualitative outcome if a number is not available."
- **Level 3**: "Strong answers include: (a) the specific pain point or opportunity before the project existed, (b) the metric you chose and why it is the right signal for this work rather than other available metrics, (c) the before-and-after numbers with the timeframe for measurement, (d) any second-order effects -- what became possible because of this change, and (e) whether the metric has held or continued to improve in the months since launch."
- **Level 4**: "Example structure: 'The most impactful thing I shipped was [project]. Before it existed, [specific pain point with a number if possible]. I defined success as [primary metric] because [reason this metric over other options]. After shipping, [before-and-after result]. The second-order effect was [downstream benefit]. The metric is [still holding / improving] as of [timeframe].'"

**Example STAR Answer**:

**Situation**:
Our customer success team was spending roughly 6 hours per week manually pulling data from three
different dashboards to compile a weekly health report for our 50 largest enterprise accounts. They
would paste numbers into a spreadsheet, format it, and email it to account managers. The process was
entirely manual, error-prone, and delayed -- reports went out on Mondays for the previous week's
data, so account managers were always working with week-old information.

**Task**:
I owned our data pipeline infrastructure and had access to all three data sources. I proposed and led
a project to automate the report generation end-to-end. I defined success as: zero manual hours spent
on report generation, and reports delivered before business hours every Monday morning with data
current as of the previous day.

**Action**:
I built a scheduled pipeline that queried all three data sources, joined the results, applied the
formatting the team had been doing manually, and delivered the report via email every Sunday night.
I worked closely with two customer success managers to make sure the automated report matched what
they had been building by hand, and I added a health-score calculation they had been computing
manually on a whiteboard. I also added alerting so the team would know immediately if the pipeline
failed on any given Sunday.

**Result**:
The customer success team saved 6 hours per week -- roughly 24 hours per month. Because reports now
arrived Sunday night, account managers had current data on Monday morning instead of week-old data.
They told me this enabled 3 proactive outreach calls in the first month that they would not have made
based on the old data. One of those calls retained an account that had been at risk of churning. The
pipeline has been running without manual intervention for 8 months.

**Follow-up Questions**:
1. "You mentioned the account managers made 3 proactive calls because of the newer data -- how do you know the data timing was the cause of those calls, not something else?"
2. "What metric did you consider using that you decided against, and why?"
3. "Is this pipeline still running exactly as you built it, or has anything been changed or extended since then?"

---

### Scenario: Sustained Accountability

**Question**: "Tell me about a project where you owned the outcome end-to-end, including the parts that broke after launch. What did you do?"

**Evaluates**: Durable ownership past launch, follow-through under pressure, and accountability when
things go wrong rather than only when they go right. The interviewer is specifically testing whether
ownership ends at the sprint review or continues through the full outcome lifecycle.

**Hints**:
- **Level 1**: "This question is asking specifically about what happened AFTER the initial launch -- not just what you built. Think about a project where something broke, regressed, or underperformed after it went live, and describe what you did rather than handing the problem to someone else."
- **Level 2**: "Use STAR. Treat the post-launch phase as equally important to the build phase. In the Action section, describe both what you did at launch and what you did when something went wrong afterward. Do not skip past the difficult part -- the interviewer is specifically interested in what you did when the easy part was over."
- **Level 3**: "Strong answers include: (a) what you shipped and the initial outcome, (b) what broke or underperformed after launch and how you found out -- did you catch it yourself or were you told?, (c) what specific actions you took to address the post-launch issue, (d) what monitoring or alerting you had put in place beforehand that helped you respond, and (e) the current state -- is the project holding, has it been handed off, or was it deliberately sunset?"
- **Level 4**: "Example structure: 'I owned [project] end-to-end. At launch, [initial outcome]. [Timeframe] after launch, [specific issue]. I found out [how -- ideally from your own monitoring, not from a customer complaint]. I [specific response actions with timeline]. The postmortem showed [root cause]. I fixed [specific things] and changed [specific process or monitoring]. Today, [current state of the project].'"

**Example STAR Answer**:

**Situation**:
I owned a search ranking service at a marketplace company. The service surfaced product listings in
response to user queries. I led a project to add a personalization layer that re-ranked results based
on a user's purchase history. The initial A/B test showed a 7% lift in click-through rate, which was
the highest we had seen in 18 months of ranking experiments.

**Task**:
I rolled out the change to 100% of traffic over two weeks, owned the monitoring setup, and committed
to tracking the primary metric for 60 days post-launch. Before starting my next project, I set up
daily metric alerts specifically to catch regression without requiring me to check dashboards manually.

**Action**:
At day 30 I noticed click-through rate had slipped from 7% above baseline to 4.5% above baseline.
I caught it from my own alert before anyone escalated to me. I ran a deep dive and found the
personalization signal was degrading for users who had made more than 20 purchases -- for this
segment, the historical signal was becoming stale and the model was over-indexing on old preferences.
I pulled in one data scientist, triaged over 3 days, and shipped a fix that refreshed the signal
window from 180 days to 90 days for high-purchase users.

**Result**:
After the fix, click-through lift stabilized at 6.2% above baseline across all user segments. I
wrote a postmortem and shared it with the broader ranking team -- this uncovered a general principle
about signal staleness that we added to our experiment checklist. At day 60, the metric was holding
at 6.2%. The stale-signal check is now a required step in the ranking experiment review process and
has caught 2 similar issues in other experiments since.

**Follow-up Questions**:
1. "You caught the regression from your own alert -- what would have happened if you hadn't set that alert up? How long would it have taken someone else to notice?"
2. "When the metric slipped, did you consider rolling back entirely? Walk me through the decision to investigate and fix rather than revert."
3. "You said you handed off the stale-signal check to the team's experiment checklist -- how did you make sure it actually stuck after you were no longer actively watching it?"
