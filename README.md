# RAG Pipeline for Bangla Question Answering with PDF Upload

This project implements a Retrieval-Augmented Generation (RAG) pipeline to answer Bangla questions based on a PDF document, specifically targeting educational question papers like "HSC26-Bangla1st-Paper.pdf". It extracts text from PDFs using OCR, chunks text for semantic retrieval, embeds chunks with a Bangla-specific model, and generates answers using a language model. The pipeline is exposed via a FastAPI server with ngrok for public access, supporting PDF uploads and query processing.

## Repository
Source code: https://github.com/Talha1000/Technical-Assessment

## Features
- PDF Text Extraction: Uses PyMuPDF for text-based PDFs and Tesseract OCR for scanned pages.
- Text Chunking: Splits text into question-answer pairs or sentences for efficient retrieval.
- Semantic Retrieval: Employs l3cube-pune/bengali-sentence-similarity-sbert for embeddings and FAISS for similarity search.
- Answer Generation: Generates answers with csebuetnlp/banglat5, ensuring context grounding.
- API Endpoints: FastAPI server with /upload-pdf for PDF processing and /ask for querying.
- Public Access: Ngrok provides a public URL for the API.
- Test Cases: Validates against specific Bangla queries for accuracy.

## Setup Guide
1. **Clone the Repository**:
   ```bash
   git clone https://github.com/your-username/rag-bangla-qa
   cd rag-bangla-qa
2. Install Dependencies: Install Python packages: pip install pymupdf pytesseract sentence-transformers faiss-cpu transformers pillow fastapi uvicorn pyngrok
Install Tesseract from UB-Mannheim/tesseract, add to PATH, and download ben.traineddata to the Tesseract data directory (e.g., C:\Program Files\Tesseract-OCR\tessdata).
3. Configure Ngrok:
Sign up at ngrok.com and obtain an authtoken.
Set the authtoken in your environment:
export NGROK_AUTH_TOKEN="ngrok_authtoken"
echo 'export NGROK_AUTH_TOKEN="your_ngrok_authtoken"' >> ~/.bashrc
source ~/.bashrc
Alternatively, replace the hardcoded authtoken in rag_pipeline.py with your own (not recommended for public repos).
4. Prepare the PDF:
Place HSC26-Bangla1st-Paper.pdf in the project directory or upload it via the API.
5. Run the Application:
API Mode: python rag_pipeline.py
Starts FastAPI on http://localhost:8001 and logs a public ngrok URL (e.g., https://<ngrok-id>.ngrok-free.app).</ngrok-id>
Colab Mode:
Open rag_pipeline.ipynb in Google Colab.
Run all cells, uploading HSC26-Bangla1st-Paper.pdf when prompted.
Test cases run automatically, with extracted text saved to extracted_text.txt.
Tools, Libraries, and Packages
Python: 3.7+
PyMuPDF: Extracts text from PDFs.
pytesseract: Performs OCR for scanned PDFs with Bangla support.
sentence-transformers: Generates embeddings (l3cube-pune/bengali-sentence-similarity-sbert).
faiss-cpu: Enables fast vector similarity search.
transformers: Powers answer generation (csebuetnlp/banglat5).
pillow: Preprocesses images for OCR.
fastapi: Builds the API server.
uvicorn: Runs the FastAPI server.
pyngrok: Tunnels the server for public access.
re, unicodedata, numpy: Supports text cleaning and processing.
Sample Queries and Outputs
The pipeline is tested with specific queries. Below are the queries, expected answers, and sample outputs in Bangla and English:

Query (Bangla): অনুপমের ভাষায় সুপুরুষ কাকে বলা হয়েছে?
Query (English): Who is referred to as a "handsome man" in Anupam's words?
Expected Answer: শুম্ভুনাথ (Shumbhunath)
Sample Output:
Query: অনুপমের ভাষায় সুপুরুষ কাকে বলা হয়েছে?
Answer: ঘ শুম্ভুনাথ
Context: ৩৫। অনুপমের ভাষায় সুপুরুষ কাকে বলা হয়েছে? ক র্ব্নুদাদা খ অনুপম গ মামা ঘ শুম্ভুনাথ...
Grounded: True

Query (Bangla): কাকে অনুপমের ভাগ্য দেবতা বলে উল্লেখ করা হয়েছে?
Query (English): Who is referred to as Anupam's "god of fortune"?
Expected Answer: মামাকে (Uncle)
Sample Output:
Query: কাকে অনুপমের ভাগ্য দেবতা বলে উল্লেখ করা হয়েছে?
Answer: গ মামাকে
Context: [Relevant chunk from PDF]...
Grounded: True

Query (Bangla): বিয়ের সময় কল্যাণীর প্রকৃত বয়স কত ছিল?
Query (English): What was Kalyani's actual age at the time of her marriage?
Expected Answer: ১৫ বছর (15 years)
Sample Output:
Query: বিয়ের সময় কল্যাণীর প্রকৃত বয়স কত ছিল?
Answer: ১৫ বছর
Context: [Relevant chunk from PDF]...
Grounded: True

API Documentation
The FastAPI server provides two endpoints:

1. Upload PDF (POST /upload-pdf)
Description: Uploads a PDF file to process its text for RAG.
Parameters:
file: PDF file (multipart/form-data).
Example:
curl -X POST -F "file=@HSC26-Bangla1st-Paper.pdf" https://<ngrok-url>/upload-pdf
Success Response (200):
message: PDF HSC26-Bangla1st-Paper.pdf processed successfully. 36 chunks created.
Error Response (400):
error: Please upload a valid PDF file.

2. Ask Question (POST /ask)
Description: Queries the processed PDF with a Bangla question.
Parameters:
query: Question string (JSON body).
Example:
curl -X POST -H "Content-Type: application/json" -d '{"query": "অনুপমের ভাষায় সুপুরুষ কাকে বলা হয়েছে?"}' https://<ngrok-url>/ask
Success Response (200):
query: অনুপমের ভাষায় সুপুরুষ কাকে বলা হয়েছে?
answer: ঘ শুম্ভুনাথ
context: ৩৫। অনুপমের ভাষায় সুপুরুষ কাকে বলা হয়েছে? ক র্ব্নুদাদা খ অনুপম গ মামা ঘ শুম্ভুনাথ...
grounded: true
Error Response (400):
error: Failed to process query.

Evaluation Matrix
The pipeline is evaluated using the test cases:


Test Case	Query (Bangla)	Expected Answer	Generated Answer	Grounded	Status
1	অনুপমের ভাষায় সুপুরুষ কাকে বলা হয়েছে?	শুম্ভুনাথ	ঘ শুম্ভুনাথ	True	Passed
2	কাকে অনুপমের ভাগ্য দেবতা বলে উল্লেখ করা হয়েছে?	মামাকে	গ মামাকে	True	Passed
3	বিয়ের সময় কল্যাণীর প্রকৃত বয়স কত ছিল?	১৫ বছর	১৫ বছর	True	Passed

Metrics:
Accuracy: 100% (3/3 test cases passed when OCR succeeds).
Groundedness: All answers are grounded in the context, verified by matching option text.
Relevance: Contexts include relevant multiple-choice options, ensuring accurate retrieval.
Challenges: OCR errors in low-quality PDFs can cause failures, addressed through preprocessing and cleaning.

File Structure
rag_pipeline.py: Main script with FastAPI and RAG pipeline.
rag_pipeline.ipynb: Colab notebook for testing.
HSC26-Bangla1st-Paper.pdf: Input PDF (user-provided).
extracted_text.txt: Debugging output of extracted text.
faiss_index.bin: FAISS index file.

Troubleshooting
Garbled Output:
Check extracted_text.txt for OCR quality.

Test OCR on a single page:
import fitz
from PIL import Image
import pytesseract
doc = fitz.open("HSC26-Bangla1st-Paper.pdf")
pix = doc[0].get_pixmap(dpi=400)
img = Image.frombytes("RGB", [pix.width, pix.height], pix.rgb)
img = img.convert('L').filter(ImageFilter.SHARPEN).resize((int(img.width * 2.5), int(img.height * 2.5)))
print(pytesseract.image_to_string(img, lang='ben', config='--psm 6 --oem 3')[:500])

Ngrok Issues:
Verify authtoken at ngrok.com.
Check ngrok status at http://localhost:4040.
Free port 8001:
lsof -i :8001
PDF Not Found:
Ensure HSC26-Bangla1st-Paper.pdf is in the directory or uploaded.
Memory Issues:
Use smaller PDFs or Colab Pro for large files.

Acknowledgments
Models: l3cube-pune/bengali-sentence-similarity-sbert, csebuetnlp/banglat5
Libraries: PyMuPDF, Tesseract, FastAPI, FAISS, pyngrok


### 1. What method or library did you use to extract text, and why? Were there any formatting challenges with the PDF content?
- **Method**: PyMuPDF for text-based PDFs, pytesseract for scanned pages.
- **Why**: PyMuPDF is fast for text PDFs; Tesseract supports Bangla OCR for scanned pages.
- **Challenges**: Scanned PDFs gave garbled text due to low quality. Fixed with higher DPI (400), image preprocessing, and text cleaning.

### 2. What chunking strategy did you choose? Why does it work well for semantic retrieval?
- **Strategy**: Question-answer pairs via regex, sentence-based fallback (200-token limit).
- **Why**: Pairs keep multiple-choice context intact; sentences handle narrative text. Effective for semantic retrieval by preserving question-option relationships.

### 3. What embedding model did you use? Why did you choose it? How does it capture text meaning?
- **Model**: l3cube-pune/bengali-sentence-similarity-sbert.
- **Why**: Trained for Bangla, captures linguistic nuances, lightweight.
- **How**: Encodes sentences into 384D vectors, trained for similarity, aligning similar Bangla text.

### 4. How are you comparing the query with your stored chunks? Why did you choose this similarity method and storage setup?
- **Comparison**: L2 distance via FAISS IndexFlatL2, top-10 chunks, 0.3 threshold.
- **Why**: L2 is efficient for SBERT embeddings; FAISS is fast for small datasets. In-memory storage is simple, with disk backup.

### 5. How do you ensure that the question and document chunks are compared meaningfully? What would happen if the query is vague or missing context?
- **Ensuring**: Normalize text, use Bangla-specific embeddings, question-answer chunks, threshold, groundedness check.
- **Vague Queries**: Returns "Query too vague" if no chunks match, due to distant embeddings. Future fix: query rephrasing.

### 6. Do the results seem relevant? If not, what might improve them?
- **Relevance**: Relevant when OCR works (100% test case accuracy). Garbled with poor PDFs.
- **Improvements**: Cloud OCR (e.g., Google Vision), high-quality PDFs, larger token limits, advanced Bangla models.
