# P2.4 — Target Job Mapping

> 目的：把 **實際 JD Requirement → Resume Evidence → GitHub Evidence → Interview Story → Gap / Risk → Application Emphasis** 串成可重複使用的投遞決策流程。  
> Positioning: **Senior Data Engineer · Data Platform Engineer · DataOps / Platform Engineer · Governed AI Platform Integration**

---

## 0. P2.4 要解決什麼

P2.1 已回答「專案怎麼講」；P2.2 已回答「履歷與 GitHub 怎麼對應」；P2.3 已回答「被 Senior-level 深挖時怎麼回答」。

P2.4 的問題更接近實際求職：

> **同一份經歷與同一組 Portfolio，面對不同 JD 時，哪些 evidence 要放前面、哪些只當加分、哪些 gap 要主動控制？**

核心流程：

```text
Job Description
      ↓
Requirement Classification
      ↓
Evidence Level
      ↓
Resume Emphasis
      ↓
GitHub Review Order
      ↓
Interview Story Selection
      ↓
Gap / Risk Handling
      ↓
Application Decision
```

重點不是讓每個職缺都看起來 100% match，而是讓 **最強 evidence 對到最重要 requirement**。

---

# 1. Evidence Level

先沿用 P2.2 的 Evidence boundary，避免為了提高 match rate 過度宣稱。

| Level | Evidence | 定義 | 面試安全說法 |
|---|---|---|---|
| **A** | Real Production Experience | 真實企業 / 公部門交付、migration、operations、量化成果 | 「我實際做過…」 |
| **B** | Portfolio Engineering Evidence | GitHub 可公開驗證的 code、tests、CI、smoke、architecture、evaluation | 「我在 public portfolio 建立並驗證…」 |
| **C** | Engineering Extension | POC、評估、正在深化的技術方向 | 「我有評估 / 延伸實作經驗…」 |
| **D** | Gap | 目前沒有足夠 evidence | 「目前不是我的主要實務強項，但我可從相鄰能力切入…」 |

**規則：A 不需要 B 才成立，B 也不能冒充 A。**

---

# 2. Target Role Archetypes

目前最適合的四個主職缺族群：

| Target Role | 核心價值 | 第一 Repo | 第二 Repo | AI 權重 |
|---|---|---|---|---:|
| **Senior Data Engineer** | DWH / ETL / Data Pipeline correctness、migration、performance | Enterprise ETL Platform | Data Platform MCP Server | 低～中 |
| **Data Platform Engineer** | Shared platform、integration、metadata、security、K8s / CI/CD | Data Platform MCP Server | Enterprise ETL Platform | 中 |
| **DataOps / Platform Engineer** | Reliability、deployment、observability、incident、governance | Agentic DataOps Copilot | Enterprise ETL Platform | 中 |
| **AI Platform / AI Integration Engineer** | Governed AI integration、RAG、MCP、Gateway、evaluation | Multi-LLM AI Gateway / Enterprise RAG | Data Platform MCP / Enterprise ETL | 中～高 |

Portfolio 的基準定位仍是：

```text
Data Engineering / Data Platform        50–60%
Platform / Cloud / DataOps              25–30%
Governed AI Integration                 15–20%
```

只有在 JD 本身明確是 AI Platform / AI Integration 時，才提高 AI evidence 的排序；不要反過來把履歷主體改造成 ML Engineer。

---

# 3. JD Requirement Classification

閱讀 JD 時，不要先看工具名稱，要先把 requirement 分成以下類型。

| Requirement Type | 典型關鍵字 | 主要 Evidence |
|---|---|---|
| **Core Data Engineering** | ETL / ELT、DWH、SQL、batch、data pipeline、migration | 700+ tables / 8TB+、Vertica / Oracle、Pentaho / DataStage、Enterprise ETL Platform |
| **Orchestration / Modernization** | Airflow、Dagster、legacy migration、workflow、dependency | Airflow / Hop responsibility、ETL Intelligence、metadata / lineage |
| **Database / Performance** | Vertica、PostgreSQL、Oracle、query tuning、MPP | production DB experience、3× representative workload improvement |
| **Platform Engineering** | Docker、Kubernetes、Helm、CI/CD、registry、release | five repos、immutable promotion、Helm baselines |
| **Reliability / Operations** | monitoring、SLO、incident、on-call、backup、restore | production operations、ETL audit / observability、DataOps Copilot |
| **Security / Governance** | RBAC、OIDC、audit、tenant、supply chain、SBOM | MCP Server、Gateway、Trivy / SBOM、immutable release |
| **Metadata / Governance** | lineage、catalog、metadata、semantic layer、Denodo | Denodo experience、ETL normalized metadata / lineage、MCP Server |
| **Cloud** | AWS、Azure、cloud-native、managed service | AWS / AKS / container / data service experience + portfolio deployment patterns |
| **RAG / Knowledge AI** | vector DB、RAG、retrieval、citation、evaluation | Enterprise RAG Platform |
| **AI Agent / Tool Integration** | MCP、tool calling、agent、workflow automation | Data Platform MCP Server、Agentic DataOps Copilot |
| **LLM Platform** | routing、fallback、quota、cost、provider abstraction | Multi-LLM AI Gateway |
| **AI Evaluation** | grounding、hallucination、eval、quality、LLMOps | ETL AI Evaluation + RAG Evaluation + Gateway usage evidence |

工具名稱只是 signal。例如 JD 寫 `Airflow`，真正需求可能是 orchestration ownership；寫 `Databricks`，真正需求可能是 scalable data processing / platform integration。先判斷能力，再判斷工具 gap。

---

# 4. JD Fit Scoring

為避免「看起來很多關鍵字都有」造成主觀誤判，使用簡單可解釋的 scoring。

## 4.1 Requirement Weight

| JD 標記 | Weight |
|---|---:|
| Must-have / Required / 核心職責 | 3 |
| Preferred / Nice-to-have | 1 |

## 4.2 Evidence Score

| Evidence | Score |
|---|---:|
| A — 真實 production / quantified experience | 3 |
| B — public executable portfolio evidence | 2 |
| C — POC / adjacent experience | 1 |
| D — 無實質 evidence | 0 |

## 4.3 計算方式

```text
Requirement Score = Weight × Evidence Score

Fit % =
  Σ Requirement Score
  ------------------- × 100
  Σ (Weight × 3)
```

另外加一個 **Critical Gap Rule**：

> 如果某項 Must-have 是職務核心責任，而且 Evidence = D，即使總分很高，也必須列為 Red Flag，不可以用其他 nice-to-have 抵銷。

建議判讀：

| Fit | 判讀 | 行動 |
|---:|---|---|
| **80–100%** | Strong fit | 優先投遞，客製 Resume Summary / Project order |
| **65–79%** | Good adjacent fit | 可投，準備 gap story |
| **50–64%** | Stretch | JD / 公司價值高才投，避免大量客製成本 |
| **<50%** | Low fit | 通常不優先，除非職涯策略刻意轉向 |

Fit score 是決策工具，不是能力評分，也不是 ATS 模擬器。

---

# 5. Target A — Senior Data Engineer

## 5.1 JD 常見核心要求

```text
SQL / DWH / ETL / ELT
Data modeling / data migration
Batch pipeline / orchestration
Performance / reliability
Data quality
Production troubleshooting
Cloud / container / CI/CD（常為加分）
```

## 5.2 Resume Evidence 優先順序

第一屏應優先呈現：

1. **700+ tables / 8TB+ DWH migration**
2. **部分代表性 workload 約 3× performance improvement**
3. Vertica / Oracle / PostgreSQL / SQL
4. DataStage / Pentaho / Apache Hop / Airflow
5. Data quality / automation
6. Production troubleshooting / batch operations
7. GitHub Enterprise ETL Platform
8. AI integration 只當 differentiation

## 5.3 GitHub Review Order

```text
1. enterprise-etl-platform
2. data-platform-mcp-server
3. agentic-dataops-copilot
4. enterprise-rag-platform
5. multi-llm-ai-gateway
```

面試官只有 5 分鐘時，先讓他看到 ETL v0.8.0。

## 5.4 優先 Interview Stories

P2.1：

- Enterprise ETL Platform 30 秒 / 2 分鐘 / 5 分鐘

P2.3 Core：

- #1 Airflow vs Hop responsibility
- #2 PostgreSQL execution truth
- #3 Immutable promotion
- #4 Observability vs execution truth

P2.3 ETL AI Extension：

- #13 Deterministic Parser vs Raw ETL → LLM
- #14 Parser correctness / synthetic ground truth
- #16 AI failure degradation

## 5.5 最適合的 Positioning

> **Senior Data Engineer with strong enterprise migration and production platform experience, plus modern ETL intelligence and governed AI integration.**

不是：

> AI Engineer who also knows ETL.

## 5.6 Gap / Risk

若 JD 強調下列能力，要額外評估：

| JD Requirement | 目前風險 | 應對 |
|---|---|---|
| Spark / Databricks 為核心 | Portfolio 並非主力 | 強調 large-scale DWH / ETL fundamentals；若為硬性多年經驗則列 Critical Gap |
| Kafka / Flink / streaming-first | 主要 evidence 偏 batch / platform | 不把 batch 經驗包裝成 streaming production experience |
| Snowflake-only ecosystem | DB fundamentals 強，但 tool-specific evidence 較弱 | 以 Vertica / Oracle / PostgreSQL MPP / SQL 能力做 adjacent mapping |
| Heavy Scala | 不是目前核心語言 | 若 Required 則降低 fit score |

---

# 6. Target B — Data Platform Engineer

## 6.1 JD 常見核心要求

```text
Shared data platform
Database / warehouse integration
Metadata / lineage / governance
Kubernetes / container
API / platform service
Identity / RBAC
CI/CD / observability
Developer self-service
```

## 6.2 Resume Evidence 優先順序

1. Enterprise Data Platform / DWH experience
2. Vertica / PostgreSQL / Oracle / Denodo
3. Linux / Docker / Kubernetes
4. CI/CD / Monitoring / Security
5. Metadata / Lineage / Data Virtualization
6. Data Platform MCP Server
7. Enterprise ETL Platform
8. Multi-LLM Gateway 作為 platform-control-plane 延伸

## 6.3 GitHub Review Order

```text
1. data-platform-mcp-server
2. enterprise-etl-platform
3. multi-llm-ai-gateway
4. agentic-dataops-copilot
5. enterprise-rag-platform
```

## 6.4 優先 Interview Stories

P2.3 Core：

- #5 AI Agent 不直接連 DB / Airflow
- #6 Read-only 不靠 Prompt
- #7 RBAC vs Tenant Isolation
- #3 Immutable promotion
- #11 Multi-LLM Gateway control plane

P2.3 ETL AI Extension：

- #13 Deterministic metadata / lineage boundary
- #18 Synthetic → private production evaluation governance

## 6.5 最適合的 Positioning

> **Data Platform Engineer who combines enterprise database / integration experience with governed platform APIs, metadata, security and production delivery.**

## 6.6 Gap / Risk

| JD Requirement | 目前風險 | 應對 |
|---|---|---|
| Terraform / IaC 深度為核心 | Portfolio deployment 偏 Helm / scripts / CI | 不過度宣稱 IaC；可強調 release / environment / Kubernetes governance |
| Service mesh 深度維運 | 有 K8s / Gateway adjacent experience，但非主 Portfolio evidence | 視 Required 程度降分 |
| Internal Developer Platform / Backstage | platform thinking 可映射，但無直接 production claim | 用 API contract / self-service capability 思路回答，不說做過 Backstage production |
| 大規模 multi-region platform | Portfolio 有 HA/failure semantics，非真實 hyperscale claim | 強調 architecture reasoning 與既有 enterprise operations |

---

# 7. Target C — DataOps / Platform Engineer

## 7.1 JD 常見核心要求

```text
Production operations
Deployment / release
Monitoring / alerting / SLO
Incident / RCA
Kubernetes
CI/CD
Security patch / CVE
Backup / restore
Automation / runbook
```

## 7.2 Resume Evidence 優先順序

1. Production troubleshooting / operations
2. Linux / Docker / Kubernetes
3. Monitoring / observability
4. CI/CD / release / security patch
5. Backup / restore / recovery
6. Enterprise ETL lifecycle / audit / SLO
7. Agentic DataOps Copilot
8. Multi-LLM Gateway distributed governance

## 7.3 GitHub Review Order

```text
1. agentic-dataops-copilot
2. enterprise-etl-platform
3. multi-llm-ai-gateway
4. data-platform-mcp-server
5. enterprise-rag-platform
```

若 JD 更偏傳統 Platform / SRE 而非 AI-assisted operations，則第一與第二互換：先看 Enterprise ETL Platform。

## 7.4 優先 Interview Stories

P2.3 Core：

- #4 Observability vs execution truth
- #8 Agent reasoning vs production authority
- #9 Risk-tiered Human Approval
- #12 Redis failure semantics
- #3 Immutable promotion
- #2 PostgreSQL execution truth

P2.3 ETL AI Extension：

- #16 AI / Gateway outage degradation
- #17 Token / cost / latency evidence methodology

## 7.5 最適合的 Positioning

> **Production-oriented Data / Platform Engineer with strong troubleshooting, observability, controlled delivery and governed automation experience.**

## 7.6 Gap / Risk

| JD Requirement | 目前風險 | 應對 |
|---|---|---|
| 純 SRE、Go 為主要語言 | Data / platform context 比 pure software SRE 強 | 若 coding language 是核心 gate，要保守評估 |
| 大規模 public-cloud infra ownership | 有 AWS / AKS / platform experience，但需區分實際 ownership 範圍 | 不暗示 hyperscale infra |
| Terraform / Pulumi 為 must-have | 直接 evidence 較少 | Criticality 高時降 fit |
| 24x7 on-call years 要求 | 只有有真實經驗才聲稱 | 不用 troubleshooting 經驗自動等同正式 on-call ownership |

---

# 8. Target D — AI Platform / AI Integration Engineer

## 8.1 JD 常見核心要求

```text
LLM / RAG integration
Model API / Gateway
Prompt / tool calling
Evaluation / grounding
MCP / Agent
Security / identity / governance
Cost / latency / observability
Kubernetes / cloud
Data integration
```

這類職缺是最需要 Claim Safety 的類型。

## 8.2 Resume Evidence 優先順序

1. **先保留 Data Platform credibility**
2. API / Gateway / Kubernetes / Cloud
3. Enterprise RAG Platform
4. Data Platform MCP Server
5. Multi-LLM AI Gateway
6. ETL AI Evaluation
7. Agentic DataOps governance
8. 真實 AI / RAG POC / evaluation experience

不要讓 public portfolio 蓋掉真實工作 evidence。

## 8.3 GitHub Review Order

如果 JD 偏 **LLM Platform / Gateway**：

```text
1. multi-llm-ai-gateway
2. data-platform-mcp-server
3. enterprise-etl-platform
4. enterprise-rag-platform
5. agentic-dataops-copilot
```

如果 JD 偏 **RAG / Knowledge AI**：

```text
1. enterprise-rag-platform
2. multi-llm-ai-gateway
3. data-platform-mcp-server
4. enterprise-etl-platform
5. agentic-dataops-copilot
```

如果 JD 偏 **Agent / Tool Integration**：

```text
1. data-platform-mcp-server
2. agentic-dataops-copilot
3. multi-llm-ai-gateway
4. enterprise-etl-platform
5. enterprise-rag-platform
```

## 8.4 優先 Interview Stories

P2.3 Core：

- #8 Reasoning vs production authority
- #11 Multi-LLM Gateway
- #12 Redis distributed control state
- #5 MCP capability boundary
- #6 Deterministic security control
- #10 Hybrid Retrieval

P2.3 ETL AI Extension：

- #13 Deterministic Truth vs Probabilistic Semantic Layer
- #14 Evaluation scope
- #15 Hallucination Guard capability boundary
- #16 AI failure degradation
- #17 Usage / Cost evidence
- #18 Synthetic → Production Evaluation

## 8.5 最適合的 Positioning

> **Data Platform Engineer extending enterprise data, API and operations experience into governed AI integration, with public evidence for RAG evaluation, MCP, AI gateway governance and deterministic-vs-probabilistic boundaries.**

這比直接宣稱多年 production LLM platform ownership 更可信。

## 8.6 Gap / Risk

| JD Requirement | 目前風險 | 應對 |
|---|---|---|
| ML model training / fine-tuning research | Portfolio 主體是 integration / platform，不是 model research | 若 JD 核心是 training，通常不是主要 target |
| PyTorch / TensorFlow 深度 | 非目前差異化 | 不因使用 LLM API 就映射成 ML engineering |
| Production LLM at hyperscale | 有 production-oriented design evidence，但非同義的 production scale evidence | 清楚區分 B 與 A |
| Advanced evaluation science / statistical experimentation | 已有 eval framework，但 production dataset 尚需 private evidence | 強調 methodology 與 claim boundary |

---

# 9. Hybrid Role Mapping

真實 JD 常常不是純單一角色。

| JD 組合 | 建議主定位 | Repo 順序 |
|---|---|---|
| Data Engineer + Platform | **Senior Data Engineer** → Data Platform | ETL → MCP → DataOps |
| Data Engineer + AI | **Senior Data Engineer with governed AI integration** | ETL → MCP / RAG → Gateway |
| Platform + AI | **Data Platform Engineer with AI control-plane extension** | MCP → Gateway → ETL |
| DataOps + AI Agent | **Production DataOps / Platform Engineer** | DataOps → ETL → Gateway |
| RAG + Data Platform | **AI Integration / Knowledge Platform Engineer** | RAG → MCP → Gateway |
| API Gateway + LLM | **AI Platform / Model Control Plane** | Gateway → MCP → DataOps |

如果 JD 同時列很多 AI buzzword，但主要職責其實是 pipeline / platform ownership，仍應以 Data / Platform 作為履歷標題。

---

# 10. Requirement → Evidence Mapping Cheatsheet

| JD 說法 | Resume Evidence | GitHub Evidence | Interview Story |
|---|---|---|---|
| Build reliable data pipelines | DWH / ETL / production batch | Enterprise ETL lifecycle | P2.3 #1 / #2 / #4 |
| Migrate legacy ETL | Pentaho / DataStage + migration experience | ETL deterministic parser / modernization | ETL AI #13 / #14 |
| Data lineage / metadata | Denodo / governance experience | ETL normalized lineage + MCP metadata | #5 / ETL AI #13 |
| Improve reliability | production troubleshooting | Audit / retry / SLO / failure semantics | #2 / #4 |
| Secure data access for AI | DB / access-control background | MCP Tool Contract / OIDC / RBAC | #5 / #6 / #7 |
| Build AI agents safely | operations governance | DataOps policy / approval / executor | #8 / #9 |
| RAG quality / grounding | RAG evaluation background | Hybrid retrieval / citation / evaluation | #10 |
| Multi-provider LLM | API / Gateway experience | Multi-LLM Gateway | #11 / #12 |
| Control hallucinations | data correctness mindset | ETL evidence validation | ETL AI #15 |
| Measure AI cost / latency | platform / cost governance | Gateway usage contract + ETL eval | ETL AI #17 |
| AI service graceful degradation | production failure handling | deterministic fallback | ETL AI #16 |
| Secure / offline delivery | controlled enterprise deployment | immutable bundle / checksum / signature | #3 |

---

# 11. Resume Customization Rule

每次投遞只需要調整 **排序與摘要**，不要為不同 JD 重寫一份互相矛盾的職涯故事。

## Senior Data Engineer Resume

首頁核心：

```text
DWH Migration → ETL / SQL → Performance → Production Reliability
                                       ↓
                              Modern ETL / AI Evidence
```

## Data Platform Resume

```text
Data Platform / Database → Integration / Metadata → K8s / CI/CD / Security
                                                ↓
                                      Governed AI Extension
```

## DataOps / Platform Resume

```text
Operations → Reliability → Deployment → Observability → Recovery
                                             ↓
                                 Governed Automation / AI
```

## AI Platform Resume

```text
Data / API / Platform Foundation
              ↓
RAG / MCP / Gateway / Evaluation
              ↓
Governance / Cost / Failure Semantics
```

同一個真實量化成果不要因 JD 不同而改變數字或範圍。

---

# 12. GitHub Review Path by Interviewer Time

## 2 分鐘

只給一個 repo：選與 JD 第一責任最接近者。

## 5 分鐘

```text
Primary Repo
   ↓
Engineering Decision / Evidence
   ↓
Second Repo showing boundary / integration
```

例如 Senior Data Engineer：

```text
Enterprise ETL
→ ETL AI Evaluation Evidence
→ MCP Server（metadata / governed integration）
```

## 15 分鐘

再補第三個 repo 展示 cross-platform thinking，但不要五個 repo 全部逐一導覽。

---

# 13. Interview Story Selection Matrix

| Interviewer Question | Primary Story | Secondary Story |
|---|---|---|
| 最大的 data migration？ | Real 700+ tables / 8TB+ | ETL Platform modernization evidence |
| Airflow / workflow 怎麼設計？ | P2.3 #1 | #2 / #4 |
| 怎麼確保 retry 正確？ | #2 Execution Truth | ETL lifecycle smoke |
| Legacy ETL 怎麼導入 AI？ | ETL AI #13 | #14 / #15 |
| AI hallucination 怎麼處理？ | ETL AI #15 | RAG #10 |
| AI 掛掉呢？ | ETL AI #16 | Gateway #11 / #12 |
| LLM 成本如何估？ | ETL AI #17 | Gateway usage governance |
| AI 怎麼安全連資料庫？ | MCP #5 / #6 | #7 |
| 為什麼需要 Gateway？ | #11 | #12 |
| Agent 能不能自動修 production？ | #8 | #9 |
| RAG quality 怎麼證明？ | #10 | ETL AI #14 的 evaluation discipline |
| 如何避免環境 drift？ | #3 | Supply-chain evidence |

---

# 14. Gap Handling Template

遇到缺乏 direct evidence 的 JD requirement，不要用「我很快可以學」作為主要答案。

使用：

```text
1. 明確承認 direct experience boundary
2. 指出最接近的 transferable engineering concept
3. 提供已存在的 adjacent evidence
4. 說明需要補的 tool-specific layer
5. 不把 adjacent experience 說成 years of direct production experience
```

例如 JD 問 Databricks：

> 我目前最強的 production evidence 是 Vertica / Oracle / ETL / DWH 與 Kubernetes data platform，不會把這些直接說成 Databricks production experience。不過在 workload design、SQL / data pipeline、migration、observability 與 platform governance 上有高度可轉移性；若職務的核心是 Spark / Databricks-specific optimization，這會是我需要補強的 tool-specific gap。

這種回答比硬湊 tool keyword 更安全。

---

# 15. Application Decision Template

每個實際 JD 建一張表：

| JD Requirement | Required? | Evidence Level | Evidence | Repo / Story | Gap | Action |
|---|---:|---|---|---|---|---|
| Example: ETL / DWH | Must | A | 700+ tables / 8TB+ | ETL | — | Lead |
| Example: Airflow | Must | B / C | Portfolio + modernization | ETL #1 | production scope需說明 | Support |
| Example: Spark | Must | D | — | — | Critical | Reconsider |
| Example: LLM evaluation | Preferred | B | ETL/RAG eval | ETL AI #14 | public synthetic only | Differentiator |

最後產生六個輸出：

```text
1. Fit Score
2. Critical Gaps
3. Resume Summary emphasis
4. Top 3 Resume bullets
5. GitHub review order
6. Top 5 interview stories
```

---

# 16. Red / Yellow / Green Decision

## Green — 優先投遞

- 主要 Must-have 有 A / B evidence
- 沒有核心 D gap
- Portfolio 可以強化而非取代履歷
- 至少有 2–3 個 strong interview stories

## Yellow — 可投但要準備 Gap Story

- 主能力相符，但有一到兩個 tool-specific gap
- 核心 concept 可 transfer
- 職缺沒有要求多年直接 production ownership

## Red — 不優先

- 核心工作本身落在 D evidence
- 例如 role 實際是 ML research / model training，但只因為 JD 寫 LLM 就投
- 或 streaming / Spark / Scala 是不可替代核心，而目前 direct evidence 明顯不足

---

# 17. Claim Safety by Target Role

| Target | 可以強調 | 不應過度宣稱 |
|---|---|---|
| Senior Data Engineer | 真實 DWH / migration / ETL / DB scale | GitHub synthetic benchmark = production accuracy |
| Data Platform Engineer | DB / platform / K8s / security + public platform evidence | 所有 portfolio platform 已在客戶 production 上線 |
| DataOps / Platform | troubleshooting / deployment / recovery + governed operations evidence | 沒有正式 ownership 時宣稱 hyperscale SRE / 24x7 ownership |
| AI Platform | public RAG / MCP / Gateway / evaluation implementation | 多年 production LLM platform / ML research experience |

---

# 18. P2.4 Completion Checklist

- [x] 建立四個 Target Role archetype
- [x] 建立 JD requirement classification
- [x] 建立 Evidence Level A / B / C / D
- [x] 建立 weighted JD Fit Score
- [x] 建立 Critical Gap Rule
- [x] 建立各職缺 Resume priority
- [x] 建立各職缺 GitHub review order
- [x] 建立各職缺 Interview Story priority
- [x] 建立 role-specific Gap / Risk matrix
- [x] 建立 Hybrid Role Mapping
- [x] 建立 Requirement → Evidence Cheatsheet
- [x] 建立 Gap Handling Template
- [x] 建立 Green / Yellow / Red 投遞判斷
- [x] 延續 P2.2 / P2.3 Claim Safety

---

# 19. P2 Portfolio Career Package

P2 現在形成完整求職 evidence chain：

```text
P2.1 — Interview Story
       專案怎麼講
          ↓
P2.2 — Resume ↔ GitHub Mapping
       經歷怎麼證明
          ↓
P2.3 — Architecture Decisions
       Senior-level 深挖怎麼回答
          ↓
P2.4 — Target Job Mapping
       不同 JD 該強調哪一組 evidence
```

下一步不需要再增加通用 Portfolio feature。

真正的使用方式是針對每一個實際 JD，複製第 15 節的 mapping table，產出該職缺專屬的：**Fit Score、Resume emphasis、GitHub order、Interview Stories 與 Gap Strategy**。