# 📊 BÁO CÁO THU HOẠCH NGHIỆM THU BÀI LAB 3 (BƯỚC 3 — SUBMISSION ARTIFACT)

> **Họ và Tên Học viên:** Võ Doanh Nhân  
> **Mã Sinh Viên / Mã Học viên:** 2A202602770  
> **Lớp:** K4A (Lớp Sáng)  
> **GitHub Repository:** https://github.com/nhanna4605/K4A-DAY03-VoDoanhNhan-2A202602770  
> **Chủ đề Lựa chọn:** Gợi ý 1.1 — Trợ lý Học vụ & Tra cứu Lịch thi VinUni (tra cứu hồ sơ học vụ + đặt lịch tư vấn với Cố vấn học tập)  

---

## 1. BẢNG CHẤM ĐIỂM AGENTIC FIT SCORING MATRIX (ĐÁNH GIÁ CHỦ ĐỀ)

| Tiêu chí Đánh giá | Mức độ (1 - 5) | Giải trình chi tiết lý do chọn điểm |
| :--- | :---: | :--- |
| **1. Multi-step Reasoning** | 4 / 5 | Có. Yêu cầu điển hình như "đặt lịch với cố vấn của tôi" buộc Agent phải tách thành chuỗi: xác minh mã sinh viên → tra cứu tên cố vấn → chuẩn hóa thời gian → đặt lịch → tổng hợp phản hồi (TC04). Không chấm 5 vì phần lớn yêu cầu chỉ dài 1–2 bước. |
| **2. Tool Interaction** | 5 / 5 | Bắt buộc. GPA, trạng thái học tập, cố vấn là dữ liệu riêng của từng sinh viên, LLM không thể biết nếu không gọi `academic_query`; việc đặt lịch là hành động ghi (write) phải thực thi qua `schedule_appointment` trên MCP Server. Chatbot thuần chỉ có thể từ chối hoặc bịa dữ liệu. |
| **3. Dynamic Decision** | 4 / 5 | Có. Kết quả Observation quyết định nhánh đi tiếp: `SUCCESS` → lấy tên cố vấn làm tham số đặt lịch; `NOT_FOUND` → dừng, không đặt lịch và báo lại sinh viên (TC05); câu hỏi chung → trả lời thẳng không gọi Tool (TC01). Số nhánh vẫn còn giới hạn nên chấm 4. |
| **4. Long Horizon Goal** | 3 / 5 | Trung bình. Agent phải giữ mục tiêu gốc ("đặt được lịch hẹn") xuyên suốt nhiều vòng ReAct trong cùng một phiên (tối đa `MAX_ITERATIONS = 5`), nhưng chưa cần ghi nhớ dài hạn qua nhiều ngày/nhiều phiên hội thoại. |
| **TỔNG ĐIỂM AGENTIC FIT** | **16 / 20** | *16 > 12/20 ⇒ Bài toán rất phù hợp triển khai ReAct Agent kết nối MCP Server thay vì Chatbot Baseline.* |

---

## 2. TRÍCH XUẤT KẾT QUẢ WATERFALL TRACE LOG (SAU KHI CHẠY TEST SUITE TRÊN API THẬT)

> ⚠️ **YÊU CẦU NGHIỆM THU:** Mở tệp `.env` điền `GEMINI_API_KEY` (hoặc `OPENAI_API_KEY`) để kết nối LLM thật trước khi thực thi `python src/app.py --all`. Bài nộp chỉ dùng Mock Offline Provider sẽ không đạt điểm nghiệm thực tế.

**Cấu hình chạy nghiệm thu:** `LLM_PROVIDER=gemini`, `LLM_MODEL=gemini-3.5-flash` (Google Gemini API thật, Native Tool Calling) — lệnh `python src/app.py --all`. Toàn bộ 11 sự kiện trong `docs/trace_waterfall.json` đều có `"model": "gemini-3.5-flash"`, **không có sự kiện nào fallback về Mock** (nếu fallback, trace sẽ ghi `Offline-Mock-Fallback`).

Đoạn trích tiêu biểu từ file `docs/trace_waterfall.json` — **TC04 (Multi-step Reasoning)**: Agent tự tra cứu cố vấn học tập, dùng kết quả Observation làm tham số cho bước đặt lịch, rồi tổng hợp Final Answer:

```json
[
  {
    "step": 1,
    "query": "Mình là sinh viên SV2026002. Mình không nhớ cố vấn học tập của mình là ai, bạn kiểm tra giúp rồi đặt lịch hẹn tư vấn với cố vấn đó vào 09:30 sáng ngày 18/09/2026 nhé.",
    "model": "gemini-3.5-flash",
    "action_type": "TOOL_EXECUTION",
    "thought": "Gemini quyết định gọi công cụ 'academic_query' với tham số: {\"student_id\": \"SV2026002\"}",
    "tool_name": "academic_query",
    "arguments": {
      "student_id": "SV2026002"
    },
    "mcp_request": {
      "jsonrpc": "2.0",
      "server": "vinuni-academic-mcp-server",
      "tool": "academic_query"
    },
    "observation": {
      "status": "SUCCESS",
      "student_id": "SV2026002",
      "data": {
        "full_name": "Trần Thị Bình",
        "class": "AI-K4",
        "gpa": 3.6,
        "email": "binh.tt@vinuni.edu.vn",
        "status": "Đang học",
        "advisor": "TS. Lê Thị B"
      }
    },
    "llm_latency_ms": 2465.39,
    "tool_latency_ms": 0.0,
    "latency_ms": 2465.39
  },
  {
    "step": 2,
    "query": "Mình là sinh viên SV2026002. Mình không nhớ cố vấn học tập của mình là ai, bạn kiểm tra giúp rồi đặt lịch hẹn tư vấn với cố vấn đó vào 09:30 sáng ngày 18/09/2026 nhé.",
    "model": "gemini-3.5-flash",
    "action_type": "TOOL_EXECUTION",
    "thought": "Gemini quyết định gọi công cụ 'schedule_appointment' với tham số: {\"advisor_name\": \"TS. Lê Thị B\", \"student_id\": \"SV2026002\", \"datetime_str\": \"09:30 18/09/2026\"}",
    "tool_name": "schedule_appointment",
    "arguments": {
      "advisor_name": "TS. Lê Thị B",
      "student_id": "SV2026002",
      "datetime_str": "09:30 18/09/2026"
    },
    "mcp_request": {
      "jsonrpc": "2.0",
      "server": "vinuni-academic-mcp-server",
      "tool": "schedule_appointment"
    },
    "observation": {
      "status": "SUCCESS",
      "booking_id": "BK-SV2026002-99",
      "student_id": "SV2026002",
      "datetime": "09:30 18/09/2026",
      "advisor": "TS. Lê Thị B",
      "message": "Đặt lịch thành công cho sinh viên SV2026002 với TS. Lê Thị B vào lúc 09:30 18/09/2026."
    },
    "llm_latency_ms": 2521.26,
    "tool_latency_ms": 0.0,
    "latency_ms": 2521.26
  },
  {
    "step": 3,
    "query": "Mình là sinh viên SV2026002. Mình không nhớ cố vấn học tập của mình là ai, bạn kiểm tra giúp rồi đặt lịch hẹn tư vấn với cố vấn đó vào 09:30 sáng ngày 18/09/2026 nhé.",
    "model": "gemini-3.5-flash",
    "action_type": "FINAL_ANSWER",
    "thought": "Gemini phản hồi trực tiếp bằng văn bản (không cần gọi công cụ).",
    "output": "Chào bạn, \n\nMình đã kiểm tra thông tin và hỗ trợ đặt lịch hẹn cho bạn thành công:\n\n* **Cố vấn học tập của bạn:** TS. Lê Thị B\n* **Thời gian hẹn:** 09:30 ngày 18/09/2026\n* **Mã lịch hẹn:** BK-SV2026002-99\n\nChúc bạn có buổi tư vấn học vụ hiệu quả với cô Lê Thị B! Nếu cần hỗ trợ thêm thông tin gì khác, bạn cứ báo mình nhé.",
    "latency_ms": 3094.68,
    "total_latency_ms": 8083.34
  }
]
```

### Tóm tắt Waterfall theo từng Test Case

| Test Case | Chuỗi ReAct thực tế (Thought → Action → Observation → Final Answer) | Tool calls | Kết quả |
| :--- | :--- | :---: | :---: |
| **TC01** — direct_query | Thought: câu hỏi kiến thức chung → **Final Answer** trực tiếp, nêu nguyên tắc khái quát và khuyến nghị đối chiếu Sổ tay Sinh viên (không bịa số liệu) | 0 | ✅ PASS |
| **TC02** — single_tool_query | `academic_query(SV2026001)` → `SUCCESS` → Final Answer liệt kê đúng hồ sơ Nguyễn Văn An, GPA 3.85 | 1 | ✅ PASS |
| **TC03** — appointment_booking | `academic_query(SV2026001)` xác minh sinh viên → `SUCCESS` → `schedule_appointment(SV2026001, "14:00 15/09/2026", "PGS.TS Nguyễn Văn A")` → `SUCCESS` (BK-SV2026001-99) → Final Answer xác nhận lịch | 2 | ✅ PASS |
| **TC04** — multi_step_reasoning | `academic_query(SV2026002)` → cố vấn `TS. Lê Thị B` → `schedule_appointment(SV2026002, "09:30 18/09/2026", "TS. Lê Thị B")` → `SUCCESS` (BK-SV2026002-99) → Final Answer | 2 | ✅ PASS |
| **TC05** — edge_case_handling | `academic_query(SV9999999)` → `NOT_FOUND` → **dừng, không gọi `schedule_appointment`** → Final Answer lịch sự đề nghị kiểm tra lại mã, không bịa GPA/cố vấn | 1 | ✅ PASS |

> 📝 **Ghi chú quan sát (Observability):** Độ trễ mỗi lượt gọi LLM khoảng 2–5 giây; thời gian thực thi Tool trên MCP Server ≈ 0 ms (dữ liệu mô phỏng in-memory). Riêng bước `FINAL_ANSWER` của TC03 có `latency_ms ≈ 39.5 s` do Gemini Free Tier trả lỗi `429 RESOURCE_EXHAUSTED` (giới hạn request/phút) — Provider tự động retry với exponential backoff (5s → 10s → 20s) rồi thành công, không phải fallback Mock.

---

## 3. TỔNG KẾT KẾT QUẢ NGHIỆM THU & NỘP BÀI

- [x] Đã điền API Key thật trong `.env` và xác nhận Agent chạy mượt mà trên LLM API thật (Gemini/OpenAI).
- **Tổng số Test Cases đã chạy thành công:** 5 / 5 test cases.
- **Số lượt gọi Tool qua MCP Server chính xác:** 6 lượt (`academic_query` × 4, `schedule_appointment` × 2 — tất cả đúng tool, đúng tham số, phản hồi chuẩn JSON-RPC 2.0).
- **Kết quả đẩy Repo nộp bài:** [x] Đã Commit và Push mã nguồn thành công lên GitHub cá nhân.

### Các cải tiến đã thực hiện so với Starter Repo
1. **`src/tools.py` (TODO 1.2):** Khai báo đầy đủ JSON Schema cho `schedule_appointment` (`student_id`, `datetime_str`, `advisor_name` — đều `required`), mô tả rõ định dạng thời gian để LLM trích xuất chính xác.
2. **`src/mcp_server.py` (TODO 2.1):** `call_tool()` gọi `dispatch_tool_call()`, parse JSON và đóng gói phản hồi chuẩn JSON-RPC 2.0 (`jsonrpc`, `server`, `tool`, `result`), có xử lý lỗi parse.
3. **`src/app.py` (ReAct Loop):** Nâng cấp thành vòng lặp ReAct nhiều bước thực sự — Observation của từng bước được nạp lại vào ngữ cảnh (ReAct scratchpad) cho lượt suy luận tiếp theo, dừng khi LLM đưa Final Answer hoặc chạm `MAX_ITERATIONS`. Trace ghi thêm `model`, `thought`, `mcp_request`, `llm_latency_ms`, `tool_latency_ms`, `total_latency_ms`.
4. **`src/providers.py`:** Retry exponential backoff cho lỗi tạm thời (429/503/mất kết nối); đánh dấu rõ `Offline-Mock-Fallback` trong trace nếu buộc phải fallback để đảm bảo tính trung thực của log.
5. **`src/prompts.py`:** Bổ sung quy tắc suy luận đa bước, chống bịa đặt, không đặt lịch khi mã sinh viên `NOT_FOUND`, chuẩn hóa định dạng thời gian.

---

> ✅ **HOÀN TẤT NỘP BÀI:** Sao chép đường link GitHub Repository cá nhân của bạn và dán vào ô nộp bài trên hệ thống LMS VLearn để hoàn tất Bài Lab 3!
