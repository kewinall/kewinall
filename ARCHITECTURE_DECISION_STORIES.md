# P2.3 — Architecture Decision Stories

> 12 個 Senior-level Architecture Deep Dive 面試故事  
> Positioning: **Senior Data Engineer · Data Platform Engineer · DataOps · Governed AI Integration**

---

## 0. 使用方式

P2.1 解決「專案怎麼講」，P2.2 解決「履歷怎麼對到 GitHub」；P2.3 則處理：

> **當面試官開始追問「為什麼這樣設計？如果掛掉怎麼辦？為什麼不用更簡單的方法？」時，要怎麼回答。**

每題固定使用：

```text
Question
  ↓
What the interviewer is testing
  ↓
Decision
  ↓
Why
  ↓
Trade-off
  ↓
Failure / Recovery
  ↓
Evidence
  ↓
Senior-level close
```

不要背逐字稿。記住每題的 **Decision / Trade-off / Failure semantics** 即可。

---

# Decision Index

| # | Architecture Decision | Primary Repo | 主題 |
|---:|---|---|---|
| 1 | 為什麼 Airflow 與 Apache Hop 要拆責任？ | Enterprise ETL Platform | Data Engineering |
| 2 | 為什麼 PostgreSQL Audit DB 是 execution truth？ | Enterprise ETL Platform | Reliability / Audit |
| 3 | 為什麼 build once / immutable promotion？ | Enterprise ETL Platform | Supply Chain |
| 4 | 為什麼 Observability 不能等同 execution truth？ | Enterprise ETL Platform | Observability |
| 5 | 為什麼 AI Agent 不直接連 DB / Airflow？ | Data Platform MCP Server | Integration / Security |
| 6 | Read-only 為什麼不能只靠 Prompt？ | Data Platform MCP Server | Defense in Depth |
| 7 | RBAC 與 Tenant Isolation 為什麼要拆？ | Data Platform MCP Server | Authorization |
| 8 | Agent reasoning 與 production authority 為什麼必須分離？ | Agentic DataOps Copilot | AI Governance |
| 9 | Human Approval 不會讓 Automation 失去價值嗎？ | Agentic DataOps Copilot | Risk / Operations |
| 10 | 為什麼 RAG 不只使用 Vector Search？ | Enterprise RAG Platform | Retrieval |
| 11 | 為什麼需要 Multi-LLM Gateway？ | Multi-LLM AI Gateway | Platform Architecture |
| 12 | Redis 掛掉時哪些 guarantee 會失效？ | Multi-LLM AI Gateway | Distributed Systems |

---

# 1. 為什麼 Airflow 與 Apache Hop 要拆責任？

**Primary Repo:** [Enterprise ETL Platform](https://github.com/kewinall/enterprise-etl-platform)

## 面試官真正想測什麼

不是在問你「會不會 Airflow / Hop」，而是在測：

- 能不能定義 control plane 與 execution runtime boundary
- 是否理解 coupling
- 是否能說出拆分後增加的 operational cost
- 故障時能不能知道責任在哪一層

## Decision

> **Airflow 負責 orchestration；Apache Hop 負責 data transformation。**

Airflow 管：

- scheduling
- dependency
- retry context
- execution context
- workflow coordination

Hop 管：

- extraction
- transformation
- loading
- data processing logic

## 為什麼不全部放 Airflow？

如果所有 transformation 都塞入 PythonOperator：

- workflow definition 與 business transformation 耦合
- 大量 ETL logic 難以獨立測試與移植
- data engineer 必須把大量 transformation 轉成 scheduler-specific code

反過來，如果全部由 Hop 處理：

- scheduler / dependency / retry / cross-workflow orchestration 能力會與 ETL runtime 混在一起
- platform control plane 不夠清楚

## Trade-off

拆分不是免費的。

代價包括：

- 多一個 runtime
- connection / configuration 管理增加
- troubleshooting 需要跨 Airflow、Hop、Audit DB
- version compatibility 要治理

所以我的判斷不是「microservice 越多越好」，而是：

> **只有在 responsibility boundary 清楚、且各自有不同 lifecycle 時才值得拆。**

## Failure / Recovery

### Airflow 正常、Hop 失敗

Airflow 應把 Hop failure 視為 task failure，依 retry policy 產生新 attempt。

### Hop 成功、Audit 寫入失敗

不能只因 transformation 成功就說整個 lifecycle 已完成，因為 durable execution evidence 不完整。

## Evidence

- `scripts/etl_lifecycle_smoke.sh`
- `docs/AUDIT_LIFECYCLE.md`
- `postgres/init/002_v0_3_audit_lifecycle.sql`

## 常見踩雷

錯誤回答：

> 因為 Airflow 是 scheduler，Hop 是 ETL，所以拆開。

這只是產品分類，不是 architecture reasoning。

## Senior-level Close

> 我拆 Airflow 與 Hop 不是因為工具名稱，而是讓 orchestration control plane 和 data-processing runtime 有清楚責任邊界；代價是 operational complexity 上升，所以必須再用 audit、observability 與 failure semantics 把兩層重新串起來。

---

# 2. 為什麼 PostgreSQL Audit DB 是 Execution Truth？

**Primary Repo:** Enterprise ETL Platform

## 面試官真正想測什麼

- 是否理解 scheduler state 與 business execution history 的差別
- Retry 是否會覆蓋 failure
- Audit / SLO / RCA 從哪裡取得資料
- Durable state 如何設計

## Decision

> **將 logical run、attempt、event 持久化到 PostgreSQL，作為 execution truth。**

例如：

```text
Logical Run #123
   ├─ Attempt 1 → FAILED
   └─ Attempt 2 → SUCCESS
```

最終狀態雖然 SUCCESS，但第一次 failure 不能消失。

## Why

只看 Airflow 最終 task state 會損失：

- retry history
- failure frequency
- recovery behavior
- attempt duration
- audit trace

這會讓：

- SLO 失真
- incident investigation 困難
- 「第一次為何失敗」無法追溯

## Trade-off

PostgreSQL 因此變成 platform dependency：

- HA
- backup
- retention
- schema migration
- capacity
- access control

都需要治理。

## Failure / Recovery

### Audit DB unavailable

這是最重要的回答。

我不會說：

> ETL 照跑就好，之後再補。

因為這取決於 business criticality。

合理做法是先定義 consistency requirement：

- 強 audit requirement：fail closed / stop execution
- 可接受 delayed audit：buffer event，再補寫
- reconciliation model：execution 完成後必須重新驗證

Portfolio 中採較保守 semantics：

> Durable execution truth 無法確認時，不把 scheduler UI 當成完整事實來源。

## Evidence

- audit lifecycle schema
- lifecycle smoke test
- observability views 直接依 audit truth 產生 metrics

## 常見踩雷

> PostgreSQL 比 Airflow metadata DB 比較好查。

這不是核心理由。

## Senior-level Close

> Execution truth 的價值不是多一張 log table，而是建立一個不會因 retry、UI 或 monitoring state 改變而消失的 durable operational history。

---

# 3. 為什麼 Build Once / Immutable Promotion？

**Primary Repo:** Enterprise ETL Platform

## 面試官真正想測什麼

- CI/CD 是否只停留在 Git branch
- 是否理解 environment drift
- supply-chain / artifact identity
- air-gapped deployment

## Decision

> **同一份 artifact 在 TEST 驗證後 promotion 到 PROD，不重新 build。**

```text
Source Commit
     ↓
Build
     ↓
Artifact Digest
     ↓
TEST validation
     ↓
Promote same digest
     ↓
PROD
```

## Why

如果 TEST、PROD 都自己 build：

即使 source commit 相同，也可能因：

- base image 更新
- package repository 改變
- dependency resolution
- timestamp / build context
- external download

得到不同 artifact。

這表示：

> TEST 驗證過的，不一定是 PROD 真正執行的。

## Trade-off

需要額外維護：

- registry lifecycle
- digest / checksum
- promotion process
- artifact retention
- offline bundle
- rollback identity

## Air-gapped Scenario

封閉網段最容易暴露這個問題。

我的策略：

```text
Connected Build Environment
        ↓
Image / Artifact Bundle
        ↓
Checksum / Identity
        ↓
Offline Transfer
        ↓
Verification
        ↓
Private Environment
```

不是進 PROD 後再重新下載 / rebuild。

## Failure / Recovery

### Checksum mismatch

**Fail closed。**

重新產 bundle 或重新確認 source artifact。

不能說：

> 大概一樣，先部署。

## Evidence

- `scripts/promote_image.sh`
- `scripts/verify_offline_bundle.sh`
- `scripts/supply_chain_smoke.sh`

## 常見踩雷

> Docker image 本來就是 immutable。

Tag 不是 immutable identity。

例如：

`app:latest`

可以被重新指向。

真正要談的是 digest / content identity。

## Senior-level Close

> CI/CD 的信任鏈不是「同一個 Git commit」，而是「被測試的 artifact 與被部署的 artifact 是同一個 identity」。

---

# 4. 為什麼 Observability 不能等同 Execution Truth？

**Primary Repo:** Enterprise ETL Platform

## 面試官真正想測什麼

- Monitoring 與 system-of-record boundary
- Prometheus failure semantics
- metrics vs audit
- SLO data source

## Decision

> **PostgreSQL 保存 execution truth；SQL Exporter / Prometheus / Grafana 是觀測平面。**

```text
Execution
   ↓
PostgreSQL Audit
   ↓
Read-only Views
   ↓
SQL Exporter
   ↓
Prometheus
   ↓
Alertmanager / Grafana
```

## Why

Prometheus 是 time-series monitoring system，不應被當成業務 execution ledger。

Monitoring stack 可能：

- restart
- scrape miss
- retention expire
- network partition
- rule evaluation delay

但這不應使 ETL history 消失。

## Trade-off

多一層資料模型：

- audit schema
- observability views
- exporter
- metrics

需要確保 metric definition 與 underlying audit semantics 一致。

## Failure / Recovery

### Grafana 掛掉

只是 visualization unavailable。

### Prometheus 掛掉

alert / SLO evaluation 暫停，但 execution truth 還在 DB。

### Audit DB 掛掉

這才是更高級別的 lifecycle dependency incident。

## Evidence

- `scripts/observability_smoke.sh`
- `postgres/init/003_v0_5_observability.sql`
- `monitoring/sql-exporter/etl_audit.collector.yml`

## Senior-level Close

> Observability 是對 truth 的衍生觀察，不應反過來成為 truth 本身；這讓 monitoring failure 與 data execution failure 可以有不同 recovery path。

---

# 5. 為什麼 AI Agent 不直接連 Database / Airflow？

**Primary Repo:** [Data Platform MCP Server](https://github.com/kewinall/data-platform-mcp-server)

## 面試官真正想測什麼

- AI integration boundary
- least privilege
- capability-oriented architecture
- backend credential governance

## Decision

> **AI Client 只能透過 MCP Tool Contract 存取被允許的 capability，而不是直接取得 backend credentials。**

## Why

如果每個 Agent 直接連：

- PostgreSQL
- Vertica
- Airflow
- OpenSearch / Loki

那每個 Agent 都必須自己處理：

- authentication
- authorization
- tenant isolation
- SQL safety
- audit
- timeout
- backend-specific APIs

更大的問題是：

> DB credential 通常授權的是「connection 能做什麼」，不是 Agent「這個任務應該能做什麼」。

MCP tool layer 可以提供較小 capability：

- describe_table
- get_lineage
- explain_sql
- get_dag_status
- search_logs

## Trade-off

MCP 增加：

- network hop
- protocol maintenance
- tool schema versioning
- adapter maintenance
- latency

所以不是所有 internal script 都需要 MCP。

當 client 少、backend 單一、風險低時，直接 API integration 可能更合理。

## Failure / Recovery

某個 backend outage：

- 對應 adapter degraded
- 不提高權限
- 不 fallback 到 unrestricted DB connection

## Evidence

- `tests/test_mcp_protocol.py`
- `tests/test_integrations.py`
- `tests/test_vertica.py`
- `tests/test_auth_audit.py`

## Senior-level Close

> 我使用 MCP 的主要原因不是「AI framework 比較新」，而是把 Agent 從 backend credential holder 變成 governed capability consumer。

---

# 6. Read-only 為什麼不能只靠 Prompt？

**Primary Repo:** Data Platform MCP Server

## 面試官真正想測什麼

- 是否把 LLM prompt 當 security control
- defense in depth
- SQL security
- fail-safe defaults

## Decision

Read-only 至少分三層：

```text
Tool Surface
   ↓
SQL Policy / AST Validation
   ↓
Database Read-only Session / User
```

## Why

System Prompt：

> 「不要執行 DELETE」

不是 authorization mechanism。

因為：

- Prompt injection
- malformed model output
- application bug
- direct API caller

都可能繞過語言層約束。

## SQL AST

用 SQLGlot 等 AST parser 檢查 query，比 regex 更合理。

因為 regex 很難正確處理：

- nested query
- comments
- CTE
- vendor syntax
- obfuscation

## Trade-off

越保守的 SQL policy：

- false positive 越多
- 某些合法分析 SQL 可能被拒絕
- dialect compatibility 需要測試

所以 policy 應該顯式 versioning / testing。

## Failure / Recovery

Policy parser 無法確定 query 是否安全：

> **Reject，而不是猜測安全。**

## Evidence

- `src/data_platform_mcp/security.py`
- `tests/test_security.py`
- DB read-only examples

## 常見踩雷

> 我用 Prompt 加 SQL allow-list，所以安全。

Prompt 只能算 model guidance，不是 security boundary。

## Senior-level Close

> LLM 是 probabilistic component，因此 security boundary 必須放在 deterministic layer；Prompt 可以降低錯誤率，但不能承擔 authorization responsibility。

---

# 7. RBAC 與 Tenant Isolation 為什麼要拆？

**Primary Repo:** Data Platform MCP Server

## 面試官真正想測什麼

- authorization model
- multi-tenancy
- role vs resource scope
- identity design

## Decision

> **RBAC 回答 what；Tenant Policy 回答 which resource / source。**

例如：

```text
Role = analyst
  → 可以 describe_table / explain_sql

Tenant = tenant-a
  → 只能看 postgres-a

Tenant = tenant-b
  → 只能看 vertica-b
```

有 analyst role 不代表可以看所有 tenant。

## Why

只做 RBAC：

```text
analyst → catalog:read
```

沒有定義：

> 哪一個 catalog？

反過來只做 tenant：

> 知道你屬於 tenant-a，但不知道能不能執行 explain_sql。

兩者是不同 authorization dimension。

## Trade-off

Policy model 會更複雜：

- identity mapping
- role mapping
- source mapping
- policy debugging
- audit reason

而且目前 Portfolio 的 tenant isolation 是 **source-level**，不應宣稱等於 row-level RLS。

## Extension

若需要 row-level：

- PostgreSQL RLS
- secure view
- query rewrite
- semantic layer policy

應下推到更接近 data boundary 的層。

## Failure / Recovery

Identity claim 缺 tenant：

- fail closed
- 不給 default global tenant

## Evidence

- auth / tenancy tests
- OIDC role mapping
- source-level tenant policy

## Senior-level Close

> Authorization 很少是一個 role 就能完整描述；operation permission 與 data boundary 應分開建模，才能清楚 audit「誰能做什麼」以及「他能對哪個資料做」。

---

# 8. Agent Reasoning 與 Production Authority 為什麼必須分離？

**Primary Repo:** [Agentic DataOps Copilot](https://github.com/kewinall/agentic-dataops-copilot)

## 面試官真正想測什麼

- AI governance maturity
- autonomy boundary
- production safety
- deterministic vs probabilistic system

## Decision

```text
Evidence
   ↓
LLM Reasoning
   ↓
Action Proposal
   ↓
Deterministic Policy
   ↓
Approval
   ↓
Explicit Executor
   ↓
Audit
```

LLM 產生的是 proposal，不是 authorization。

## Why

模型可能因：

- hallucination
- incomplete evidence
- prompt injection
- tool poisoning
- reasoning error

產生錯誤 action。

如果 reasoning output 直接等於 command：

> 模型錯誤就直接變 production mutation。

## Trade-off

- automation latency 上升
- policy rule 要維護
- executor 要 individually govern
- full autonomy 降低

但 enterprise operations 的目標不是 autonomy 最大化，而是：

> **在可接受風險下提高 operator throughput。**

## Failure / Recovery

- malformed plan → policy reject
- policy unavailable → no allow = no execution
- approval reject → retain audit
- executor failure → record failure → retry / rollback / manual recovery

## Evidence

- `tests/test_governance.py`
- `src/agentic_dataops_copilot/governance/policy.py`
- `src/agentic_dataops_copilot/governance/audit.py`

## Senior-level Close

> Intelligence 可以 probabilistic，但 authority 必須 deterministic。這是我設計 production AI Agent 最重要的 boundary。

---

# 9. Human Approval 不會讓 Automation 失去價值嗎？

**Primary Repo:** Agentic DataOps Copilot

## 面試官真正想測什麼

- 能否避免 security absolutism
- 是否理解 risk-based automation
- human-in-the-loop design

## 錯誤極端

### 極端 A

全部 auto-remediation。

風險太高。

### 極端 B

所有事情都必須人手 approve。

那 Agent 只是比較複雜的 chatbot。

## Decision

應採 **risk-tiered authority**。

例如：

| Risk | Example | Authority |
|---|---|---|
| Low | read logs / get metadata | automatic |
| Medium | bounded retry / restart candidate | policy controlled |
| High | schema change / destructive mutation | human approval |
| Critical | irreversible production change | multi-party / change process |

## Why

Automation 的價值不只在最後一個「執行按鈕」。

Agent 可以先自動完成：

- evidence gathering
- log correlation
- timeline
- probable RCA
- runbook retrieval
- proposed remediation

把 30 分鐘調查縮成 3 分鐘，即使最後仍由人 approve，也有很大價值。

## Trade-off

需要：

- risk classification
- policy
- escalation
- approval SLA
- ownership

## Failure / Recovery

Approver timeout：

- action remains pending
- 不因 timeout 自動升權
- 可依 incident severity escalation 到另一 approver

## Evidence

- governance tests
- separation-of-duties
- explicit executor registry
- hash-chained audit

## Senior-level Close

> Human-in-the-loop 不應放在每一個步驟，而應只放在 authority boundary；前面的 evidence gathering 與 reasoning 仍可高度自動化。

---

# 10. 為什麼 Enterprise RAG 不只用 Vector Search？

**Primary Repo:** [Enterprise RAG Platform](https://github.com/kewinall/enterprise-rag-platform)

## 面試官真正想測什麼

- IR fundamentals
- RAG 是否只停在 framework
- quality vs latency trade-off
- evaluation thinking

## Decision

```text
Vector Retrieval
      +
BM25 Retrieval
      ↓
RRF Fusion
      ↓
Optional Reranker
```

## Why

Vector Search 強在 semantic similarity。

但對：

- error code
- product ID
- table name
- policy number
- exact entity
- uncommon acronym

BM25 / lexical match 可能更穩。

Hybrid 可以降低單一路徑 blind spot。

## 為什麼 RRF？

Vector score 與 BM25 score 的尺度不同。

直接 normalize / weighted sum 很容易受 score distribution 影響。

RRF 主要根據 ranking position fusion，較簡單且 robust。

## Reranker Trade-off

Reranker 可以提高 precision，但會增加：

- latency
- compute
- cost

所以不是所有 query 都需要 rerank。

## Failure / Recovery

Retrieval evidence 不足：

> 明確回答 evidence insufficient。

不要讓 LLM 用流暢文字補空白。

## Evidence

- retrieval evaluation
- answer evaluation
- adversarial evaluation
- tenancy / filter tests

## 常見踩雷

> Hybrid 一定比 Vector 好。

不一定。

要用 evaluation dataset 證明 quality gain 是否值得 complexity。

## Senior-level Close

> Retrieval architecture 應由 evaluation data 決定，而不是由目前最流行的 embedding model 決定。

---

# 11. 為什麼需要 Multi-LLM Gateway？

**Primary Repo:** [Multi-LLM AI Gateway](https://github.com/kewinall/multi-llm-ai-gateway)

## 面試官真正想測什麼

- platform centralization trade-off
- control plane
- abstraction cost
- SPOF thinking

## Problem

如果五個 application 各自接三個 provider：

```text
5 applications ×
  identity
  API key
  retry
  fallback
  budget
  rate limit
  logging
  provider SDK
```

治理會快速碎片化。

## Decision

建立 centralized model control plane：

```text
Applications
    ↓
AI Gateway
    ├─ Identity
    ├─ Policy
    ├─ Budget
    ├─ Routing
    ├─ Retry / Fallback
    ├─ Cost
    └─ Observability
          ↓
   Model Providers
```

## Why

集中處理 cross-cutting concern：

- provider selection
- cost
- quota
- identity
- audit
- resilience

application 專注 business logic。

## Trade-off

最大代價：

> Gateway 自己變成 critical path。

因此必須處理：

- capacity
- HA
- readiness
- PDB
- observability
- security
- deployment ownership

另一個 trade-off 是 abstraction leakage。

OpenAI-compatible API 無法完美代表所有 provider-native capability。

## Failure / Recovery

Gateway pod failure：

- multiple replicas
- readiness
- Service reroute
- PDB

整個 Gateway unavailable：

- 是否允許 application direct-to-provider bypass？

企業治理情境通常不建議無條件 bypass，否則 policy / budget / audit 全失效。

## Evidence

- router tests
- governance tests
- identity tests
- Helm hardening
- observability implementation

## Senior-level Close

> Gateway 的價值不是多包一層 API，而是把 provider integration 從 application concern 變成 platform governance concern；前提是承認並治理新的 critical path。

---

# 12. Redis 掛掉時哪些 Distributed Guarantee 會失效？

**Primary Repo:** Multi-LLM AI Gateway

## 面試官真正想測什麼

這題是在測 distributed systems maturity。

不是問：

> Redis 要怎麼重啟？

而是：

> 你知不知道哪些 state 不能當 cache？

## Shared State

Gateway 多 replicas 時，Redis 保存：

- rate-limit state
- round-robin cursor
- circuit-breaker state
- usage
- budget

## Key Insight

Redis 在這裡不是單純 cache。

如果 Redis unavailable，而每個 pod 改成 local memory：

```text
Pod A budget = 50%
Pod B budget = 50%
```

兩個 pod 各自以為還有額度，global budget guarantee 失效。

同樣問題也會發生在：

- rate limit
- circuit breaker
- provider rotation

## Decision

> Redis unavailable 應被視為 **distributed control dependency incident**。

而不是：

> cache miss，繼續就好。

## Failure Strategy

真正 production 要依 business policy 決定：

### Option A — Fail Closed

適合：

- strict financial budget
- compliance policy
- hard quota

### Option B — Degraded Local Mode

適合：

- availability 比 global consistency 更重要
- 可以接受短時間 quota overshoot

但一定要明確標示：

> global guarantee temporarily unavailable.

## Trade-off

Fail closed：

- governance strong
- availability lower

Degraded mode：

- availability higher
- global consistency weaker

沒有 universally correct answer。

## Evidence

- `tests/test_redis_state.py`
- budget / policy / rate-limit implementation
- Kubernetes deployment

## Senior-level Close

> Distributed system failure 最重要的不是「服務有沒有回 200」，而是 failure 發生後哪些 guarantee 還成立、哪些已經不成立。

---

# 13. Cross-Question Patterns

面試官常會換個方式問同一件事。

## Pattern A — 「為什麼不更簡單？」

回答順序：

```text
先承認簡單方案可以成立
→ 說明什麼規模 / governance requirement 下開始失效
→ 說明目前 decision 的 trade-off
```

例如：

> 如果只有一個 Agent、一個 DB，而且是 read-only internal POC，我不一定會先做 MCP。當 client、backend、identity 與 audit requirement 增加後，才值得把 integration boundary 抽出來。

這比：

> MCP 是最佳實務。

更像 Senior Engineer。

---

## Pattern B — 「如果 dependency 掛掉呢？」

不要只回答：

> 做 HA。

先回答：

1. 哪個 guarantee 失效？
2. fail open 還是 fail closed？
3. 是否可 degraded？
4. truth 還在哪裡？
5. recovery 後要不要 reconciliation？

---

## Pattern C — 「這是不是 over-engineering？」

推薦回答：

> 有可能，所以我會先看 scale、risk、team size、compliance 與 operational ownership。Portfolio 是刻意展示完整 production concern；實際專案不會不分規模全部照搬。

這句很重要。

它可以避免 GitHub 作品過於完整反而讓面試官認為你會 over-engineer。

---

# 14. Senior Interview Answer Template

臨場可以用以下 90 秒模板：

```text
我當時先定義的核心問題是 ______。

我選擇 ______，主要是因為 ______。

更簡單的方案是 ______，在 ______ 情境其實可以成立，
但當 ______ 出現後，它的限制會變成 ______。

我的方案代價是 ______，所以我另外需要處理 ______。

如果 ______ dependency 掛掉，最重要的是 ______ guarantee 會失效，
因此我會選擇 fail closed / degraded / retry / reconciliation。

在 Portfolio 裡，我用 ______ test / smoke / audit evidence 驗證這個 claim。
```

---

# 15. Architecture Decision Ranking by Target Role

## Senior Data Engineer

優先熟練：

1. Airflow vs Hop responsibility
2. PostgreSQL execution truth
3. Immutable promotion
4. Observability vs truth
5. Hybrid Retrieval

## Data Platform Engineer

優先熟練：

1. MCP integration boundary
2. Read-only defense-in-depth
3. RBAC vs Tenant isolation
4. Immutable promotion
5. Multi-LLM Gateway

## DataOps / Platform Engineer

優先熟練：

1. Observability vs execution truth
2. Agent reasoning vs authority
3. Human approval
4. Redis distributed state
5. Immutable promotion

## AI Platform / AI Integration

優先熟練：

1. Agent reasoning vs authority
2. Hybrid Retrieval
3. MCP boundary
4. Multi-LLM Gateway
5. Redis failure semantics
6. Read-only defense-in-depth

---

# 16. Claim Safety

P2.3 講的是 **architecture reasoning**。

請維持 P2.2 的 Evidence boundary：

### 可以說

> 我在公開 Portfolio 中採這個設計，並透過 regression / smoke tests 驗證。

### 真實工作有直接經驗時可以說

> 我在 production 遇過類似 reliability / migration / operations 問題，因此 Portfolio 把它抽象成可公開 reference implementation。

### 不要說

> 這 12 個 architecture decision 全部都已在同一個客戶 production system 落地。

---

# 17. P2.3 Completion Checklist

- [x] 12 個 Senior-level architecture deep-dive stories
- [x] Data Engineering architecture decisions
- [x] Data Platform / MCP decisions
- [x] Security / identity / tenant decisions
- [x] Agent governance decisions
- [x] RAG retrieval decision
- [x] Multi-LLM control plane decision
- [x] Distributed Redis failure semantics
- [x] 每題包含 Trade-off
- [x] 每題包含 Failure / Recovery
- [x] 每題包含 GitHub Evidence
- [x] 建立 cross-question patterns
- [x] 建立 90 秒臨場回答模板
- [x] 建立 role-specific priority
- [x] 延續 P2.2 Claim Safety

**Next: P2.4 — Target Job Mapping**

P2.4 不再增加通用故事，而是拿實際 JD 做：

```text
Job Requirement
      ↓
Resume Evidence
      ↓
Best GitHub Repo
      ↓
Interview Story
      ↓
Gap / Risk
      ↓
Resume / GitHub emphasis
```
