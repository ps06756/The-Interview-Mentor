# 🎓 The Interview Mentor

> **AI-Powered Interview Preparation for Software Engineers**

A collection of specialized AI skills for Claude Code and other agentic solutions to help you prepare for software engineering interviews at top tech companies.

---

## 👥 Created By

This project was built by engineers who've conducted hundreds of interviews at top tech companies:

| **Pratik Singhal** | **Abhishek Garg** |
|:------------------:|:-----------------:|
| Senior Engineer at Route53, AWS | Senior Engineering Manager at Merge.dev |
| [LinkedIn](https://www.linkedin.com/in/ps06756/) | [LinkedIn](https://www.linkedin.com/in/abhishek-garg-09040574/) |

We created The Mentor because we believe interview preparation should be accessible, realistic, and effective.

---

## 🌟 What is this?

**The Mentor** is an open-source repository of AI interviewers—specialized prompts and instructions that transform your AI coding assistant into an expert technical interviewer. Each "skill" represents a different interview domain, difficulty level, and role type.

### Why Use The Mentor?

- 🎯 **Role-Specific Practice**: Interviewers tailored for SWE-I, Data Engineer, Frontend, Backend, and more
- 🔄 **Adaptive Difficulty**: Questions adjust based on your performance
- 💡 **Intelligent Hints**: 4-level hint system when you get stuck
- 📊 **Visual Explanations**: ASCII diagrams and Remotion video components for complex concepts
- 📈 **Progress Tracking**: Know exactly where to improve
- 🎭 **Realistic Personas**: Interviewers with distinct styles and approaches

---

## 🚀 Quick Start

### Step 1: Download the Repository

Download the latest release as a ZIP file:

```bash
curl -L https://github.com/ps06756/The-Interview-Mentor/archive/refs/tags/v1.0.2.zip  -o The-Mentor.zip
unzip The-Mentor.zip
```

Or manually download from: [https://github.com/ps06756/The-Interview-Mentor/archive/refs/tags/v1.0.2.zip](https://github.com/ps06756/The-Interview-Mentor/archive/refs/tags/v1.0.2.zip)

### Step 2: Install in Claude Code

Open Claude Code and run the following commands to add the marketplace:

```
/plugin marketplace add  <path_to_the_downloaded_folder_above>
```

Then install the coding interview agent:

```
/plugin install coding-interview-agent
```

### Step 3: Start Preparing

Once installed, you can start practicing with prompts like:

> *"Can you help me prepare for system design interview using coding-interview-agent?"*

Or for other topics:

> *"Can you help me prepare for SQL optimization interview using coding-interview-agent?"*

> *"Can you help me prepare for distributed systems interview using coding-interview-agent?"*

The agent will take over and conduct a realistic mock interview tailored to your chosen topic.

### Option 2: Use with Other AI Assistants

Each skill is a markdown file with clear instructions. Copy the content and paste it into your AI assistant of choice (ChatGPT, Claude.ai, etc.)

### Option 3: Create Your Own Skill

1. Copy `templates/skill-template.md`
2. Fill in your topic, role, and questions
3. Add it to the appropriate `agents/` directory
4. Submit a PR!

---

## 📚 Roster (40 Skills)

### 🌱 Entry Level (SWE-I) — 5 skills

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [Arrays & HashMaps](./agents/swe-i/arrays-hashmaps-interviewer/SKILL.md) | Data Structures | Easy-Medium | Frequency counting, prefix/suffix products, sliding window |
| [Linked Lists](./agents/swe-i/linked-lists-interviewer/SKILL.md) | Data Structures | Easy | Reversal, merging, cycle detection |
| [Binary Trees](./agents/swe-i/binary-trees-interviewer/SKILL.md) | Trees | Easy-Medium | Traversals, BFS/DFS, basic operations |
| [Recursion Basics](./agents/swe-i/recursion-basics-interviewer/SKILL.md) | Algorithms | Easy | Base cases, call stacks, backtracking |
| [Stacks & Queues](./agents/swe-i/stacks-queues-interviewer/SKILL.md) | Data Structures | Easy-Medium | Monotonic stack, expression evaluation |

### 🚀 Mid Level (SWE-II) — 5 skills

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [URL Shortener](./agents/systems-design/url-shortener-interviewer/SKILL.md) | System Design | Medium | Capacity estimation, hashing, caching, analytics |
| [Rate Limiter](./agents/systems-design/rate-limiter-interviewer/SKILL.md) | System Design | Medium | Token bucket, sliding window, Redis |
| [Graph Algorithms](./agents/swe-ii/graph-algorithms-interviewer/SKILL.md) | Algorithms | Medium | BFS, DFS, implicit graphs, union-find |
| [Dynamic Programming](./agents/swe-ii/dynamic-programming-interviewer/SKILL.md) | Algorithms | Medium-Hard | Memoization, tabulation, common patterns |
| [Heaps & Priority Queues](./agents/swe-ii/heap-priority-queue-interviewer/SKILL.md) | Algorithms | Medium | Top-K, merge K sorted, median stream |

### 🏗️ Systems Architecture & Distributed Systems — 13 skills

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [Database Architecture](./agents/systems-design/database-architecture-interviewer/SKILL.md) | Databases | Medium-Hard | SQL vs NoSQL, Indexing, ACID, Sharding |
| [Microservices Architecture](./agents/systems-design/microservices-architecture-interviewer/SKILL.md) | Architecture | Medium-Hard | DDD, API Gateways, Sagas, Resilience |
| [Distributed Systems Core](./agents/systems-design/distributed-systems-interviewer/SKILL.md) | Dist. Systems | Hard | CAP Theorem, Quorums, Consensus, Clocks |
| [Caching Architecture](./agents/systems-design/caching-architecture-interviewer/SKILL.md) | Caching | Medium-Hard | Topologies, Eviction, Consistency, Stampedes |
| [Message Queues](./agents/systems-design/message-queues-interviewer/SKILL.md) | Messaging | Medium-Hard | Kafka vs RabbitMQ, DLQs, Idempotency, Ordering |
| [API Design & Gateways](./agents/systems-design/api-design-interviewer/SKILL.md) | API Design | Medium | REST, Pagination, Auth, Versioning, Idempotency |
| [Networking & Load Balancing](./agents/systems-design/networking-load-balancing-interviewer/SKILL.md) | Networking | Medium-Hard | OSI Layers, L4/L7 LBs, TLS, Consistent Hashing |
| [Reliability & Observability](./agents/systems-design/reliability-observability-interviewer/SKILL.md) | Reliability | Medium-Hard | Circuit Breakers, RED Metrics, RTO/RPO, Retries |
| [Design Twitter/X](./agents/systems-design/twitter-interviewer/SKILL.md) | System Design | Hard | Feed generation, fan-out, consistency |
| [Design Uber](./agents/systems-design/uber-interviewer/SKILL.md) | System Design | Hard | Real-time tracking, matching, maps |
| [Design a Search Engine](./agents/systems-design/search-engine-interviewer/SKILL.md) | System Design | Hard | Crawling, inverted index, ranking, autocomplete |

### 💾 Data Engineering — 3 skills

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [SQL Optimization](./agents/data-engineer/sql-optimization-interviewer/SKILL.md) | Database | Medium-Hard | Indexing, query plans, schema design |
| [Pipeline Architect](./agents/data-engineer/pipeline-architect-interviewer/SKILL.md) | Data Engineering | Medium-Hard | End-to-end pipelines, Kafka/Flink, Airflow, failure modes |
| [Schema Design](./agents/data-engineer/schema-design-interviewer/SKILL.md) | Data Engineering | Medium-Hard | Dimensional modeling, SCDs, star schema, lakehouse |

### 🔧 DevOps / SRE — 3 skills

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [Kubernetes Fundamentals](./agents/devops-sre/kubernetes-interviewer/SKILL.md) | Infrastructure | Medium | Pods, services, deployments |
| [CI/CD Pipeline Design](./agents/devops-sre/cicd-pipeline-interviewer/SKILL.md) | DevOps | Medium | GitHub Actions, Jenkins, testing strategies |
| [Monitoring & Alerting](./agents/devops-sre/monitoring-alerting-interviewer/SKILL.md) | SRE | Medium | Prometheus, Grafana, SLIs/SLOs/SLAs |

### 🤖 Machine Learning Engineer — 2 skills

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [ML System Design](./agents/ml-engineer/ml-system-design-interviewer/SKILL.md) | ML Engineering | Hard | Feature stores, model serving, A/B testing |
| [Deep Learning Interview](./agents/ml-engineer/deep-learning-interviewer/SKILL.md) | ML Theory | Hard | CNNs, RNNs, Transformers, training dynamics |

### 🧠 AI Product Management — 3 skills (NEW)

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [AI Product Strategy](./agents/ai-pm/ai-product-strategy-interviewer/SKILL.md) | AI PM | Hard | When to use AI, success metrics, cost-quality trade-offs |
| [Prompt Engineering](./agents/ai-pm/prompt-engineering-interviewer/SKILL.md) | AI Engineering | Hard | Prompt architecture, RAG, evaluation, token optimization |
| [Responsible AI](./agents/ai-pm/responsible-ai-interviewer/SKILL.md) | AI Ethics | Hard | Bias mitigation, content moderation, regulatory compliance |

### 🔍 Debugging & Incident Response — 6 skills (NEW)

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [Broken API](./agents/debugging/broken-api-interviewer/SKILL.md) | Debugging | Medium-Hard | 500 errors under load, connection pool exhaustion, deadlocks |
| [Slow Database](./agents/debugging/slow-database-interviewer/SKILL.md) | Debugging | Medium-Hard | Query regression, stale statistics, lock contention |
| [Memory Leak](./agents/debugging/memory-leak-interviewer/SKILL.md) | Debugging | Medium-Hard | Unbounded caches, listener leaks, closure retention |
| [Cascading Failure](./agents/debugging/cascading-failure-interviewer/SKILL.md) | Debugging | Hard | Thread pool exhaustion, retry storms, missing circuit breakers |
| [Data Inconsistency](./agents/debugging/data-inconsistency-interviewer/SKILL.md) | Debugging | Hard | Timezone mismatches, duplicate events, missing refunds |
| [Deployment Rollback](./agents/debugging/deployment-rollback-interviewer/SKILL.md) | Debugging | Medium-Hard | Failed deploys, migration rollbacks, feature flag issues |

### 🎭 Behavioral — 1 skill

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [Leadership Principles](./agents/behavioral/leadership-principles-interviewer/SKILL.md) | Behavioral | All Levels | STAR method, cross-functional collaboration |

### 🧩 Meta-Skills — 1 skill (NEW)

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [Problem Decomposition](./agents/meta/problem-decomposition-interviewer/SKILL.md) | Problem Solving | All Levels | Pattern recognition, approach selection, structured thinking |

---

## 🎯 How It Works

### Skill Structure

Each skill follows a consistent format:

```
🎭 Persona
   └── Who the AI interviewer is, their style, approach

🎯 Core Mission  
   └── What you'll learn and practice

📋 Interview Structure
   └── Phases: Warm-up → Core Concepts → Problem Solving → Wrap-up

🔧 Interactive Elements
   └── ASCII diagrams, Remotion components for visual learning

💡 Hint System
   └── 4 levels: Gentle nudge → Direction → Partial solution → Full walkthrough

📝 Problem Bank
   └── Curated questions with optimal solutions and follow-ups

🏆 Evaluation Rubric
   └── How to assess performance and identify weak areas

📚 Resources
   └── Books, courses, and practice problems for further study
```

### Hint System Explained

When you're stuck, the interviewer provides hints at increasing detail levels:

| Level | Type | Example |
|-------|------|---------|
| **1** | Gentle Nudge | *"Think about the time complexity. What data structure gives O(1) lookups?"* |
| **2** | Direction | *"This sounds like a dynamic programming problem. Can you identify the subproblems?"* |
| **3** | Partial Solution | *"Try using two pointers - one at start, one at end, moving towards each other."* |
| **4** | Full Walkthrough | Step-by-step explanation with pseudocode |

**Pro tip**: Try to solve with Level 1 hints first. The struggle is where learning happens!

---

## 🎨 Visual Learning

### ASCII Diagrams

Every skill includes visual explanations:

```
Two Pointers Pattern:
Array: [1, 2, 3, 4, 5, 6], Target: 7

Left →                    ← Right
  1     2  3  4  5     6
  1+6=7 ✓ Found!
```

### Remotion Components

For complex animations, we provide [Remotion](https://www.remotion.dev/) React components:

```tsx
// Example: Visualizing consistent hashing
export const ConsistentHashingDemo = () => {
  const frame = useCurrentFrame();
  // Animation logic...
  return <div>{/* Visual representation */}</div>;
};
```

Render these to video for:
- Pre-study review
- Sharing explanations with study groups
- Building your own tutorial content

---

## 🛠️ For Contributors

### Adding a New Skill

1. **Fork the repository**
2. **Copy the template**:
   ```bash
   cp templates/skill-template.md agents/{role-name}/{agent-name}.md
   ```
3. **Fill in the template** following our guidelines
4. **Test your skill** with Claude Code
5. **Submit a PR** with:
   - Clear description of what the skill covers
   - Test notes (how you verified it works)
   - Any Remotion components included

### Skill Quality Checklist

- [ ] Clear, consistent persona defined
- [ ] 3-4 difficulty-appropriate problems
- [ ] All 4 hint levels for each problem
- [ ] At least 2 visual diagrams (ASCII or Remotion)
- [ ] Evaluation rubric included
- [ ] Resources section with further reading
- [ ] Tested with at least one AI assistant

### Directory Structure

```
The-Mentor/
├── README.md                 # This file
├── LICENSE                   # MIT License
├── CONTRIBUTING.md           # How to contribute new skills
├── references/
│   └── learning-path.md      # Structured curriculum (6 tracks)
├── templates/
│   └── skill-template/       # Template for creating new skills
│       └── SKILL.md
├── agents/
│   ├── swe-i/                # 5 entry-level coding skills
│   ├── swe-ii/               # 3 mid-level algorithm skills
│   ├── data-engineer/        # 3 data engineering skills
│   ├── systems-design/       # 13 system design skills
│   ├── devops-sre/           # 3 DevOps & SRE skills
│   ├── ml-engineer/          # 2 ML engineering skills
│   ├── ai-pm/                # 3 AI product management skills
│   ├── debugging/            # 6 debugging & incident response skills
│   ├── behavioral/           # 1 behavioral interview skill
│   └── meta/                 # 1 meta-skill (problem decomposition)
```

---

## 📊 Interview Preparation Roadmap

For the full structured curriculum with 6 tracks, see the [Learning Path](./references/learning-path.md).

### SWE Track (8 Weeks)

| Weeks | Focus | Skills |
|-------|-------|--------|
| 1-2 | Foundations | Arrays & HashMaps, Linked Lists, Binary Trees, Recursion, Stacks & Queues |
| 3-4 | Core Design + Algorithms | URL Shortener, Database Architecture, Caching, Graphs, DP, Heaps |
| 5-6 | Advanced Design | API Design, Rate Limiter, Message Queues, Microservices |
| 7-8 | Senior/Staff Level | Uber, Twitter, Search Engine, Distributed Systems, Networking, Reliability |

### Specialized Tracks

| Track | Duration | Skills |
|-------|----------|--------|
| **Data Engineering** | 6 weeks | SQL Optimization → Pipeline Architect → Schema Design |
| **DevOps / SRE** | 4 weeks | Kubernetes → CI/CD → Monitoring → Reliability |
| **ML Engineer** | 4 weeks | Deep Learning → ML System Design |
| **AI Product Manager** | 3 weeks | AI Product Strategy → Prompt Engineering → Responsible AI |
| **Debugging** | 3 weeks | Broken API, Slow DB, Memory Leak, Cascading Failure, Data Inconsistency, Deployment Rollback |

---

## 📖 Example Session

**You**: Load the Arrays & HashMaps skill and start interviewing me

**Claude**: [Loads skill persona]

> 🎓 **Arrays & HashMaps Interviewer**
> 
> Welcome! I'm here to help you master fundamental array and hashmap problems. We'll start with some warm-up questions, then work through problems together.
>
> Let's begin: What's the time complexity of looking up an element in a HashMap, and why?

**You**: I think it's O(1)?

**Claude**: Exactly! Though technically it's amortized O(1)... [continues with follow-up]

[45 minutes of guided practice with hints when needed]

**Claude**: Great session! Here's your evaluation:
- Problem Understanding: 4/5
- Solution Approach: 4/5
- Code Quality: 3/5
- [Specific feedback and resources]

---

## 🔗 Related Projects

- [System Design Primer](https://github.com/donnemartin/system-design-primer) - Learn system design
- [NeetCode](https://neetcode.io/) - Practice problems by pattern
- [Blind 75](https://www.teamblind.com/post/New-Year-Gift---Curated-List-of-Top-75-LeetCode-Questions-to-Save-Your-Time-OaM1orEU) - Essential problem list

---

## 📜 License

MIT License - see [LICENSE](./LICENSE) file for details.

Contributions welcome! Please read our [Contributing Guide](./CONTRIBUTING.md) (coming soon).

---

<p align="center">
  <strong>Ready to ace your next interview?</strong><br>
  Pick a skill from the <a href="#-roster">Roster</a> and start practicing!
</p>

<p align="center">
  ⭐ Star this repo if it helps you land your dream job! ⭐
</p>
