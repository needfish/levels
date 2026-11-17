Levels is a CV Analysis tool that compares a profile with a target Job Description, and offers skillset supplement through YouTube/MOOC content recommendation.


---

## Key Features

- **CV Parser**: Automatically extracts competency information with high accuracy (supports scanned files/images).
- **Skillset Analysis**: Compares current capabilities against target Job Description requirements to pinpoint exact skill gaps.
- **Certificate Roadmap**: Recommends professional certificates and courses (Coursera, Udemy, YouTube) to fill skill gaps for the selected job.
- **Market Insights**: Updated trends and demand for skills based on crawled job board data.

---

## Technology Stack

- **Backend**: FastAPI, Celery, Redis, PostgreSQL (pgvector)
- **Frontend**: Next.js + React + Tailwind
- **AI Backend**: OpenAI GPT-4o, Chandra OCR (Optional) / PDF Fallback, LangGraph, Semantic Search, LiteLLM.

---

## Setup Guide

### 1. Infrastructure Setup (Docker)
Requirements: Docker & Docker Compose.
```bash
cd backend
cp .env.example .env  # Update OPENAI_API_KEY in .env
docker-compose up -d --build
```

### 2. Configure AI Inference Hub (Chandra OCR) - Optional
This service handles advanced CV data extraction (supports image/scan files). 

**Note**: If you skip this step, the system will automatically use the PDF fallback library to extract text from standard PDF files.
```bash
cd ai_inference_hub
python3 -m venv venv
source venv/bin/activate  # Or venv\Scripts\activate on Windows
pip install -r requirements_ai.txt
python setup_chandra.py    # Download model weights (~5GB)
python main.py             # Run AI Hub on port 8080
```

### 3. Initialize Data (Run inside Docker)
After Step 1 completes (containers are up), run the following commands to initialize the system:
```bash
# 1. Initialize Database & Admin (Run once)
docker exec -it advisor_worker_crawler python scripts/setup_db.py

# 2. Load seed data (Courses & Skills)
# Tip: Use --limit to quickly import a small amount (e.g. 20 courses)
docker exec -it advisor_worker_crawler python scripts/seed_all.py --limit 20

# 3. Load job data (TopCV) - Sample 20 job listings
# Note: TopCV blocks Datacenter IP ranges. If running on Cloud (AWS/GCP...),
# configure PROXY_LIST (preferably residential proxies) in Admin Settings before crawling.
docker exec -it advisor_worker_crawler celery -A worker.celery_app call worker.tasks.crawler_tasks.crawl_topcv_jobs_task --args="[20, true]"
```

### 4. Run Frontend
```bash
cd frontend
npm install
npm run dev

# Access local frontend at http://localhost:3000
```


---

## User Flow

1.  **Analyze CV**: Upload your CV file (PDF/Image). AI will automatically extract your skill set and experience.
2.  **Select a Position (JD)**: Choose a target job (e.g. AI Engineer) from listings crawled (this one crawls from TopCV), or paste a JD for comparison manually.
3.  **Gap Analysis**: The system displays a Radar Chart comparing your skills against the actual JD requirements, calculating your current Match Percentage.
4.  **Obtain Certificate Roadmap**: Explore a list of professional certificates and courses (Coursera/YouTube) suggested to fill missing skills.
5.  **Demand Forecast**: View metrics for market demand for skills suggested (based on # of job descriptions having the skill listed)

---

## Documentation

- **Pitch Slides (Google Slides)**: [Pitch Deck](https://docs.google.com/presentation/d/1n9wfL9EEjTd_q0k-bvhSoeOxadimsC5Z/edit?usp=drive_link&ouid=105688789480489939544&rtpof=true&sd=true)
- **Product Demo Video (YouTube)**: [Demo Walkthrough](https://youtu.be/LCOgX_Bs6No)
- **Architecture**: Microservices and Data Flow details.
- **Setup Guide**: Environment setup and data loading instructions.
- **AI Agent Guidelines**: Instructions for AI coding assistants.

---

