# The Mentor

> **30 AI Mock Interviewers for Software Engineering Interview Prep**

Practice realistic technical interviews with AI agents that adapt to your level, give progressive hints when you're stuck, and evaluate your performance with detailed scorecards.

**30 skills** | **7 domains** | **Easy to Staff+ difficulty** | **Works with Claude Code, ChatGPT, Cursor, and more**

---

## Quick Start

### Option 1: Clone and run locally (recommended)

```bash
git clone https://github.com/ps06756/The-Mentor.git
cd The-Mentor
```

Then start a mock interview by loading any skill category:

```bash
# System design interviews (13 skills)
claude --plugin-dir ./agents/systems-design

# Entry-level coding interviews (4 skills)
claude --plugin-dir ./agents/swe-i

# Mid-level algorithm interviews (2 skills)
claude --plugin-dir ./agents/swe-ii

# Data engineering interviews (5 skills)
claude --plugin-dir ./agents/data-engineer

# DevOps & SRE interviews (3 skills)
claude --plugin-dir ./agents/devops-sre

# ML engineering interviews (2 skills)
claude --plugin-dir ./agents/ml-engineer

# Behavioral interviews (1 skill)
claude --plugin-dir ./agents/behavioral
```

Once loaded, start your session:

> *"Use the uber-interviewer skill and start my mock interview."*

### Option 2: Use with ChatGPT, Claude.ai, or any AI assistant

1. Open any skill's `SKILL.md` file (e.g., [`agents/systems-design/uber-interviewer/SKILL.md`](./agents/systems-design/uber-interviewer/SKILL.md))
2. Copy the entire contents
3. Paste it into your AI assistant as the system prompt or first message
4. Say: *"Start the interview"*

### Option 3: Use with Cursor or Cline (VS Code)

1. Copy a skill directory into your workspace (e.g., `agents/systems-design/uber-interviewer/`)
2. Reference the SKILL.md file: *"Use the instructions in `@agents/systems-design/uber-interviewer/SKILL.md` to conduct a mock interview with me."*

---

## Where Do I Start?

With 30 skills, here's how to pick the right one:

| Your situation | Start here |
|---------------|------------|
| **New grad / first SWE job** | [Arrays & HashMaps](./agents/swe-i/arrays-hashmaps-interviewer/SKILL.md), then [Linked Lists](./agents/swe-i/linked-lists-interviewer/SKILL.md) |
| **Preparing for SWE-II / mid-level** | [URL Shortener](./agents/systems-design/url-shortener-interviewer/SKILL.md) (classic starter), then [Dynamic Programming](./agents/swe-ii/dynamic-programming-interviewer/SKILL.md) |
| **Targeting Senior / Staff** | [Design Uber](./agents/systems-design/uber-interviewer/SKILL.md), [Design Twitter](./agents/systems-design/twitter-interviewer/SKILL.md), [Distributed Systems](./agents/systems-design/distributed-systems-interviewer/SKILL.md) |
| **Data Engineer role** | [SQL Optimization](./agents/data-engineer/sql-optimization-interviewer/SKILL.md), then [Pipeline Architect](./agents/data-engineer/pipeline-architect-interviewer/SKILL.md) |
| **DevOps / SRE role** | [Kubernetes](./agents/devops-sre/kubernetes-interviewer/SKILL.md), [Monitoring & Alerting](./agents/devops-sre/monitoring-alerting-interviewer/SKILL.md) |
| **ML Engineer role** | [ML System Design](./agents/ml-engineer/ml-system-design-interviewer/SKILL.md), [Deep Learning](./agents/ml-engineer/deep-learning-interviewer/SKILL.md) |
| **Behavioral / culture fit prep** | [Leadership Principles](./agents/behavioral/leadership-principles-interviewer/SKILL.md) |

For a structured 8-week curriculum, see the [Learning Path](./references/learning-path.md).

---

## All 30 Skills

### Entry Level (SWE-I / SWE-Intern)

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [Arrays & HashMaps](./agents/swe-i/arrays-hashmaps-interviewer/SKILL.md) | Data Structures | Easy-Medium | Two pointers, sliding window, frequency counting |
| [Linked Lists](./agents/swe-i/linked-lists-interviewer/SKILL.md) | Data Structures | Easy | Reversal, merging, cycle detection |
| [Binary Trees](./agents/swe-i/binary-trees-interviewer/SKILL.md) | Trees | Easy-Medium | Traversals, BFS/DFS, basic operations |
| [Recursion Basics](./agents/swe-i/recursion-basics-interviewer/SKILL.md) | Algorithms | Easy | Base cases, call stacks, simple problems |

### Mid Level (SWE-II / Backend)

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [URL Shortener](./agents/systems-design/url-shortener-interviewer/SKILL.md) | System Design | Medium | Distributed systems, scaling, trade-offs |
| [Rate Limiter](./agents/systems-design/rate-limiter-interviewer/SKILL.md) | System Design | Medium | Token bucket, sliding window, Redis |
| [Graph Algorithms](./agents/swe-ii/graph-algorithms-interviewer/SKILL.md) | Algorithms | Medium | BFS, DFS, Dijkstra, topological sort |
| [Dynamic Programming](./agents/swe-ii/dynamic-programming-interviewer/SKILL.md) | Algorithms | Medium-Hard | Memoization, tabulation, common patterns |

### Data Engineering

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [SQL Optimization](./agents/data-engineer/sql-optimization-interviewer/SKILL.md) | Database | Medium-Hard | Indexing, query plans, schema design |
| [Pipeline Architect](./agents/data-engineer/pipeline-architect-interviewer/SKILL.md) | Data Engineering | Medium-Hard | End-to-end pipelines, Kafka/Flink, scaling, failure modes |
| [Schema Design](./agents/data-engineer/schema-design-interviewer/SKILL.md) | Data Engineering | Medium-Hard | Dimensional modeling, SCDs, star schema, query optimization |
| [Data Pipeline Design](./agents/data-engineer/data-pipeline-design-interviewer/SKILL.md) | Data Engineering | Medium | ETL/ELT, Apache Airflow, data quality |
| [Data Modeling](./agents/data-engineer/data-modeling-interviewer/SKILL.md) | Data Engineering | Medium | Star schema, snowflake, data warehouses |

### Systems Architecture & Distributed Systems

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [Database Architecture](./agents/systems-design/database-architecture-interviewer/SKILL.md) | Databases | Medium-Hard | SQL vs NoSQL, Indexing, ACID, Sharding |
| [Microservices Architecture](./agents/systems-design/microservices-architecture-interviewer/SKILL.md) | Architecture | Medium-Hard | DDD, API Gateways, Sagas, Resilience |
| [Distributed Systems Core](./agents/systems-design/distributed-systems-interviewer/SKILL.md) | Dist. Systems | Hard | CAP Theorem, Quorums, Consensus, Clocks |
| [Caching Architecture](./agents/systems-design/caching-architecture-interviewer/SKILL.md) | Caching | Medium-Hard | Topologies, Eviction, Consistency, Stampedes |
| [Message Queues](./agents/systems-design/message-queues-interviewer/SKILL.md) | Messaging | Medium-Hard | Kafka vs RabbitMQ, DLQs, Idempotency, Ordering |
| [API Design & Gateways](./agents/systems-design/api-design-interviewer/SKILL.md) | API Design | Medium | REST, Pagination, Auth, API Gateways |
| [Networking & Load Balancing](./agents/systems-design/networking-load-balancing-interviewer/SKILL.md) | Networking | Medium-Hard | OSI Layers, L4/L7 LBs, TLS, Consistent Hashing |

### DevOps / SRE

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [Reliability & Observability](./agents/systems-design/reliability-observability-interviewer/SKILL.md) | Reliability | Medium-Hard | Circuit Breakers, RED Metrics, RTO/RPO, Retries |
| [Kubernetes Fundamentals](./agents/devops-sre/kubernetes-interviewer/SKILL.md) | Infrastructure | Medium | Pods, services, deployments |
| [CI/CD Pipeline Design](./agents/devops-sre/cicd-pipeline-interviewer/SKILL.md) | DevOps | Medium | GitHub Actions, Jenkins, testing strategies |
| [Monitoring & Alerting](./agents/devops-sre/monitoring-alerting-interviewer/SKILL.md) | SRE | Medium | Prometheus, Grafana, SLIs/SLOs/SLAs |

### Machine Learning Engineer

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [ML System Design](./agents/ml-engineer/ml-system-design-interviewer/SKILL.md) | ML Engineering | Hard | Feature stores, model serving, A/B testing |
| [Deep Learning Interview](./agents/ml-engineer/deep-learning-interviewer/SKILL.md) | ML Theory | Hard | CNNs, RNNs, Transformers, training dynamics |

### Senior+ Level (SWE-III / Senior / Staff)

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [Design Twitter/X](./agents/systems-design/twitter-interviewer/SKILL.md) | System Design | Hard | Feed generation, fan-out, consistency |
| [Design Uber](./agents/systems-design/uber-interviewer/SKILL.md) | System Design | Hard | Real-time tracking, matching, maps |
| [Design a Search Engine](./agents/systems-design/search-engine-interviewer/SKILL.md) | System Design | Hard | Indexing, ranking, query understanding |
| [Leadership Principles](./agents/behavioral/leadership-principles-interviewer/SKILL.md) | Behavioral | All Levels | STAR method, cross-functional collaboration |

---

## How It Works

### What happens in a session

1. **You invoke a skill** — The AI adopts a specific interviewer persona
2. **Warm-up (5-10 min)** — Foundational questions to calibrate your level
3. **Core concepts (15-20 min)** — Deep dive with visual explanations and follow-ups
4. **Problem solving (20-30 min)** — Realistic problems with progressive hints when you're stuck
5. **Scorecard (5 min)** — Evaluation across multiple dimensions (Novice / Intermediate / Expert) with specific strengths, improvement areas, and resource recommendations

### The hint system

When you're stuck, ask for a hint. The interviewer provides hints at increasing levels:

| Level | Type | Example |
|-------|------|---------|
| **1** | Gentle Nudge | *"Think about the time complexity. What data structure gives O(1) lookups?"* |
| **2** | Direction | *"This sounds like a dynamic programming problem. Can you identify the subproblems?"* |
| **3** | Partial Solution | *"Try using two pointers - one at start, one at end, moving towards each other."* |
| **4** | Full Walkthrough | Step-by-step explanation with pseudocode |

**Pro tip**: Try to solve with Level 1 hints first. The struggle is where learning happens!

### Visual learning

Every skill includes ASCII diagrams for complex concepts:

```
Two Pointers Pattern:
Array: [1, 2, 3, 4, 5, 6], Target: 7

Left ->                    <- Right
  1     2  3  4  5     6
  1+6=7  Found!
```

Each skill also includes [Remotion](https://www.remotion.dev/) React components in `references/remotion-components.md` that can be rendered to video for study groups or tutorial content.

---

## Example Session

**You**: Use the arrays-hashmaps-interviewer skill and start interviewing me

**AI**: Hey there! Let's jump right in with a warm-up. What's the time complexity of looking up an element in a HashMap, and why?

**You**: I think it's O(1)?

**AI**: Exactly! Though technically it's amortized O(1) — in the worst case with many collisions, it could be O(n). Good. Now, when would you NOT want to use a HashMap?

*[45 minutes of guided practice with hints when needed]*

**AI**: Great session! Here's your evaluation:

| Area | Rating | Notes |
|------|--------|-------|
| Problem Understanding | Expert | Asked excellent clarifying questions |
| Solution Approach | Intermediate | Considered trade-offs but missed optimal |
| Code Quality | Intermediate | Clean and readable |
| Edge Cases | Novice | Missed empty input handling |

**Strengths**: Strong problem decomposition, good communication...
**Improve**: Edge case coverage, complexity analysis of alternative approaches...
**Resources**: LeetCode 76 (Minimum Window Substring), NeetCode sliding window playlist...

---

## Interview Preparation Roadmap

For the full structured curriculum, see the [Learning Path](./references/learning-path.md).

### 8-Week Plan (SWE Track)

| Weeks | Focus | Skills |
|-------|-------|--------|
| 1-2 | Foundations | Arrays & HashMaps, Linked Lists, Binary Trees, Recursion |
| 3-4 | Core System Design | URL Shortener, Database Architecture, Caching |
| 5-6 | Advanced Design | API Design, Rate Limiter, Message Queues, Microservices |
| 7-8 | Senior/Staff Level | Uber, Twitter, Distributed Systems, Networking, Reliability |

### Data Engineering Track

SQL Optimization -> Data Pipeline Design -> Pipeline Architect -> Data Modeling -> Schema Design

### DevOps / SRE Track

Kubernetes -> CI/CD Pipeline -> Monitoring & Alerting -> Reliability & Observability

### ML Engineer Track

Deep Learning -> ML System Design

### Tips for Best Results

- **One skill per session** — Don't rush through multiple interviewers in one sitting
- **Think out loud** — Practice verbalizing your thought process as in a real interview
- **Try Level 1 hints first** — The struggle is where learning happens
- **Review your scorecard** — Focus next practice on your lowest-rated dimensions
- **Revisit after 2 weeks** — Spaced repetition solidifies understanding

---

## For Contributors

### Adding a New Skill

1. **Fork the repository**
2. **Copy the template**:
   ```bash
   cp -r templates/skill-template agents/{category}/{skill-name}-interviewer
   ```
3. **Fill in the template** — Replace all `{{placeholders}}` with your content
4. **Test your skill** with `claude --plugin-dir ./agents/{category}`
5. **Submit a PR** with test notes showing it works

### Skill Quality Checklist

- [ ] Clear, consistent persona defined
- [ ] 3-4 difficulty-appropriate problems with 4-level hints each
- [ ] At least 2 ASCII diagrams
- [ ] Evaluation rubric (Novice / Intermediate / Expert format)
- [ ] Resources section with further reading
- [ ] Tested with at least one AI assistant

### Directory Structure

```
The-Mentor/
├── README.md
├── LICENSE
├── references/
│   └── learning-path.md          # 8-week curriculum
├── templates/
│   └── skill-template/           # Template for creating new skills
│       ├── SKILL.md
│       └── references/
│           ├── problems.md
│           └── remotion-components.md
├── agents/
│   ├── swe-i/                    # 4 entry-level skills
│   ├── swe-ii/                   # 2 mid-level algorithm skills
│   ├── data-engineer/            # 5 data engineering skills
│   ├── systems-design/           # 13 system design skills
│   ├── devops-sre/               # 3 DevOps & SRE skills
│   ├── ml-engineer/              # 2 ML engineering skills
│   └── behavioral/               # 1 behavioral skill
```

Each skill directory contains:
```
skill-name-interviewer/
├── SKILL.md                      # Main interviewer instructions
└── references/
    ├── problems.md               # Problem bank with walkthroughs
    └── remotion-components.md    # Animation components
```

---

## Related Projects

- [System Design Primer](https://github.com/donnemartin/system-design-primer) - Learn system design
- [NeetCode](https://neetcode.io/) - Practice problems by pattern
- [Blind 75](https://www.teamblind.com/post/New-Year-Gift---Curated-List-of-Top-75-LeetCode-Questions-to-Save-Your-Time-OaM1orEU) - Essential problem list

---

## License

MIT License - see [LICENSE](./LICENSE) file for details.

---

<p align="center">
  <strong>Ready to ace your next interview?</strong><br>
  Pick a skill from the roster above and start practicing!
</p>
