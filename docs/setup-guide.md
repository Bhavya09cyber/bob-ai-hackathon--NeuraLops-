# Setup Guide

> **This file is read by the automated evaluation pipeline. Be precise and complete.**

## Prerequisites

Prerequisites

Before you begin, ensure you have the following installed:

 [Python 3.10 or newer]
 [Git]
 [A modern web browser]
 [An IBM Cloud account with watsonx.ai access]

## Environment Variables

IBM watsonx.ai is optional for the local demonstration.

If you want to enable IBM watsonx.ai / IBM Granite runtime analysis, create a .env file based on .env.example and configure the following values:

| Variable | Description | Required |
|---|---|---|
| `WATSONX_API_KEY` | Your IBM watsonx.ai API key | Yes |
| `WATSONX_PROJECT_ID` | Your watsonx.ai project ID | Yes |
| `DATABASE_URL` | PostgreSQL connection string | Yes |
| `SLACK_WEBHOOK_URL` | Slack webhook for alerts | No |

## Installation

```bash
# 1. Clone the repository
git clone https://github.com/Bhavya09cyber/bob-ai-hackathon--NeuraLops-.git
cd bob-ai-hackathon--NeuraLops-

# 2. Install backend dependencies
[pip install -r requirements.txt
]

# 3. Install frontend dependencies (if applicable)
[your command — e.g.: cd frontend && npm install]

# 4. Set up the database (if applicable)
[SQLite is already included with the project.
]
```

## Running the Application

```bash
# Start the backend
[python backend/app.py]

# Start the frontend (in a separate terminal, if applicable)
[your command — e.g.: cd frontend && npm run dev]
```

The application will be available at: `http://127.0.0.1:5000`

## Running Tests

```bash
[python -m py_compile backend/app.py backend/database.py backend/models.py backend/gap_detector.py backend/ai_analyzer.py backend/seed_data.py]
```

## Quick Demo (Optional)
After starting the application:

Open the Patients page.
Select Margaret Thompson.
Open her Care Journey.
Review the sequence:
Doctor consultation
Test ordered
Test completed
Report available
Notice that the expected follow-up step is missing.
Open the Alerts page.
Review the high-priority care-gap alert.
Review the AI Care-Process Analysis explaining how the gap was detected.
Open Robert Davies to see a complete care journey with no open care gap.

Document Upload Demo

CareSentinel supports healthcare document processing for:

PDF
TXT
DOCX

To demonstrate document processing:

Open the Upload Document page.
Select a supported document.
Upload the document for the selected patient.
The system stores the uploaded document metadata.
Relevant care-process information can be extracted as healthcare events.
The extracted events can be included in the patient's care journey and care-gap detection.

```bash
[e.g.: python demo/seed_demo_data.py]
[e.g.: open http://localhost:8000/demo]
```

## Troubleshooting

| Issue | Solution |
|---|---|
| [ModuleNotFoundError: flask] | [Run pip install -r requirements.txt from the src directory.] |
| [IBM watsonx.ai authentication error] | [Check WATSONX_APIKEY and WATSONX_PROJECT_ID in your local environment configuration.] |
| [Document upload does not work] | [Verify that the document is PDF, TXT, or DOCX and that the required dependencies are installed.] |
