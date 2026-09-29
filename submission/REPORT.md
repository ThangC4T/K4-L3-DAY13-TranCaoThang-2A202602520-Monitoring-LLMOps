# Bao cao ca nhan - K4-L3A Day 13 Monitoring & LLMOps

## 1. Thong tin hoc vien

- **Ho va ten:** Tran Cao Thang
- **MSSV:** 2A202602520
- **Lop:** K4-L3A
- **Repository URL:** https://github.com/ThangC4T/K4-L3-DAY13-TranCaoThang-2A202602520-Monitoring-LLMOps
- **Commit SHA cuoi:** se lay bang `git log -1 --oneline` sau commit cuoi va nop tren LMS/Codelabs
- **Challenge ID:** chua co file challenge chinh thuc tu Lab Coach
- **Ten project Langfuse ca nhan:** `day13-k4-l3a-2A202602520`

## 2. Evidence index

| Evidence | Duong dan |
|---|---|
| Pytest cuoi | `evidence/01-pytest.txt` |
| Log validator | `evidence/02-log-validator.txt` |
| Dashboard validator | `evidence/03-dashboard-validator.txt` |
| Structured log | `evidence/04-structured-log.txt` |
| PII redaction | `evidence/05-pii-redaction.txt` |
| Trace list | `evidence/06-trace-list.png`; run log `evidence/06-trace-list-production-run.txt` |
| Trace waterfall | `evidence/07-trace-waterfall.png` |
| Trace metadata | `evidence/08-trace-metadata.png` |
| Prompt versions | `evidence/09-prompt-versions.png`; text check `evidence/09-prompt-versions.txt` |
| Prompt rollback | `evidence/10-prompt-rollback.png`; text check `evidence/10-prompt-rollback.txt` |
| Dashboard runtime | `evidence/11-dashboard-overview.png`; text summary `evidence/11-dashboard-overview.txt` |
| Incident metric | Chua co vi chua nhan challenge chinh thuc |
| Incident log | Chua co vi chua nhan challenge chinh thuc |
| Incident trace | Chua co vi chua nhan challenge chinh thuc |

## 3. Ket qua ky thuat

| Noi dung | Baseline | Ket qua cuoi | Nhan xet |
|---|---|---|---|
| `validate_logs.py` | Baseline starter chua dat CP1 | 100/100 | Log co correlation ID, enrichment va khong con PII tho |
| `validate_dashboard.py` | Dashboard contract co san | 6/6 panel hop le | Du 6 panel latency, traffic, errors, cost, tokens, quality |
| `pytest` | Chua xac nhan truoc khi sua | 22 passed | Tat ca public tests pass |
| So traces hop le | Chua co trace truoc khi cau hinh Langfuse | >=20 request da tao trace/observations | Evidence `06-trace-list.png` cho thay trace list co du lieu trong project ca nhan |
| So PII leak | Chua xac nhan truoc CP1 | 0 | Validator khong phat hien PII leak trong `data/logs.jsonl` |
| Latency P95 / TTFT P95 | Chua xac nhan truoc CP1 | 1419ms / 50ms tren workload local | Xem `evidence/11-dashboard-overview.txt` |
| Retrieval success rate | Chua xac nhan truoc CP1 | 100% tren workload local | Xem `evidence/11-dashboard-overview.txt` |

## 4. Logging va PII

- **Cach tao/nhan va truyen correlation ID:** `CorrelationIdMiddleware` xoa contextvars dau moi request, nhan `x-request-id` neu dung format `req-<8 hex>`, neu khong thi sinh ID moi bang `uuid4`. ID duoc bind vao structlog contextvars, gan vao `request.state.correlation_id`, tra lai qua header `x-request-id` va kem `x-response-time-ms`.
- **Cac metadata duoc ghi vao structured log:** endpoint `/chat` bind `user_id_hash`, `session_id`, `feature`, `model`, `env`. Log `request_received`, `response_sent`, `request_failed` deu co context chung de noi request voi dashboard va trace.
- **Cach bao dam PII duoc scrub truoc khi ghi:** `scrub_event` duoc dat trong processor chain truoc `JsonlFileProcessor` va `JSONRenderer`, nen payload string bi scrub truoc khi render/ghi vao `data/logs.jsonl`. `summarize_text` cung goi `scrub_text` truoc khi cat preview.
- **Cach kiem chung ket qua:** da chay `python scripts/validate_logs.py`, ket qua 100/100; evidence o `evidence/02-log-validator.txt`, `evidence/04-structured-log.txt`, `evidence/05-pii-redaction.txt`.

## 5. Tracing va prompt versioning

- **Cach xac nhan traces do chinh toi tao trong project ca nhan:** `.env` da co Langfuse keys, workload da chay voi `tracing_enabled=True`. Cac request production mau co correlation ID: `req-fbdf124c`, `req-321f368a`, `req-b4df9fe1`, `req-cc532ca9`, `req-c81dbbe4`. Evidence `06-trace-list.png` cho thay trace list trong project `day13-k4-l3a-2A202602520`.
- **Cau truc root/retrieval/generation observations:** `LabAgent.run` la root observation `lab-agent-run`; ben trong co child observation `retrieval` voi `doc_count`, `latency_ms`; child observation `llm-generate` kieu generation co `model`, `usage_details`, `cost_details`, `ttft_ms`, `cost_usd`.
- **Cach noi trace voi log:** metadata trace va child observations co `correlation_id`; structured logs cung co field `correlation_id`, nen co the loc log bat thuong roi tim trace tuong ung.
- **Prompt name:** `day13-chat`
- **Version/label baseline:** version 1, labels `baseline` va `production`.
- **Version/label candidate:** version 2, label `candidate`.
- **Trace ID cua moi version:** Trace v2 da chup trong `07-trace-waterfall.png` co trace ID `9bbba358a82287a5dc5af92c3a280014` va correlation ID `req-200f0ae4`. Correlation IDs v1: `req-fbdf124c`, `req-321f368a`; correlation IDs v2 khi promote candidate len production: `req-f1232814`, `req-45f51757`, `req-200f0ae4`.
- **Cach promote va rollback `production`:** Da promote `production` sang version 2, chay 3 request, sau do rollback `production` ve version 1; evidence text o `evidence/10-prompt-rollback.txt`.

## 6. Dashboard, SLO va alerts

- **Dashboard va sau panel:** `config/dashboard.yaml` hop le voi 6 panel: latency/TTFT, traffic, errors/retrieval success, cost, tokens, quality. Validator tra ve `HOP LE: 6/6 panel`.
- **SLO va ly do chon:** `config/slo.yaml` dinh nghia SLO `fast_successful_requests`: good event la `response_sent` co `latency_ms <= 3000`, total event la `request_received`, target 99.5% trong cua so 28 ngay. Nguong 3000ms phu hop cho API chat lab vi cham hon muc nay nguoi dung cam nhan ro.
- **Cach tinh error budget:** target 99.5% tuong duong error budget 0.5%; trong 28 ngay, toi da 0.5% request duoc phep vuot latency hoac khong thanh cong truoc khi can hanh dong.
- **Ba alert va runbook tuong ung:** `config/alert_rules.yaml` co `high_latency_p95`, `elevated_error_rate`, `low_retrieval_success`; runbook chi tiet o `docs/alerts.md`.

## 7. Dieu tra challenge

- **Challenge ID:** Chua co file challenge chinh thuc tu Lab Coach.
- **Khoang thoi gian dieu tra:** Chua thuc hien challenge chinh thuc.
- **Trieu chung tu metrics:** Chua co du lieu challenge chinh thuc.
- **Log line va correlation ID lien quan:** Chua co du lieu challenge chinh thuc.
- **Trace ID va span gay anh huong:** Chua co du lieu challenge chinh thuc.
- **Root cause:** Chua ket luan khi chua co challenge chinh thuc.
- **Fix action:** Se dua ra sau khi co evidence metric -> log -> trace cua challenge.
- **Preventive measure:** Se dua ra sau khi co root cause cua challenge.

## 8. Giai thich va tu danh gia

- **Mot quyet dinh ky thuat quan trong va ly do:** PII scrubbing duoc dat trong logging processor chain truoc file writer/JSON renderer thay vi chi scrub o endpoint. Cach nay giam rui ro khi cac log event moi duoc them vao sau nay.
- **Mot loi/blocker da gap:** Khi them Langfuse generation metadata, ban dau payload dung field `usage`, trong khi Langfuse SDK v4 nhan `usage_details` va `cost_details`. Da sua helper de dung dung signature SDK v4 va van fallback an toan khi chua cau hinh key.
- **Cach tim nguyen nhan va xu ly:** Chay pytest thay loi `/chat` tra 500, xem traceback thay `update_current_generation` nhan tham so sai, inspect signature SDK, sua payload va chay lai tests.
- **Cach hieu luong Metrics -> Logs -> Traces:** Metrics cho biet trieu chung va khoang thoi gian; logs loc request cu the bang `correlation_id`; trace cung `correlation_id` cho biet span retrieval hay generation gay cham/loi; tu do ket luan root cause.
- **Vai tro cua prompt version, token/cost, SLO hoac rollback trong van hanh LLM:** Prompt version giup so sanh va rollback hanh vi model; token/cost giup phat hien cost spike; SLO/error budget bien chat luong van hanh thanh nguong canh bao cu the.
- **Dieu quan trong nhat da hoc:** Observability cho LLM API can noi duoc metric, log va trace bang cung mot correlation ID, dong thoi khong lam lo PII.
- **Han che hoac phan chua hoan thanh, neu co:** Chua co challenge evidence vi chua nhan file challenge chinh thuc tu Lab Coach.

## 9. Checklist truoc khi nop

- [ ] Ket qua va evidence thuoc commit SHA cuoi.
- [x] Tat ca anh/output mo duoc bang duong dan tuong doi.
- [ ] Incident evidence noi dung metric -> log -> trace, sau khi Lab Coach mo challenge chinh thuc.
- [x] Trace/prompt evidence thuoc project Langfuse ca nhan va anh khong lo key/secret.
- [x] Repository chay lai duoc theo README.
- [x] Khong commit `.env`, API key, `.venv`, `data/logs.jsonl` hoac `config/challenge.json`.
- [ ] URL repo va commit SHA cuoi da duoc nop tren LMS/Codelabs.
