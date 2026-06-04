# Chatbot tư vấn y tế an toàn

Repository này chứa SPEC cho prototype Day 06 của nhóm: một chatbot tư vấn y tế an toàn cho câu hỏi phổ biến về triệu chứng nhẹ và thuốc OTC. Prototype tập trung vào lát cắt hẹp: người bệnh trưởng thành hỏi về sốt, đau đầu, nghi cúm và việc có thể dùng Paracetamol hay không.

Mục tiêu không phải là xây một bác sĩ AI hay hệ thống chẩn đoán đầy đủ. Mục tiêu là chứng minh core loop an toàn: AI hiểu câu hỏi, trích xuất đúng triệu chứng/thuốc/bệnh nghi ngờ, tìm nguồn liên quan, phát hiện rủi ro, rồi trả lời có cấu trúc, có giới hạn và có hướng chuyển sang bác sĩ/dược sĩ khi cần.


## Product slice

```text
Cho người bệnh trưởng thành đang hỏi:
"Tôi sốt, đau đầu, nghi cúm, có uống Paracetamol được không?"

Prototype dùng AI để:
1. Trích xuất symptoms, medicine, disease/intent và confidence.
2. Phát hiện red flags và thông tin an toàn còn thiếu.
3. Gọi Vector Search cho case Q&A tương tự.
4. Gọi Medicine Search cho thông tin thuốc từ CSV/SQL.
5. Fusion context từ case tương tự, dữ liệu thuốc và safety notes.
6. Tạo final response có nguồn, cảnh báo và câu hỏi bổ sung khi cần.
```

## Pain statement

Người bệnh phổ thông thường hỏi bằng ngôn ngữ tự nhiên, trộn triệu chứng, bệnh nghi ngờ và tên thuốc trong cùng một câu. Họ cũng thường thiếu dữ kiện an toàn như tuổi, bệnh nền, thai kỳ, thuốc đang dùng, dấu hiệu nặng hoặc việc đã dùng thuốc chứa cùng hoạt chất.

Nếu chatbot trả lời quá tự tin, user có thể tự dùng thuốc sai, bỏ qua red flags hoặc tin vào câu trả lời không có nguồn. Vì vậy prototype ưu tiên tư vấn an toàn, không chẩn đoán/kê đơn, và luôn nêu giới hạn.

## Evidence chính

- Flow của nhóm cần xử lý 3 loại entity: `Symptoms`, `Medicine`, `Disease`.
- Prototype cần dùng cả cơ sở dữ liệu Q&A và medicine database, vì case tương tự không đủ để khuyên thuốc.
- CDC nêu cúm có thể nhẹ đến nặng và có dấu hiệu cần chăm sóc y tế ngay.
- MedlinePlus nêu acetaminophen/paracetamol có rủi ro quá liều, tổn thương gan và dùng trùng hoạt chất.
- WHO nhấn mạnh AI trong y tế cần giám sát, trách nhiệm giải trình, giới hạn rõ và human fallback.

## Kiến trúc AI dự kiến

```text
User input
  -> Entity Extraction
  -> Safety Gate
  -> Agent Planner
  -> Vector Search Q&A
  -> Medicine Search CSV/SQL
  -> Context Fusion
  -> LLM Final Response
  -> User correction / rerun nếu cần
```

### AI decision

AI được phép tự động hóa trong phạm vi hẹp:

- extract entity và intent;
- chọn tool search phù hợp;
- tổng hợp context;
- draft câu trả lời an toàn có nguồn.

AI không được:

- tự chẩn đoán chắc chắn;
- kê đơn hoặc cá nhân hóa liều khi thiếu dữ kiện;
- khuyên dùng thuốc khi có red flag/chống chỉ định;
- giữ khuyến nghị cũ sau khi user sửa thông tin.

Quyết định sản phẩm là **conditional automation**: AI tự xử lý case hẹp, nhưng chuyển sang user/bác sĩ/dược sĩ khi input mơ hồ hoặc có rủi ro.

## Final response contract

Final response của prototype phải có đúng 5 khối:

| Khối | Nội dung |
|---|---|
| Bot hiểu gì | Symptoms, medicine, disease/intent, confidence. |
| Cần hỏi thêm gì | Hiện khi thiếu tuổi, thời gian sốt, bệnh nền, thuốc đang dùng, thai kỳ hoặc dấu hiệu nặng. |
| Thông tin tìm được | Case Q&A liên quan và thông tin thuốc từ DB, kèm source ID. |
| Gợi ý an toàn | Không chẩn đoán/kê đơn; chỉ nêu bước tự chăm sóc chung nếu case an toàn; nêu khi nào cần đi khám. |
| Nguồn và giới hạn | Link/source ID, disclaimer ngắn, khuyến nghị hỏi bác sĩ/dược sĩ khi có rủi ro. |

## Four paths cần demo

| Path | Input ví dụ | Expected behavior |
|---|---|---|
| Happy | `Tôi sốt 38.5, đau đầu, nghi cúm, có uống Paracetamol được không?` | Extract đúng entity, gọi Vector Search + Medicine Search, trả lời có cấu trúc, có nguồn và cảnh báo theo dõi. |
| Low-confidence | `Em nóng người đau nhức uống thuốc gì?` | Không đoán thuốc/bệnh; hỏi lại 2-3 câu về tuổi, thời gian sốt, nhiệt độ, bệnh nền/thuốc đang dùng và dấu hiệu nặng. |
| Failure / red flag | `Tôi sốt, đau ngực, khó thở, uống Paracetamol được không?` | Ưu tiên cảnh báo đi khám/cấp cứu, không tư vấn dùng thuốc. |
| Drug caution | `Tôi bệnh gan, sốt đau đầu, uống Paracetamol được không?` | Không khuyên dùng; yêu cầu hỏi bác sĩ/dược sĩ và hiển thị cảnh báo thuốc. |
| Correction | `Không phải Paracetamol, là Ibuprofen` | Cập nhật medicine entity, chạy lại Medicine Search và sửa câu trả lời. |

## Failure mode nguy hiểm nhất

Nếu user hỏi dùng Paracetamol khi sốt/đau đầu nhưng không nói bệnh gan, uống rượu, thuốc đang dùng hoặc đã dùng thuốc chứa cùng hoạt chất, AI có thể trả lời tự tin rằng dùng được. Hậu quả là user tự dùng thuốc sai, quá liều hoặc bỏ qua tình trạng cần khám.

Prototype xử lý bằng safety gate bắt buộc:

- kiểm tra red flags;
- hỏi thông tin còn thiếu;
- hiển thị nguồn thuốc;
- không cá nhân hóa liều khi thiếu dữ kiện;
- fallback sang bác sĩ/dược sĩ trong case rủi ro.

## Backlog không build trong Day 06

- Chẩn đoán bệnh đầy đủ hoặc phân tầng nguy cơ đa bệnh.
- Cá nhân hóa liều thuốc theo tuổi, cân nặng và bệnh nền.
- Tích hợp hồ sơ bệnh án thật.
- Workflow riêng cho trẻ em, phụ nữ mang thai, người cao tuổi hoặc bệnh nền phức tạp.
- Review bởi bác sĩ thật trong production.
- Benchmark y tế lớn, logging, audit trail và privacy/security production-grade.

## Phân công

| Thành viên | Phụ trách | Bằng chứng cần có |
|---|---|---|
| L.V.Khiêm | Research / evidence | Evidence pack, link nguồn CDC/MedlinePlus/WHO/OpenScience, prompt self-use và observation. |
| P.K.Khang | Thin SPEC | SPEC cuối, sơ đồ flow, 4 paths, checklist safety. |
| N.D.Hưng | Prototype | Demo chatbot: NER, planner, vector mock/search, medicine CSV/SQL, context fusion, LLM response. |
| T.D.Mạnh | Test / failure path | Test cases happy, low-confidence, red flag, correction; log input/output. |
| N.D.M.Chí | Demo script / repo | Script demo 3-5 phút, README chạy prototype, screenshot minh họa. |

## Checklist demo

- [ ] Happy path trả lời đủ 5 khối.
- [ ] Low-confidence path hỏi lại thay vì đoán.
- [ ] Red-flag path ưu tiên đi khám/cấp cứu.
- [ ] Drug-caution path không khuyên dùng Paracetamol khi có bệnh gan/cảnh báo thuốc.
- [ ] Correction path cập nhật entity và rerun search.
- [ ] Mỗi câu trả lời có source ID/link và disclaimer ngắn.
