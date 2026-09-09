# Senior Data Engineer · Data Platform Engineer

**Enterprise Data Engineering · Data Platform · DataOps · Governed AI Integration**

我專注於企業級 **Data Engineering / Data Platform** 的設計、建置與維運，從 ETL / DWH、Workflow Orchestration、Data Integration，到 Observability、Security、CI/CD 與 Production Operations。

目前的技術發展方向，是在穩定的 Data Platform 基礎上整合 **RAG、AI Agent、MCP 與 Multi-LLM Gateway**，讓 AI 能力具備企業環境需要的治理、安全、稽核與可維運性。

> **My core is Data Engineering.**  
> AI / RAG / Agent / MCP capabilities in this portfolio are designed as governed extensions of an enterprise data platform rather than standalone AI experiments.

> **🌐 Portfolio Landing Page**  
> [Open the interactive portfolio](https://kewinall.github.io/kewinall/) — 五個核心專案、Portfolio Architecture、Live Project Guides 與面試官快速導覽集中於單一入口。

---

## Featured Portfolio

| Project | Portfolio Role | What it demonstrates | Release |
|---|---|---|---:|
| **[Enterprise ETL Platform](https://github.com/kewinall/enterprise-etl-platform)** · [Live Guide](https://kewinall.github.io/enterprise-etl-platform/) | **Enterprise Data Engineering Platform** | Airflow → Apache Hop ETL lifecycle、PostgreSQL Audit、Retry、Immutable Supply Chain、Air-Gapped Delivery、Prometheus/Grafana、SLO | `v0.5.0` |
| **[Data Platform MCP Server](https://github.com/kewinall/data-platform-mcp-server)** · [Live Guide](https://kewinall.github.io/data-platform-mcp-server/) | **Tool / Integration Platform** | PostgreSQL、Vertica、Airflow、Logs、Metadata、Lineage 的標準化 MCP Tool Layer，搭配 OIDC、RBAC、Multi-tenancy | `v0.4.0` |
| **[Agentic DataOps Copilot](https://github.com/kewinall/agentic-dataops-copilot)** · [Live Guide](https://kewinall.github.io/agentic-dataops-copilot/) | **AI Reasoning / DataOps Operations** | Incident Triage、Evidence Correlation、Root Cause Analysis、Policy、Human Approval、Tamper-evident Audit | `v0.5.0` |
| **[Enterprise RAG Platform](https://github.com/kewinall/enterprise-rag-platform)** · [Live Guide](https://kewinall.github.io/enterprise-rag-platform/) | **Knowledge AI Platform** | Ingestion、Hybrid Retrieval、Reranking、Grounded Generation、Citation、Evaluation、Knowledge Governance | `v0.6.0` |
| **[Multi-LLM AI Gateway](https://github.com/kewinall/multi-llm-ai-gateway)** · [Live Guide](https://kewinall.github.io/multi-llm-ai-gateway/) | **Model Control Plane** | OpenAI-compatible API、Multi-provider Routing、Fallback、Streaming、Policy、Quota / Cost、OIDC、Observability | `v0.5.0` |

每個核心專案皆提供 README、版本化 Release，以及適合面試官快速瀏覽的 **single-file interactive HTML project guide**。

> **🎯 Interview Story Playbook / P2.1**  
> [30 秒 / 2 分鐘 / 5 分鐘面試講法](INTERVIEW_STORIES.md) — 依 Data Engineering、Platform、DataOps、RAG / AI Integration 問題快速選擇案例，並以 Problem → Decision → Trade-off → Failure / Recovery → Evidence 組織回答。

> **🧭 Resume ↔ GitHub Mapping / P2.2**  
> [履歷能力 ↔ Portfolio Evidence 對照](RESUME_GITHUB_MAPPING.md) — 將真實工作成果、GitHub Evidence、Resume Bullet、職缺排序與 Claim Safety 串成同一條證據鏈，避免把 Portfolio implementation 過度宣稱成客戶 production 經驗。

---

## Portfolio Architecture

```text
                   Enterprise Data Sources
                           |
                           v
              +---------------------------+
              |  Enterprise ETL Platform  |
              | Airflow / Hop / Audit     |
              | Retry / Supply Chain / SLO|
              +-------------+-------------+
                            |
                            v
                  Enterprise Data Platform
                            |
              +-------------+-------------+
              |                           |
              v                           v
 +------------------------+    +------------------------+
 | Data Platform          |    | Enterprise RAG        |
 | MCP Server             |    | Platform              |
 |                        |    |                        |
 | Metadata / Lineage     |    | Enterprise Knowledge  |
 | SQL / Airflow / Logs   |    | Retrieval / Citation  |
 +-----------+------------+    +-----------+------------+
             |                             |
             v                             |
 +------------------------+                |
 | Agentic DataOps        |                |
 | Copilot                |                |
 |                        |                |
 | Triage / RCA / Policy  |                |
 | Human Approval / Audit |                |
 +-----------+------------+                |
             |                             |
             +-------------+---------------+
                           |
                           v
              +---------------------------+
              | Multi-LLM AI Gateway      |
              |                           |
              | Routing / Fallback        |
              | Policy / Cost / Identity  |
              +-------------+-------------+
                            |
              +-------------+-------------+
              |             |             |
           OpenAI       Anthropic       Gemini
                                         / Local
```

這五個 Repository 並不是五個互相重複的 AI Demo，而是刻意拆分成不同的 **enterprise platform responsibilities**。

---


## Engineering Decision Matrix

P1 將五個作品統一成 **Problem → Engineering Decision → Trade-off → Failure / Recovery → Evidence** 的面試證據鏈；每個 Live Guide 都有對應的 `Engineering Decisions & Production Evidence` 區塊。

| Project | Key Engineering Decision | Main Trade-off | Production Evidence |
|---|---|---|---|
| **[Enterprise ETL Platform](https://kewinall.github.io/enterprise-etl-platform/#engineering-decisions)** | Airflow orchestration + Hop processing + PostgreSQL execution truth + immutable promotion | 多 runtime 與 audit DB dependency | lifecycle / observability / supply-chain smoke |
| **[Data Platform MCP Server](https://kewinall.github.io/data-platform-mcp-server/#engineering-decisions)** | MCP Tool Contract + Adapter + RBAC/Tenant + read-only defense-in-depth | Protocol/schema maintenance、backend normalization | auth/audit、security、protocol、integration、Helm tests |
| **[Agentic DataOps Copilot](https://kewinall.github.io/agentic-dataops-copilot/#engineering-decisions)** | Reasoning 與 authority 分離；Policy → Approval → Explicit Executor → Audit | 非 full-auto remediation、approval latency | governance、multi-agent、RAG eval、MCP、hash-chained audit |
| **[Enterprise RAG Platform](https://kewinall.github.io/enterprise-rag-platform/#engineering-decisions)** | Hybrid Retrieval + Citation + Evaluation + Tenant Governance | Latency / tuning / eval dataset maintenance | tenancy、adversarial、retrieval/answer/agent eval、budget/rate |
| **[Multi-LLM AI Gateway](https://kewinall.github.io/multi-llm-ai-gateway/#engineering-decisions)** | Centralized model control plane + routing/fallback + policy/budget + Redis state | Gateway / Redis 成為新 production dependency | router、Redis integration、governance、identity、Helm validation |

這個矩陣的重點不是列技術名稱，而是展示：**為什麼這樣設計、犧牲了什麼、系統會在哪裡失敗，以及有什麼 evidence 可以驗證 claim。**

---

## Engineering Focus

| Area | Focus |
|---|---|
| **Data Engineering** | ETL / ELT、DWH、Batch Processing、Workflow Orchestration、Retry / Recovery |
| **Data Platform** | Metadata、Lineage、Data Integration、API / Tool Layer、Platform Governance |
| **Database** | Vertica、PostgreSQL、Oracle、SQL Optimization、Data Migration |
| **Platform Engineering** | Docker、Kubernetes / AKS / OKD、Linux、GitLab CI/CD、Environment Promotion |
| **Observability** | Prometheus、Grafana、OpenTelemetry、Audit Log、SLO、Alerting |
| **Security & Supply Chain** | RBAC、OIDC、Secret Management、Trivy、SBOM、Immutable Artifact、Air-Gapped Deployment |
| **AI Integration** | RAG、MCP、AI Agent、Multi-LLM Gateway、Model Routing、AI Governance |
| **Cloud** | AWS、Azure、Container / Data / Integration Services |

---

## Selected Engineering Experience

| Area | Experience |
|---|---|
| **Enterprise DWH Migration** | 700+ tables、約 8 TB 資料規模，完成資料平台遷移與效能優化，核心 workload 約達原環境 3× performance |
| **BI Modernization** | 完成 244 份既有 BI Report 的平台轉換與驗證 |
| **Operational Analytics** | 參與建立 149 份營運與決策分析 Report |
| **Production Data Platforms** | Vertica、Oracle、PostgreSQL、Apache Airflow、Apache Hop、Pentaho、DataStage、Denodo |
| **Platform Operations** | Linux、Docker、Kubernetes、Monitoring、Security Patch、Backup / Restore、CI/CD 與 Production Troubleshooting |

我重視的不只是「功能可以執行」，而是系統是否具備：

**Deployability → Reliability → Observability → Security → Auditability → Operability**

---

## Technology Stack

| Domain | Technologies |
|---|---|
| **Data Engineering** | Apache Airflow · Apache Hop · Pentaho · DataStage · Python · SQL |
| **Database / DWH** | Vertica · PostgreSQL · Oracle |
| **Data Virtualization** | Denodo |
| **Container Platform** | Docker · Kubernetes · AKS · OKD |
| **DevOps** | GitHub Actions · GitLab CI/CD · Container Registry |
| **Observability** | Prometheus · Grafana · OpenTelemetry · Zabbix |
| **Cloud** | AWS · Azure |
| **AI / Integration** | FastAPI · RAG · Qdrant · MCP · Multi-LLM Gateway |
| **Security** | OIDC · RBAC · Trivy · SBOM · Secret Management |

---

## Engineering Principles

```text
Production-oriented over demo-oriented
Explicit boundaries over monolithic architecture
Governance over unrestricted automation
Observability over black-box execution
Immutable promotion over environment rebuilds
Synthetic examples over exposed enterprise data
Human approval over uncontrolled mutation
Repeatable deployment over manual operations
```

所有公開作品均以 **synthetic / generic data、hostname、credential 與 configuration** 為原則，不包含客戶資料、公司內部資訊、真實帳號密碼或 private endpoint。

---

## Recommended Review Path

1. **[enterprise-etl-platform](https://github.com/kewinall/enterprise-etl-platform)** — 先看核心 Data Engineering 能力與 Production ETL lifecycle。
2. **[data-platform-mcp-server](https://github.com/kewinall/data-platform-mcp-server)** — 再看如何把 Data Platform 能力標準化成安全的 Integration Layer。
3. **[agentic-dataops-copilot](https://github.com/kewinall/agentic-dataops-copilot)** — 查看 AI 如何在治理框架內協助 DataOps。
4. **[enterprise-rag-platform](https://github.com/kewinall/enterprise-rag-platform)** — 查看企業知識如何進入可評測、可治理的 RAG Platform。
5. **[multi-llm-ai-gateway](https://github.com/kewinall/multi-llm-ai-gateway)** — 最後查看多個 AI workload 如何透過統一 Model Control Plane 使用不同 LLM Provider。

若只有幾分鐘時間，每個核心 Repository README 頂部皆提供 **Interactive Project Guide**，可直接查看 Architecture、Features、Security、CI/CD、Release evolution 與 Interviewer Quick Review。

---

## Certifications

**AWS Solutions Architect · PMP · Denodo C.D. Associate · ISO 9001**

---

## Current Direction

```text
Traditional Data Engineering
          +
Enterprise Platform Engineering
          +
Governed AI Integration
          =
Production-oriented Data Platform
```

我的目標不是把所有技術堆進同一個 Repository，而是展示如何從 **Data Engineering foundation** 出發，把資料處理、平台整合、維運治理、企業知識與 AI Runtime 拆成具有清楚責任邊界的系統。

---

### English Summary

Senior Data Engineer / Data Platform Engineer focused on enterprise data engineering, platform architecture, production operations, observability, security, and governed AI integration.

The portfolio demonstrates a layered enterprise architecture covering **ETL orchestration, data-platform integration, DataOps reasoning, enterprise RAG, and multi-LLM model governance**, with an emphasis on deployability, auditability, security, and operational readiness.
