# Senior Data Engineer · Data Platform Engineer

**Enterprise Data Engineering · Data Platform · DataOps · Governed AI Integration**

我專注於企業級 **Data Engineering / Data Platform** 的設計、建置與維運，涵蓋 ETL / DWH、Workflow Orchestration、Data Integration、Observability、Security、CI/CD 與 Production Operations。

目前的技術發展方向，是在穩定的 Data Platform 基礎上整合 **RAG、AI Agent、MCP 與 Multi-LLM Gateway**，讓 AI 能力具備企業環境需要的治理、安全、稽核與可維運性。

> **核心仍是 Data Engineering。**  
> 本 Portfolio 中的 AI / RAG / Agent / MCP 能力，定位為 Enterprise Data Platform 上的 governed extension，而不是彼此獨立的 AI demo。

> **🌐 Portfolio Landing Page**  
> [開啟互動式作品集](https://kewinall.github.io/kewinall/) — 五個核心專案、Portfolio Architecture 與面試官快速導覽集中於單一入口。

---

## 核心 Portfolio

| 專案 | Portfolio 定位 | 主要展示內容 | 版本 |
|---|---|---|---:|
| **[Enterprise ETL Platform](https://github.com/kewinall/enterprise-etl-platform)** · [Live Guide](https://kewinall.github.io/enterprise-etl-platform/) | **Enterprise Data Engineering Platform** | Legacy Pentaho / Hop modernization、deterministic metadata / lineage、ETL AI evaluation、Airflow → Hop runtime、PostgreSQL Audit / Retry、Immutable Supply Chain、Air-Gapped Delivery、Prometheus / Grafana SLO | `v0.8.0` |
| **[Data Platform MCP Server](https://github.com/kewinall/data-platform-mcp-server)** · [Live Guide](https://kewinall.github.io/data-platform-mcp-server/) | **Tool / Integration Platform** | PostgreSQL、Vertica、Airflow、Logs、Metadata、Lineage 的標準化 MCP Tool Layer，搭配 OIDC、RBAC、Multi-tenancy | `v0.5.0` |
| **[Agentic DataOps Copilot](https://github.com/kewinall/agentic-dataops-copilot)** · [Live Guide](https://kewinall.github.io/agentic-dataops-copilot/) | **AI Reasoning / DataOps Operations** | Incident Triage、Evidence Correlation、Root Cause Analysis、Policy、Human Approval、Tamper-evident Audit | `v0.5.0` |
| **[Enterprise RAG Platform](https://github.com/kewinall/enterprise-rag-platform)** · [Live Guide](https://kewinall.github.io/enterprise-rag-platform/) | **Knowledge AI Platform** | Ingestion、Hybrid Retrieval、Reranking、Grounded Generation、Citation、Evaluation、Knowledge Governance | `v0.6.0` |
| **[Multi-LLM AI Gateway](https://github.com/kewinall/multi-llm-ai-gateway)** · [Live Guide](https://kewinall.github.io/multi-llm-ai-gateway/) | **Model Control Plane** | OpenAI-compatible API、Multi-provider Routing、Fallback、Streaming、Policy、Quota / Cost、OIDC、Observability | `v0.5.0` |

每個核心專案皆提供工程師導向的 README、版本化 Release，以及面試官可快速理解專案價值的 **single-file interactive HTML GitHub Pages**。

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

這五個 Repository 並不是五個互相重複的 AI demo，而是刻意拆分成不同的 **enterprise platform responsibilities**。

---

## 工程決策矩陣

五個作品統一以 **Problem → Engineering Decision → Trade-off → Failure / Recovery → Evidence** 的方式呈現工程判斷與可驗證依據。

| 專案 | 關鍵工程決策 | 主要 Trade-off | 可驗證 Evidence |
|---|---|---|---|
| **[Enterprise ETL Platform](https://kewinall.github.io/enterprise-etl-platform/#engineering-decisions)** | Deterministic parser 掌握 ETL structural truth；AI 僅作 evidence-grounded advisory；runtime 採 Airflow + Hop + PostgreSQL execution truth + immutable promotion | Parser / plugin coverage、evaluation corpus maintenance、multi-runtime dependency | migration、parser metrics、AI guardrail、lifecycle、observability、supply-chain evidence |
| **[Data Platform MCP Server](https://kewinall.github.io/data-platform-mcp-server/#engineering-decisions)** | MCP Tool Contract + Adapter + RBAC / Tenant + read-only defense-in-depth | Protocol / schema maintenance、backend normalization | auth / audit、security、protocol、integration、Helm tests |
| **[Agentic DataOps Copilot](https://kewinall.github.io/agentic-dataops-copilot/#engineering-decisions)** | Reasoning 與 authority 分離；Policy → Approval → Explicit Executor → Audit | 非 full-auto remediation、approval latency | governance、multi-agent、RAG evaluation、MCP、hash-chained audit |
| **[Enterprise RAG Platform](https://kewinall.github.io/enterprise-rag-platform/#engineering-decisions)** | Hybrid Retrieval + Citation + Evaluation + Tenant Governance | Latency / tuning / evaluation dataset maintenance | tenancy、adversarial、retrieval / answer / agent evaluation、budget / rate |
| **[Multi-LLM AI Gateway](https://kewinall.github.io/multi-llm-ai-gateway/#engineering-decisions)** | Centralized Model Control Plane + routing / fallback + policy / budget + Redis state | Gateway / Redis 成為 production dependency | router、Redis integration、governance、identity、Helm validation |

重點不是技術名稱本身，而是說明：**為什麼這樣設計、犧牲什麼、失敗時怎麼處理，以及有什麼 Evidence 可以驗證。**

---

## Portfolio Evidence Story

五個 Repository 的共同主軸不是「AI feature 越多越好」，而是把不同類型的 truth 與 authority 放在正確的位置：

```text
Deterministic Data / Metadata Truth
            |
            v
Governed Integration / AI Assistance
            |
            v
Evaluation / Grounding / Failure Semantics
            |
            v
Runtime / Audit / Observability
            |
            v
Immutable / Controlled Delivery
```

`enterprise-etl-platform` v0.8.0 是目前最完整的示例：Legacy ETL 先經 deterministic parser 形成 normalized metadata / lineage，再由 AI 做 semantic interpretation；AI 結果需回到 parser evidence 驗證，並以 synthetic ground truth、CI regression、failure fallback 與 Gateway usage contract 證明可控性。沒有 live token / pricing evidence 時，成本欄位維持 `null`，不把估算包裝成 production benchmark。

---

## 工程能力重點

| 領域 | 技術重點 |
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

## 代表性工程經驗

| 領域 | 經驗 |
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

| 領域 | Technologies |
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

## 工程原則

```text
Production-oriented > demo-oriented
Explicit boundaries > monolithic architecture
Governance > unrestricted automation
Observability > black-box execution
Immutable promotion > environment rebuilds
Synthetic examples > exposed enterprise data
Human approval > uncontrolled mutation
Repeatable deployment > manual operations
```

所有公開作品均以 **synthetic / generic data、hostname、credential 與 configuration** 為原則，不包含客戶資料、公司內部資訊、真實帳號密碼或 private endpoint。

---

## 建議瀏覽順序

1. **[enterprise-etl-platform](https://github.com/kewinall/enterprise-etl-platform)** — 核心 Data Engineering 能力與 Production ETL lifecycle。
2. **[data-platform-mcp-server](https://github.com/kewinall/data-platform-mcp-server)** — Data Platform capability 如何標準化成安全 Integration Layer。
3. **[agentic-dataops-copilot](https://github.com/kewinall/agentic-dataops-copilot)** — AI 如何在治理框架內協助 DataOps。
4. **[enterprise-rag-platform](https://github.com/kewinall/enterprise-rag-platform)** — 企業知識如何進入可評測、可治理的 RAG Platform。
5. **[multi-llm-ai-gateway](https://github.com/kewinall/multi-llm-ai-gateway)** — 多個 AI workload 如何透過統一 Model Control Plane 使用不同 LLM Provider。

若只有幾分鐘時間，每個核心 Repository README 頂部都有 **GitHub Pages 互動式說明**，可快速查看架構、核心能力、工程決策、Evidence 與責任邊界。

---

## Certifications

**AWS Solutions Architect · PMP · Denodo C.D. Associate · ISO 9001**

---

## 目前發展方向

```text
Traditional Data Engineering
          +
Enterprise Platform Engineering
          +
Governed AI Integration
          =
Production-oriented Data Platform
```

目標不是把所有技術堆進同一個 Repository，而是從 **Data Engineering foundation** 出發，把資料處理、平台整合、維運治理、企業知識與 AI Runtime 拆成具有清楚責任邊界、可驗證 Evidence 與 production-oriented failure semantics 的系統。
