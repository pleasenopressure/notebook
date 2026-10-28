# 🧭 Intelligent Agents — Lecture 1 Notes
> Course: COMP6203 — Intelligent Agents  
> Lecturer: Enrico Marchioni  
> Topic: Introduction to Intelligent Agents  
> Keywords: Agents, Intelligence, Autonomy, Multi-Agent Systems, Game Theory  
> pdf link:[lecture 2 week 1](file:///D:/EdgeDownload/OneDrive%20-%20University%20of%20Southampton/Document/intelligent_agents.pdf)

---

## 1. Overview

This lecture introduces the concept of **Intelligent Agents (IA)** and how it leads to the study of **Multi-Agent Systems (MAS)** — systems where autonomous entities interact to achieve goals.

We explore:
- How the idea of agent systems emerged
- What makes an agent “intelligent”
- How agents interact in environments and societies
- The relevance of this field to broader AI and computer science

---

## 2. Five Trends Leading to Agent Systems

In the last three decades, **five key trends** in computing have converged toward agent-based research:

### 2.1 Ubiquity
- Computing devices have become **cheap, powerful, and everywhere**.
- From PCs to smartphones → now each person carries a “supercomputer” in their pocket.
- **Edge computing** allows data to be processed locally, near where it is generated.

> 🧩 *Example*:  
> 20–25 years ago, it was unimaginable to have such computational power in your pocket.  
> Now, your phone performs operations that required entire PCs decades ago.

---

### 2.2 Interconnection
- Almost all devices are now **networked** — they share data and decisions.
- **Distributed systems** cooperate and collaborate to perform tasks.

> 🧩 *Examples*:
> - **Drones** coordinating to survey disaster areas  
> - **Stock-trading agents** that automatically negotiate or trade based on market data  

---

### 2.3 Intelligence
- Systems now perform tasks that were once only possible for humans:
  - Planning, decision-making, and adapting.
- These systems can be considered **intelligent** if they make complex, goal-driven decisions.

> 🧩 *Examples*:
> - **Self-driving cars** making real-time navigation decisions  
> - **AI investment software** selecting portfolios based on preferences  
> - **Autopilots** in aircraft operating without human input  

---

### 2.4 Delegation
- We **trust computers to act on our behalf**, even in safety-critical scenarios.
- Delegation means humans give systems goals, and systems act independently to achieve them.

> 🧩 *Examples*:  
> - Autopilot aircraft, autonomous buses, delivery robots  
> - Recommender systems choosing products for users

---

### 2.5 Human Orientation
- Systems must understand human **preferences**, **goals**, and **communication**.
- **Explainable AI (XAI)**: users should understand system actions without needing to know internal mechanics.
- AI is not only about automation — it **augments human intelligence**.

> 🧩 *Example discussed in lecture*:  
> A robot waiter that delivers drinks to customers —  
> It must interpret orders, identify tables, and interact meaningfully with people.  

---

### → Summary
These five trends together led to the rise of **multi-agent systems (MAS)** — networks of intelligent, autonomous entities acting independently and interacting socially.

---

## 3. What Is an Agent?

> **Definition:**  
> “A computer system capable of autonomous action in some environment, in order to achieve its delegated goals.”

### Key Features
- **Autonomy** → acts independently, no continuous human control.  
- **Delegation** → acts on behalf of a user or owner.  
- **Situatedness** → perceives and interacts with its environment continuously.

> 🧩 *Example*: **Thermostat**
> - Goal: Maintain room temperature.  
> - Perception: Temperature sensor.  
> - Action: Turns heater on/off.  
> - It’s autonomous and goal-driven, but not “intelligent”.

---

## 4. What Makes an Agent Intelligent?

An **intelligent agent** must demonstrate **three behaviors**:

### 4.1 Reactive
- Constantly monitors the environment and responds in real time.  
- Must handle a **dynamic**, changing world.

> *The thermostat* is a **reactive** agent —  
> It senses temperature and reacts by toggling heat.

---

### 4.2 Proactive
- Not just reacting — but **setting and pursuing goals**.  
- **Goal-oriented behavior**: generates plans to achieve objectives.

> 🧩 *Example*:  
> A delivery robot plans a route to deliver drinks efficiently, not just reacting to obstacles.  
> It might replan when it detects blocked paths or delayed customers.

---

### 4.3 Social
- Operates in environments with other agents or humans.
- Needs to **communicate**, **cooperate**, **coordinate**, or **negotiate** with others.

> 🧩 *Examples*:
> - Trading bots negotiating market prices  
> - Drones sharing data to cover search areas efficiently  
> - Game-playing AIs coordinating in a team  

---

## 5. Additional Properties

### Rationality
- Acts to **maximize expected utility**.
- Chooses the option that best satisfies its preferences.

### Learning / Adaptation
- Learns from the environment to improve decisions over time.
- Example: Reinforcement learning agents refining strategies through feedback.

---

## 6. Key Design Challenges
1. **Representing goals and preferences** — how to formalize human desires into computational form (utility functions).  
2. **Optimizing decision-making** — finding efficient strategies.  
3. **Learning and adapting** — updating models through experience.  

> 🧩 *Example from lecture*:  
> When a robot waiter needs to know which table to deliver to —  
> it must understand that “the preferred outcome” is delivering to the *right* table, not any table.

---

## 7. Multi-Agent Systems (MAS)

> A system with multiple agents interacting in a shared environment.

### Core Abilities:
- **Cooperation** → work together toward shared goals.  
- **Coordination** → avoid conflicts and redundancies.  
- **Negotiation** → reach mutually acceptable agreements.

> 🧩 *Examples*:
> - Teams of **rescue drones** coordinating search patterns  
> - **Stock-trading bots** with competing goals influencing markets  
> - **Autonomous vehicles** negotiating intersections  

---

## 8. Collaboration vs Competition

| Type | Description | Example |
|------|--------------|----------|
| **Collaborative MAS** | Agents owned/designed by one organization. Share goals and protocols. | A fleet of drones surveying an area together. |
| **Self-Interested / Competitive MAS** | Agents designed by different owners, each pursuing distinct goals. | Trading agents from different companies competing on a market. |

### Cooperation and Conflict
- Even **self-interested agents** may cooperate if it helps them reach better outcomes (second-best payoffs).  
- Requires **mechanisms or rules** to resolve conflicts — often designed into the system.

> 💬 *Lecture example:*  
> Competing stock-market agents might temporarily cooperate to influence market thresholds —  
> illustrating negotiation and coordination between competitive entities.

---

## 9. Interdisciplinary Links
The field draws from:
- **Artificial Intelligence (AI)**
- **Economics & Game Theory**
- **Philosophy & Logic**
- **Robotics**
- **Social Sciences**

> Strength: cross-pollination of ideas  
> Weakness: varied interpretations across disciplines  

---

## 10. Agents and Artificial Intelligence

Agents **combine multiple AI techniques**:
- **Planning algorithms** for sequential decision-making  
- **Machine learning & vision** for perception and prediction  
- **Knowledge representation & reasoning** for decision logic  

> 🧩 *Examples*:
> - **Robotics**: vision + planning  
> - **E-commerce bots**: search + machine learning  
> - **Software assistants**: scheduling + ML personalization  

---

## 11. Next Topics in Course

After this lecture, the module moves to:
1. **Utility and Strategic-form Games**  
   - Representing preferences with utility functions.  
2. **Game Theory**  
   - Agents’ strategic interactions.  
3. **Negotiation and Cooperation Models**  
   - How agents can coordinate or compete effectively.  

> 📘 Tip:  
> Review the worksheet on **basic game theory exercises** — they reflect exam question style.  

---

## 12. Summary

| Concept | Description |
|----------|--------------|
| **Agent** | Autonomous computer system acting in an environment to achieve delegated goals. |
| **Intelligent Agent** | Reactive, proactive, and social. |
| **Multi-Agent System** | Multiple agents interacting — cooperative or competitive. |
| **Key Challenges** | Representation, optimization, adaptation. |
| **Connection to AI** | Builds upon planning, ML, reasoning, and human–AI interaction. |

> 🧩 In short:  
> Intelligent agents mark the shift from **tools that follow instructions**  
> → to **entities that act autonomously and intelligently** in dynamic, social environments.

---
