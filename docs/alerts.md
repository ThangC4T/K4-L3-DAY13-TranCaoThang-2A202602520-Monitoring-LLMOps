# Alert va Runbook

Moi alert dua tren trieu chung nguoi dung/SLO, co duration de tranh paging khi chi co mot request le. Khi dieu tra, luon di theo thu tu Metrics -> Logs -> Traces.

## Alert 1 High Latency P95

- Ten: `high_latency_p95`
- Severity: page
- Duration: 5m
- Kenh thong bao: Slack `#llmops-alerts`
- SLI/SLO lien quan: fast successful requests, `response_sent.latency_ms <= 3000`
- Dieu kien va thoi gian duy tri: `p95(latency_ms) > 3000` tren event `response_sent` trong 5 phut
- Anh huong toi nguoi dung: phan hoi chat cham, co nguy co vuot SLO latency
- Ba buoc kiem tra dau tien:
  1. Mo dashboard latency, xac dinh khoang thoi gian P95/P99 tang va TTFT co tang cung khong.
  2. Loc `data/logs.jsonl` trong khoang do, lay mot `correlation_id` co `latency_ms` cao.
  3. Mo trace co cung `correlation_id`, so sanh thoi gian span retrieval va generation.
- Mitigation tam thoi: neu retrieval cham, giam concurrency hoac rollback cau hinh retrieval; neu generation cham/cost tang, chuyen label prompt ve version on dinh va giam output length.
- Owner: llmops-oncall

## Alert 2 Elevated Error Rate

- Ten: `elevated_error_rate`
- Severity: page
- Duration: 5m
- Kenh thong bao: Slack `#llmops-alerts`
- SLI/SLO lien quan: request thanh cong va error budget
- Dieu kien va thoi gian duy tri: `count(request_failed) / count(request_received) * 100 > 2` trong 5 phut
- Anh huong toi nguoi dung: mot phan request tra loi loi HTTP 500 thay vi cau tra loi chat
- Ba buoc kiem tra dau tien:
  1. Mo panel errors, xac dinh `error_type` chiem nhieu nhat.
  2. Loc log `request_failed`, lay `correlation_id`, `error_type`, `tool_name`, `tool_success`.
  3. Mo trace cung `correlation_id`, kiem tra span nao loi hoac dung lau bat thuong.
- Mitigation tam thoi: neu loi den tu retrieval, bat fallback response va retry sau; neu loi den tu prompt/LLM, rollback label `production` ve prompt version gan nhat da on dinh.
- Owner: llmops-oncall

## Alert 3 Low Retrieval Success

- Ten: `low_retrieval_success`
- Severity: ticket
- Duration: 10m
- Kenh thong bao: Slack `#llmops-alerts`
- SLI/SLO lien quan: retrieval success guardrail >= 90%
- Dieu kien va thoi gian duy tri: `count(tool_success == true) / count(tool_success != null) * 100 < 90` trong 10 phut
- Anh huong toi nguoi dung: cau tra loi co the thieu ngu canh, chat luong giam hoac request that bai khi retrieval timeout
- Ba buoc kiem tra dau tien:
  1. Mo panel errors/retrieval, kiem tra success rate va error count theo thoi gian.
  2. Loc log co `tool_name == "retrieval"` va `tool_success == false`, lay `correlation_id`.
  3. Mo trace cung `correlation_id`, xem retrieval span co timeout, loi, hay tra ve doc count bat thuong.
- Mitigation tam thoi: bat fallback answer khi retrieval loi, giam concurrency, kiem tra vector store/index, va rollback thay doi query/prompt neu su co bat dau sau deploy prompt moi.
- Owner: llmops-oncall
