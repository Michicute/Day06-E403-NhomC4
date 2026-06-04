# Trợ lý y tế RAG

Prototype mô phỏng một website nhà thuốc số có trợ lý AI dạng chat popup. Người dùng có thể hỏi về thuốc, triệu chứng, bệnh thường gặp, tác dụng phụ, thành phần thuốc, đồng thời yêu cầu tìm nhà thuốc, bệnh viện hoặc phòng khám gần vị trí hiện tại.

## Tính năng chính

- Giao diện Streamlit mô phỏng trang bán thuốc với danh mục, sản phẩm mẫu, khuyến mãi và góc sức khỏe.
- Chatbot RAG truy xuất dữ liệu từ hai file local: medical Q&A và catalog thuốc.
- Trả lời bằng TF-IDF retrieval khi không có API key.
- Có thể dùng OpenAI để sinh câu trả lời tự nhiên hơn nếu cấu hình `OPENAI_API_KEY`.
- Hiển thị nguồn tham khảo đã retrieve kèm điểm tương đồng và metadata.
- Hỗ trợ ngữ cảnh hội thoại cho các câu hỏi tiếp nối như "thuốc này dùng được không?".
- Tìm cơ sở y tế gần người dùng bằng vị trí trình duyệt/IP và OpenStreetMap Overpass API.

## Cấu trúc source code

```text
codebase/
├── app.py                 # Giao diện Streamlit và flow chat/location
├── src/rag_agent.py       # RAG agent: load data, retrieve, trả lời CLI/LLM
├── data_clean.csv         # Bộ dữ liệu medical Q&A đã làm sạch
├── medicine_clean.csv     # Bộ dữ liệu thuốc đã làm sạch
├── Medicine_Details.csv   # Dữ liệu thuốc gốc
├── train.csv              # Dữ liệu Q&A gốc
├── eda_outputs/           # Biểu đồ và thống kê EDA
├── requirements.txt       # Thư viện cần cài
├── RAG_AGENT.md           # Ghi chú kỹ thuật riêng cho RAG agent
└── .streamlit/config.toml # Theme Streamlit
```

## Cài đặt

Yêu cầu Python 3.10+.

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Cấu hình môi trường

Ứng dụng chạy được không cần API key, khi đó chatbot trả về câu trả lời dạng extractive từ dữ liệu local.

Nếu muốn dùng OpenAI để sinh câu trả lời tự nhiên hơn, tạo file `.env`:

```bash
OPENAI_API_KEY=your_api_key_here
OPENAI_MODEL=gpt-4o-mini
```

Lưu ý: không commit `.env` lên repo.

## Chạy giao diện demo

```bash
streamlit run app.py
```

Sau khi chạy, mở URL Streamlit hiển thị trong terminal. Nút "Trợ lý" nằm ở góc phải màn hình, dùng để mở chatbot.

## Chạy RAG agent bằng CLI

Không dùng LLM:

```bash
python -m src.rag_agent --no-llm "What are the side effects of Augmentin 625 Duo Tablet?"
```

Dùng OpenAI, yêu cầu có `OPENAI_API_KEY`:

```bash
python -m src.rag_agent --llm "What are the treatments for breast cancer?"
```

## Cách hoạt động

1. `src/rag_agent.py` đọc `data_clean.csv` và `medicine_clean.csv`.
2. Mỗi dòng dữ liệu được chuyển thành một document có `source`, `title`, `text` và `metadata`.
3. Agent dùng `TfidfVectorizer` của scikit-learn để tạo index truy xuất local.
4. Khi user hỏi, agent lấy top-k document liên quan nhất bằng cosine similarity.
5. Nếu có OpenAI, app gửi câu hỏi, lịch sử hội thoại và context đã retrieve vào model.
6. Nếu không có OpenAI, app trả về các record liên quan nhất để người dùng tự kiểm tra nguồn.

## Công cụ và API đã dùng

- Streamlit: xây giao diện web prototype.
- streamlit-geolocation: xin vị trí từ trình duyệt.
- pandas: đọc và xử lý CSV.
- scikit-learn: TF-IDF vectorizer và cosine similarity.
- OpenAI API: sinh câu trả lời RAG khi có API key.
- python-dotenv: đọc biến môi trường từ `.env`.
- OpenStreetMap Overpass API: tìm nhà thuốc, bệnh viện, phòng khám gần vị trí người dùng.
- ipapi.co / ip-api.com: lấy vị trí gần đúng theo IP khi không dùng định vị trình duyệt.

## Dữ liệu

- `data_clean.csv`: câu hỏi/đáp y tế theo loại câu hỏi như symptoms, treatments, exams and tests, susceptibility.
- `medicine_clean.csv`: tên thuốc, thành phần, công dụng, tác dụng phụ, nhà sản xuất và tỷ lệ review.
- `eda_outputs/`: các biểu đồ và bảng thống kê phục vụ phân tích dữ liệu.

## Lưu ý an toàn

Đây là prototype học tập, không thay thế bác sĩ, dược sĩ hoặc tư vấn y tế chuyên môn. Với triệu chứng nặng, tình huống khẩn cấp, quyết định dùng thuốc, liều dùng hoặc điều trị cá nhân, người dùng cần liên hệ cơ sở y tế hoặc chuyên gia y tế.

## Phân công công việc

| Thành viên | Phụ trách |
|---|---|
| Phung Kim Khang | SPEC/Product canvas |
| Nguyen Dinh Minh Chi | Xử lý dữ liệu và RAG agent |
| Nguyen Duy Hung | Giao diện Streamlit |
| Tran Duc Manh | Kiểm thử prompt, demo và repo |
| Le Van Khiem | Kiểm thử prompt, demo và repo |
