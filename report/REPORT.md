# Báo Cáo Lab 7: Embedding & Vector Store

**Họ tên:** [Tên sinh viên]
**Nhóm:** [Tên nhóm]
**Ngày:** [Ngày nộp]

---

## 1. Warm-up (5 điểm)

### Cosine Similarity (Ex 1.1)

**High cosine similarity nghĩa là gì?**
> Hai đoạn văn bản có cosine similarity cao khi hai vector embedding của chúng chỉ về gần cùng một hướng trong không gian nhiều chiều. Điều đó nghĩa là chúng mang ý nghĩa/chủ đề gần nhau, bất kể độ dài câu chữ khác nhau.

**Ví dụ HIGH similarity:**
- Sentence A: "Con mèo đang ngủ trên chiếc ghế sofa."
- Sentence B: "Chú mèo nằm nghỉ trên ghế."
- Tại sao tương đồng: Cùng chủ đề (một con mèo đang nghỉ ngơi trên ghế), dùng các từ gần nghĩa nên hai vector gần như cùng hướng.

**Ví dụ LOW similarity:**
- Sentence A: "Lãi suất ngân hàng tăng mạnh trong quý này."
- Sentence B: "Con mèo đang ngủ trên ghế."
- Tại sao khác: Một câu nói về tài chính, một câu nói về vật nuôi — không liên quan về ngữ nghĩa nên hai vector lệch hướng nhau.

**Tại sao cosine similarity được ưu tiên hơn Euclidean distance cho text embeddings?**
> Cosine đo góc (hướng) giữa hai vector nên bất biến với độ dài (magnitude) — văn bản dài hay ngắn không làm sai lệch kết quả. Với text, điều ta quan tâm là "cùng hướng nghĩa" chứ không phải khoảng cách tuyệt đối; Euclidean lại bị ảnh hưởng bởi độ lớn vector nên kém phù hợp.

### Chunking Math (Ex 1.2)

**Document 10,000 ký tự, chunk_size=500, overlap=50. Bao nhiêu chunks?**
> Phép tính: `num_chunks = ceil((doc_length - overlap) / (chunk_size - overlap))`
> `= ceil((10000 - 50) / (500 - 50)) = ceil(9950 / 450) = ceil(22.11) = 23`
> **Đáp án: 23 chunks.** (Đã kiểm chứng khớp với vòng lặp trong `FixedSizeChunker.chunk`, bước nhảy `step = chunk_size - overlap = 450`.)

**Nếu overlap tăng lên 100, chunk count thay đổi thế nào? Tại sao muốn overlap nhiều hơn?**
> `ceil((10000 - 100) / (500 - 100)) = ceil(9900 / 400) = ceil(24.75) = 25 chunks` — tăng từ 23 lên **25**. Lý do: overlap lớn hơn → bước nhảy nhỏ hơn → nhiều chunk hơn. Ta muốn overlap nhiều hơn để giữ ngữ cảnh ở ranh giới chunk, tránh cắt ngang câu/ý làm mất thông tin khi retrieval.

---

## 2. Document Selection — Nhóm (10 điểm)

### Domain & Lý Do Chọn

**Domain:** [ví dụ: Customer support FAQ, Vietnamese law, cooking recipes, ...]

**Tại sao nhóm chọn domain này?**
> *Viết 2-3 câu:*

### Data Inventory

| # | Tên tài liệu | Nguồn | Số ký tự | Metadata đã gán |
|---|--------------|-------|----------|-----------------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

### Metadata Schema

| Trường metadata | Kiểu | Ví dụ giá trị | Tại sao hữu ích cho retrieval? |
|----------------|------|---------------|-------------------------------|
| | | | |
| | | | |

---

## 3. Chunking Strategy — Cá nhân chọn, nhóm so sánh (15 điểm)

### Baseline Analysis

Chạy `ChunkingStrategyComparator().compare()` trên 2-3 tài liệu:

| Tài liệu | Strategy | Chunk Count | Avg Length | Preserves Context? |
|-----------|----------|-------------|------------|-------------------|
| | FixedSizeChunker (`fixed_size`) | | | |
| | SentenceChunker (`by_sentences`) | | | |
| | RecursiveChunker (`recursive`) | | | |

### Strategy Của Tôi

**Loại:** [FixedSizeChunker / SentenceChunker / RecursiveChunker / custom strategy]

**Mô tả cách hoạt động:**
> *Viết 3-4 câu: strategy chunk thế nào? Dựa trên dấu hiệu gì?*

**Tại sao tôi chọn strategy này cho domain nhóm?**
> *Viết 2-3 câu: domain có pattern gì mà strategy khai thác?*

**Code snippet (nếu custom):**
```python
# Paste implementation here
```

### So Sánh: Strategy của tôi vs Baseline

| Tài liệu | Strategy | Chunk Count | Avg Length | Retrieval Quality? |
|-----------|----------|-------------|------------|--------------------|
| | best baseline | | | |
| | **của tôi** | | | |

### So Sánh Với Thành Viên Khác

| Thành viên | Strategy | Retrieval Score (/10) | Điểm mạnh | Điểm yếu |
|-----------|----------|----------------------|-----------|----------|
| Tôi | | | | |
| [Tên] | | | | |
| [Tên] | | | | |

**Strategy nào tốt nhất cho domain này? Tại sao?**
> *Viết 2-3 câu:*

---

## 4. My Approach — Cá nhân (10 điểm)

Giải thích cách tiếp cận của bạn khi implement các phần chính trong package `src`.

### Chunking Functions

**`SentenceChunker.chunk`** — approach:
> Dùng regex `re.split(r'(?<=[.!?])\s+', text.strip())` với lookbehind để tách câu **sau** các dấu kết câu `.` `!` `?` mà vẫn giữ lại dấu. Sau đó strip và bỏ chuỗi rỗng, rồi gộp mỗi `max_sentences_per_chunk` câu thành một chunk (join bằng khoảng trắng). Edge case: text rỗng hoặc toàn khoảng trắng → trả về `[]`.

**`RecursiveChunker.chunk` / `_split`** — approach:
> Đệ quy đi qua danh sách separator theo thứ tự ưu tiên `["\n\n", "\n", ". ", " ", ""]` (lớn → nhỏ). **Base case:** đoạn văn ≤ `chunk_size` thì giữ nguyên; nếu hết separator hoặc gặp separator rỗng `""` thì cắt cứng theo ký tự. Ngược lại split theo separator hiện tại, phần nào vẫn quá dài thì đệ quy với phần separator còn lại — nhờ vậy ưu tiên giữ ranh giới ngữ nghĩa lớn (đoạn, dòng) trước, mới tới ranh giới nhỏ (câu, từ).

### EmbeddingStore

**`add_documents` + `search`** — approach:
> Mỗi `Document` được `_make_record` chuẩn hoá thành dict `{id, content, embedding, metadata}`, trong đó embedding tính bằng `embedding_fn` (mặc định mock md5 đã chuẩn hoá), rồi append vào list in-memory `self._store`. `search` embed query một lần, tính **dot product** giữa query và từng embedding (vì vector đã chuẩn hoá nên dot product = cosine), sort giảm dần theo score và lấy `top_k`, trả về list `{content, score, metadata}`.

**`search_with_filter` + `delete_document`** — approach:
> `search_with_filter` lọc metadata **trước** (pre-filtering) rồi mới chạy similarity search trên tập đã lọc, nên kết quả luôn đúng phòng ban/ngôn ngữ yêu cầu; `metadata_filter=None` thì dùng toàn bộ records. `delete_document` lọc bỏ mọi record có `metadata["doc_id"] == doc_id` và trả `True` nếu số lượng giảm — điều này chạy được nhờ `_make_record` đã chủ động nhét `doc_id` vào metadata khi lưu.

### KnowledgeBaseAgent

**`answer`** — approach:
> Retrieve `top_k` chunk liên quan qua `store.search(question)`, ghép phần `content` của chúng thành một khối context (ngăn cách bằng dòng trống), rồi dựng prompt theo cấu trúc `Context: ... / Question: ... / Answer:` và gọi `llm_fn(prompt)` để sinh câu trả lời (kiểu string). Đây chính là pattern RAG: retrieve → augment prompt → generate.

### Test Results

```
$ python3 -m unittest discover -s tests -v
... (42 tests) ...
----------------------------------------------------------------------
Ran 42 tests in 0.003s

OK
```

> Ghi chú: môi trường máy không cài sẵn `pytest` cho Python 3.14 nên chạy bằng `unittest` (cùng bộ test, vì suite viết trên `unittest`). Khi nộp nên `pip install -r requirements.txt` rồi chạy `pytest tests/ -v` để có output chuẩn.

**Số tests pass:** 42 / 42

---

## 5. Similarity Predictions — Cá nhân (5 điểm)

| Pair | Sentence A | Sentence B | Dự đoán | Actual Score | Đúng? |
|------|-----------|-----------|---------|--------------|-------|
| 1 | | | high / low | | |
| 2 | | | high / low | | |
| 3 | | | high / low | | |
| 4 | | | high / low | | |
| 5 | | | high / low | | |

**Kết quả nào bất ngờ nhất? Điều này nói gì về cách embeddings biểu diễn nghĩa?**
> *Viết 2-3 câu:*

---

## 6. Results — Cá nhân (10 điểm)

Chạy 5 benchmark queries của nhóm trên implementation cá nhân của bạn trong package `src`. **5 queries phải trùng với các thành viên cùng nhóm.**

### Benchmark Queries & Gold Answers (nhóm thống nhất)

| # | Query | Gold Answer |
|---|-------|-------------|
| 1 | | |
| 2 | | |
| 3 | | |
| 4 | | |
| 5 | | |

### Kết Quả Của Tôi

| # | Query | Top-1 Retrieved Chunk (tóm tắt) | Score | Relevant? | Agent Answer (tóm tắt) |
|---|-------|--------------------------------|-------|-----------|------------------------|
| 1 | | | | | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |

**Bao nhiêu queries trả về chunk relevant trong top-3?** __ / 5

---

## 7. What I Learned (5 điểm — Demo)

**Điều hay nhất tôi học được từ thành viên khác trong nhóm:**
> *Viết 2-3 câu:*

**Điều hay nhất tôi học được từ nhóm khác (qua demo):**
> *Viết 2-3 câu:*

**Nếu làm lại, tôi sẽ thay đổi gì trong data strategy?**
> *Viết 2-3 câu:*

---

## Tự Đánh Giá

| Tiêu chí | Loại | Điểm tự đánh giá |
|----------|------|-------------------|
| Warm-up | Cá nhân | / 5 |
| Document selection | Nhóm | / 10 |
| Chunking strategy | Nhóm | / 15 |
| My approach | Cá nhân | / 10 |
| Similarity predictions | Cá nhân | / 5 |
| Results | Cá nhân | / 10 |
| Core implementation (tests) | Cá nhân | 30 / 30 (42/42 tests pass) |
| Demo | Nhóm | / 5 |
| **Tổng** | | **/ 100** |
