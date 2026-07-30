# Agentic AI - Important Interview Questions

## Foundational Concepts

### 1. What is an AI Agent? How is it different from a traditional LLM chatbot?

**Answer:** An AI agent is an autonomous system that perceives its environment, reasons about goals, and takes actions to achieve those goals. Unlike a traditional LLM chatbot (which simply responds to prompts), an agent has:
- **Autonomy**: Operates without step-by-step human guidance
- **Tool use**: Can call APIs, run code, query databases
- **Memory**: Maintains state across interactions
- **Planning**: Breaks down complex goals into sub-tasks
- **Self-correction**: Can reflect, evaluate, and retry

### 2. What are the core components of an agentic system?

| Component | Description |
|-----------|-------------|
| **Perception** | Receives input from environment (user query, sensor data, system state) |
| **Reasoning/Planning** | LLM or planner decomposes goals into steps |
| **Memory** | Stores context, history, and learned information |
| **Action/Tools** | Functions the agent can invoke (APIs, code exec, search) |
| **Feedback Loop** | Evaluates action outcomes and adjusts |

### 3. Explain the ReAct (Reasoning + Acting) pattern

**Answer:** ReAct interleaves reasoning traces with actions in a loop:
1. **Thought**: "I need to find the user's account balance"
2. **Action**: `get_account_balance(user_id)`
3. **Observation**: `{"balance": 1200}`
4. **Thought**: "Balance is sufficient, proceed with payment"
5. **Action**: `process_payment(amount=500)`

This is more robust than pure chain-of-thought because the agent can incorporate real-time observations.

### 4. What is the Plan-and-Solve pattern? How does it differ from ReAct?

**Answer:** Plan-and-Solve separates planning from execution:
- **Phase 1 (Plan)**: Agent creates a complete step-by-step plan upfront
- **Phase 2 (Solve)**: Executes each step, potentially re-planning on failure

**vs ReAct**: ReAct is interleaved (thought → act → observe → loop), more flexible but can be less efficient. Plan-and-Solve is better when steps are well-defined upfront.

### 5. What types of memory do agents use?

- **Short-term/Working Memory**: Current conversation context, fits in LLM context window
- **Long-term Memory**: Persistent storage (vector DB, key-value store) for facts across sessions
- **Episodic Memory**: Past experiences, successful strategies, past mistakes
- **Procedural Memory**: Knowledge of how to use tools, stored as system prompts or fine-tuned knowledge

---

## Tool Use & Function Calling

### 6. Explain how tool/function calling works in LLM agents

**Answer:**
1. Developer defines tools with JSON schemas (name, description, parameters)
2. System prompt includes tool descriptions
3. LLM outputs a structured JSON call like `{"name": "search_web", "arguments": {"query": "..."}}`
4. Runtime parses the call, executes the function, returns result
5. Result is fed back to LLM as an observation

```
User: "What's the weather in Tokyo?"
→ LLM: {function: "get_weather", args: {city: "Tokyo"}}
→ Runtime: get_weather("Tokyo") → "72°F, sunny"
→ LLM: "The weather in Tokyo is 72°F and sunny."
```

### 7. How do you handle tool selection when many tools are available?

- **Descriptive naming & descriptions** so LLM can distinguish tools
- **Categorization**: Group tools; use a router agent to pick category first
- **Retrieval-augmented tool selection**: Embed tool descriptions, retrieve top-k relevant tools per query
- **Hierarchical agents**: Parent agent delegates to child agents with narrower tool sets

### 8. What security concerns exist with agent tool use?

- **Command injection**: Malicious input in tool arguments
- **Over-privileged tools**: Agent accessing tools it doesn't need
- **Data exfiltration**: Agent reading sensitive data and passing to external tools
- **Infinite loops**: Agent calling the same tool repeatedly
- **Hallucinated tool calls**: LLM inventing tool names or arguments

**Mitigations**: Least-privilege tool design, input validation, human-in-the-loop for destructive actions, rate limiting, output sanitization.

---

## Multi-Agent Systems

### 9. What are multi-agent architectures? When would you use them?

**Types:**
- **Orchestrator-Worker**: One agent delegates to specialized workers
- **Debate/Collaborative**: Agents discuss and reach consensus
- **Pipeline**: Output of one agent feeds into another
- **Competitive**: Agents compete, best answer wins (adversarial)

**When to use**: Complex tasks requiring diverse expertise, tasks that benefit from specialization, when you need verification/critique loops.

### 10. How do agents communicate in a multi-agent system?

- **Shared memory/message bus**: Agents read/write to a shared store
- **Direct messaging**: Agent-to-agent via structured messages
- **Orchestrator-mediated**: Central coordinator routes messages
- **Tool-based**: Agent A writes to a tool, Agent B reads from it

Common formats: JSON messages with `from`, `to`, `type`, `content`, `timestamp`.

### 11. What is the "orchestrator" pattern?

**Answer:** A supervising agent that:
1. Receives the user's goal
2. Decomposes it into sub-tasks
3. Assigns each sub-task to a specialized worker agent
4. Collects results
5. Synthesizes final output

The orchestrator handles error recovery, re-planning, and conflict resolution.

---

## Planning & Reasoning

### 12. How do agents handle task decomposition?

- **LLM-generated plans**: "Break down 'plan a trip' into flights, hotels, itinerary"
- **Predefined DAGs**: Fixed workflow graphs for known task types
- **Hierarchical planning**: High-level plan → each sub-task decomposed further
- **Dynamic re-planning**: If a step fails, regenerate the remaining plan

### 13. What is "reflection" in agentic systems?

**Answer:** The agent evaluates its own outputs for correctness and improvement:
- **Self-reflection**: "Did my last action produce the expected outcome?"
- **Critique**: "Is this answer logically consistent?"
- **Revision**: "Let me correct the mistake"

```
Action: search("population of France") → "65 million"
Reflection: "That seems low, France population is ~68M. Let me verify."
Action: search("France population 2025") → "68.4 million"
Final: "The population of France is ~68.4 million."
```

### 14. Explain the concept of "grounding" in agents

**Answer:** Grounding ensures agent outputs are tied to verifiable reality:
- **Tool grounding**: Facts come from tool calls, not parametric memory
- **Citation grounding**: Output includes source references
- **Environmental grounding**: Agent actions have observable consequences
- **Human grounding**: Verification by human when confidence is low

---

## Advanced Topics

### 15. What is Agentic RAG? How is it different from standard RAG?

| Standard RAG | Agentic RAG |
|--------------|-------------|
| Single retrieval step | Multiple retrieval strategies |
| Fixed chunk size | Dynamic chunking based on query |
| No query reformulation | Agent rewrites/decomposes query |
| No tool choice | Agent decides search, code, or tool |
| Passive results used as-is | Agent reflects, re-queries, synthesizes |

### 16. How do you evaluate agent performance?

**Metrics:**
- **Task Success Rate**: % of tasks completed successfully
- **Tool Call Efficiency**: Number of tool calls per task
- **Cost**: Total tokens + API calls consumed
- **Latency**: Time from user request to completion
- **Recovery Rate**: % of failed steps successfully recovered
- **Safety violations**: Actions that breached guardrails

**Evaluation tools**: AgentBench, GAIA, SWE-bench, custom rubric evaluation

### 17. What are the most common failure modes of LLM-powered agents?

1. **Tool hallucination**: Calling non-existent functions
2. **Task drift**: Forgetting the original goal mid-execution
3. **Confirmation bias**: Ignoring contradictory observations
4. **Infinite loops**: Repeating the same unsuccessful action
5. **Context overflow**: Losing early context as history grows
6. **Brittle parsing**: Failing on slightly malformed tool responses
7. **Over-confidence**: Executing destructive actions without verification

### 18. What frameworks exist for building agents?

| Framework | Key Feature | Best For |
|-----------|-------------|----------|
| **LangChain/LangGraph** | Graph-based agent orchestration | Complex workflows, cycles |
| **CrewAI** | Role-based multi-agent | Collaborative agent teams |
| **AutoGen (Microsoft)** | Multi-agent conversation | Agent-to-agent debate |
| **Semantic Kernel** | Microsoft ecosystem | Enterprise .NET/Azure |
| **OpenAI Assistants API** | Hosted, managed agents | Quick prototyping |
| **Anthropic Tool Use** | Native function calling | Simpler agent needs |
| **smolagents (HuggingFace)** | Code-as-actions agents | Code-generation agents |

### 19. What is the "agentic workflow" concept popularized by Andrew Ng?

**Answer:** Four design patterns:
1. **Reflection**: Agent critiques its own output and improves
2. **Tool Use**: Agent uses external tools to gather info/take action
3. **Planning**: Agent decomposes complex tasks
4. **Multi-agent**: Multiple specialized agents collaborate

Ng's argument: Agentic workflows can dramatically outperform single-shot prompting even with weaker models.

### 20. How would you design a robust agent for a production banking application?

**Key considerations:**
- **Human-in-the-loop**: Require approval for transactions > threshold
- **Least-privilege tools**: Read-only queries by default, explicit opt-in for writes
- **Timeouts & circuit breakers**: Prevent runaway loops
- **Audit logging**: Every action logged (user, agent, tool, timestamp, outcome)
- **Confidence thresholds**: If agent confidence < 0.8, escalate to human
- **Session limits**: Max steps per task, max tokens per session
- **Fail-closed**: On error, default to rejecting the action
- **Monitoring**: Track success rate, latency, tool error rates

### 21. How would you design a customer support agent for an e-commerce platform?

**Key considerations:**
- **Tiered escalation**: Agent handles returns/refunds/tracking autonomously; escalates to human for account bans, fraud, or legal issues
- **Tool set**: `get_order_status`, `initiate_return`, `lookup_inventory`, `schedule_pickup`, `apply_discount`
- **Sentiment detection**: If customer sentiment drops below threshold, transfer to human immediately
- **Order context injection**: Automatically inject user's recent order history into system prompt
- **Guardrails**: Cannot issue refunds > $500 without manager approval; cannot modify prices
- **Multi-language**: Detect language from user message, route to language-specific prompts/tools
- **Session persistence**: Maintain conversation state across reconnects (store in Redis with TTL)
- **Human handoff**: Summarize the entire agent conversation in a structured handoff note for the human agent
- **Feedback loop**: After resolution, ask "Was this resolved?" and log success/failure for continuous improvement

### 22. How would you design a code-generation agent for an internal development team?

**Key considerations:**
- **Sandboxed execution**: Generated code runs in isolated Docker containers with network policies
- **Scaffold + iterate**: Generate initial code scaffold, then let developer request modifications
- **Context injection**: Agent has access to repo structure, existing code patterns, and style guide
- **Review loop**: Agent generates code → runs linter + type checker → fixes issues → presents to developer
- **Dependency awareness**: Agent checks package.json/requirements.txt before adding new imports
- **Test generation**: Every code generation includes corresponding unit tests
- **Commit-ready output**: Generated code must pass CI checks before human review
- **Rollback support**: Each generation creates a git commit that can be easily reverted
- **Security scanning**: Scan generated code for secrets, SQL injection, XSS, vulnerable dependencies

### 23. How would you design a research assistant agent that analyzes competitor data?

**Key considerations:**
- **Multi-source retrieval**: Searches web, internal wikis, Crunchbase, news APIs
- **Structured extraction**: Agent fills a predefined schema (pricing, features, funding, market share)
- **Cross-source validation**: Compare claims across sources; flag contradictions
- **Temporal awareness**: "As of [date]" annotations on every data point; track changes over time
- **Report generation**: Produce structured markdown/PDF with citations and confidence scores
- **Scheduled execution**: Run weekly with diff report showing what changed
- **Bias mitigation**: Agent explicitly notes source credibility and potential biases
- **Recursive deep-dive**: If initial search is shallow, agent identifies gaps and re-queries

### 24. How would you design a healthcare triage agent that patients interact with?

**Key considerations:**
- **Regulatory compliance**: HIPAA-compliant, data never leaves approved boundary, audit trail
- **Scope limitation**: Agent explicitly cannot diagnose; it collects symptoms and assigns urgency level
- **Disclaimers**: Every response includes "This is not medical advice. Consult your physician."
- **Urgency detection**: Keywords + symptom severity → immediate escalation to human (chest pain, difficulty breathing)
- **PHI handling**: All patient data encrypted; agent memory wiped after session unless consent given
- **Structured intake**: Agent fills structured form (symptoms, duration, severity, medications) for doctor review
- **Handoff to specialist**: Based on symptoms, route to cardiology/dermatology/pediatrics queue
- **Appointment scheduling**: Agent checks doctor availability and books slots
- **Fallback to human**: If agent confidence < 0.85 on any symptom, escalate to nurse

### 25. How would you design a supply chain optimization agent for logistics?

**Key considerations:**
- **Real-time data integration**: Agent reads from inventory DB, shipping APIs, weather feeds, port schedules
- **Constraint-aware planning**: Respects warehouse capacity, truck weight limits, driver hours-of-service rules
- **What-if analysis**: Agent can simulate scenarios ("what if we reroute through Memphis?")
- **Alerting**: Proactive alerts for delays, stockouts, capacity issues
- **Multi-objective optimization**: Minimize cost + time simultaneously; present tradeoffs
- **Human confirmation**: Route changes require dispatcher approval; cost changes > 10% require manager
- **Historical learning**: Agent learns from past disruptions and suggests preemptive actions
- **Dashboard generation**: Agent maintains a live dashboard of KPIs (on-time delivery %, cost/mile, inventory turnover)

### 26. How would you design a compliance monitoring agent for a fintech company?

**Key considerations:**
- **Regulatory rule engine**: Encode rules from KYC/AML/BSA regulations as machine-readable constraints
- **Transaction monitoring**: Agent reviews flagged transactions against regulatory rules
- **Evidence collection**: For each alert, agent gathers supporting documents (user history, transaction chain, geolocation)
- **Narrative generation**: Agent writes structured SAR (Suspicious Activity Report) narrative with all evidence
- **Audit trail**: Every decision logged — what rules fired, what evidence was considered, final disposition
- **Human review queue**: Agent flags but does not block; human compliance officer makes final call
- **False positive feedback**: Agent records which flags were dismissed; adjusts rules
- **Regulatory update ingestion**: Agent reads regulatory updates, suggests rule changes, requires human approval to deploy

### 27. How would you design an interview scheduler agent for a recruiting platform?

**Key considerations:**
- **Calendar integration**: Read/write access to interviewers' calendars; check availability
- **Constraint handling**: Timezone-aware, buffer between interviews, preferred time windows
- **Reschedule intelligence**: If a candidate cancels, agent finds the next available slot without manual re-coordination
- **Natural language parsing**: "Next Tuesday afternoon" → generates candidate-specific time options
- **Multi-participant coordination**: Find overlapping availability across 3-5 interviewers
- **Reminder automation**: Send reminders 24h and 1h before; detect no-shows and trigger reschedule flow
- **Template management**: Schedule with company-specific email templates and branding
- **Fallback**: If no slot found within N days, alert recruiting coordinator

### 28. How would you design a fraud detection agent that augments a rule-based system?

**Key considerations:**
- **Hybrid approach**: Rules catch known patterns; LLM agent catches novel/anomalous patterns
- **Context gathering**: For flagged transactions, agent gathers user history, device fingerprint, IP geolocation, typical behavior
- **Reasoning**: Agent writes a natural language explanation of why this transaction is suspicious
- **Confidence scoring**: Agent outputs fraud probability + confidence level
- **Action matrix**:
  | Confidence | Risk | Action |
  |------------|------|--------|
  | High | High | Auto-block + alert human |
  | Medium | High | Block, queue for human review within 1hr |
  | Low | High | Allow, flag for review |
  | Any | Low | Allow, no action |
- **Explainability**: Every decision includes human-readable reasoning for compliance
- **Continuous learning**: Agent's reasoning is compared against human investigator decisions; fine-tune periodically
- **Adversarial awareness**: Agent checks for prompt injection in transaction descriptions

### 29. How would you design a data pipeline agent for a data engineering team?

**Key considerations:**
- **Natural language → pipeline**: "Join sales data with customer demographics and compute churn risk by region"
- **Data catalog awareness**: Agent has schema, descriptions, and lineage for all tables
- **SQL generation + validation**: Agent generates SQL, runs it against a shadow copy, verifies row counts and null rates
- **Performance guardrails**: Auto-terminate queries that exceed cost or time thresholds
- **Dry-run mode**: Preview the result set before materializing the pipeline
- **Dependency management**: Agent detects upstream dependencies and schedules jobs accordingly
- **Error handling**: If source table schema changed mid-pipeline, agent detects and re-plans
- **Documentation generation**: Agent writes data dictionary and lineage docs for each pipeline

### 30. How would you design a content moderation agent for a social media platform?

**Key considerations:**
- **Tiered moderation**:
  | Tier | Model | Action |
  |------|-------|--------|
  | 1 | Fast classifier | Auto-approve safe content, auto-reject clear violations |
  | 2 | LLM agent | Ambiguous content reviewed with context |
  | 3 | Human | Edge cases escalated |
- **Context window**: Agent reads post + comments + user history before deciding
- **Policy encoding**: Company moderation policies encoded as instructions + examples
- **Appeal handling**: User can appeal; agent re-evaluates with fresh context
- **Tone detection**: Distinguish between "I'll kill this bug" (hyperbole) and genuine threats
- **Regional nuance**: Same content may be acceptable in one region and violating in another
- **Consistency logging**: Log all moderation decisions to detect bias or inconsistency
- **Bypass detection**: Check for homoglyphs, Unicode tricks, image-based text

---

### 31. How would you design a DevOps incident response agent?

**Key considerations:**
- **Alert ingestion**: Agent connects to PagerDuty/Datadog/Grafana, reads alerts and runbooks
- **Automated diagnosis**: Agent checks logs, metrics, recent deployments, and error rates before proposing actions
- **Runbook execution**: Agent follows existing runbooks step-by-step, reporting progress per step
- **Blast radius assessment**: Agent evaluates "how many users affected?", "is this canary vs production?"
- **Action types**:
  | Action | Auto-approve? | Condition |
  |--------|--------------|-----------|
  | Rollback last deploy | Yes | If canary, error rate > 5% |
  | Scale up replicas | Yes | If CPU > 80%, < 10 min since alert |
  | Restart service | No | Always requires human |
  | Modify firewall rules | No | Always requires human |
- **Slack channel integration**: Agent posts status updates, @-oncall for approvals, runs /commands
- **Post-mortem generation**: After resolution, agent creates a timeline of events, actions taken, and improvement suggestions
- **Learning from incidents**: Agent archives successful resolution patterns for future automated responses
- **Chaos engineering compatibility**: Agent distinguishes real incidents from chaos experiments

### 32. How would you design a personalized learning/tutoring agent?

**Key considerations:**
- **Student model**: Agent maintains a knowledge graph of what the student knows vs. gaps
- **Adaptive pacing**: If student answers 3 in a row correctly → increase difficulty; if struggling → offer hints
- **Socratic teaching**: Agent does not give answers directly; asks guiding questions
- **Multi-modal support**: Agent can generate diagrams, code examples, quizzes, and flashcards
- **Misconception detection**: "I think you're confusing X with Y. Here's how they differ..."
- **Spaced repetition**: Agent schedules review of past topics at optimal intervals (Leitner system)
- **Plagiarism prevention**: For coding questions, agent asks student to explain their approach before giving solution
- **Progress reports**: Weekly summary for student and parent/teacher
- **Content safety**: Agent refuses to generate answers for take-home exams; flags academic dishonesty
- **Session limit**: Max 1 hour per session; agent winds down conversation naturally

### 33. How would you design a legal document review agent for contract analysis?

**Key considerations:**
- **Document ingestion**: Agent parses PDF/DOCX, extracts clauses, parties, dates, obligations
- **Clause classification**: Classify each clause (indemnification, termination, liability, confidentiality, etc.)
- **Risk scoring**: Agent assigns risk score per clause (1-5) based on deviation from standards
- **Redline generation**: Agent compares against preferred terms and generates suggested edits
- **Obligation extraction**: Structured output — "Party A must [action] by [date] or face [penalty]"
- **Jurisdiction awareness**: Different rules for GDPR, CCPA, UK, Singapore, etc.
- **Contradiction detection**: "Section 4.1 says 30-day notice but Section 12.3 says 60-day — conflict"
- **Confidence threshold**: If clause meaning is ambiguous (< 80% confidence), flag for human review
- **Audit trail**: Every annotation linked to specific text + reasoning
- **Version comparison**: Agent diffs contract versions and highlights meaningful changes (not formatting)

### 34. How would you design a real-time voice assistant agent (e.g., for call centers)?

**Key considerations:**
- **Speech-to-text → Agent → Text-to-speech**: End-to-end latency target < 500ms
- **Turn-taking detection**: Agent knows when caller has finished speaking (VAD + silence detection)
- **Barge-in handling**: Allow caller to interrupt; agent stops speaking and listens
- **Emotion detection**: Real-time sentiment from tone + word choice
- **Partial response streaming**: Agent starts speaking initial acknowledgment while generating full response
- **Script adherence**: Agent follows call scripts but can deviate when appropriately phrased
- **Hold music integration**: "Let me look that up" → plays hold music → agent returns with answer
- **Verbatim capture**: For compliance, log exact transcript; agent responses tagged as AI-generated
- **Escalation**: Warm transfer to human with full conversation summary
- **Language switching**: Detect code-switching mid-call; route to bilingual agent
- **Accent/dialect robustness**: Fine-tune ASR for regional accents

### 35. How would you design an automated A/B testing analysis agent?

**Key considerations:**
- **Experiment governance**: Agent validates experiment setup before launch (sample size, duration, metrics)
- **Real-time monitoring**: Agent watches experiment dashboards and alerts on anomalies (peeking, Simpson's paradox)
- **Statistical rigor**: Agent applies proper statistical tests (Mann-Whitney, Bayesian, sequential testing)
- **Segment discovery**: Agent automatically finds segments where treatment effect is different (e.g. "works for mobile users but not desktop")
- **Narrative generation**: "Variant B increased click-through rate by 3.2% (p=0.01, 95% CI [1.1%, 5.3%]). The effect is strongest on new users (+5.1%)"
- **Confounding detection**: Agent checks if randomization failed (imbalanced cohorts)
- **Recommendation engine**: Based on results, agent suggests "ship", "iterate", "stop", or "run longer"
- **Meta-analysis**: Agent maintains a knowledge base of past experiments; identifies patterns
- **Automated reporting**: Agent posts summary to Slack/email with key charts and interpretation

### 36. How would you design a document extraction agent for accounts payable?

**Key considerations:**
- **Multi-format parsing**: PDF invoices, scanned images, email attachments, EDI, CSV
- **Field extraction with validation**: Extract invoice #, PO #, amounts, tax, vendor — validate against rules
- **Three-way matching**: Agent matches invoice → PO → receiving report; flags discrepancies
- **Duplicate detection**: Compare vendor + invoice # + amount against database; flag duplicates
- **Approval routing**: Based on amount and department, route to appropriate approver
- **Error recovery**: If OCR confidence is low on a field, agent asks human or fetches original
- **Vendor communication**: Agent emails vendor for missing fields or mismatches
- **GL coding**: Agent suggests GL codes based on invoice description, learns from corrections
- **Payment scheduling**: Agent schedules payment based on terms, discounts, cash flow
- **Audit compliance**: Every extraction includes confidence scores and source bounding box

### 37. How would you design a sales prospecting agent that researches leads?

**Key considerations:**
- **Lead enrichment**: Given company name, agent fetches industry, size, funding, tech stack, recent news
- **Contact discovery**: Agent finds decision-makers (title → LinkedIn/Crunchbase → email pattern guess)
- **Personalization engine**: Agent writes draft outreach emails referencing specific company events
- **Intent signals**: Agent monitors job postings, funding announcements, leadership changes as triggers
- **Multi-channel sequencing**: Email → LinkedIn → phone call with configurable delays
- **Reply handling**: Agent processes replies; if positive → suggest meeting; if negative → mark DNC; if ambiguous → hand to human
- **CRM sync**: Agent reads/writes Salesforce/HubSpot records
- **Compliance**: CAN-SPAM, GDPR — opt-out handling, do-not-contact list
- **Performance tracking**: Open rate, reply rate, meeting booked rate per template
- **A/B testing outreach**: Agent tests subject lines, message lengths, and CTAs

### 38. How would you design a SQL database administration (DBA) agent?

**Key considerations:**
- **Query optimization**: Agent explains slow queries, suggests indexes, rewrites inefficient JOINs
- **Schema change management**: "Add a column" → agent checks for breaking changes, generates migration, schedules during maintenance window
- **Anomaly detection**: Agent monitors query latency, lock waits, deadlocks, replication lag
- **Backup verification**: Agent tests restore from backup periodically; reports RPO/RTO compliance
- **Capacity planning**: Agent projects storage growth, recommends sharding or archival
- **Security audit**: Agent checks for unsecured credentials, excessive privileges, missing encryption
- **Self-service for devs**: "Give me a read-only replica of production with these columns masked" → agent provisions
- **Rollback automation**: If migration fails, agent reverts to previous state automatically
- **Permission to act**:
  | Action | Level |
  |--------|-------|
  | Read queries | Always allowed |
  | CREATE INDEX | Only if `max_parallel_workers` available |
  | DROP/ALTER | Require ticket # and manager approval |
  | TRUNCATE | NEVER — escalate to human DBA |

### 39. How would you design a video content generation agent?

**Key considerations:**
- **Script generation**: Agent writes narrative script based on topic + target audience + tone
- **Scene planning**: Agent decomposes script into scenes with visual descriptions
- **Asset generation pipeline**:
  ```
  Script → Scene breakdown → Text-to-image → Image-to-video → Voiceover (TTS) → Music → Composite
  ```
- **Consistency**: Maintain character appearance, branding, and style across scenes
- **Quality gates**: Each frame checked for artifacts, text legibility, brand compliance
- **Human review points**: Script approval, keyframe approval, final cut approval
- **Length constraints**: Agent trims or expands content to hit exact duration target
- **Localization**: Agent generates versions in multiple languages with lip-sync
- **Cost optimization**: Agent chooses model size based on scene complexity (simple text → cheap model)
- **Templates**: Reusable template library for common formats (explainer, tutorial, ad, social clip)
- **Versioning**: Store all generation parameters for reproducibility

### 40. How would you design a cybersecurity SOC (Security Operations Center) analyst agent?

**Key considerations:**
- **Alert triage**: Agent ingests SIEM alerts (Splunk, Sentinel, Elastic), prioritizes by severity + asset criticality
- **Investigation playbooks**: Agent follows MITRE ATT&CK framework; checks IOCs against threat intel feeds
- **Log investigation**: Agent writes KQL/Splunk SPL queries to hunt for related evidence
- **Context enrichment**: Agent pulls asset owner, vulnerability scans, past incidents, user behavior baselines
- **Decision matrix**:
  | Finding | Action |
  |---------|--------|
  | True positive, low criticality | Auto-create ticket, notify team |
  | True positive, critical | Block IP, isolate host, page on-call, write incident report |
  | False positive | Add to allowlist, suppress similar alerts, document reasoning |
  | Inconclusive | Escalate to Tier-2 analyst with investigation summary |
- **Threat intel**: Agent correlates alerts with CVE database, known IoC feeds, dark web mentions
- **Evidence preservation**: Agent creates forensic snapshot, chain-of-custody log
- **Incident timeline**: Agent auto-generates a chronological timeline of events
- **Post-incident report**: Agent writes root cause analysis with remediation steps
- **Red team awareness**: Agent detects red team exercises and adjusts response

### 41. How would you design a real-time trading assistant agent?

**Key considerations:**
- **Data feeds**: Agent subscribes to market data, news feeds, earnings calls, social sentiment
- **Strategy execution**: Agent executes predefined strategies with strict risk parameters
- **Risk guardrails**:
  - Max position size per asset
  - Max daily loss limit (hard stop)
  - Max leverage ratio
  - Min liquidity requirement
- **News reaction**: Agent reads earnings call transcript in real-time, extracts sentiment, suggests hedge
- **Explainability**: Every trade logged with human-readable rationale (e.g., "Sold 1000 shares of TSLA because P/E ratio exceeded 200x")
- **Circuit breakers**: If VIX > 30, reduce position sizes by 50%. If volatility exceeds threshold, pause trading entirely
- **Regulatory compliance**: Audit trail for SEC/MiFID II; best execution documentation
- **Shadow mode**: Agent runs in simulation alongside live trader; compare decisions
- **Latency requirements**: Agent decision-to-execution must be < 10ms; co-location if needed
- **Kill switch**: Emergency stop button that liquidates positions and disables agent
- **Multi-strategy isolation**: Each strategy runs in isolated container; one failure doesn't cascade

### 42. How would you design a drug discovery research agent?

**Key considerations:**
- **Literature mining**: Agent reads PubMed, patents, clinical trial registries continuously
- **Hypothesis generation**: "Compound X binds to receptor Y because of structural similarity to known drug Z"
- **Molecular property prediction**: Agent calls computational models for ADMET, binding affinity, toxicity
- **Retrosynthesis planning**: "Target molecule can be synthesized from precursors A + B via reaction C"
- **Experiment design**: Agent proposes in-vitro experiments with controls, sample sizes, and success criteria
- **Data extraction**: Agent reads experimental results from PDFs, extracts IC50, EC50, selectivity data
- **Reproducibility check**: Agent flags missing metadata, statistical flaws, potential p-hacking
- **Collaboration across teams**: Biology agent → Chemistry agent → Clinical agent (pipeline)
- **Multi-objective optimization**: Optimize for efficacy + safety + synthesizability simultaneously
- **Regulatory readiness**: Agent structures data in FDA/EMA submission format
- **IP awareness**: Agent checks patent landscape; flags potential infringement

### 43. How would you design a restaurant management agent (for a chain)?

**Key considerations:**
- **Inventory management**: Agent forecasts ingredient needs based on historical sales, weather, local events
- **Dynamic pricing**: Agent adjusts menu prices based on demand, competitor pricing, time of day
- **Staff scheduling**: Agent creates schedules respecting labor laws, employee preferences, and predicted foot traffic
- **Kitchen optimization**: Agent analyzes ticket times and suggests prep list changes
- **Customer feedback**: Agent scans Yelp/Google reviews, extracts themes, flags urgent complaints
- **POS integration**: Agent reads real-time sales data, identifies top/worst sellers, suggests menu changes
- **Supplier management**: Agent compares supplier prices, auto-orders when stock hits reorder point
- **Health code compliance**: Agent tracks expiration dates, fridge temperatures, cleaning schedules
- **Multi-location coordination**: Transfer inventory between locations; standardize recipes across chain
- **Anomaly detection**: "Store #42 had 3x normal waste this week" → agent investigates
- **Manager dashboard**: Daily briefing — sales vs forecast, labor cost %, waste %, customer sentiment

### 44. How would you design an accessibility testing agent for web applications?

**Key considerations:**
- **Automated scanner**: Agent runs axe-core, WAVE, Lighthouse, checks WCAG 2.2 AA/AAA compliance
- **Screen reader simulation**: Agent uses NVDA/JAWS API to verify announcements, focus order, ARIA labels
- **Keyboard navigation**: Agent tab-throughs entire page; verifies focus indicators, skip links, trap prevention
- **Color contrast**: Agent checks all text/background combinations against ratio requirements
- **Visual impairment simulation**: Agent passes page through blur, color blindness, tunnel vision filters
- **Report generation**: Agent produces prioritized fix list with code snippets and before/after screenshots
- **Remediation suggestions**: "Button missing aria-label. Suggested: `<button aria-label="Close dialog">X</button>`"
- **Regression testing**: Agent re-tests after each deployment; blocks PR if critical issues found
- **User story creation**: Agent converts issues into accessibility user stories for the backlog
- **Manual testing handoff**: For issues that can't be automated (e.g., logical reading order), agent creates instructions for human QA

---

### 45. How do you reduce latency in an agentic system?

**Answer:** Latency in agents comes from three sources: LLM inference, tool execution, and planning overhead.

#### Model-Level
| Technique | How it helps |
|-----------|-------------|
| **Smaller models for sub-tasks** | Use fast/cheap model for routing, classification; large model only for complex reasoning |
| **Speculative decoding** | Draft tokens with a small model, verify with large model |
| **KV-cache / prefix caching** | Cache common prefix prompts (tool descriptions, system instructions) across requests |
| **Quantization** | INT8/FP8 inference reduces compute per token |
| **Shorter generations** | Limit `max_tokens`, prefer structured JSON output over free text |

#### Prompt-Level
- **Compress system prompts**: Remove redundant tool descriptions, use abbreviated examples
- **Lazy context loading**: Only inject relevant history/tools, not everything
- **Tool pruning**: Embed tool descriptions and dynamically inject only top-k relevant tools per query

#### Parallelization
- **Parallel tool calls**: Execute independent tool calls concurrently (e.g. fetch weather + fetch calendar simultaneously)
- **Speculative tool execution**: Predict which tool will be called and pre-fetch (e.g. pre-search while user is still typing)
- **Streaming**: Stream LLM tokens to user while tool calls happen asynchronously in background

#### Caching
- **Semantic caching**: Cache responses for semantically similar queries (embedding similarity > threshold)
- **Response cache**: Cache deterministic tool results (e.g. `get_stock_price("AAPL")` cached for N seconds)
- **Full-turn cache**: Cache the complete agent response for exact repeated queries

#### Architectural
- **Early termination**: If confidence is high after N steps, skip reflection/re-planning
- **Fallback chain**: Try fast model first; only escalate to slower model if confidence < threshold
- **Batching**: Batch non-urgent agent requests; run inference on grouped prompts
- **Shorten the loop**: Reduce the number of required ReAct cycles by improving the planner

#### Infrastructure
- **Fast inference providers**: Use providers with lower P50/P95 latency (e.g. Groq for open models)
- **Cold start prevention**: Keep agents/embeddings/models warm (keep-alive pings)
- **Edge/region proximity**: Deploy inference close to users
- **Async I/O**: Non-blocking tool calls, connection pooling for databases/APIs

---

## System Design Framework

Use this structured approach when answering any design question in an interview.

### Step 1: Clarify Scope & Constraints
- What is the primary goal? (automate, assist, replace?)
- Who are the users? (internal employees, external customers, other systems?)
- What are the non-negotiables? (latency, cost, compliance, accuracy?)
- What is the blast radius if the agent fails?

### Step 2: Define Agent Boundaries
- **In scope**: Actions the agent can take autonomously
- **Requires human**: Actions that need approval
- **Never allowed**: Actions the agent must refuse
- **Explicit triggers** for escalation (confidence < threshold, sensitive data, first-time errors)

### Step 3: Design Tools & Permissions
| Tool | Read/Write | Rate Limit | Approval Required |
|------|-----------|------------|-------------------|
| `search_knowledge_base` | Read | 100/min | No |
| `send_email` | Write | 10/min | No (templates only) |
| `refund_order` | Write | 5/min | Yes, if > $100 |
| `delete_account` | Write | 1/min | Always |

### Step 4: Memory & Context Strategy
- **Within session**: Full conversation in context window (with sliding window if long)
- **Across sessions**: Summarize key facts into persistent store (Redis/vector DB)
- **What to forget**: PII after session ends, stale data after TTL
- **Context injection**: Only inject relevant history (retrieve top-k past interactions)

### Step 5: Error Recovery & Fallbacks
- **Transient errors** (API timeout, rate limit): Retry with exponential backoff
- **Permanent errors** (invalid input, missing data): Graceful message to user, log for analysis
- **Agent confusion** (low confidence, repeated failures): Escalate to human with full trace
- **Infinite loop detection**: Max steps per task, max consecutive same-action limit
- **State corruption**: Snapshot agent state before each action; rollback on failure

### Step 6: Evaluation & Monitoring
| Metric | Target | Alert If |
|--------|--------|----------|
| Task success rate | > 90% | < 80% |
| Avg steps per task | < 5 | > 10 |
| Human escalation rate | < 15% | > 25% |
| Avg latency per step | < 2s | > 5s |
| Cost per task | < $0.05 | > $0.20 |

---

## Key Tradeoffs (Common Interview Discussions)

### ReAct vs Plan-and-Solve
| Dimension | ReAct | Plan-and-Solve |
|-----------|-------|----------------|
| **When to use** | Open-ended, dynamic tasks | Well-defined, predictable tasks |
| **Latency** | Faster first action | Faster overall if plan is correct |
| **Flexibility** | Adapts mid-stream | Brittle if plan fails early |
| **Context usage** | Efficient (only relevant reasoning) | Expensive (stores full plan) |
| **Parallelism** | Difficult (sequential by nature) | Easy (sub-tasks can run in parallel) |

**Interview answer**: "I default to ReAct for most agents because it's more robust. I switch to Plan-and-Solve only when the task structure is well-known and parallel execution matters."

### Single Agent vs Multi-Agent
| Dimension | Single Agent | Multi-Agent |
|-----------|-------------|-------------|
| **Complexity** | Simple to build and debug | Complex orchestration, harder to debug |
| **Specialization** | Jack of all trades | Each agent can be optimized independently |
| **Latency** | No inter-agent overhead | Communication overhead between agents |
| **Cost** | One large model call | Multiple calls (can be more expensive) |
| **Failure mode** | Single point of failure | Partial failure (some agents may fail) |

**Interview answer**: "I start with a single agent and split only when I hit specific pain points — context overflow, conflicting tools, or need for specialized evaluation."

### Large Model vs Small Model + Router
| Dimension | Single Large Model | Small Model + Router |
|-----------|-------------------|---------------------|
| **Accuracy** | Higher (general capability) | Lower on complex tasks |
| **Latency** | Higher (slower generation) | Lower (fast router + fast small model) |
| **Cost** | Higher (expensive per token) | Lower (cheap models for most tasks) |
| **Maintenance** | One model to update | Two or more models + routing logic |
| **Best for** | Complex reasoning tasks | High-volume, simple tasks with occasional complexity |

**Interview answer**: "Use a router pattern: small model handles 80% of simple requests, large model handles the remaining 20% that need deep reasoning. This cuts cost by 60-70%."

### Deterministic vs Agentic Workflows
| Dimension | Deterministic (DAG) | Agentic (LLM-driven) |
|-----------|-------------------|---------------------|
| **Predictability** | High — exact same path every time | Low — path depends on LLM decisions |
| **Debuggability** | Trivial — just follow the DAG | Hard — need tracing and replay |
| **Flexibility** | Rigid — every path must be predefined | Flexible — adapts to new situations |
| **When to use** | Regulatory, compliance, audit-heavy use cases | Exploratory, creative, or variable tasks |

**Interview answer**: "I use deterministic DAGs for anything regulated or audited (KYC, loan processing) and agentic workflows for open-ended tasks (research, customer support, data analysis)."

---

## Common Gotchas & Pitfalls

### 1. Agents look smart but fail silently
- **Problem**: Agent produces a plausible-sounding but wrong answer
- **Fix**: Always require citations/tool call evidence. Use a validation agent to verify claims. Implement confidence thresholds that trigger human review.

### 2. Cost explosion from runaway loops
- **Problem**: Agent gets stuck in a loop, calling tools repeatedly
- **Fix**: Hard cap on steps per task (e.g. max 15). Budget tracking per session (kill if cost > $X). Detect repeated same-action patterns and force escalation.

### 3. Non-determinism makes testing hard
- **Problem**: Same input produces different outputs across runs
- **Fix**: 
  - Seed the random/LLM call if possible
  - Use deterministic evaluation (does the tool get called correctly, not what the final text is)
  - Run N times and measure distribution of outcomes
  - Snapshot agent traces for replay debugging

### 4. Context window limits with long histories
- **Problem**: Agent runs out of context after many steps
- **Fix**: Sliding window (keep last N messages + summary of earlier). Summarize instead of truncate. Offload non-critical context to vector storage and retrieve on demand.

### 5. Tool hallucination
- **Problem**: Agent calls a tool that doesn't exist or passes invalid arguments
- **Fix**: Strict schema validation before execution. Reduce number of available tools (only inject relevant ones per task). Use constrained decoding (JSON mode, grammar-based sampling).

### 6. Prompt injection through tool results
- **Problem**: External data returned by a tool contains instructions that override the agent's system prompt
- **Fix**: 
  - Isolate tool outputs from system instructions (use special delimiters)
  - Strip or escape control tokens from tool results
  - Re-assert system prompt after every tool response
  - Rate-limit and sanitize data from untrusted sources

### 7. Over-reliance on a single tool
- **Problem**: Agent keeps calling the same tool even when it's not helpful
- **Fix**: Track tool effectiveness per session. If a tool returns "no results" more than N times, suggest alternative tools. Implement tool call budgets.

### 8. Compliance blindspots
- **Problem**: Agent stores PII in memory/logs, violating GDPR/HIPAA
- **Fix**: Auto-detect and redact PII before storing. Session-scoped memory (auto-expire). Audit trails that anonymize user data. Never log raw tool inputs/outputs for sensitive fields.

### 9. Degradation over time
- **Problem**: Agent performance degrades as context grows (loses focus on original goal)
- **Fix**: Periodic goal reminder injection. Step-level summaries instead of full history. Re-inject the original user goal every N steps.

### 10. Human-in-the-loop fatigue
- **Problem**: Too many escalations overwhelm human operators
- **Fix**: Batch review (show 10 cases at once). Tier escalation (junior → senior). Auto-approve low-risk items based on historical patterns. Give humans structured forms not free text.

---

## Emerging Topics to Watch

### MCP (Model Context Protocol)
- Standardized way for agents to discover and connect to tools/servers
- Like "USB-C for AI agents" — plug-and-play tool integration
- Key concepts: MCP servers expose resources/tools/prompts; MCP clients connect and negotiate capabilities

### A2A (Agent-to-Agent Protocol)
- Google's proposed standard for inter-agent communication
- Agents advertise capabilities, negotiate tasks, share context
- Enables agent marketplaces and federated agent networks

### Agent Evaluation Benchmarks
| Benchmark | Focus | Limitation |
|-----------|-------|------------|
| **GAIA** | Multi-step reasoning + tool use | Single-turn, limited tool set |
| **SWE-bench** | Real GitHub issues | Code-only, narrow domain |
| **AgentBench** | 8 diverse environments | Synthetic tasks |
| **ToolBench** | Tool use across APIs | Only tool selection, no planning |
| **WebArena** | Web navigation | Simulated browser only |

### Security in Agent Systems
- **Prompt injection** — adversarial input hijacks agent behavior
- **Indirect injection** — malicious data in retrieved documents triggers actions
- **Tool poisoning** — compromised APIs return data that misleads agent
- **Escape sequences** — special characters in tool results break parsing
- **Mitigations**: Least privilege, input/output sanitization, human confirmation for writes, sandboxing code execution

### Agent Observability (Tracing)
- **Key data to capture**: LLM calls, tool calls, reasoning traces, timing, cost, outcomes
- **Tools**: LangSmith, LangFuse, Weights & Biases, custom logging
- **Debugging workflow**: Replay agent traces step-by-step, inspect LLM reasoning at each decision point, compare expected vs actual tool calls

### Agent Fine-Tuning
- Fine-tune base models on agent trajectories (ReAct traces)
- Improves tool selection accuracy and reduces hallucinated calls
- Tradeoff: requires high-quality trajectory data, can overfit to specific tool sets

