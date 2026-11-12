# Ghi chú phân tích · Day 04

Họ và tên: Lương Thị Hồng Hạnh  
MSSV: 2A202603748

## 1. Case được chọn

- ID case: `E1`
- Câu hỏi: GitHub Actions hiện có sự cố không? Hãy xem cả incident gần đây và dẫn nguồn.
- Điểm lần chạy 1: **100/100** (`raw_judge_score: 100`), trạng thái `scored`.
- Kết quả agent: `completed`. Agent kết luận snapshot GitHub ngày 09/11/2026 lúc 03:50 UTC có overall `none`, Actions `operational`; các incident Actions gần đây được kiểm tra đều `resolved`.

## 2. Bằng chứng trong trace

`get_status` trả overall `All Systems Operational`, component Actions `operational`, cùng năm incident gần nhất. Kết quả cũng ghi `matching_incidents_in_last_50: 23` và cảnh báo chỉ tìm trong 50 incident gần nhất. Agent gọi `get_incident_details` cho bốn incident Actions:

| Incident | Trạng thái trong tool result | URL |
|---|---|---|
| `t89g2q1800fr` | `resolved` | https://stspg.io/lv4q3xm9r0rk |
| `h02n9572mx9y` | `resolved` | https://stspg.io/7pwv28hvkmz8 |
| `xr3myzj7ms51` | `resolved` | https://stspg.io/pjgptg3cfc6n |
| `pfbmpxxfq1cy` | `resolved` | https://stspg.io/s5bs6jn41fwh |

Judge chấm 100 điểm, không có vi phạm mechanical; lý do chấm ghi nhận câu trả lời khớp snapshot, có nguồn và không nhầm incident lịch sử là sự cố đang diễn ra. Trace có 5 tool calls qua 3 bước có tool call; eval ghi nhận 4 model calls.

## 3. Phân tích và cải tiến

- E1 không cho thấy lỗi chức năng cần sửa: trạng thái hiện tại, các incident được tra cứu và kết luận đều nhất quán với tool result.
- Điểm có thể cải thiện là giới hạn phạm vi. Tool trả 5 incident trong kết quả và báo 23 kết quả khớp trong 50 incident gần nhất, nhưng câu trả lời nên nói rõ danh sách chi tiết đã kiểm tra chỉ là các incident được trả về/đọc; không nên để “đều resolved” bị hiểu thành tất cả 23 incident.
- Một số diễn đạt như “nguyên nhân không phải do các incident trên” nên giới hạn thành “các incident đã kiểm tra đều resolved tại snapshot; dữ liệu này không xác định nguyên nhân của lỗi hiện tại”.
- `agent.py` hiện có giới hạn bảo đảm nguồn: khi incident details thành công, agent yêu cầu câu trả lời cuối đưa URL đã xác nhận. Đây là kiểm tra sự hiện diện URL, không xác minh từng claim hoặc tự đảm bảo model nêu giới hạn tìm kiếm.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | `null` (10/22 case được chấm; partial mean 100) | `null` (0/22 case được chấm) |
| Điểm E1 | 100 | `null` |
| Kết quả E1 | `completed`, được chấm | `error`, unscored |
| Model calls / tool calls E1 | 4 / 5 | 1 / 0 |
| Token E1 | 9,499 agent tokens; judge 4,862 tokens | Không có |

- `eval_before.json` dùng OpenRouter model `nvidia/nemotron-3-ultra-550b-a55b:free`; E1 có điểm nhưng 12 case khác không được chấm nên điểm tổng vẫn `null`.
- `eval_after.json` dùng cùng cấu hình; E1 dừng ở bước đầu vì OpenRouter trả HTTP 429. Không thể dùng lần chạy này để kết luận cải tiến làm điểm E1 tăng hay giảm.
- Lỗi agent được thấy trong E1: không có lỗi rõ ràng. Cơ hội cải thiện chủ yếu là diễn đạt cẩn trọng hơn về giới hạn dữ liệu và tránh khẳng định nhân quả ngoài bằng chứng.

## 5. Giới hạn

- Điểm 100 là đánh giá của judge cho một lần chạy; không chứng minh độ ổn định và có thể chịu thiên lệch của model.
- Snapshot được chụp ngày 09/11/2026, không phản ánh trạng thái GitHub hiện tại.
- Tool chỉ tìm trong tối đa 50 incident gần nhất và chỉ trả một phần danh sách; không tìm thấy hoặc không đọc chi tiết một incident không chứng minh incident đó không tồn tại.
- Lần chạy sau bị HTTP 429 nên không có điểm E1 để so sánh trước/sau. Cần chạy lại khi provider hoạt động để có so sánh hợp lệ.
