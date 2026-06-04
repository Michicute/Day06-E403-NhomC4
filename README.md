# Day06-E403-NhomC4

Repository bài nộp Day 06 - AI Product Hackathon của nhóm C4, lớp E403.

Sản phẩm prototype là **Trợ lý y tế RAG**: một website mô phỏng nhà thuốc số có chatbot AI hỗ trợ hỏi đáp về triệu chứng nhẹ, thuốc OTC, tác dụng phụ, thành phần thuốc và gợi ý tìm cơ sở y tế gần người dùng.

> Lưu ý: Đây là prototype học tập, không thay thế bác sĩ, dược sĩ hoặc tư vấn y tế chuyên môn.

## Thành viên và phân công

| Mã học viên | Thành viên | Phụ trách | Bằng chứng cần có |
|---|---|---|---|
| 2A202600542 | L.V.Khiêm | Research / evidence | Evidence pack, link nguồn CDC/MedlinePlus/WHO/OpenScience, prompt self-use và observation. |
| 2A202600882 | P.K.Khang | Thin SPEC | SPEC cuối, sơ đồ flow, demo paths, checklist safety. |
| 2A202600820 | N.D.M.Chí | Prototype | Demo chatbot: NER, planner, vector mock/search, medicine CSV/SQL, context fusion, LLM response. |
| 2A202600734 | T.D.Mạnh | Test / failure path | Test cases happy, low-confidence, red flag, correction; log input/output. |
| 2A202600578 | N.D.Hưng | Demo script / repo | Script demo 3-5 phút, README chạy prototype, screenshot minh họa. |

## Product slice

Người dùng mục tiêu là người trưởng thành hỏi bằng ngôn ngữ tự nhiên về các triệu chứng phổ biến và thuốc thông dụng, ví dụ:

```text
Tôi sốt, đau đầu, nghi cúm, có uống Paracetamol được không?
```

Prototype dùng AI/RAG để:

1. Trích xuất triệu chứng, thuốc, bệnh/ý định hỏi và mức độ tự tin.
2. Phát hiện dấu hiệu rủi ro hoặc thông tin an toàn còn thiếu.
3. Truy xuất Q&A y tế liên quan từ dữ liệu local.
4. Truy xuất thông tin thuốc từ medicine database.
5. Tổng hợp context và tạo câu trả lời có cấu trúc, có nguồn, có giới hạn an toàn.

## Tính năng chính

- Giao diện Streamlit mô phỏng website nhà thuốc số.
- Chatbot popup hỏi đáp về thuốc, triệu chứng, bệnh thường gặp, công dụng và tác dụng phụ.
- RAG local bằng TF-IDF trên hai nguồn dữ liệu: medical Q&A và catalog thuốc.
- Có thể chạy không cần API key; nếu có `OPENAI_API_KEY`, chatbot sinh câu trả lời tự nhiên hơn.
- Hiển thị nguồn tham khảo đã retrieve kèm metadata/điểm liên quan.
- Hỗ trợ ngữ cảnh hội thoại cho câu hỏi tiếp nối.
- Tìm nhà thuốc, bệnh viện hoặc phòng khám gần người dùng qua vị trí trình duyệt/IP và OpenStreetMap Overpass API.

## Cấu trúc repo

```text
Day06-E403-NhomC4/
├── README.md              # README chính: mô tả sản phẩm, thành viên, phân công, cách chạy
├── hackathon-rules.md     # Luật hackathon và cách chấm
├── spec/                  # SPEC sản phẩm và các demo paths
└── codebase/              # Source code prototype Streamlit/RAG
```

Chi tiết source code nằm trong [`codebase/README.md`](codebase/README.md). SPEC đầy đủ nằm trong [`spec/README.md`](spec/README.md).

## Cài đặt và chạy demo

Yêu cầu Python 3.10+.

```bash
cd codebase
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
streamlit run app.py
```

Sau khi chạy, mở URL Streamlit hiển thị trong terminal. Nút **Trợ lý** nằm ở góc phải màn hình để mở chatbot.

Ứng dụng chạy được không cần API key. Nếu muốn dùng OpenAI, tạo file `codebase/.env`:

```bash
OPENAI_API_KEY=your_api_key_here
OPENAI_MODEL=gpt-4o-mini
```

Không commit file `.env` lên repo.

## Dữ liệu và công nghệ

| Hạng mục | Nội dung |
|---|---|
| Framework UI | Streamlit |
| Retrieval | scikit-learn TF-IDF + cosine similarity |
| LLM optional | OpenAI API |
| Data processing | pandas |
| Location search | streamlit-geolocation, OpenStreetMap Overpass API, IP geolocation fallback |
| Q&A data | `codebase/data_clean.csv`, từ `train.csv` đã làm sạch |
| Medicine data | `codebase/medicine_clean.csv`, từ `Medicine_Details.csv` đã làm sạch |
| EDA outputs | `codebase/eda_outputs/` |

## Luồng AI

```text
User input
  -> Entity Extraction / intent understanding
  -> Safety Gate
  -> RAG Retriever
  -> Medicine Search
  -> Context Fusion
  -> Final Response
  -> User correction / rerun nếu cần
```

AI được dùng để hiểu câu hỏi, chọn nguồn dữ liệu liên quan, tổng hợp context và draft câu trả lời. AI không được chẩn đoán chắc chắn, kê đơn, cá nhân hóa liều thuốc khi thiếu dữ kiện, hoặc bỏ qua dấu hiệu nguy hiểm.

## Demo paths

| Path | Input ví dụ | Expected behavior |
|---|---|---|
| Happy | `Tôi sốt 38.5, đau đầu, nghi cúm, có uống Paracetamol được không?` | Extract đúng entity, retrieve Q&A + medicine, trả lời có cấu trúc, có nguồn và cảnh báo theo dõi. |
| Low-confidence | `Em nóng người đau nhức uống thuốc gì?` | Không đoán thuốc/bệnh; hỏi lại 2-3 câu về tuổi, nhiệt độ, thời gian sốt, bệnh nền/thuốc đang dùng và dấu hiệu nặng. |
| Red flag | `Tôi sốt, đau ngực, khó thở, uống Paracetamol được không?` | Ưu tiên cảnh báo đi khám/cấp cứu, không tư vấn dùng thuốc. |
| Drug caution | `Tôi bệnh gan, sốt đau đầu, uống Paracetamol được không?` | Không khuyên dùng; yêu cầu hỏi bác sĩ/dược sĩ và hiển thị cảnh báo thuốc. |
| Correction | `Không phải Paracetamol, là Ibuprofen` | Cập nhật medicine entity, chạy lại Medicine Search và sửa câu trả lời. |

## Tài liệu liên quan

| Folder / file | Nội dung |
|---|---|
| [`spec/README.md`](spec/README.md) | SPEC sản phẩm, product slice, safety gate, demo paths, phân công chi tiết |
| [`codebase/README.md`](codebase/README.md) | Cách chạy prototype, cấu trúc source code, công cụ/API đã dùng |
| [`codebase/RAG_AGENT.md`](codebase/RAG_AGENT.md) | Ghi chú kỹ thuật riêng cho RAG agent |
| [`hackathon-rules.md`](hackathon-rules.md) | Luật chơi, lịch demo và cách chấm |

## Checklist nộp bài

- [x] Có README chính liệt kê thành viên, mã học viên và phân công.
- [x] Có SPEC trong `spec/`.
- [x] Có prototype chạy được trong `codebase/`.
- [x] Có ít nhất một flow AI/RAG chạy thật.
- [x] Có mô tả cách cài đặt và chạy demo.
- [ ] Điền đầy đủ mã học viên vào bảng phân công.
