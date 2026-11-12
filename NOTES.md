# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Nam Khánh
MSSV: 2A202603871

## 1. Case được chọn

- ID case: H4 
- Câu trả lời của agent: "answer": "Biển ru giấc mây trôi,  \nSóng khâu ánh bạc vào trời mênh mang.  \nThuyền về nghiêng bóng chiều vàng,  \nCát nghe vỏ ốc kể ngàn khơi xa."
- Điểm ở lần chạy 1: 50

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text
        "trace": [
          {
            "event": "model",
            "step": 1,
            "finish": "tool_calls"
          },
          {
            "event": "tool",
            "step": 1,
            "name": "get_status",
            "arguments": {
              "provider": "github"
            },
            "result": {
              "provider": "github",
              "captured_at": "2026-11-09T03:50:47.065020+00:00",
              "overall": {
                "indicator": "none",
                "description": "All Systems Operational"
              },
              "components": [],
              "incidents": [
                {
                  "id": "pyynvrm1vskw",
                  "name": "Increase in timeouts for some unauthenticated users",
                  "status": "resolved",
                  "updated_at": "2026-11-08T12:32:35.646Z"
                },
                {
                  "id": "pfbmpxxfq1cy",
                  "name": "Degraded performance for multiple GitHub services",
                  "status": "resolved",
                  "updated_at": "2026-11-06T00:20:49.369Z"
                },
                {
                  "id": "t89g2q1800fr",
                  "name": "Incident with Actions",
                  "status": "resolved",
                  "updated_at": "2026-11-05T17:42:35.455Z"
                },
                {
                  "id": "h02n9572mx9y",
                  "name": "Incident with Actions",
                  "status": "resolved",
                  "updated_at": "2026-11-07T01:10:18.506Z"
                },
                {
                  "id": "xr3myzj7ms51",
                  "name": "Incident with Actions and Pages",
                  "status": "resolved",
                  "updated_at": "2026-11-07T01:04:38.647Z"
                }
              ],
              "matching_incidents_in_last_50": 50,
              "note": "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố.",
              "source": "https://www.githubstatus.com"
            }
          },
          {
            "event": "model",
            "step": 2,
            "finish": "stop"
          }
        ],

```

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Thêm dòng "Chỉ gọi tools khi yêu cầu của người dùng cần dữ liệu trạng thái hoặc incident, với các yêu cầu
ngoài chủ đề, trả lời trưc tiếp nếu ngoài phạm vi. Không gọi tools, không đưa ra khẳng định trạng thái
khi chưa được xác minh ở system prompt"
- Vì sao chọn thay đổi này: Mặc dù prompt của người dùng yêu cầu không gọi tools, agent vẫn có 1 bước gọi tool để kiểm tra
trạng thái, có thể agent cần chỉ dẫn cụ thể hơn rằng những yêu cầu không liên quan thì không phải sử dụng đến việc gọi tool
dù việc gọi tool có thể được nhắc đến.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 97.05/100 | 94.77/100 |
| Điểm case đã chọn | H4: 50/100 | H4: 100/100 |
| Số model calls | 68 (agent, toàn bộ 22 case; không gồm judge) | 67 (agent, toàn bộ 22 case; không gồm judge) |
| Token, chi phí nếu có | Agent: 79,673 token (71,414 input + 8,259 output); không gồm judge. File không ghi chi phí. | Agent: 81,659 token (74,601 input + 7,058 output); không gồm judge. File không ghi chi phí. |

- Lỗi đã giải quyết: Trong lần chạy sau, H4 trả lời yêu cầu viết thơ trực tiếp, không gọi status tools trái yêu cầu của người dùng. Số tool calls giảm từ 1 xuống 0; điểm tăng từ 50 lên 100.
- Lỗi còn lại hoặc mới xuất hiện: A6 giảm từ 100 xuống 0 vì agent hết 8 bước và trả `max_steps`, không có câu trả lời cuối. Sau khi đọc incident ban đầu, agent gọi `get_incident_details` cho cùng incident liên quan `xr3myzj7ms51` ở các bước 2–8 với cùng arguments và nhận lại dữ liệu bị cắt (`truncated: true`), thay vì tổng hợp bằng chứng hiện có và nêu giới hạn. Ngoài ra, E5 chưa nêu rõ giới hạn tìm kiếm khi nói “gần đây nhất”; A1 còn bỏ sót một incident Actions. Điểm tổng giảm 2.28 điểm dù H4 cải thiện. Một lần chạy mỗi phiên bản chưa đủ để kết luận thay đổi prompt gây ra lỗi A6.

## 5. Giới hạn

- Điểm judge là ước lượng; agent và judge cùng dùng `gpt-6-luna` nên có thể thiên lệch. `completed` không đảm bảo câu trả lời đúng.
- Snapshot không phản ánh realtime. Tool giới hạn số incident và nội dung update; tìm không thấy không chứng minh không có sự cố, và `truncated` nghĩa là chưa có toàn bộ nội dung.
- Mỗi phiên bản chỉ chạy một lần, chưa chứng minh độ ổn định hay quan hệ nhân quả giữa thay đổi prompt và điểm số. H4 cải thiện nhưng A6 hết bước; không phải mọi case đều tốt hơn.
- Validation chỉ yêu cầu ít nhất một URL incident đã quan sát, không xác minh mọi claim. Tests offline không thay thế đánh giá model thật; thống kê token mục 4 chưa gồm judge và chưa có chi phí thực tế.
- `max_steps` chỉ giới hạn model calls, không giới hạn riêng từng tool; A6 cho thấy agent vẫn có thể gọi lặp đến hết budget mà chưa trả lời.
