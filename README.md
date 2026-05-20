# PaperLens

PaperLens is a research paper analysis app that turns PDFs into a searchable, question-answerable knowledge base. It uses GROBID for structured PDF extraction, Pinecone for semantic retrieval, and Gemini to generate grounded answers.[1][2][3]

## Stack

- **Frontend:** Angular
- **Backend:** Node.js, Express
- **Storage:** AWS S3
- **Processing:** AWS Lambda
- **PDF Extraction:** GROBID service on EC2
- **Vector DB:** Pinecone
- **LLM:** Gemini 2.5 Flash[1][4][3][5]

## Features

- Upload research papers as PDFs
- Extract structured paper text with GROBID
- Chunk and embed extracted text
- Index embeddings in Pinecone
- Ask natural-language questions about a paper
- Retrieve relevant chunks before answer generation
- Return grounded answers through a simple UI

## Architecture

```text
Angular UI
   -> Node.js API
   -> S3 upload
   -> Lambda 1 (ingestion trigger)
   -> Lambda 2 (call GROBID on EC2 + chunking + embeddings)
   -> Lambda 3 (Pinecone upsert)

Query flow:
Angular UI
   -> Node.js API
   -> Pinecone retrieval
   -> Gemini 2.5 Flash
   -> Answer
```

S3 can trigger Lambda functions automatically when files are uploaded, which makes the ingestion flow event-driven.[4] GROBID is commonly run as a web service in Docker, and EC2 provides the compute layer to host that extraction service reliably outside Lambda’s runtime constraints.[3][5][6]

## How it works

1. A user uploads a PDF from the Angular frontend.
2. The Node.js API stores the file in S3.[4]
3. Lambda 1 starts the ingestion pipeline.
4. Lambda 2 sends the PDF to a GROBID service running on EC2, which extracts structured text from the paper.[3][7][8]
5. The extracted text is cleaned, chunked, and converted into embeddings.
6. Lambda 3 stores the embeddings and chunk metadata in Pinecone.[9][2]
7. When the user asks a question, the backend retrieves the most relevant chunks from Pinecone and sends them to Gemini as context.[9][1]
8. Gemini returns a grounded answer, which is displayed in the UI.[1]

## API overview

### `POST /upload`
Uploads a PDF and starts the processing pipeline.

### `GET /papers`
Returns indexed papers available in the system.

### `POST /query`
Takes a paper ID and a question, retrieves relevant chunks, and returns the generated answer.

## GROBID on EC2

PaperLens uses GROBID as a dedicated extraction service instead of running PDF parsing inside Lambda. GROBID is designed to run as a server, typically through Docker, and exposes an HTTP API for processing scholarly PDFs into structured TEI/XML output.[3][10][8]

Using EC2 for GROBID keeps the extraction layer separate from the rest of the serverless pipeline and avoids packing a heavier PDF extraction stack directly into Lambda. Amazon EC2 is designed to run cloud applications with scalable compute capacity, which makes it a practical fit for hosting long-running services such as GROBID.[11][5][6]

## Example metadata

Each chunk stored in Pinecone includes metadata like:

```json
{
  "paper_id": "paper-123",
  "chunk_index": 4,
  "total_chunks": 18,
  "processed_date": "2026-03-30T12:34:56Z",
  "title": "Example Paper Title"
}
```

Pinecone supports listing vector IDs and fetching records with metadata, which is useful for managing indexed paper chunks.[9][2]

## Running locally

```bash
# frontend
cd frontend
npm install
ng serve

# backend
cd backend
npm install
npm run dev
```

The backend needs environment variables for AWS, the EC2-hosted GROBID endpoint, Pinecone, and Gemini.

## Why this project matters

PaperLens is a practical RAG application built around real document workflows rather than a generic chatbot. It demonstrates full-stack development, serverless orchestration, external document extraction with GROBID, vector retrieval, and grounded LLM answer generation in one system.[1][4][3][5]