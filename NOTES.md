# Ghi chú phân tích · Day 04
- Họ và tên : Tô Minh Thành  
- MSSV : 2A202603439

## 1. Case được chọn

- ID case: A5
- Câu trả lời lần 1: Agent nói không có incident GitHub Actions nào trong năm 2025, dù dữ liệu chỉ gồm các incident gần đây từ snapshot.
- Điểm lần 1: 40/100.

## 2. Bằng chứng trong trace

Trace lần 1 cho thấy `get_status` tìm kiếm trong tối đa 50 incident gần nhất và trả ghi chú rằng không tìm thấy không chứng minh là không có sự cố. Agent còn gọi `get_incident_details` cho các incident gần đây, nhưng những dữ liệu đó không chứng minh danh sách incident của năm 2025 là đầy đủ. Câu trả lời vẫn đưa ra kết luận phủ định cho cả năm.

```json
{"event":"tool","step":1,"name":"get_status","arguments":{"provider":"github","component":"GitHub Actions"},"result":{"matching_incidents_in_last_50":11,"note":"Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố."}}
```

## 3. Thay đổi đã thực hiện

- Trong `agent.py`, bổ sung chỉ dẫn rõ: status tool chỉ tìm trong 50 incident gần nhất; không suy ra không có incident cũ hơn hoặc lịch sử theo năm là đầy đủ; nói rõ giới hạn nếu câu hỏi vượt dữ liệu.
- Thêm test trong `tests/test_lab.py` để bảo đảm system prompt luôn nêu giới hạn này.
- Khi kiểm tra agent, phát hiện và sửa lỗi liên quan trong `agent.py`: giữ nguyên assistant message có `tool_calls` khi gửi tool result về model. Nếu không giữ message, lượt gọi kế tiếp bị OpenAI từ chối.
- Tests offline: 38 tests passed.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---:|---:|
| Điểm tổng | 79.32/100 | 79.09/100 |
| Điểm case A5 | 40/100 | 90/100 |
| Model calls của A5 | 5 | 2 |
| Tool calls của A5 | 6 | 1 |
| Agent calls / tokens toàn bộ | 53 / 53,095 | 50 / 43,377 |
| Judge calls / tokens toàn bộ | 22 / 44,666 | 22 / 36,377 |
| Tổng tokens ghi nhận | 97,761 | 79,754 |
| Chi phí | Không có dữ liệu giá trong báo cáo | Không có dữ liệu giá trong báo cáo |

- Lỗi đã cải thiện: A5 lần 2 nói rõ không thể cung cấp danh sách đầy đủ cho 2025 vì giới hạn 50 incident; điểm tăng 50 điểm và số model calls giảm.
- Lỗi còn lại/biến động: điểm trung bình toàn bộ giảm nhẹ 0.23 điểm. Các case H1/H6 còn yếu về việc theo incident liên quan và nêu nguyên nhân; một số case khác cũng dao động giữa hai lần chạy. A5 vẫn liệt kê incident gần đây sau khi nêu giới hạn, nên cần đọc chúng như ví dụ từ dữ liệu gần đây, không phải danh sách năm 2025.
- Hai lần chạy dùng cùng cấu hình: provider/judge OpenAI, cùng model, dataset core và `max_steps=8`. Mỗi phiên bản chỉ chạy một lần nên chênh lệch tổng thể không chứng minh xu hướng ổn định.

## 5. Giới hạn

- Điểm LLM judge là ước lượng, có thể thiên lệch hoặc không nhất quán; cần đối chiếu expected, trace và evidence.
- Snapshot là dữ liệu cố định và chỉ bao phủ tối đa 50 incident gần nhất; không đại diện cho status live hay lịch sử đầy đủ.
- Một lần chạy cho mỗi phiên bản không chứng minh cải tiến ổn định. Token usage được ghi lại nhưng không có bảng giá để suy ra chi phí.
