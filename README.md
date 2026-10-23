# Excel RAG Question-Answering System

A Retrieval-Augmented Generation (RAG) notebook that lets you ask natural-language questions about an employee database stored in an Excel file. It retrieves relevant text from the spreadsheet using semantic similarity, then uses a Groq-hosted LLM to generate an answer grounded in the retrieved context.

## How It Works

1. **Load the Excel file** with LangChain's `UnstructuredExcelLoader`.
2. **Split the loaded content** into text chunks using `RecursiveCharacterTextSplitter`.
3. **Create embeddings** with the `sentence-transformers/all-MiniLM-L6-v2` model.
4. **Build a vector index** using FAISS so relevant chunks can be retrieved by similarity.
5. **Retrieve context** for a user's question (the notebook retrieves the top 3 matches by default).
6. **Generate an answer** with Groq's `llama-3.3-70b-versatile` model, using the retrieved context. The prompt instructs the model to say it does not know when the context does not contain the answer.

## Tech Stack

- Python
- Google Colab and Google Drive
- LangChain
- `UnstructuredExcelLoader` for Excel ingestion
- Hugging Face Sentence Transformers (`all-MiniLM-L6-v2`) for embeddings
- FAISS for vector similarity search
- Groq API with `llama-3.3-70b-versatile` for answer generation

## Requirements

- A Google account for Google Colab and Google Drive
- The Excel file `Sample of Employee Database.xlsx`
- A Groq API key
- The Python packages installed in the notebook:
  - `langchain-community`
  - `unstructured`
  - `langchain-groq`
  - `faiss-cpu`
  - `sentence-transformers`

The notebook also imports `IPython.display` and LangChain text-splitting and embedding/vector-store classes.

## Getting Started

### 1. Open the notebook

Upload `RAG_N (1).ipynb` to Google Colab, or open it from Google Drive.

### 2. Prepare the Excel file

Place `Sample of Employee Database.xlsx` in the notebook's working directory, or update the file path in the notebook to match its location.

The notebook currently contains a Google Drive mount and a working-directory change specific to the original author's Drive. If you use a different Drive folder, update or remove that `%cd` line.

### 3. Configure your Groq API key

In Google Colab, add a secret named `Groq` containing your Groq API key. The notebook reads it with:

```python
from google.colab import userdata
api_key = userdata.get("Groq")
```

Do not hard-code your API key in the notebook or commit it to a public repository.

### 4. Install dependencies

Run the notebook's installation cell:

```bash
pip install langchain-community unstructured langchain-groq faiss-cpu sentence-transformers -q
```

### 5. Run the notebook in order

Run the cells from top to bottom. The notebook first demonstrates loading, chunking, indexing, retrieval, and answer generation, then defines reusable functions for preparing the Excel knowledge base and asking questions.

## Example Questions

The notebook includes these example queries:

```python
"Retrieve the marketing employees"
"Who has a car and is living in madinaty"
```

You can ask other natural-language questions about information present in the employee spreadsheet. Results depend on how well the Excel loader extracts the spreadsheet's contents and how relevant the retrieved chunks are to the question.

## Reusable Functions

### `prepare_excel(file_path)`

Loads an Excel file, splits its contents into chunks, creates embeddings, and builds a FAISS vector index. It returns the index for later queries.

```python
db_excel = prepare_excel("Sample of Employee Database.xlsx")
```

### `rag(query, db_faiss, k=3)`

Searches the FAISS index for the `k` most relevant chunks, combines them into context, and sends the context plus the question to the Groq model. The function displays the generated answer as Markdown.

```python
rag("Who has a car and is living in madinaty", db_excel)
```

Change `k` to retrieve a different number of chunks:

```python
rag("Retrieve the marketing employees", db_excel, k=5)
```

## Important Notes and Limitations

- **This is a notebook prototype**, not a deployed application or web interface.
- The current implementation handles Excel files; although the notebook's original folder name mentions PDF, no PDF-loading pipeline is implemented in the code shown.
- The notebook uses a fixed chunk size of 10,000 characters with 1,000 characters of overlap. Depending on the spreadsheet structure, smaller chunks or row-aware processing may improve retrieval.
- RAG answers are not guaranteed to be correct. Check important answers against the source spreadsheet.
- The prompt asks the model to admit when information is missing, but this instruction does not guarantee that every answer will be accurate or fully grounded.
- Employee databases may contain personal information. Use only data you are authorized to process, and avoid publishing the dataset, API keys, or sensitive outputs.
- Package versions can change; if imports fail, check compatibility between the installed LangChain packages.

## Project Structure

```text
.
├── RAG_N (1).ipynb
└── Sample of Employee Database.xlsx   # Required input data; not included here
```

## Possible Improvements

- Add support for multiple Excel sheets and other file formats.
- Preserve spreadsheet row and column structure during ingestion.
- Add source-row or sheet metadata to retrieved results.
- Show retrieved evidence alongside each answer.
- Add error handling for missing files, missing API keys, and empty retrieval results.
- Pin compatible dependency versions in a `requirements.txt`.
- Evaluate retrieval and answer quality using a small set of test questions.

## License

No license is specified in the notebook. Add a license file if you intend to distribute or reuse this project publicly.
