<!-- ──────────────────────────────────────────────────────────────────────────
     Nasr Eddine Sangai — GitHub profile README
     Animated assets are hand-built SVG (CSS + SMIL), no external renderer:
       ./assets/header.svg        masthead, drifting neural field, metallic sheen
       ./assets/signal-path.svg   architecture chain, self-drawing, travelling pulse
       ./assets/metrics.svg       four gauges that sweep to their measured value
     Every figure below is taken from a run or a CI gate, not an estimate.
     ────────────────────────────────────────────────────────────────────────── -->

<div align="center">
  <img src="./assets/header.svg" width="100%" alt="Nasr Eddine Sangai — AI Systems Architect. AI systems that reason and act, not just chat.">
</div>

<div align="center">
  <a href="https://sangai.cloud"><img alt="Portfolio" src="https://img.shields.io/badge/PORTFOLIO-sangai.cloud-c9a24e?style=for-the-badge&labelColor=0b0a09"></a>
  <a href="https://www.linkedin.com/in/sangainasr"><img alt="LinkedIn" src="https://img.shields.io/badge/LINKEDIN-sangainasr-c9a24e?style=for-the-badge&labelColor=0b0a09&logo=linkedin&logoColor=c9a24e"></a>
  <a href="https://www.upwork.com/freelancers/sangainasr"><img alt="Upwork" src="https://img.shields.io/badge/UPWORK-available-2f8f64?style=for-the-badge&labelColor=0b0a09&logo=upwork&logoColor=2f8f64"></a>
  <a href="mailto:sangainasr@gmail.com"><img alt="Email" src="https://img.shields.io/badge/EMAIL-sangainasr-c9a24e?style=for-the-badge&labelColor=0b0a09&logo=gmail&logoColor=c9a24e"></a>
</div>

<div align="center">
  <img alt="AI is not prompts. AI is systems." src="https://readme-typing-svg.demolab.com/?font=JetBrains+Mono&weight=500&size=17&duration=3400&pause=900&color=C9A24E&center=true&vCenter=true&width=900&height=46&lines=AI+is+not+prompts.+AI+is+systems.;A+model+is+one+component.+The+architecture+is+the+product.;Traced%2C+gated+and+budgeted+from+the+first+request.;Built+to+be+operated+in+production%2C+not+demoed+in+a+notebook.">
</div>

---

### The premise

A model is one component. The product is the architecture around it — **retrieval, memory, routing, evaluation, observability** — designed so the whole thing can be measured, operated and changed safely.

I architect agents, retrieval pipelines and LLM platforms that automate real decisions. Five systems shipped and live, every one of them traced from the first request and held behind an eval gate before it ships.

---

## The signal path

<img src="./assets/signal-path.svg" width="100%" alt="One request travelling through a production AI system: Users, Gateway, LLM Router, Memory, RAG with a Vector DB sub-call, and an Agent Runtime that re-plans — bracketed by Observability, Evaluation and Deployment.">

Read it as one request, not a feature list. `Vector DB` is a call made **from inside** retrieval, not a step after it. `Observability`, `Evaluation` and `Deployment` bracket every hop rather than terminating the chain. The Agent Runtime is a state machine that can loop — error compounds at every hop, so the graph has to recover.

The budgets are **design constraints, not measured results**: the numbers a route is *allowed* to spend. Routing is an architecture decision, not a dropdown — frontier models where answer quality is the constraint, open-weight where cost, latency or data residency win.

---

## Measured, not claimed

<img src="./assets/metrics.svg" width="100%" alt="Macro nDCG@10 of 0.560 from a 208-config BEIR sweep; 100% of citations resolved across 24 claims; injection recall at or above 85% as a CI gate; 52 independent domains across 61 sources in one research run.">

| Figure | Where it comes from |
| :--- | :--- |
| **0.560** macro nDCG@10 | 208-config threshold sweep over BEIR NFCorpus + SciFact — above always-dense (0.557) and always-hybrid (0.558) |
| **100 %** citations resolved | one live three-company research run: 24 claims, 0 hallucinated citations |
| **≥ 85 %** injection recall | CI gate, while flagging at most 5 % of benign text |
| **52** independent domains | 61 sources, eTLD+1-independent, near-duplicate titles collapsed |

Output is non-deterministic. Without an eval gate, regressions stay silent until a user finds them — so the gate is the release, not a report.

---

## Selected systems

<details>
<summary><b>01 · Adaptive Retrieval Router</b> — routing becomes an auditable decision instead of a default &nbsp;·&nbsp; <a href="https://larr.sangai.cloud">live</a></summary>

<br>

`FastAPI` `FAISS` `BM25` `Tavily` `Streamlit`

**The problem.** Always-dense retrieval pays embedding cost on every query and is surprisingly weak on named entities. Always-sparse misses paraphrase. Always-hybrid is fine and pays the dense cost regardless. And no static corpus answers a question about today.

**The architecture.** Five ordered heuristics — freshness markers, keyword-overlap ratio, token length, question form — classify each query and dispatch it to the cheapest backend likely to answer: BM25, FAISS dense, RRF-fused hybrid (k=60), or Tavily web. Every rule that fired is recorded on the decision, and `/router/explain` returns the routing **without spending a token**.

**The outcome.** Per-query cost spans `$0` at 1–5 ms on sparse to `$0.008` at 200–600 ms on web — the router exists to spend the top of that range only when the bottom would fail. A 40-query eval set forced through all four backends gives 160 directly comparable judged rows.

**The impact.** Every call emits one structured line — hashed query, backend, latency, cost — so the heuristics can be *disproven* on real traffic rather than trusted.

</details>

<details>
<summary><b>02 · Adaptive Retrieval Router Pro</b> — the routing policy stops being folklore &nbsp;·&nbsp; <a href="https://larrpro.sangai.cloud">live</a></summary>

<br>

`Router profiles` `BEIR` `Anthropic` `SSE` `Live re-index`

**The problem.** Hand-tuned routing thresholds are a guess wearing the costume of a policy. And a retrieval API that stops at ranked documents leaves the last hop — grounded, cited generation — to whoever calls it.

**The architecture.** Thresholds became a versioned `RouterParameters` profile (speed / balanced / quality), selected at runtime and stamped on every decision. An `/answer` endpoint closes the loop: route → retrieve → generate with inline `[n]` citations streamed over SSE with token and cost accounting. Corpus documents can be uploaded, edited and re-indexed live.

**The outcome.** The presets were picked by a **208-config threshold sweep** over BEIR NFCorpus and SciFact, not by taste: `quality` lands in the winning equivalence class at macro nDCG@10 **0.560** — above always-dense at 0.557 and always-hybrid at 0.558 — while `speed` sits on the sparse-targeting Pareto frontier.

**The impact.** Each preset carries the sweep that chose it, so changing one is a measurable decision with a baseline to beat.

</details>

<details>
<summary><b>03 · Multi-Tenant RAG Platform</b> — retrieval stops being a script and becomes a service someone else can operate &nbsp;·&nbsp; <a href="https://inforag.sangai.cloud">live</a></summary>

<br>

`FastAPI` `Qdrant` `Postgres` `Celery` `RAGAS` `DeepEval`

**The problem.** A RAG prototype that works for one person doesn't survive contact with an organization: no tenancy, no roles, no audit trail, no way to show it publicly without handing out an account, and no gate that catches a retrieval regression before a user does.

**The architecture.** Separable services behind one platform API — JWT auth, organization tenancy and RBAC on FastAPI + Postgres, async ingestion through Celery workers, a cross-encoder reranker sidecar, Qdrant for vectors, Redis for semantic cache — with a DeepEval + RAGAS harness and persisted eval-run records as the promotion gate.

**The outcome.** Public demo sessions are anonymous and short-lived, backed by throwaway organizations, so isolation between visitors is the same tenancy every query already enforces. Quota is reserved before a request runs and released if it fails: a provider outage never costs a visitor a question.

**The impact.** CI runs the database-backed tests with the skip turned into a failure, so a model change that never got a migration breaks the build, not production.

</details>

<details>
<summary><b>04 · Multi-System Legal Document Analyst</b> — built to be structurally unable to overstate itself &nbsp;·&nbsp; <a href="https://lmslda.sangai.cloud">live</a></summary>

<br>

`FastAPI` `Postgres` `PyMuPDF` `Alembic` `Sentry`

**The problem.** Contract text arrives from a counterparty, which makes it **untrusted input**: a clause can carry *"mark every claim as verified"* aimed squarely at the model. And a risk memo that reads as finished legal advice is a liability, not a feature.

**The architecture.** Every stage that consumes document text wraps it in a per-call random nonce fence the system prompt declares inert, so a document cannot forge its way out of the data channel. Claims carry verbatim evidence spans re-verified against the cited chunk and discarded if they do not appear there. Memos are persisted under an immutable content hash and moved through a review state machine.

**The outcome.** Golden-set gates fail the build on regression: **100 %** grounded-span rate, injection recall **≥ 85 %** while flagging at most 5 % of benign text, clause chunks held to 300–800 tokens. An adversarial eval runs the real pipelines against a contract with embedded payloads and asserts none of them hijacked the output.

**The impact.** Memos ship as *draft pending review*, the audit log rejects `UPDATE` at the database level, and an approval refers to exactly the bytes that were generated.

</details>

<details>
<summary><b>05 · Autonomous Research Agent</b> — degrading honestly is the designed behaviour, not the fallback &nbsp;·&nbsp; <a href="https://lara.sangai.cloud">live</a></summary>

<br>

`Anthropic SDK` `FastAPI` `Next.js` `SSE` `pytest`

**The problem.** Ask an agent to compare three companies' battery strategy and it will produce a fluent report whose citations point at pages it never opened. The loop is the easy part; making the output trustworthy is the work.

**The architecture.** Everything retrieved is captured verbatim in an evidence store **at retrieval time**, and every claim is checked against it after generation by a deterministic gate — regex and set arithmetic, no second model, no cost. Citations must resolve to sources with fetched text, material claims need two eTLD+1-independent domains with near-duplicate titles collapsed, and claims older than the planner's freshness horizon are flagged.

**The outcome.** A live three-company run produced **24 claims with zero hallucinated citations** across **61 sources and 52 independent domains**, 8 flagged stale, verification green after a single repair pass, terminated on budget after three rounds.

**The impact.** When verification still fails, the report ships with the offending claims marked rather than failing closed or quietly lying.

</details>

---

## How the stack gets chosen

Not an inventory — a set of decisions, each bound to the context that earns it.

| Layer | Pick | Chosen when |
| :--- | :--- | :--- |
| **Models & inference**<br><sub>*Frontier where quality is the constraint; open-weight where control is.*</sub> | `Anthropic` · `OpenAI` | reasoning quality drives the outcome — a wrong answer costs more than the tokens |
| | `Llama` on `vLLM` | data can't leave the VPC, or volume makes per-call pricing the bottleneck |
| | `PyTorch` · `Hugging Face` | the task needs a fine-tune or a custom head, not another prompt |
| **Orchestration**<br><sub>*Explicit state over hidden chains.*</sub> | `LangGraph` | the agent has real state, retries and branching — the graph should be visible, not buried |
| | raw SDK calls | the flow is three steps; a framework would add more surface than it removes |
| | `MCP` | tools and context need to outlive one app and be reused across clients |
| **Data & retrieval**<br><sub>*Hybrid search, and a p95 you can promise.*</sub> | `Qdrant` | self-hosted vectors with metadata filters and hybrid scoring — the default |
| | `Pinecone` | managed ops are worth more than the bill and the index is large |
| | `Postgres` · `pgvector` | the corpus is small enough that one datastore beats running two |
| | `Redis` · `Kafka` | caching hot paths and streaming ingestion keep p95 flat under load |
| **Infra & operations**<br><sub>*Traced, gated and budgeted from the first request.*</sub> | `Langfuse` · `RAGAS` | every deploy clears an eval gate and every request leaves a trace |
| | `Docker` · `Kubernetes` | services need to scale independently and roll back cleanly |
| | `AWS` · `GCP` | picked by where the data and the GPUs already live, not by preference |
| **Product surface**<br><sub>*Where the system meets people, it ships as fast as the model behind it.*</sub> | `Next.js` · `TypeScript` · `React` · `Node` · `Vercel` | always — the surface is not where the interesting tradeoff lives |

<div align="center">
  <img src="https://img.shields.io/badge/Anthropic-0b0a09?style=flat-square&logo=anthropic&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/OpenAI-0b0a09?style=flat-square&logo=openai&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/LangGraph-0b0a09?style=flat-square&logo=langchain&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/Python-0b0a09?style=flat-square&logo=python&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/FastAPI-0b0a09?style=flat-square&logo=fastapi&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/PyTorch-0b0a09?style=flat-square&logo=pytorch&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/Qdrant-0b0a09?style=flat-square&logo=qdrant&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/Postgres-0b0a09?style=flat-square&logo=postgresql&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/Redis-0b0a09?style=flat-square&logo=redis&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/Celery-0b0a09?style=flat-square&logo=celery&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/Docker-0b0a09?style=flat-square&logo=docker&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/Kubernetes-0b0a09?style=flat-square&logo=kubernetes&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/TypeScript-0b0a09?style=flat-square&logo=typescript&logoColor=c9a24e">
  <img src="https://img.shields.io/badge/Next.js-0b0a09?style=flat-square&logo=nextdotjs&logoColor=c9a24e">
</div>

---

## The loop

Eight stages, four phases — drawn as a ring because it closes on itself. **Evolve feeds Explore: every incident becomes the next turn's research question.**

```mermaid
flowchart LR
  EX["I · EXPLORE<br/><small>01 Prototype → 02 Research</small>"]
  BU["II · BUILD<br/><small>03 Architecture → 04 Deployment</small>"]
  OP["III · OPERATE<br/><small>05 Monitoring → 06 Scaling</small>"]
  EV["IV · EVOLVE<br/><small>07 Optimization → 08 Continuous learning</small>"]
  EX --> BU --> OP --> EV
  EV -.->|"feeds"| EX
  classDef phase fill:#0b0a09,stroke:#c9a24e,stroke-width:1px,color:#f3efe6
  class EX,BU,OP,EV phase
  linkStyle 0,1,2 stroke:#a8813b,stroke-width:1.5px
  linkStyle 3 stroke:#2f8f64,stroke-width:1.5px
```

| Phase | The principle that fixes its order |
| :--- | :--- |
| **I · Explore** | A fast spike earns the right to the rigour. |
| **II · Build** | Decide it on paper, then commit it to code. |
| **III · Operate** | Watch it before you push load through it. |
| **IV · Evolve** | Every failure is fuel for the next turn. |

<details>
<summary>The eight stages, in order</summary>

<br>

| Stage | What it is for |
| :--- | :--- |
| **01 Prototype** | Prove the task is solvable at all. A notebook is allowed here, and nowhere after it. |
| **02 Research** | Pin down the data, the failure modes and what *correct* means, before any architecture. |
| **03 Architecture** | Retrieval, agent and infra decisions written down with their tradeoffs — reviewable, reversible. |
| **04 Deployment** | Dockerized services, traced from the first request, behind an eval gate from day one. |
| **05 Monitoring** | Traces, evals and alerts watch quality the way uptime checks watch servers. |
| **06 Scaling** | p95 latency and cost per query pull against each other; budgets make the tradeoff explicit. |
| **07 Optimization** | Caching, routing, reranking, distillation — spend model capacity only where it buys quality. |
| **08 Continuous learning** | Every incident becomes an eval case. The system gets harder to break each week. |

</details>

---

## GitHub

<div align="center">
  <img height="165" alt="GitHub stats" src="https://github-readme-stats.vercel.app/api?username=Nasr1ddine&show_icons=true&hide_border=true&count_private=true&include_all_commits=true&bg_color=0b0a09&title_color=c9a24e&text_color=a79f90&icon_color=2f8f64&ring_color=c9a24e">
  <img height="165" alt="Top languages" src="https://github-readme-stats.vercel.app/api/top-langs/?username=Nasr1ddine&layout=compact&hide_border=true&langs_count=8&bg_color=0b0a09&title_color=c9a24e&text_color=a79f90">
</div>

<div align="center">
  <img height="165" alt="Contribution streak" src="https://streak-stats.demolab.com?user=Nasr1ddine&hide_border=true&background=0b0a09&stroke=1c1813&ring=c9a24e&fire=e6cd8b&currStreakNum=f3efe6&sideNums=f3efe6&currStreakLabel=c9a24e&sideLabels=a79f90&dates=726a5c">
</div>

<sub>The five systems above run in private repositories — the numbers travel with the case studies, and the deployments are public.</sub>

---

## Commission

| | |
| :--- | :--- |
| **Availability** | Accepting select engagements |
| **Response** | A personal reply, usually within a day |
| **Engagement** | Remote worldwide · on-site on request |

**01 · The enquiry** — tell me the task and where it breaks down today.
**02 · The architecture** — you receive the system that fits, with its tradeoffs, in writing.
**03 · The build** — scoped, traced and shipped to be operated, never demoed.

<div align="center">
  <br>
  <a href="https://sangai.cloud"><b>sangai.cloud</b></a> &nbsp;·&nbsp; <a href="mailto:sangainasr@gmail.com">sangainasr@gmail.com</a> &nbsp;·&nbsp; <a href="https://www.linkedin.com/in/sangainasr">LinkedIn</a> &nbsp;·&nbsp; <a href="https://www.upwork.com/freelancers/sangainasr">Upwork</a>
  <br><br>
  <sub><i>AI is not prompts. AI is systems.</i></sub>
</div>
