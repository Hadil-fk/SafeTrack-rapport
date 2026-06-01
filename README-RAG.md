# SafeTrack RAG API - Testing Guide

## Overview

SafeTrack is a **Child Safety Analysis System** powered by Retrieval Augmented Generation (RAG). It analyzes child location data, detects safety risks (geofencing violations, night activity, abnormal speeds), and provides AI-powered guidance using LLM and knowledge base retrieval.

---

## Setup & Installation

### Prerequisites

- Python 3.8+
- Ollama running locally (for LLM and embeddings)
- Virtual environment (recommended)

### 1. Install Dependencies

```bash
cd RAG-safetrack
python -m venv venv

# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate

pip install -r requirements.txt
```

### 2. Start Ollama

Ollama must be running in the background. Make sure you have the required models:

```bash
# In a separate terminal
ollama serve

# In another terminal, pull models if needed
ollama pull gemma2:9b
ollama pull nomic-embed-text
```

Verify Ollama is running:
```bash
curl http://localhost:11434
```

### 3. Start the API Server

```bash
python app.py
```

You should see:
```
============================================================
SafeTrack RAG API - Starting Server
============================================================

API Documentation:
  - Swagger UI: http://localhost:8000/docs
  - ReDoc: http://localhost:8000/redoc

Server starting on http://localhost:8000
============================================================
```

---

## API Endpoints

### 1. **Health Check** - Verify System Status

**Endpoint:** `GET /health`

**Description:** Check if the API and RAG system are operational.

**Example - cURL:**
```bash
curl -X GET "http://localhost:8000/health"
```

**Example - Python:**
```python
import requests

response = requests.get("http://localhost:8000/health")
print(response.json())
```

**Response:**
```json
{
  "status": "operational",
  "ollama_configured": true,
  "knowledge_base_size": 1
}
```

---

### 2. **Generate Test Data** - Create Sample Location Data

**Endpoint:** `POST /generate-data`

**Description:** Generates simulated child location tracking data and safety zones. This simulates a day in the life of a child including movements between home, school, and a danger zone.

**Example - cURL:**
```bash
curl -X POST "http://localhost:8000/generate-data"
```

**Example - Python:**
```python
import requests

response = requests.post("http://localhost:8000/generate-data")
print(response.json())
```

**Response:**
```json
{
  "success": true,
  "message": "Test data generated successfully",
  "zones_count": 4,
  "locations_count": 60
}
```

---

### 3. **Analyze Locations** - Detect Safety Risks

**Endpoint:** `POST /analyze`

**Description:** Analyzes the generated location data and detects safety risks:
- Geofencing violations (entering danger zones)
- Night activity (outside safe zones during night hours 9 PM - 6 AM)
- Abnormal speeds (suspicious vehicle activity)

**Example - cURL:**
```bash
curl -X POST "http://localhost:8000/analyze"
```

**Example - Python:**
```python
import requests

response = requests.post("http://localhost:8000/analyze")
risks = response.json()
print(f"Total risks found: {risks['total_risks']}")
for risk in risks['risks']:
    print(f"\n- {risk['type']} ({risk['severity'].upper()})")
    print(f"  Location: {risk['location']}")
    print(f"  Description: {risk['description']}")
```

**Response:**
```json
{
  "total_risks": 2,
  "risks": [
    {
      "type": "Critical: Danger Zone Entry",
      "description": "Child entered prohibited zone: Industrial Area (DANGER)",
      "location": "35.842, 10.632",
      "timestamp": "2026-04-21T16:00:00",
      "severity": "high"
    },
    {
      "type": "Alert: Night Activity",
      "description": "Child is active during prohibited night hours",
      "location": "35.801, 10.601",
      "timestamp": "2026-04-21T22:30:00",
      "severity": "medium"
    }
  ]
}
```

---

### 4. **RAG Analysis** - Get AI-Powered Safety Guidance

**Endpoint:** `POST /rag-analyze`

**Description:** Takes a safety risk and generates AI-powered guidance using the knowledge base and LLM.

**Request Body:**
```json
{
  "type": "Critical: Danger Zone Entry",
  "description": "Child entered prohibited zone: Industrial Area",
  "location": "35.85, 10.65",
  "severity": "high",
  "timestamp": "2026-04-21T16:00:00"
}
```

**Example - cURL:**
```bash
curl -X POST "http://localhost:8000/rag-analyze" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "Critical: Danger Zone Entry",
    "description": "Child entered prohibited industrial zone",
    "location": "35.85, 10.65",
    "severity": "high"
  }'
```

**Example - Python:**
```python
import requests
import json

risk_data = {
    "type": "Critical: Danger Zone Entry",
    "description": "Child entered prohibited industrial zone",
    "location": "35.85, 10.65",
    "severity": "high"
}

response = requests.post(
    "http://localhost:8000/rag-analyze",
    json=risk_data
)

result = response.json()
print(f"Risk Type: {result['risk_type']}")
print(f"Severity: {result['severity']}")
print(f"\nGuidance:\n{result['guidance']}")
```

**Response:**
```json
{
  "risk_type": "Critical: Danger Zone Entry",
  "severity": "high",
  "guidance": "**Immediate Risk Assessment**: This is a CRITICAL situation. Industrial areas pose severe safety risks including machinery hazards, toxic substances, traffic dangers, and potential criminal activity...\n\n**Action Plan for Parent**: 1. Contact emergency services immediately (911). 2. Provide child's location coordinates...",
  "timestamp": "2026-04-21T16:05:30.123456"
}
```

---

### 5. **Full Analysis Pipeline** - Complete Workflow

**Endpoint:** `POST /full-analysis`

**Description:** Orchestrates the entire workflow:
1. Generates test data
2. Analyzes location data for risks
3. Runs RAG analysis on each detected risk
4. Returns comprehensive results

**Example - cURL:**
```bash
curl -X POST "http://localhost:8000/full-analysis"
```

**Example - Python:**
```python
import requests

response = requests.post("http://localhost:8000/full-analysis")
results = response.json()

print(f"Status: {results['status']}")
print(f"Total Risks: {results['total_risks']}")

for i, result in enumerate(results['results'], 1):
    print(f"\n--- Risk #{i} ---")
    print(f"Type: {result['risk']['type']}")
    print(f"Severity: {result['risk']['severity']}")
    print(f"\nAI Guidance:\n{result['guidance']}")
```

**Response:**
```json
{
  "status": "complete",
  "total_risks": 2,
  "results": [
    {
      "risk": {
        "type": "Critical: Danger Zone Entry",
        "description": "Child entered prohibited zone: Industrial Area (DANGER)",
        "location": "35.842, 10.632",
        "timestamp": "2026-04-21T16:00:00",
        "severity": "high"
      },
      "guidance": "**Immediate Risk Assessment**:\nThis is a CRITICAL situation. Industrial areas pose severe safety risks...\n\n**Action Plan for Parent**:\n1. Contact emergency services immediately..."
    }
  ],
  "timestamp": "2026-04-21T16:05:30"
}
```

---

### 6. **Knowledge Base Status** - Check KB Info

**Endpoint:** `GET /kb-status`

**Description:** Returns information about the vector database and loaded knowledge documents.

**Example - cURL:**
```bash
curl -X GET "http://localhost:8000/kb-status"
```

**Example - Python:**
```python
import requests

response = requests.get("http://localhost:8000/kb-status")
print(response.json())
```

**Response:**
```json
{
  "status": "operational",
  "documents_in_db": 1,
  "knowledge_files_available": 1,
  "knowledge_file_paths": ["guidelines.txt"]
}
```

---

### 7. **Initialize Knowledge Base** - Load/Reload Documents

**Endpoint:** `POST /initialize-kb`

**Description:** Loads all `.txt` files from `data/safety_knowledge/` into the vector database.

**Example - cURL:**
```bash
curl -X POST "http://localhost:8000/initialize-kb"
```

**Example - Python:**
```python
import requests

response = requests.post("http://localhost:8000/initialize-kb")
print(response.json())
```

**Response:**
```json
{
  "status": "success",
  "message": "Knowledge base initialized with 1 files",
  "documents_count": 1
}
```

---

## Interactive Testing

### Option 1: Swagger UI (Recommended)

Open your browser and navigate to:
```
http://localhost:8000/docs
```

This provides an interactive interface where you can:
- View all endpoints
- Execute API calls directly
- See request/response schemas
- Try different parameters

### Option 2: ReDoc

For API documentation in a different format:
```
http://localhost:8000/redoc
```

### Option 3: Python Script - Complete Test Flow

Create a file `test_api.py`:

```python
#!/usr/bin/env python3
"""
Complete API testing script for SafeTrack RAG System
"""

import requests
import json
import time

BASE_URL = "http://localhost:8000"

def print_section(title):
    print(f"\n{'='*60}")
    print(f"  {title}")
    print(f"{'='*60}\n")

def test_health():
    """Test health endpoint"""
    print_section("1. Health Check")
    
    response = requests.get(f"{BASE_URL}/health")
    print(f"Status Code: {response.status_code}")
    print(f"Response:\n{json.dumps(response.json(), indent=2)}")
    return response.status_code == 200

def test_generate_data():
    """Test data generation"""
    print_section("2. Generate Test Data")
    
    response = requests.post(f"{BASE_URL}/generate-data")
    print(f"Status Code: {response.status_code}")
    data = response.json()
    print(f"Response:\n{json.dumps(data, indent=2)}")
    return response.status_code == 200

def test_analyze():
    """Test risk analysis"""
    print_section("3. Analyze Locations for Risks")
    
    response = requests.post(f"{BASE_URL}/analyze")
    print(f"Status Code: {response.status_code}")
    data = response.json()
    print(f"Total Risks Found: {data['total_risks']}")
    print(f"\nRisks:")
    for risk in data['risks']:
        print(f"  - {risk['type']} ({risk['severity'].upper()})")
        print(f"    Location: {risk['location']}")
        print(f"    Timestamp: {risk['timestamp']}")
    return response.status_code == 200

def test_kb_status():
    """Test knowledge base status"""
    print_section("4. Knowledge Base Status")
    
    response = requests.get(f"{BASE_URL}/kb-status")
    print(f"Status Code: {response.status_code}")
    print(f"Response:\n{json.dumps(response.json(), indent=2)}")
    return response.status_code == 200

def test_rag_single_risk():
    """Test RAG analysis on a single risk"""
    print_section("5. RAG Analysis - Single Risk")
    
    risk_payload = {
        "type": "Critical: Danger Zone Entry",
        "description": "Child detected in industrial area with machinery",
        "location": "35.85, 10.65",
        "severity": "high"
    }
    
    print(f"Request:\n{json.dumps(risk_payload, indent=2)}\n")
    
    response = requests.post(
        f"{BASE_URL}/rag-analyze",
        json=risk_payload
    )
    
    print(f"Status Code: {response.status_code}")
    data = response.json()
    print(f"Risk Type: {data['risk_type']}")
    print(f"Severity: {data['severity']}")
    print(f"\nAI Guidance:\n{data['guidance']}")
    return response.status_code == 200

def test_full_analysis():
    """Test complete analysis pipeline"""
    print_section("6. Full Analysis Pipeline")
    
    print("Running complete workflow...")
    print("(This may take a few moments due to LLM processing)\n")
    
    response = requests.post(f"{BASE_URL}/full-analysis")
    print(f"Status Code: {response.status_code}")
    data = response.json()
    
    print(f"Status: {data['status']}")
    print(f"Total Risks Analyzed: {data['total_risks']}")
    
    for i, result in enumerate(data['results'], 1):
        print(f"\n--- Risk #{i} ---")
        print(f"Type: {result['risk']['type']}")
        print(f"Severity: {result['risk']['severity'].upper()}")
        print(f"Location: {result['risk']['location']}")
        print(f"\nAI Guidance (first 300 chars):\n{result['guidance'][:300]}...")
    
    return response.status_code == 200

def main():
    """Run all tests"""
    print("\n" + "="*60)
    print("  SafeTrack RAG API - Complete Test Suite")
    print("="*60)
    
    tests = [
        ("Health Check", test_health),
        ("Generate Data", test_generate_data),
        ("Analyze Risks", test_analyze),
        ("KB Status", test_kb_status),
        ("RAG Analysis", test_rag_single_risk),
        ("Full Pipeline", test_full_analysis),
    ]
    
    results = []
    
    for name, test_func in tests:
        try:
            success = test_func()
            results.append((name, "✓ PASS" if success else "✗ FAIL"))
        except Exception as e:
            print(f"\n✗ Error: {e}")
            results.append((name, "✗ ERROR"))
        
        time.sleep(1)  # Small delay between tests
    
    # Summary
    print_section("Test Summary")
    for name, status in results:
        print(f"{name:<20} {status}")
    
    passed = sum(1 for _, s in results if "PASS" in s)
    total = len(results)
    print(f"\nTotal: {passed}/{total} tests passed")

if __name__ == "__main__":
    main()
```

**Run the test script:**
```bash
python test_api.py
```

### Option 4: Using httpie (if installed)

```bash
# Health check
http GET http://localhost:8000/health

# Generate data
http POST http://localhost:8000/generate-data

# Analyze
http POST http://localhost:8000/analyze

# RAG Analysis
http POST http://localhost:8000/rag-analyze \
  type="Critical: Danger Zone Entry" \
  description="Child in industrial area" \
  location="35.85, 10.65" \
  severity="high"
```

---

## Troubleshooting

### ❌ Error: "Connection refused" on port 8000

**Solution:** Make sure the API server is running:
```bash
python app.py
```

### ❌ Error: "Ollama not available" / "503 Service Unavailable"

**Solution:** Start Ollama in a separate terminal:
```bash
ollama serve
```

**Verify it's running:**
```bash
curl http://localhost:11434
```

### ❌ Error: "Model not found" (gemma2:9b or nomic-embed-text)

**Solution:** Pull the required models:
```bash
ollama pull gemma2:9b
ollama pull nomic-embed-text
```

### ❌ Slow RAG responses

The first request may be slow as the model loads. Subsequent requests will be faster.

---

## Project Structure

```
RAG-safetrack/
├── app.py                          # FastAPI application
├── main.py                         # CLI interface
├── rag_engine.py                   # RAG logic
├── analyzer.py                     # Risk detection
├── vector_db.py                    # Vector database (Chroma)
├── data_generator.py               # Test data generation
├── config.py                       # Configuration
├── requirements.txt                # Dependencies
├── data/
│   ├── chroma_db/                 # Vector database storage
│   └── safety_knowledge/
│       └── guidelines.txt          # Knowledge base document
├── safetrack.locations.json        # Generated location data
└── zones.json                      # Generated safety zones
```

---

## API Flow

```
User Request
    ↓
[FastAPI Server] (app.py)
    ↓
┌─────────────────────────────────┐
│   Data Generation               │
│ (data_generator.py)             │
│ → Creates location & zone data  │
└─────────────────────────────────┘
    ↓
┌─────────────────────────────────┐
│   Risk Analysis                 │
│ (analyzer.py)                   │
│ → Geofencing, night, speed      │
└─────────────────────────────────┘
    ↓
┌─────────────────────────────────┐
│   RAG Processing                │
│ (rag_engine.py)                 │
│ → Vector DB query (vector_db)   │
│ → LLM inference (Ollama)        │
│ → Generates guidance            │
└─────────────────────────────────┘
    ↓
User Response (JSON)
```

---

## Key Features

✅ **Geofencing Detection** - Alerts when child enters danger zones
✅ **Night Activity Monitoring** - Detects activity outside safe hours
✅ **Speed Anomaly** - Identifies suspicious vehicle activity
✅ **RAG-Powered Guidance** - AI-generated safety recommendations
✅ **Swagger Documentation** - Interactive API exploration
✅ **Knowledge Base Integration** - Contextual safety protocols
✅ **Extensible** - Easy to add new risk types and protocols

---

## Next Steps

1. Start the API server: `python app.py`
2. Open Swagger UI: http://localhost:8000/docs
3. Run the test script: `python test_api.py`
4. Add custom safety knowledge in `data/safety_knowledge/`
5. Extend risk detection in `analyzer.py`

---

**Version:** 1.0.0  
**Last Updated:** April 21, 2026
