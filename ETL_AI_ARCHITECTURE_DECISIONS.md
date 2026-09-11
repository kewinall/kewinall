# P2.3 Extension — ETL AI Architecture Decision Stories

> 延伸既有 12 個 Senior-level Architecture Decision Stories，補上 `enterprise-etl-platform v0.8.0` 的 ETL AI / Evaluation / Evidence 深度追問。  
> Positioning: **Senior Data Engineer · Data Platform Engineer · Governed AI Integration**

---

## 使用方式

這份文件不是把 ETL 專案重新包裝成 AI 專案，而是回答一個更成熟的問題：

> **當 Legacy ETL modernization 引入 AI 時，哪些 truth 必須 deterministic、哪些工作適合 AI、怎麼驗證、失敗時哪些 guarantee 還成立？**

建議與主文件 [ARCHITECTURE_DECISION_STORIES.md](ARCHITECTURE_DECISION_STORIES.md) 一起使用。

```text
Legacy ETL
   ↓
Deterministic Parser
   ↓
Normalized Metadata / Lineage
   ↓
AI Semantic Layer
   ↓
Evaluation / Guardrail
   ↓
Runtime / Audit / Delivery Evidence
```

---

# Decision Index — ETL AI Extension

| # | Architecture Decision | Primary Repo | 主題 |
|---:|---|---|---|
| 13 | 為什麼 structural truth 要先用 deterministic parser，而不是 Raw ETL → LLM？ | Enterprise ETL Platform | AI / Data Engineering Boundary |
| 14 | Parser correctness 要怎麼量測？為什麼 synthetic ground truth 有價值但不等於 production accuracy？ | Enterprise ETL Platform | Evaluation |
| 15 | Hallucination Guard 能保證什麼？又不能保證什麼？ | Enterprise ETL Platform | AI Reliability |
| 16 | AI / Gateway 掛掉時，ETL modernization 應該怎麼 degraded？ | Enterprise ETL Platform + Multi-LLM AI Gateway | Failure Semantics |
| 17 | 243 pipelines 的 token / cost / latency 要怎麼估，為什麼沒有 evidence 時寧可留 null？ | Enterprise ETL Platform + Multi-LLM AI Gateway | Cost / Capacity |
| 18 | Synthetic evaluation 要如何走向 production evaluation，而不洩漏客戶資料？ | Enterprise ETL Platform | Production Evidence |

---

# 13. 為什麼 Structural Truth 要先用 Deterministic Parser，而不是 Raw ETL → LLM？

**Primary Repo:** [Enterprise ETL Platform](https://github.com/kewinall/enterprise-etl-platform)

## 面試官真正想測什麼

- 是否知道 LLM 適合 semantic reasoning，但不是可靠 parser 的替代品
- 能不能區分 structural truth 與 semantic interpretation
- 是否理解 repeatability / auditability / migration correctness
- 是否能解釋為什麼不是所有事情都丟給模型

## Decision

> **Step、Source、Target、SQL、Table、Parameter、Variable、Workflow Dependency 等 structural facts 由 deterministic parser 負責；AI 只在 parser evidence 上做 semantic enrichment。**

```text
Pentaho / Hop Artifact
        ↓
Deterministic Parser
        ↓
Normalized Metadata
        ↓
Evidence-bound AI Context
        ↓
Semantic Summary / Interpretation
```

## Why

如果直接做 Raw ETL → LLM：

- 相同輸入不一定每次得到相同結構
- 模型可能漏 step 或誤解 plugin
- source / target / SQL 可能被改寫或補齊
- 很難對單一 structural field 做精確 regression
- migration correctness 變成 probabilistic

而 deterministic parser 可以：

- 相同 artifact 產出相同 metadata
- 對 field-level output 建 ground truth
- 明確標示 unsupported component
- 讓後續 migration validator / lineage / AI 共用同一 structural truth

## Trade-off

Parser-first 的代價是：

- 要維護 Pentaho / Hop schema / plugin coverage
- vendor-specific component 需要額外 parser
- parser 不懂的 semantic business meaning 仍需人工或 AI 補強

也就是把複雜度從「模型不確定性」換成「明確可治理的 parser coverage」。

## Failure / Recovery

### 遇到未知 Plugin

不要讓 AI 猜它等同哪個 Hop component。

應該：

```text
Unsupported Component
       ↓
Explicit Flag
       ↓
Manual Review / New Parser Fixture
```

### Parser 無法解析 Artifact

保留 parse failure，不能讓 AI 自動產生一份看起來合理的 metadata 取代 structural truth。

## Evidence

- `etl_intelligence/parser.py`
- `etl_intelligence/metadata.py`
- `evaluation/dataset.json`
- `tests/test_p2_evaluation.py`
- `docs/ETL_AI_EVALUATION.md`

## 常見踩雷

> LLM 現在 context 很大，所以直接讀完整 KTR / KJB 應該比較簡單。

Context window 解決的是「放不放得進去」，不是 correctness、repeatability 或 auditability。

## Senior-level Close

> 我不是反對 Raw ETL → LLM，而是不把它當 structural truth mechanism。對 modernization correctness，我需要 deterministic facts；AI 的價值放在 deterministic truth 之上的解釋、摘要與輔助判斷。

---

# 14. Parser Correctness 要怎麼量測？為什麼 Synthetic Ground Truth 不等於 Production Accuracy？

**Primary Repo:** Enterprise ETL Platform

## 面試官真正想測什麼

- 是否真的懂 evaluation，而不是只寫 unit test
- 是否能定義 precision / recall / exact-match / failure rate
- 是否知道 benchmark scope 與 external validity
- 是否會避免用小型測試集過度宣稱準確率

## Decision

建立 repository-owned synthetic corpus，對每個 ETL case 定義 deterministic ground truth。

目前 v0.8.0 corpus 涵蓋：

- simple extraction
- join
- lookup
- filter
- aggregation
- SQL-heavy
- multi-pipeline dependency
- invalid / partial definition
- unsupported component
- complex parameters

Ground truth 不只寫「預期成功」，而是包含：

- expected steps
- sources / targets
- dependencies
- SQL
- tables
- parameters / variables
- unsupported component
- expected parse failure

## Metrics

Parser 至少量：

- step extraction precision / recall / F1
- source / target extraction
- dependency extraction
- SQL extraction
- table extraction
- unsupported node detection
- structural exact-match rate
- expected / unexpected parsing failure rate

## 為什麼 Synthetic Corpus 有價值？

因為它：

- 可公開
- 可 version control
- 可 CI 重複
- 可精準定義 edge case
- 不洩漏 customer artifact
- 可以作為 parser regression gate

## 為什麼不能宣稱 Production Accuracy？

因為 10-case corpus 不代表：

- 真實企業所有 plugin 分布
- 不同 Pentaho / Hop 版本差異
- 長期累積的奇怪 legacy configuration
- custom Java / script / vendor plugin
- 真實參數、repository connection 與 runtime context

所以「synthetic corpus exact-match = 100%」只代表：

> **目前這 10 個 regression contract 全部通過。**

不代表：

> **Production parser accuracy = 100%。**

## Trade-off

Synthetic corpus 容易測、可公開，但 external validity 有限。

Production corpus realism 高，但：

- 可能含機敏資料
- ground truth 建置成本高
- 不一定可以進 public repo

成熟策略應該是兩層：

```text
Public Synthetic Regression
          +
Private / Sanitized Production-like Evaluation
```

## Failure / Recovery

新增 parser feature 時如果 regression 掉：

- CI fail
- 不調低 expected result 來讓測試通過
- 先確認 parser change 是 regression 還是 ground truth 需要合理 versioning

## Evidence

- `evaluation/dataset.json`
- `etl_intelligence/evaluation.py`
- `reports/baseline/`
- GitHub Actions `p2-etl-ai-evaluation`

## Senior-level Close

> Evaluation 的價值不在漂亮分數，而在把 claim scope 說清楚。我可以證明 synthetic regression corpus 的 correctness，但 production accuracy 必須用更具代表性的資料另外驗證。

---

# 15. Hallucination Guard 能保證什麼？又不能保證什麼？

**Primary Repo:** Enterprise ETL Platform

## 面試官真正想測什麼

- 是否把 hallucination 當成一句 prompt 就能解決
- 是否理解 structured validation 與 general factuality 的差別
- 能不能明確描述 guardrail 的 capability boundary

## Decision

> **AI 回傳的 structural claim 必須能對應 deterministic parser evidence；未知 Source / Target / SQL / Dependency / Evidence Reference 直接 reject 或 fallback。**

例如 parser 只提供：

```text
source = staging.orders
target = mart.orders
```

AI 如果輸出：

```text
source = crm.customers
```

不能因為語意合理就接受。

## 能保證什麼

這個 guard 可以控制：

- structured source / target fabrication
- SQL reference 不一致
- dependency node fabrication
- evidence reference 不存在
- parser digest / context contract 不一致

## 不能保證什麼

它不是 universal truth verifier。

例如：

> 「這個 transformation 主要用於財務月結。」

如果 parser metadata 沒有 business context，就不能只靠 structural guard 判斷這句自然語言是否真的正確。

所以應區分：

```text
Structural Claim
  → deterministic validation possible

Semantic / Business Claim
  → needs source evidence, RAG, human review or lower confidence
```

## Trade-off

越嚴格的 validation：

- hallucination risk 越低
- AI 可自由摘要的空間越小
- 某些合理 inference 可能被拒絕

因此 inference 應明確標示為 inference，而不是偷偷混入 parser fact。

## Failure / Recovery

### Structured Output Invalid

有限次 validation retry。

### Retry 後仍 Invalid

reject / deterministic fallback。

### AI 加入未知 Structural Fact

reject，不把它 merge 回 canonical metadata。

## Evidence

- `etl_intelligence/analyzer.py`
- `tests/test_p2_evaluation.py`
- `docs/ETL_AI_EVALUATION.md`

## 常見踩雷

> 有 grounding prompt，所以模型不會 hallucinate。

Prompt 是 guidance，不是 proof。

## Senior-level Close

> Hallucination guard 的成熟做法不是宣稱「模型不會亂講」，而是定義哪些 claim 可以 deterministic validate、哪些只能標示 inference，並讓 canonical truth 永遠不被模型自行改寫。

---

# 16. AI / Gateway 掛掉時，ETL Modernization 應該怎麼 Degraded？

**Primary Repos:** Enterprise ETL Platform + Multi-LLM AI Gateway

## 面試官真正想測什麼

- AI 是否被放進 critical correctness path
- dependency failure semantics
- application vs platform responsibility boundary
- retry / fallback 是否重複實作

## Decision

> **AI 是 advisory semantic layer，不是 deterministic parser / migration validator 的必要 dependency。**

因此：

```text
AI Available
Parser → Metadata → AI Semantic Enrichment → Evaluation

AI Unavailable
Parser → Metadata → Deterministic Result / Manual Review
```

ETL structural truth 不因模型 unavailable 而消失。

## Application Layer 負責什麼

Enterprise ETL Platform 負責：

- structured output validation
- parser evidence validation
- bounded validation retry
- reject / deterministic fallback

## Gateway Layer 負責什麼

Multi-LLM Gateway 負責：

- provider timeout handling
- rate limit
- retry / fallback provider
- routing
- usage / cost
- provider observability

ETL repo 不應再重新實作一套 provider router。

## Trade-off

這樣的 degraded mode 代表：

- AI summary / explanation 暫時不可用
- modernization UX 下降
- manual review 可能增加

但 correctness 不應一起下降。

## Failure / Recovery

### Gateway Timeout

ETL layer 接到 transport failure後走 deterministic fallback，不做 application-level provider hopping。

### AI 回傳 malformed JSON

這是 semantic contract failure，可以在 analyzer 做有限次 retry；仍失敗就 reject。

### 整個 AI Gateway outage

parser / metadata / lineage / migration validation 仍工作；AI-dependent output 顯示 unavailable，而不是 fabricate substitute。

## Evidence

- `etl_intelligence/analyzer.py`
- `etl_intelligence/gateway.py`
- `tests/test_p2_evaluation.py`
- Multi-LLM Gateway routing / failure responsibility boundary

## Senior-level Close

> 如果 AI outage 會讓我連 source / target 都不知道，那 architecture boundary 就放錯了。我希望 AI failure 影響的是 enrichment quality，而不是 deterministic ETL truth。

---

# 17. 243 Pipelines 的 Token / Cost / Latency 要怎麼估？為什麼沒有 Evidence 時寧可留 Null？

**Primary Repos:** Enterprise ETL Platform + Multi-LLM AI Gateway

## 面試官真正想測什麼

- 是否有 LLM cost / capacity thinking
- 是否知道 estimate 與 measurement 的差別
- 是否會杜撰 token count
- 能不能設計 per-request → batch projection

## Decision

Evaluation framework 收集：

- pipeline count
- AI requests
- input tokens
- output tokens
- parser duration
- AI latency
- total duration
- failures / retries
- provider / model
- estimated cost

然後做：

```text
Per Pipeline Evidence
        ↓
Batch Aggregation
        ↓
Provider / Model Aggregation
        ↓
243-Pipeline Projection
```

但 token / cost 只有在 live Gateway 提供 usage / pricing evidence 時才有值。

## 為什麼不自己用字元數估 Token？

因為：

- tokenizer 依 model 不同
- prompt template 可能變
- provider accounting 可能含額外 token category
- pricing 隨 provider / model / cache policy 改變

粗估可以用於 planning，但不能標成 measured evidence。

## 為什麼 Baseline Cost 是 Null？

因為 checked-in synthetic baseline 沒有 live Gateway usage。

所以正確資料是：

```json
{
  "input_tokens": null,
  "output_tokens": null,
  "estimated_cost_usd": null
}
```

不是為了讓報表好看而補一個推測值。

## 243 Pipelines 怎麼估？

有實際 representative sample 後，可以：

1. 取得 per-pipeline token / cost distribution
2. 依 pipeline complexity 分 cohort
3. 分別計算 p50 / p95 latency 與 cost
4. 用 batch concurrency 模型估 throughput
5. 做 243 pipeline scenario projection
6. 把 pricing / model version 固定在 report metadata

如果 sample 不具代表性，要標示 uncertainty。

## Trade-off

Evidence discipline 會讓 dashboard 有些欄位看起來「不完整」。

但：

> **Unknown 明確標示 unknown，比 fabricated precision 更有工程價值。**

## Failure / Recovery

Gateway pricing_unknown：

- 保留 token usage
- cost = null
- 不用舊 pricing 偷算成 current cost

provider/model 變更：

- 分 bucket 報告
- 不把不同 model 混成一個平均值而失去可解釋性

## Evidence

- `etl_intelligence/evaluation.py`
- `etl_intelligence/gateway.py`
- `scripts/evaluate_etl_ai.py --live-gateway`
- Multi-LLM Gateway usage / pricing contract

## Senior-level Close

> LLM cost estimation 的重點不是算出一個漂亮數字，而是知道這個數字來自 measured usage、pricing contract 還是 planning assumption，並讓三者在報表中不能混淆。

---

# 18. Synthetic Evaluation 要如何走向 Production Evaluation，而不洩漏客戶資料？

**Primary Repo:** Enterprise ETL Platform

## 面試官真正想測什麼

- 能否從 public demo 走到 enterprise validation
- privacy / confidentiality awareness
- dataset representativeness
- evaluation lifecycle

## Decision

採分層 evaluation strategy：

```text
Layer 1 — Public Synthetic Regression
  Stable / versioned / shareable

Layer 2 — Sanitized Production-like Corpus
  Representative structures, no secrets / customer identifiers

Layer 3 — Private Production Evaluation
  Internal only, access-controlled, real distribution
```

三層用途不同，不能互相冒充。

## Layer 1 的目的

Public synthetic corpus 用於：

- parser regression
- CI
- architecture evidence
- edge-case contract
- open portfolio review

## Layer 2 的目的

Sanitized production-like corpus 用於：

- plugin distribution realism
- large pipeline complexity
- parameter / variable combinations
- deeply nested workflow dependency
- representative SQL complexity

去識別時應處理：

- hostname
- schema / table business name
- credential / connection metadata
- customer name / project ID
- internal endpoint
- SQL literal 中的敏感內容

## Layer 3 的目的

Private production evaluation 才能回答：

- 真實 plugin coverage
- production parse failure distribution
- human review rate
- real latency / token / cost distribution
- business usefulness

但這些結果即使能公開，也只能公開 aggregated metric，不應公開 raw artifact。

## Production Metrics 還要增加什麼？

除了 synthetic parser metrics，建議增加：

- unsupported component rate
- manual-review rate
- migration reconciliation pass rate
- AI reject / fallback rate
- human acceptance / correction rate
- per-complexity cohort accuracy
- p50 / p95 latency
- token / cost distribution
- version-to-version regression

## Trade-off

越接近 production：

- representativeness 越高
- 資料治理 / ground truth 成本也越高

所以不能因為 production corpus 建置很麻煩，就把 public synthetic score 當 production score。

## Failure / Recovery

Sanitization 無法確定是否完整：

- 不進 public repo
- 留在 private evaluation environment

Ground truth 有爭議：

- 標記 ambiguous
- 不強行計入 deterministic accuracy
- 建立 adjudication / reviewer process

## Evidence

目前 public repository 已具備 Layer 1：

- `evaluation/dataset.json`
- `reports/baseline/`
- CI-generated evaluation artifact

Layer 2 / Layer 3 屬下一階段 production validation，不在 public repo 偽造結果。

## Senior-level Close

> 從 synthetic 到 production 的關鍵不是把 dataset 做大，而是提高 representativeness，同時維持 confidentiality、ground-truth quality 與 claim scope。Public portfolio 證明 methodology，private evaluation 才證明 production fit。

---

# Cross-Question Pattern — ETL AI

面試官可能把上面六題混在一起問，例如：

> 為什麼不用直接把 Pentaho XML 丟給 GPT？

建議回答順序：

```text
1. 先說 structural truth 必須 deterministic
2. 再說 AI 放在哪一層
3. 說 evaluation 怎麼做
4. 說 hallucination guard 的能力邊界
5. 說 AI outage 時哪些功能仍成立
6. 最後說 cost / latency 只有 actual evidence 才能量化
```

90 秒版本：

> 我會先把 Legacy ETL 的 step、source、target、SQL 和 dependency 用 deterministic parser 正規化，因為這些是 migration correctness 的 structural truth。AI 只在這份 metadata 上做 semantic interpretation，輸出的 structural claim 必須回到 parser evidence 驗證。Repository 用 synthetic ground truth 做 parser / semantic regression，但我明確不把 10-case 100% exact-match 說成 production accuracy。AI 或 Gateway 掛掉時 parser / migration validator 仍可工作；而 token / cost 沒有 live Gateway evidence 時就保持 null。我的目標不是讓 AI 取代 ETL correctness，而是讓它在可驗證的 deterministic boundary 內提高理解效率。

---

# Role-specific Priority Update

## Senior Data Engineer

新增優先熟練：

1. Deterministic Parser vs Raw ETL → LLM
2. Parser Ground Truth / Accuracy
3. AI Failure Degradation
4. 原 P2.3 的 Airflow vs Hop
5. PostgreSQL Execution Truth
6. Immutable Promotion

## Data Platform Engineer

新增優先熟練：

1. Deterministic Metadata / Lineage Boundary
2. AI vs MCP / Gateway Responsibility Boundary
3. Production Evaluation Data Governance
4. MCP Integration Boundary
5. RBAC / Tenant Isolation

## AI Platform / AI Integration

新增優先熟練：

1. Hallucination Guard Capability Boundary
2. Deterministic Truth vs Probabilistic Semantic Layer
3. AI Outage / Gateway Failure Semantics
4. Usage / Token / Cost Evidence
5. Synthetic vs Production Evaluation
6. Multi-LLM Gateway

---

# Claim Safety Update

### 可以說

> 在 public synthetic 10-case regression corpus 上，parser structural exact-match 與 parser-truth preservation 均通過目前 baseline。

### 不應說

> Parser production accuracy 是 100%。

### 可以說

> Repository 已建立 243-pipeline cost projection framework，可在 live Gateway 提供 token / pricing evidence 後產生 projection。

### 不應說

> 我已量測 243 條 production ETL 的實際 LLM 成本。

### 可以說

> Hallucination guard 會拒絕 parser evidence 未提供的 structured source / target / SQL / dependency claim。

### 不應說

> 這套 guard 可以證明所有自然語言敘述都是真實的。

---

# P2.3 Extension Completion Checklist

- [x] Deterministic parser vs Raw ETL → LLM decision
- [x] Parser evaluation / ground-truth decision
- [x] Synthetic benchmark claim boundary
- [x] Structured hallucination guard decision
- [x] AI outage / deterministic degradation semantics
- [x] ETL application vs Multi-LLM Gateway responsibility boundary
- [x] Token / cost / latency evidence discipline
- [x] 243-pipeline projection methodology
- [x] Synthetic → sanitized → private production evaluation path
- [x] Role-specific interview priority update
- [x] Claim Safety update

**Next: P2.4 — Target Job Mapping**
