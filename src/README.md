Place all CareSentinel project's source code in this folder.

## Structure Guidelines

CareSentinel is a web application with a Flask backend and HTML/CSS/JavaScript frontend.

```text
src/
├── backend/              ← Flask API server and core application logic
│   ├── app.py            ← Main Flask application and REST APIs
│   ├── database.py       ← SQLite database setup and operations
│   ├── models.py         ← Data models
│   ├── gap_detector.py   ← Care-gap detection logic
│   ├── ai_analyzer.py    ← IBM watsonx.ai / Granite AI analysis
│   ├── seed_data.py      ← Synthetic demonstration data
│   ├── data/             ← SQLite database
│   └── uploads/          ← Uploaded healthcare documents
│
├── frontend/             ← CareSentinel user interface
│   ├── index.html
│   ├── patients.html
│   ├── patient.html
│   ├── journey.html
│   ├── alerts.html
│   ├── documents.html
│   ├── upload.html
│   ├── add-event.html
│   ├── settings.html
│   ├── css/
│   │   └── style.css
│   └── js/
│       ├── api.js
│       ├── alerts.js
│       ├── timeline.js
│       └── upload.js
│
├── requirements.txt      ← Python dependency manifest
├── start.bat             ← Windows startup script
├── .env.example          ← Environment variable template
└── README.md             ← Source code documentation

## Important Files to Include

requirements.txt — Python dependencies required by the Flask backend
.env.example — Template for IBM watsonx.ai environment variables
start.bat — Windows script for installing dependencies and starting the application
backend/data/caresentinel.db — SQLite demonstration database
backend/uploads/ — Directory for uploaded healthcare documents

## What NOT to Include in src/

.env files containing real IBM credentials or secrets
__pycache__/
.venv/ or venv/
node_modules/
Build artifacts such as dist/ or build/
Other unnecessary generated files
