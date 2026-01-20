# API Documentation

Serverless API endpoints for the portfolio website, deployed on Vercel.

## Endpoints

### POST `/api/analyze-company`

Generates a personalized 90-day plan for ML engineering roles based on company and job description analysis.

**Request:**
```json
{
  "companyName": "Company Name",
  "jobDescription": "Job description text or URL"
}
```

**Response:**
```json
{
  "plan": "Generated 90-day plan...",
  "jobFit": "Analysis of job fit...",
  "metadata": {
    "latency": 25000,
    "model": "gpt-4",
    "tokens": { "input": 1500, "output": 2000, "total": 3500 }
  }
}
```

### POST `/api/chatbot`

Interactive chatbot for answering questions about projects and experience.

**Request:**
```json
{
  "message": "User message",
  "conversationHistory": []
}
```

**Response:**
```json
{
  "response": "Chatbot response..."
}
```

## Configuration

Set environment variables in Vercel:
- `OPENAI_API_KEY` or `ANTHROPIC_API_KEY` for LLM features
