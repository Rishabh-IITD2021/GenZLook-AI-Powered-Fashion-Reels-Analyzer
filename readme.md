# GenZLook: AI-Powered Fashion Reels Analyzer

**GenZLook** is an AI-powered fashion intelligence system designed to analyze fashion reels, detect clothing items, and match them with similar products from a catalog while identifying aesthetic "vibes" or styles. It combines state-of-the-art deep learning models and scalable APIs to deliver fast, accurate fashion insights.

---

## Key Features

- **Video Fashion Item Detection** — YOLO-based object detection on fashion reels.
- **Visual Product Matching** — CLIP embeddings + FAISS index for product similarity search.
- **Aesthetic Vibe Classification** — Style tagging using Sentence Transformers.
- **FastAPI Backend** — Lightweight Fast API with background job processing.
- **Efficient Indexing Pipeline** — From catalog images to searchable fashion embeddings.

---

## 📁 Project Structure

```
genzlook/
├── api/                     # FastAPI backend
│   ├── main.py              # Entry point
│   ├── schemas.py           # Request/response schemas
│   └── video_processor.py   # Core video analysis logic
├── data/                    # Raw product data
│   ├── product_data.csv
│   └── images.csv
├── models/                  # Model weights (populated at runtime)
│   ├── yolos-fashionpedia/
│   ├── sentence-transformer/
│   └── clip-vit-b-32.pt
├── output/                  # Output artifacts (embeddings, index, metadata)
├── outputs/                 # API result files for each job
├── src/                     # Indexing pipeline
│   ├── data_loader.py
│   ├── embedding_generator.py
│   ├── faiss_indexer.py
│   ├── image_processor.py
│   └── main.py
├── requirements.txt         # Python dependencies
└── run_api.sh               # Script to start API server
```

---

## Setup Instructions

### 1. Prerequisites

- Python 3.9+
- pip
- virtualenv (recommended)

### 2. Create a Virtual Environment

```bash
python -m venv venv
source venv/bin/activate        # Linux / macOS
venv\Scripts\activate         # Windows
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

---

## Preparing the Product Index

1. **Place product data** in the `data/` directory:
   - `product_data.csv`: Contains product metadata.
   - `images.csv`: Contains image URLs for catalog products.

2. **Run the indexing pipeline** to generate embeddings and build the FAISS index:

```bash
python -m src.main
```

### Pipeline Steps

- **Data Loading**: Loads product metadata and image URLs.
- **Image Downloading**: Downloads product images locally.
- **Embedding Generation**: Generates CLIP embeddings per product.
- **Index Creation**: Builds a FAISS index for visual similarity search.
- **Metadata Export**: Saves metadata including color and visual embeddings.

### Output Files

The following files will be created in the `output/` directory:

- `faiss_index_products.clip`: FAISS index file for similarity search
- `product_metadata.json`: Combined metadata and embedding info
- `catalog_images/`: Downloaded product images

---

## Running the API Server

Start the FastAPI backend server:

```bash
chmod +x run_api.sh
./run_api.sh
```

---

## 🌐 API Endpoints

| Endpoint                | Method | Description                         | Parameters            | Response        |
|------------------------|--------|-------------------------------------|------------------------|-----------------|
| `/upload`              | POST   | Uploads a video for processing      | `file`: MP4 video file | `job_id`        |
| `/status/{job_id}`     | GET    | Returns processing status           | `job_id`: UUID         | Status JSON     |
| `/results/{job_id}`    | GET    | Fetches processing results          | `job_id`: UUID         | Results JSON    |
| `/download/{job_id}`   | GET    | Downloads result JSON               | `job_id`: UUID         | JSON file       |

---

## Example Usage

### 1. Upload a video

```bash
curl -X POST -F "file=@reel.mp4" http://localhost:8000/upload
```

**Response:**

```json
{
  "status": "started",
  "message": "Processing started in background",
  "job_id": "77d261a6-26b0-44aa-8f16-c3911d2f85d3"
}
```

---

### 2. Check Job Status

```bash
curl http://localhost:8000/status/77d261a6-26b0-44aa-8f16-c3911d2f85d3
```

**Sample Response:**

```json
{
  "status": "processing",
  "message": "Detecting objects in frames",
  "job_id": "77d261a6-26b0-44aa-8f16-c3911d2f85d3",
  "progress": 30,
  "elapsed_time": 45.2
}
```

---

### 3. Get Results

```bash
curl http://localhost:8000/results/77d261a6-26b0-44aa-8f16-c3911d2f85d3
```

**Sample Response:**

```json
{
  "video_id": "reel_001",
  "vibes": ["Coquette", "Party Glam"],
  "products": [
    {
      "type": "Dress",
      "matched_product_id": 42076,
      "match_type": "similar",
      "confidence": 0.8026
    },
    {
      "type": "Trouser",
      "matched_product_id": 15166,
      "match_type": "similar",
      "confidence": 0.8166
    }
  ]
}
```

---

### 4. Download Result JSON

```bash
curl -O http://localhost:8000/download/77d261a6-26b0-44aa-8f16-c3911d2f85d3
```

---

## Processing Pipeline Overview

1. **Frame Extraction**: Extracts 1 frame/sec from uploaded video.
2. **Object Detection**: YOLO (fashionpedia variant) detects fashion items.
3. **Duplicate Removal**: CLIP embeddings deduplicate similar objects.
4. **Product Matching**: FAISS index finds visually similar products.
5. **Vibe Classification**: Sentence Transformers identify aesthetic vibes.
6. **Result Compilation**: Outputs final matched product and style tags as JSON.

---
