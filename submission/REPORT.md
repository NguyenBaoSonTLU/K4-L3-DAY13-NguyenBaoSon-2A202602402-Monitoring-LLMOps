# Báo cáo cá nhân — K4-L3B Day 13 Monitoring & LLMOps

> Mỗi học viên hoàn thiện một file duy nhất này. Khi dẫn evidence, dùng đường dẫn tương đối, ví dụ `evidence/07-trace-waterfall.png`.

## 1. Thông tin học viên

- **Họ và tên:** Nguyen Bao Son
- **MSSV:** 2A202602402
- **Lớp:** K4-L3B
- **Repository URL:** https://github.com/NguyenBaoSonTLU/K4-L3-DAY13-NguyenBaoSon-2A202602402-Monitoring-LLMOps
- **Commit SHA cuối:** Sẽ cập nhật sau commit CP4.
- **Challenge ID:** `day13-k4-l3b-monitoring-llmops-v1`
- **Tên project Langfuse cá nhân:** `day13-k4-l3b-2A202602402`

## 2. Evidence index

Điền đúng đường dẫn tới evidence thực tế. Có thể đổi tên hoặc dùng nhiều ảnh nếu cần.

| Evidence | Đường dẫn |
|---|---|
| Pytest cuối | `evidence/01-pytest.txt` |
| Log validator | `evidence/02-log-validator.txt` |
| Dashboard validator | `evidence/03-dashboard-validator.txt` |
| Structured log | `evidence/04-structured-log.png` |
| PII redaction | `evidence/05-pii-redaction.png` |
| Trace list | `evidence/06-trace-list.png` |
| Trace waterfall | `evidence/07-trace-waterfall.png` |
| Trace metadata | `evidence/08-trace-metadata.png` |
| Prompt versions | `evidence/09-prompt-versions.png` |
| Prompt rollback | `evidence/10-prompt-rollback.png` |
| Dashboard runtime | `evidence/11-dashboard-overview.png` |
| Incident metric | `evidence/12-incident-metric.png` |
| Incident log | `evidence/13-incident-log.png` |
| Incident trace | `evidence/14-incident-trace.png` |

## 3. Kết quả kỹ thuật

| Nội dung | Baseline | Kết quả cuối | Nhận xét |
|---|---|---|---|
| `validate_logs.py` | Baseline CP0 chưa đạt | `100/100` | 75 records, 37 correlation IDs, 0 PII leak |
| `validate_dashboard.py` | Contract chưa hoàn thiện | `6/6 panel` | Dashboard YAML hợp lệ |
| `pytest` | 22 tests | `24 passed` | Có thêm test CCCD và thẻ thanh toán |
| Số traces hợp lệ | 0 | 15 challenge observations / 5 trace trees đã xác nhận qua Observations v2 | Cần chụp trace list để chứng minh tối thiểu 10 trace trên UI |
| Số PII leak | Chưa đo | `0` | Validator độc lập không phát hiện PII |
| Latency P95 / TTFT P95 | Baseline median khoảng `381 ms` | Challenge median `2877 ms`, max `3527 ms` | TTFT cụ thể cần lấy từ dashboard runtime |
| Retrieval success rate | Chưa đo | Các request thành công có `tool_success=true` | Panel errors đã bao gồm response và failure events |

## 4. Logging và PII

- **Cách tạo/nhận và truyền correlation ID:** Middleware xóa context cũ, nhận `x-request-id` hoặc sinh `req-<8-hex>`, bind vào structlog contextvars, truyền vào agent/trace và trả lại qua `x-request-id`.
- **Các metadata được ghi vào structured log:** `ts`, `level`, `service`, `event`, `correlation_id`, `user_id_hash`, `session_id`, `feature`, `model`, `env`, latency, TTFT, token, cost, quality và tool status.
- **Cách bảo đảm PII được scrub trước khi ghi:** `scrub_event` chạy trước `JsonlFileProcessor` và JSON renderer; scrub đệ quy string trong dict/list bằng pattern email, phone VN, CCCD và credit card.
- **Cách kiểm chứng kết quả:** `validate_logs.py` đạt `100/100`, 0 PII leak; test PII có email, phone, CCCD và thẻ đạt `4 passed`.

## 5. Tracing và prompt versioning

- **Cách xác nhận traces do chính tôi tạo trong project cá nhân:** Workload challenge dùng session `k4-l3b-challenge-s01` đến `s05` và được truy vấn bằng Langfuse Observations v2 trong project cá nhân.
- **Cấu trúc root/retrieval/generation observations:** Mỗi trace có `lab-agent-run` làm root, `retrieval` loại `RETRIEVER` và `generation` loại `GENERATION` cùng trace ID.
- **Cách nối trace với log:** Log dùng `correlation_id`; trace root nhận metadata này qua `propagate_attributes`. Challenge trace đã đối chiếu với session/log `req-375b7196`.
- **Prompt name:** `day13-chat`
- **Version/label baseline:** Cần chụp và ghi version thực tế sau khi tạo prompt v1 trên Langfuse.
- **Version/label candidate:** Cần chụp và ghi version thực tế sau khi tạo prompt v2 trên Langfuse.
- **Trace ID của mỗi version:** Chưa thu thập trace riêng cho prompt v1/v2; không tự điền ID giả. Trace challenge đã xác nhận: `b5a775be498f2ca186f1a528186c2e52`.
- **Cách promote và rollback `production`:** Thực hiện trên Langfuse Prompt Management bằng cách chuyển label `production` sang v2 rồi chuyển lại v1; cần bổ sung ảnh 10a/10b và hai trace ID sau thao tác.

## 6. Dashboard, SLO và alerts

- **Dashboard và sáu panel:** `latency`, `traffic`, `errors`, `cost`, `tokens`, `quality`, nguồn `data/logs.jsonl`, time range 60 phút và refresh 30 giây; contract đã đạt `6/6`.
- **SLO và lý do chọn:** `fast_successful_requests`, 99.5% request có response trong 3000 ms trên cửa sổ 28 ngày, phù hợp với mục tiêu tail latency của API.
- **Cách tính error budget:** `10,000 * (1 - 0.995) = 50` request được phép không đạt SLO.
- **Ba alert và runbook tương ứng:** `HighLatencyP95`, `ElevatedErrorRate`, `LowRetrievalSuccess`; mỗi alert có duration, severity, owner, Slack channel và link tới `docs/alerts.md`.

> Ví dụ cách viết error budget: "SLO 99.5% trong 28 ngày nghĩa là error budget 0.5%. Nếu workload có 10,000 request thì tối đa 50 request được phép lỗi hoặc chậm hơn ngưỡng SLO."

## 7. Điều tra challenge

- **Challenge ID:** `day13-k4-l3b-monitoring-llmops-v1`
- **Khoảng thời gian điều tra:** `2026-09-30T17:04:19Z` đến `2026-09-30T17:04:35Z` UTC.
- **Triệu chứng từ metrics:** Baseline có median `latency_ms` khoảng `381 ms`; challenge có median khoảng `2877 ms`, cao nhất `3527 ms`, vượt ngưỡng `2000 ms`.
- **Log line và correlation ID liên quan:** Session `k4-l3b-challenge-s02` có `response_sent.latency_ms=3527` và `correlation_id=req-375b7196`.
- **Trace ID và span gây ảnh hưởng:** Trace `b5a775be498f2ca186f1a528186c2e52` có root khoảng `3528 ms`, span `retrieval` khoảng `2501 ms`, và span `generation` khoảng `152 ms`.
- **Root cause:** Incident `rag_slow` làm bước retrieval chậm khoảng `2.5 s`; generation không phải bottleneck vì chỉ mất khoảng `152 ms`.
- **Fix action:** Tắt incident `rag_slow` bằng `python scripts/inject_incident.py --scenario rag_slow --disable`, sau đó chạy lại baseline và xác nhận retrieval latency giảm.
- **Preventive measure:** Giữ alert `HighLatencyP95`, lọc log theo `correlation_id` rồi mở retrieval span trong runbook; bổ sung regression test cho retrieval timeout/latency và theo dõi P95/P99.

> Gợi ý cách viết ngắn, không thay cho evidence thực tế: "Metric cho thấy `[latency/error/cost/quality]` bất thường trong `[khoảng thời gian]`. Log line `[event]` có `correlation_id=[...]` đại diện cho request bị ảnh hưởng. Trace cùng `correlation_id` cho thấy span `[retrieval/generation/prompt/tool]` có dấu hiệu `[chậm/lỗi/token tăng]`. Root cause là `[nguyên nhân suy ra từ evidence]`. Fix action là `[hành động khôi phục]`; preventive measure là `[alert/runbook/test/guardrail để ngăn tái diễn]`."

## 8. Giải thích và tự đánh giá

- **Một quyết định kỹ thuật quan trọng và lý do:** Tắt capture input/output raw trên Langfuse và chỉ lưu preview đã scrub, vì message có thể chứa PII.
- **Một lỗi/blocker đã gặp:** `load_test.py` trả WinError 10061 khi API chưa chạy; `.env` thiếu cũng làm Uvicorn dừng.
- **Cách tìm nguyên nhân và xử lý:** Tạo `.env` local từ `.env.example`, khởi động Uvicorn trước, kiểm tra `/health`, rồi chạy workload ở terminal thứ hai.
- **Cách hiểu luồng Metrics → Logs → Traces:** Metrics khoanh vùng latency và thời gian; logs chọn request qua `correlation_id`; trace so sánh span để kết luận retrieval là bottleneck.
- **Vai trò của prompt version, token/cost, SLO hoặc rollback trong vận hành LLM:** Version/label giúp truy nguyên prompt; token/cost phát hiện regression; SLO định lượng độ tin cậy; rollback khôi phục version ổn định.
- **Điều quan trọng nhất đã học:** Root cause phải được chứng minh bởi ba nguồn cùng chỉ về một request hoặc một khoảng thời gian.
- **Hạn chế hoặc phần chưa hoàn thành, nếu có:** Chưa có screenshot UI cho dashboard, prompt versions/promote/rollback và trace evidence; cần chụp thủ công trong project Langfuse cá nhân trước khi nộp.

## 9. Checklist trước khi nộp

- [ ] Kết quả và evidence thuộc commit SHA cuối.
- [ ] Tất cả ảnh/output mở được bằng đường dẫn tương đối.
- [x] Incident evidence nối đúng metric → log → trace.
- [ ] Trace/prompt evidence thuộc project Langfuse cá nhân và ảnh không lộ key/secret.
- [x] Repository chạy lại được theo README.
- [x] Không có secret, API key, PII thô hoặc evidence của người khác/lớp khác.
- [ ] URL repo và commit SHA cuối đã được nộp trên LMS/Codelabs.
