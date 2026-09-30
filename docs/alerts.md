# Template Alert và Runbook

Mỗi alert phải dựa trên triệu chứng người dùng hoặc SLO, không dựa trực tiếp vào tên implementation nội bộ.

## Alert mẫu để tham khảo

Ví dụ dưới đây minh họa mức độ cụ thể cần có. Học viên không cần copy nguyên, nhưng ba alert trong bài nộp nên rõ ràng tương tự: điều kiện là gì, kéo dài bao lâu, ảnh hưởng tới user ra sao và người trực cần kiểm tra gì trước.

- Tên: `HighLatencyP95`
- Severity: `warning`
- Duration: `5m`
- Kênh thông báo: Slack `#k4-l3b-alerts`
- SLI/SLO liên quan: latency P95 của `response_sent.latency_ms`
- Điều kiện và thời gian duy trì: `p95(latency_ms) > 3000ms` trong 5 phút
- Ảnh hưởng tới người dùng: người dùng phải chờ lâu hơn trước khi nhận câu trả lời
- Ba bước kiểm tra đầu tiên:
  1. Mở dashboard latency để xác nhận P95/P99 và khoảng thời gian tăng.
  2. Lọc `data/logs.jsonl` trong khoảng đó, lấy một `correlation_id` có `latency_ms` cao.
  3. Mở trace cùng `correlation_id` trên Langfuse, so sánh các span chính để xác định bước nào bất thường.
- Mitigation tạm thời: dựa trên evidence thực tế để rollback prompt, khôi phục cấu hình liên quan, tắt practice scenario hoặc giảm tải khi demo.
- Owner: `student-<MSSV>`

## Alert 1

- Tên: `HighLatencyP95`
- Severity: `warning`
- Duration: `5m`
- Kênh thông báo: Slack `#k4-l3b-alerts`
- SLI/SLO liên quan: P95 latency trong SLO `fast_successful_requests`
- Điều kiện và thời gian duy trì: `p95(response_sent.latency_ms) > 3000` trong 5 phút
- Ảnh hưởng tới người dùng: câu trả lời đến chậm hơn ngưỡng SLO
- Ba bước kiểm tra đầu tiên: xác nhận P95/P99 trên dashboard; lọc log lấy `correlation_id` chậm; mở trace và so sánh retrieval/generation
- Mitigation tạm thời: rollback prompt candidate hoặc tắt scenario gây tải sau khi xác nhận evidence
- Owner: `student-2A202602402`

## Alert 2

- Tên: `ElevatedErrorRate`
- Severity: `critical`
- Duration: `5m`
- Kênh thông báo: Slack `#k4-l3b-alerts`
- SLI/SLO liên quan: error budget của `fast_successful_requests`
- Điều kiện và thời gian duy trì: error rate của `request_failed/request_received` lớn hơn 2% trong 5 phút
- Ảnh hưởng tới người dùng: request thất bại hoặc không nhận được câu trả lời
- Ba bước kiểm tra đầu tiên: xác nhận error rate và error type; lọc log theo `request_failed`; mở trace cùng `correlation_id` để xác định span lỗi
- Mitigation tạm thời: tắt incident practice, khôi phục dependency/config vừa thay đổi, rồi xác nhận error rate giảm
- Owner: `student-2A202602402`

## Alert 3

- Tên: `LowRetrievalSuccess`
- Severity: `warning`
- Duration: `10m`
- Kênh thông báo: Slack `#k4-l3b-alerts`
- SLI/SLO liên quan: guardrail retrieval success tối thiểu 90%
- Điều kiện và thời gian duy trì: retrieval success rate nhỏ hơn 90% trong 10 phút
- Ảnh hưởng tới người dùng: câu trả lời thiếu context hoặc dùng fallback không phù hợp
- Ba bước kiểm tra đầu tiên: xác nhận panel errors; lọc log theo `tool_name` và `tool_success`; mở trace để kiểm tra retrieval span
- Mitigation tạm thời: giảm tải, sửa/khôi phục retriever config và kiểm tra lại query mẫu
- Owner: `student-2A202602402`
