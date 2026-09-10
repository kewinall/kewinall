# P2.1 — Interview Story Playbook

> 五個核心作品集的 30 秒 / 2 分鐘 / 5 分鐘面試講法  
> Positioning: **Senior Data Engineer · Data Platform Engineer · DataOps · Governed AI Integration**

## 使用原則

這份文件不是要背逐字稿，而是讓同一組作品可以依面試官問題快速切換深度。

- **30 秒**：回答「這個專案在做什麼？」或自我介紹時快速帶過。
- **2 分鐘**：回答「請挑一個專案說明」；必須包含 Problem → Decision → Trade-off → Evidence。
- **5 分鐘**：回答 Senior-level 深挖；加入 Failure / Recovery、替代方案與 production consideration。
- **Evidence 優先**：任何 claim 優先指向 repository 中可執行的 test、smoke script、CI、deployment 或 audit evidence。
- **不要把 portfolio project 說成客戶 production 上線經驗**：可說「我以 production-oriented 方式設計並建立可執行驗證」，真實工作成果則另外用履歷案例回答。

---

# 0. Portfolio Master Story

## 30 秒總覽

我的核心仍然是 **Data Engineering / Data Platform**。我把作品集拆成五個不同責任層：Enterprise ETL Platform 負責 Legacy ETL modernization、deterministic metadata / lineage、AI-assisted interpretation 與 ETL runtime lifecycle；Data Platform MCP Server 把資料平台能力標準化成安全的 tool layer；Agentic DataOps Copilot 展示 AI 如何在 policy、approval 與 audit 下協助維運；Enterprise RAG Platform 處理企業知識 retrieval、citation 與 evaluation；Multi-LLM AI Gateway 則集中管理 model routing、fallback、cost、identity 與 policy。Portfolio 的共同主軸是 **deterministic truth → governed AI → measurable evaluation → production evidence**，而不是堆 AI feature。

## 面試題 → 優先案例

| 面試官問題 | 第一案例 | 第二案例 |
|---|---|---|
| ETL / ELT、Airflow、資料管線、Retry | **Enterprise ETL Platform** | Agentic DataOps Copilot |
| Legacy ETL modernization、Pentaho → Hop | **Enterprise ETL Platform** | Data Platform MCP Server |
| Parser accuracy、AI hallucination、AI evaluation | **Enterprise ETL Platform** | Enterprise RAG Platform / Multi-LLM Gateway |
| Data Platform、Integration、Metadata / Lineage | **Data Platform MCP Server** | Enterprise ETL Platform |
| Production Operations、Incident、RCA | **Agentic DataOps Copilot** | Enterprise ETL Platform |
| RAG、Vector Search、Knowledge AI | **Enterprise RAG Platform** | Multi-LLM AI Gateway |
| LLM Platform、Model Routing、Cost / Quota | **Multi-LLM AI Gateway** | Enterprise RAG Platform |
| Security / Governance | MCP Server / DataOps Copilot | Multi-LLM Gateway |
| Observability / SLO | Enterprise ETL Platform | Multi-LLM Gateway |
| Kubernetes / Platform Engineering | MCP Server / Multi-LLM Gateway | Enterprise ETL Platform |

---

# 1. Enterprise ETL Platform

**Role:** Enterprise Data Engineering Platform  
**Release:** v0.8.0  
**Repository:** https://github.com/kewinall/enterprise-etl-platform

## 30 秒

這個專案現在不只回答「ETL 能不能穩定執行」，而是把 **Legacy ETL modernization → deterministic metadata / lineage → AI semantic interpretation → evaluation → runtime / delivery evidence** 串成一條完整鏈。Pentaho / Hop definition 先由 deterministic parser 抽出 step、source、target、SQL 與 dependency，structural truth 不交給 LLM；AI 只做 semantic enrichment，且輸出必須回到 parser evidence 驗證。Repo 另外保留 Airflow + Hop + PostgreSQL execution truth、immutable promotion、observability 與 air-gapped delivery。v0.8.0 有 10 組 synthetic evaluation cases、parser metrics、hallucination guard、AI failure fallback 與 CI report，所以我可以證明的不只是架構，而是 correctness、failure semantics 與 evidence discipline。

## 2 分鐘

這個專案有兩類 production 問題。第一類是 Legacy ETL modernization：如果直接把 Pentaho / Hop definition 丟給 LLM，要怎麼知道它沒有漏 step、看錯 source / target、改寫 SQL，或憑空產生 dependency？第二類是 runtime：即使 migration 正確，第一次 execution failure、retry、artifact promotion、monitoring failure 又要怎麼追蹤？

所以我先做一個很明確的 responsibility boundary：**deterministic parser owns structural truth，AI owns semantic interpretation。** Parser 把 step、source、target、dependency、SQL、table、parameter / variable 正規化成 metadata / lineage evidence；AI 只能基於這些 evidence 做摘要與解釋。若 AI 回傳 parser 沒有提供的 source、target、SQL 或 dependency，就會被 validation reject，而不是當成新事實。

接著 runtime 層仍維持 **Airflow orchestration + Apache Hop processing + PostgreSQL execution truth**。Retry 不覆蓋第一次 failure，monitoring 從 read-only audit evidence 匯出；delivery 則使用 build once / immutable promotion，air-gapped bundle 有 checksum / signature verification。

v0.8.0 再把 AI layer 做成可評測的工程問題。Repository 有 **10 組 synthetic / generic cases**，其中 9 組為有效 definition、1 組是預期 parse failure。checked-in regression baseline 在這個小型 synthetic corpus 上 structural exact-match 是 100%，unexpected parser failure 是 0，semantic structured validity / parser-truth preservation 也是 100%，unsupported structural claim 是 0。這些數字只代表 repository regression corpus，不宣稱 production accuracy。

成本部分也刻意不造數字。若沒有 live Multi-LLM Gateway 回傳 token / pricing evidence，cost 就保持 null；243 pipelines 的 projection 只有在實際 usage evidence 存在時才計算。這個 project 的核心價值是：**AI 可以幫忙理解 Legacy ETL，但 correctness、release gate 與 structural truth 仍由 deterministic evidence 控制。**

## 5 分鐘

### 1) Problem — Legacy modernization 不能把 correctness 外包給 LLM

企業 ETL modernization 不只是把 KTR / KJB 轉成另一種格式。我要先知道原本有哪些 steps、資料從哪裡來、寫到哪裡、SQL 是什麼、workflow dependency 是什麼，以及哪些 plugin 我其實無法可靠理解。這些都是 structural truth，不能因為模型回答流暢就當作正確。

同時，modernization 完成之後還要回答 runtime 問題：retry history 是否完整、TEST / PROD 是否真的是同一 artifact、監控掛掉時 execution evidence 是否還在、離線環境怎麼交付。

### 2) Decision — Deterministic parser first, AI second

第一個 decision 是 **先 parser，再 AI**。Parser 對 Pentaho / Hop definition 做 deterministic extraction，建立 normalized metadata / lineage，包括 Pipeline / Step、Source / Target、SQL / Table、Parameter / Variable、Workflow dependency 與 Unsupported component。

AI context 由這份 metadata 產生，而不是直接把 raw ETL artifact 當作唯一真相。AI 可以做 summary、semantic explanation 或 modernization assistance，但 source / target / SQL / dependency 等 structured claim 必須能回到 parser evidence。

這個 decision 的 trade-off 是 parser coverage 要自己維護，遇到 vendor-specific plugin 不能假裝理解；但換來的是 repeatability、auditability 與可做 regression test。

### 3) Decision — Runtime truth 與 delivery truth 也要 deterministic

第二個 decision 是 **Airflow orchestration + Hop processing + PostgreSQL execution truth**。Logical run 與 attempt 分開，因此第一次 FAILED 和後續 retry SUCCESS 都能保存。

第三個 decision 是 **build once, promote same immutable artifact**。TEST 驗證完成後不在 PROD rebuild；跨環境靠 image identity / checksum，air-gapped delivery 再加 bundle signature verification。

### 4) Evaluation — AI 必須可以量測，也必須允許「沒有數字」

v0.8.0 建立 synthetic evaluation corpus，涵蓋 extraction、join、lookup、filter、aggregation、SQL-heavy、multi-pipeline dependency、invalid / partial definition、unsupported component、complex parameters。

Parser evaluation 量 step / source / target / dependency / SQL / table extraction、precision / recall / F1 / exact-match、unsupported node detection 與 expected / unexpected failure rate。

Semantic evaluation 量 structured output validity、factual consistency、grounding、completeness、parser-truth preservation 與 unsupported structural claims。

我特別保留一個重要原則：**沒有 live Gateway evidence，就沒有假 token / cost benchmark。** Baseline 的 token / cost 是 null；只有 Multi-LLM Gateway 實際回傳 usage / pricing 時才做 per-pipeline、batch、provider/model 與 243-pipeline projection。

### 5) Failure / Recovery

- **AI Gateway unavailable**：Parser、metadata、lineage、migration validation 仍可使用；回 deterministic fallback。
- **AI structured output invalid**：做有限次 validation retry；仍不合法就 reject / fallback。
- **AI hallucinated source / target / SQL / dependency**：與 parser evidence 比對後 reject。
- **Provider timeout / rate limit**：provider routing / retry / fallback 留給 Multi-LLM Gateway，不在 ETL repo 重做。
- **Unsupported ETL plugin**：標記 unsupported / manual review，不讓 AI 自動宣稱可 migration。
- **Hop execution fail**：Airflow retry，但 PostgreSQL 保存每個 attempt。
- **Artifact identity mismatch**：promotion fail closed。

### 6) Evidence / Close

除了原有 etl_lifecycle_smoke.sh、observability_smoke.sh、supply_chain_smoke.sh，現在還有 evaluation/dataset.json、etl_intelligence/evaluation.py、tests/test_p2_evaluation.py、reports/baseline/、docs/ETL_AI_EVALUATION.md 與 GitHub Actions p2-etl-ai-evaluation artifact。

所以我在面試時不會說「AI 可以自動幫我理解所有 ETL」。我會說：**我先定義哪些 structural facts 必須 deterministic，再把 AI 限制在 advisory / semantic layer，最後用 synthetic ground truth、failure semantics、usage evidence 與 CI regression 去驗證它。**

這個故事同時可以回答 Data Engineering、Legacy Modernization、Data Platform、AI Integration 與 LLM evaluation，而核心仍然是 Data Engineering correctness。

### 可延伸追問

- Parser 準確率怎麼定義與驗證？
- 為什麼 structural truth 不直接交給 LLM？
- Raw ETL → LLM 與 Parser → LLM 怎麼比較？
- AI hallucination 如何在 application layer 擋掉？
- AI / Gateway 掛掉時 ETL modernization 是否還能工作？
- 243 pipelines 的 token / cost 要怎麼估，而且如何避免假 benchmark？
- Unsupported Pentaho / Hop plugin 要怎麼處理？
- 100% synthetic regression 為什麼不能宣稱 production accuracy？
- Exactly-once 做得到嗎？哪些地方其實只能做到 idempotent / at-least-once？
- Air-gapped promotion 如何防止 artifact drift？

---
# 2. Data Platform MCP Server

**Role:** Tool / Integration Platform  
**Repository:** https://github.com/kewinall/data-platform-mcp-server

## 30 秒

這個專案的核心問題是：**AI Agent 不應該直接拿 PostgreSQL、Vertica、Airflow 或 Log backend 的原生權限。** 我用 MCP Tool Contract 做統一 integration boundary，再用 Adapter 隔離 backend 差異，並把 OIDC、RBAC、tenant policy、SQL read-only defense-in-depth 和 audit 放在 server side。這樣 AI client 只取得被治理的 capability，而不是 backend credential。Repo 裡有 protocol、auth、security、Vertica、integration、observability 與 Helm regression tests。

## 2 分鐘

我把這個專案定位成 Data Platform 的 **integration boundary**。如果每個 AI Agent 都直接連資料庫或 Airflow，authentication、authorization、tenant isolation、SQL safety、audit，以及每個 backend 的 API 差異都會被複製到 client，而且很容易發生 privilege expansion。

所以我採用 **MCP Tool Contract + Adapter Pattern**。對 client 來說只有像 list schemas、describe table、lineage、EXPLAIN、DAG status、log search 這些標準 capability；PostgreSQL、Vertica、Airflow、OpenSearch/Loki 的差異則封裝在 adapter。

安全上我沒有把「read-only」交給 prompt。第一層是 tool surface 本身不提供任意 write capability；第二層用 SQLGlot AST 做 SQL policy；第三層 database session 仍設 read-only。Identity 則用 OIDC/JWT + JWKS，RBAC 回答「可以做什麼」，tenant policy 回答「可以碰哪個 source」。

這樣做的 trade-off 是多了一層 protocol / schema maintenance，backend error 也要 normalization，而且目前 tenant isolation 是 source-level，不等於 row-level RLS。我會明確說這是 scope boundary，而不是過度宣稱。

Evidence 包含 `test_mcp_protocol.py`、`test_auth_audit.py`、`test_security.py`、`test_vertica.py`、`test_integrations.py` 和 Helm chart validation。這個案例適合回答 Data Platform、AI Integration、security boundary 與 platform API design。

## 5 分鐘

### 1) Problem

「AI Agent 直接拿 database credential」看似最快，但會讓每個 client 都重新實作 auth、SQL safety、tenant policy、audit，而且 backend permission 通常比 Agent 真正需要的 capability 更大。

### 2) Decision

我把 **MCP Tool Contract** 當成 canonical capability boundary。client 不需要知道 Vertica 跟 PostgreSQL metadata API 差異，也不直接處理 Airflow v2 API 或 log backend query semantics。

backend 使用 Adapter Pattern，讓 tool contract 與 implementation 解耦。新增 source 時，理想狀態是增加 adapter，而不是改所有 Agent。

Security 上是 defense-in-depth：

1. Tool surface 只暴露允許 capability。
2. RBAC / scope 控制 operation。
3. Tenant policy 控制可見 source。
4. SQLGlot AST 拒絕 write / unsafe SQL。
5. DB connection 本身仍是 read-only。
6. Audit 記錄 invocation，敏感 SQL 可記 fingerprint 而非 raw text。

### 3) Trade-off

MCP server 本身變成一個需要維護 compatibility、availability 與 schema evolution 的平台元件。另外 source-level tenant isolation 不能取代 database row-level security。如果需求進一步要求 table/row 級隔離，我會把 policy 下推到 DB RLS、view 或 query rewriting layer，而不是宣稱現有 layer 已經解決全部問題。

### 4) Failure / Recovery

- Invalid token / JWKS failure：**fail closed**，不能 fallback 成 privileged anonymous。
- Cross-tenant source request：tenant policy reject。
- Malformed / write SQL：執行 DB 前 reject，DB session 仍是第二層保護。
- 某 backend outage：只讓該 adapter capability degraded，不因 outage 放寬權限或換高權限 connection。

### 5) Evidence / Close

Repo 裡 protocol、auth、security、Vertica/integration、OTel/audit 都有 regression tests，也有 Kubernetes Helm、NetworkPolicy、External Secrets 與 air-gapped delivery baseline。這代表我不是把 MCP 當新名詞，而是把它當 **enterprise integration contract**。

### 可延伸追問

- MCP 相較 REST API 的價值與成本？
- Tool schema 如何 versioning？
- 怎麼避免 prompt injection 透過 tool 呼叫變成 SQL injection？
- Tenant source isolation 如何提升到 row-level？
- OIDC/JWKS outage 是否需要 cache？cache 的安全邊界怎麼設？

---

# 3. Agentic DataOps Copilot

**Role:** AI Reasoning / DataOps Operations  
**Repository:** https://github.com/kewinall/agentic-dataops-copilot

## 30 秒

這個專案不是做「會自動修 production 的 AI Agent」，而是刻意解決 **reasoning 與 mutation authority 必須分離**。Agent 可以收集 evidence、做 triage、RCA、提出 action plan，但真正執行前仍要經 deterministic policy、risk gate、human approval 和 explicit executor，所有結果進 hash-chained audit。這讓 AI 可以提升 DataOps 調查效率，又不把 hallucination 直接轉成 production change。

## 2 分鐘

DataOps Agent 最大的風險不是 LLM 回答錯一句，而是它的回答如果直接取得 production write authority，hallucination、prompt injection 或權限誤配就可能變成真實事故。

所以我的第一個設計決策是 **reasoning 與 execution authority 分離**。Agent 可以做 evidence correlation、incident triage、RCA 和 remediation proposal，但 proposal 不是 command。

第二個決策是把 **deterministic Policy Engine 放在 LLM reasoning 之後**。模型先提出 plan，再由 policy 根據 role、risk、allow-list 判斷。高風險 action 需要 human approval，而且採 separation of duties，不能自己 proposal、自己 approve。

第三個特別保守的設計是 **mutation executor registry 預設為空**。也就是 reasoning demo 可以完整跑，但如果沒有明確註冊、測試與治理 executor，就不可能產生真實 mutation。這是我刻意做的安全 default。

代價是它不是 full-auto remediation，會多 approval latency，也要維護 policy。但對企業 DataOps，我認為「可控 automation」比「完全 autonomous」更合理。

Evidence 有 governance regression test、multi-agent orchestration test、MCP integration test、RAG evidence evaluation，以及 SHA-256 hash-chained audit。這個案例最適合回答 AI Agent 如何安全落地到 production operations。

## 5 分鐘

### 1) Problem

我先把問題定義成 authority，而不是 intelligence：LLM 再聰明也不是 production authorization system。DataOps 流程應該把 evidence gathering、reasoning、policy、approval、execution、audit 分成不同責任。

### 2) Decision

整體流程是：

**Evidence → Triage / RCA → Action Plan → Deterministic Policy → Human Approval → Explicit Executor → Audit**

Reasoning 可以是 probabilistic，但 authorization 必須 deterministic。這也代表「模型說可以」永遠不是 execution condition。

Separation of duties 讓 requester / agent 不可以 self-approve 高風險 action。Executor 則是明確 registry，預設沒有 production mutation capability。

### 3) Trade-off

Human approval 一定會增加 latency，但不是所有 action 都需要同等流程。實務上我會依 risk tier 做不同 gate：read-only investigation 可以自動；low-risk bounded action 可考慮 policy auto-approve；high-risk / destructive action 才需要 human approval。這樣不是放棄 automation，而是把 automation 跟 authority 分級。

### 4) Failure / Recovery

- Hallucinated action plan：policy / risk gate 擋住，回到 reasoning。
- Policy service 無法 allow：fail closed，不 bypass。
- Approval timeout / reject：維持 pending / rejected，保留 audit。
- Executor failure：紀錄 failure outcome，再由 operator retry / rollback / manual remediation。
- MCP / evidence source outage：RCA confidence 降低，不能用模型猜測填補 evidence。

### 5) Audit Evidence

我用 SHA-256 hash-chained audit 讓 policy、approval、dry-run、execution outcome 形成 tamper-evident sequence。我要強調它不是完整 SIEM 或 immutable WORM storage 的替代品；正式環境仍需要 durable retention、access control 與 external log sink。但 repository 能夠證明事件鏈被修改時 hash validation 會失效。

所以這個專案的 Senior-level 重點是：**AI 可以提升 reasoning throughput，但 production authority 應留在 deterministic governance layer。**

### 可延伸追問

- Human-in-the-loop 如何避免成為 bottleneck？
- 哪些 action 可以 auto-remediate？
- Policy Engine rule 如何 versioning / testing？
- Audit hash chain 與真正 immutable audit storage 差異？
- Agent hallucination、prompt injection、tool poisoning 怎麼處理？

---

# 4. Enterprise RAG Platform

**Role:** Knowledge AI Platform  
**Repository:** https://github.com/kewinall/enterprise-rag-platform

## 30 秒

這個專案把 RAG 從「Vector DB Demo」提升成可評測、可治理的 Knowledge AI Platform。我採 **Vector + BM25 + RRF hybrid retrieval**，可選 reranker，再要求 grounded answer、citation、retrieval / answer / agent evaluation，同時把 tenant isolation、rate / budget guardrail、prompt/tool injection test 納入。重點是：RAG 品質不能靠感覺，要能量化，而且 security boundary 不能因為想提高 recall 就拿掉。

## 2 分鐘

我做 RAG 時最先避免一個常見陷阱：只要 Vector Search 找到幾段文字，模型回答看起來合理，就當作成功。但企業 RAG 真正要回答的是 retrieval recall、answer grounding、citation correctness、tenant isolation、adversarial behavior，以及品質是否可量化。

Retrieval 我採 **Vector + BM25，再用 RRF fusion**。Vector 擅長 semantic similarity，但 exact entity、代碼、專有名詞不一定穩；BM25 則補 exact keyword。必要時再加 reranker，換取更好的 ordering，但我會明確承認它增加 latency 和 compute cost。

Generation 端要求 grounded answer 與 citation，並建立 retrieval、answer、agent evaluation dataset。Citation 的價值不是讓畫面好看，而是當 evidence 不夠時能暴露問題。

另一個重點是 multi-tenancy。tenant filter 可能降低 recall，但不能為了「回答比較多」就關掉 security boundary。Budget、rate limit、tool allow-list、destructive action approval 也都是 runtime governance。

Repo 有 tenancy、roles、filters、adversarial security、retrieval/answer/agent eval、budget/rate-limit、agent approval tests。這讓我可以把 RAG 從模型功能題，轉成 Data / Platform / Security 的工程題來回答。

## 5 分鐘

### 1) Problem

Pure vector demo 最大問題是「結果看起來對」但無法證明。企業場景需要知道 query 是否取到正確 evidence、answer 是否只使用 evidence、citation 是否可追溯、不同 tenant 是否真的隔離，以及 prompt injection 是否能誘導 tool 做未授權行為。

### 2) Retrieval Decision

我採 Hybrid Retrieval：

**Vector retrieval + BM25 → RRF fusion → optional reranker**

Vector 處理語意，BM25 處理 exact term，RRF 讓兩種排名不必直接比較 raw score。Reranker 再針對 candidate set 做 relevance refinement。

Trade-off 是 latency、index maintenance 和 tuning 都比 pure vector 高。所以 reranker 應該依 use case、quality gain 和 latency budget決定，不是「有模型就一定加」。

### 3) Grounding / Evaluation

Grounded generation 需要 citation，但 citation 本身不代表正確。我會分至少三層 evaluation：

- Retrieval：expected evidence 是否進 candidate set。
- Answer：答案是否正確、grounded、supported。
- Agent / Tool：在有 tool 的情境是否遵守 policy。

再加入 adversarial test，把 prompt injection / tool injection 變成 regression case，而不是只在 system prompt 加一句「不要被騙」。

### 4) Multi-tenancy / Runtime Governance

Tenant scope 應存在 retrieval、state、cache key、budget / rate 等層級。這一定增加 complexity，也可能因 filter 降低 recall，但 security boundary 不應為品質指標讓路。真正要改善 recall，應該從 ingestion、chunking、indexing、query expansion 或 tenant-specific corpus 著手。

### 5) Failure / Recovery

- Evidence insufficient：明確回覆不足，不讓 fluency 冒充 correctness。
- Cross-tenant request：reject / isolate，不 disable filter。
- Vector backend outage：degraded 或 fail，不能繞過 security。
- Budget exceeded：provider call 前 gate。
- Adversarial regression fail：修 policy / prompt / tool boundary，再重新通過 test。

這個案例的核心訊息是：**RAG 是 Information Retrieval + Evaluation + Governance 的工程系統，不只是把 LLM 接上 Vector DB。**

### 可延伸追問

- Chunk size / overlap 如何調？
- Hybrid Retrieval 為何選 RRF？
- Reranker 什麼情況不值得？
- 如何定義 groundedness / faithfulness？
- Evaluation dataset 如何避免 overfitting？
- Tenant filter 與 cache / vector index 怎麼設計？

---

# 5. Multi-LLM AI Gateway

**Role:** Model Control Plane  
**Repository:** https://github.com/kewinall/multi-llm-ai-gateway

## 30 秒

這個專案把 OpenAI、Anthropic、Gemini 等 provider 的 routing、fallback、rate limit、budget、cost、identity、policy、audit 和 observability 集中成 **Model Control Plane**。上游使用 OpenAI-compatible contract，provider failure 由 retry / fallback / circuit breaker 處理，multi-replica governance state 放 Redis。代價是 Gateway 和 Redis 變成新的 production dependency，所以我也做 Kubernetes HA/hardening、identity、governance 和 Redis integration tests。

## 2 分鐘

如果每個 RAG / Agent application 都自己整合 OpenAI、Anthropic、Gemini，model routing、API key、retry、cost、rate limit、policy、audit 都會重複實作，而且 provider outage 或 cost spike 很難集中治理。

所以我做了一個 **centralized Model Control Plane**。上游透過 OpenAI-compatible API 呼叫，Gateway 根據 alias、pool、routing policy 選 provider，並支援 retry、fallback、circuit breaker、streaming、usage / cost accounting。

Policy、budget 和 rate limit 都放在 provider call 之前，目的是 request 不合規或超預算時，在真正產生成本前就 reject。身份可用 API key，也支援 OIDC/JWT + JWKS，並做 RBAC / client policy。

為了支援多 replica，rate limit、round-robin cursor、circuit state、usage / budget 需要 shared state，所以放 Redis。這也是最大的 trade-off：Gateway 和 Redis 都變成 critical production dependency。我的設計不是忽略這點，而是透過 readiness、replicas、PDB、hardened Helm baseline，以及 Redis integration tests 明確處理。

這個案例適合回答 AI Platform / Platform Engineering：如何降低 application 與 provider coupling，同時集中治理 cost、resilience 與 security。

## 5 分鐘

### 1) Problem

沒有 Gateway 時，每個 application 都會有一份 provider SDK integration、一份 API key、一套 retry / fallback、一套 cost accounting。時間久了會變成 governance fragmentation。

### 2) Control Plane Decision

我把 common concern 集中到 Gateway：

**Identity → Policy → Rate / Budget → Routing → Provider → Usage / Cost / Audit / Observability**

上游用 OpenAI-compatible contract，降低 provider coupling。Provider-specific capability不是完全免費抽象；例如某些 native streaming 或 advanced feature 可能需要 normalization，甚至無法 1:1 暴露，這是 abstraction trade-off。

### 3) Resilience

Router 支援 priority / round-robin / random / cost-aware，搭配 retry、fallback、circuit breaker。遇到 timeout 或 429，不讓單一 provider failure 直接擴散。

但 fallback 不代表「永遠成功」。不同 provider 可能有 model capability、latency、price、context window 和 safety behavior 差異，所以 routing policy 要同時考慮 quality、latency、cost 與 compliance。

### 4) Distributed State / Failure Semantics

多 replica 下，如果每個 pod 自己算 rate limit / budget，就會失去一致性，所以 Redis 保存 shared governance state。也因此 Redis outage 不是普通 cache miss：**rate、budget、circuit 等 cross-replica guarantee 都可能不可信。** 我的原則是把它視為 control dependency incident，而不是 silently fallback 到完全無限制模式。

Gateway pod failure 則由 readiness、multiple replicas、PDB 與 Kubernetes Service 降低影響。

### 5) Evidence / Close

Repo 有 router regression、Redis integration、governance / budget / policy、OIDC identity、Helm hardening、Prometheus / OTel evidence。這個專案展示的不是「支援很多 LLM」，而是 **如何把模型使用變成可治理的平台能力**。

### 可延伸追問

- Gateway 會不會成為 bottleneck / SPOF？
- Redis 要 strong consistency 嗎？
- Cost-aware routing 是否會犧牲 quality？
- Provider fallback 怎麼處理不同 model capability？
- SSE partial response 如何 retry？
- OpenAI-compatible abstraction 會犧牲什麼？

---

# 6. 面試現場選擇策略

## Data Engineer / Senior Data Engineer

建議順序：

1. **Enterprise ETL Platform**
2. Data Platform MCP Server
3. Agentic DataOps Copilot
4. Enterprise RAG Platform
5. Multi-LLM AI Gateway

主軸句：

> 我的核心是 Data Engineering。我先證明 pipeline lifecycle、retry、audit、immutable promotion 和 observability，再延伸到 Data Platform integration 與 governed AI。

## Data Platform Engineer / Platform Engineer

建議順序：

1. **Data Platform MCP Server**
2. Enterprise ETL Platform
3. Multi-LLM AI Gateway
4. Agentic DataOps Copilot
5. Enterprise RAG Platform

主軸句：

> 我習慣先定義 responsibility boundary、identity、policy、observability 和 failure semantics，再決定具體 framework。

## DataOps / MLOps / AI Platform

建議順序：

1. **Agentic DataOps Copilot**
2. Multi-LLM AI Gateway
3. Data Platform MCP Server
4. Enterprise ETL Platform
5. Enterprise RAG Platform

主軸句：

> AI reasoning 可以是 probabilistic，但 production authority、policy、identity、audit 與 operational recovery 必須 deterministic。

## RAG / AI Integration

建議順序：

1. **Enterprise RAG Platform**
2. Multi-LLM AI Gateway
3. Data Platform MCP Server
4. Agentic DataOps Copilot
5. Enterprise ETL Platform

主軸句：

> 我不把 RAG 或 Agent 當獨立 demo，而是把 retrieval、evaluation、identity、tool boundary、cost 與 operations 放進同一個 enterprise governance context。

---

# 7. Senior-level 回答公式

當面試官臨時問一個沒有準備過的深度題時，可固定用：

```text
1. Problem
   這個問題真正的 production risk 是什麼？

2. Decision
   我選了哪個 architecture / boundary？

3. Why
   為什麼不用更簡單或更常見的方案？

4. Trade-off
   我因此多付出了什麼成本？

5. Failure
   哪個 dependency 掛掉時 guarantee 會失效？

6. Recovery
   系統要 fail open、fail closed、retry、rollback 還是 degrade？

7. Evidence
   Repo 裡哪個 test / smoke / CI / audit 可以證明？
```

Senior-level 的差異通常不在「知道更多工具名稱」，而在能否清楚回答 **boundary、trade-off、failure semantics、recovery 與 evidence**。

---

# 8. P2.1 完成條件

- [x] 五專案各有 30 秒版本
- [x] 五專案各有 2 分鐘版本
- [x] 五專案各有 5 分鐘版本
- [x] 每個案例都包含 Problem / Decision / Trade-off / Failure / Evidence
- [x] 建立面試題與專案選擇 mapping
- [x] 建立 Data Engineering / Platform / DataOps / AI Integration 四種面試切換順序
- [x] 明確區分 portfolio evidence 與真實 production work experience

**Next:** P2.2 — Resume ↔ GitHub Mapping
