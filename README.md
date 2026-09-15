# 🚀 [CareSentinel: Intelligent AI Based Healthcare Journey Monitoring & Care Gap Detection]

> ⚠️ **Replace everything in `[ ]` brackets with your actual content before submission.**

---

## 👥 Team

| Field | Value |
|---|---|
| **Team Name** | [NeuraLops] |
| **Track** | [AI] |
| **Team Lead** | [Bhavya Shah] — [25ce108@charusat.edu.in] |
| **Members** | [Vidhan Shah], [Richa Shah], [Vaidehi Shah] |

---

## 🎯 Problem Statement

> In 2–3 sentences: What problem does your project solve? Who experiences this problem?

Healthcare for elderly people often involves a chain of connected steps—consultations, tests, reports, and follow-ups—but a missed step can easily go unnoticed when no one is continuously monitoring the entire journey. Elderly patients living alone, along with their caregivers and healthcare providers, face the risk of these unnoticed care gaps leading to delays or interrupted care.
---

## 💡 Solution

> In 2–3 sentences: What did you build? How does it solve the problem above?

CareSentinel creates a living timeline of a patient's healthcare journey, connecting consultations, tests, reports, follow-ups, and other care events into one continuous view. AI understands the expected sequence and relationships between these events, identifies completed, pending, delayed, or missing steps, and detects where the patient's care journey may have broken down. It transforms scattered healthcare events into an understandable, continuously monitored care journey.

---

## ✨ Key Features

-**AI-Powered Care Journey Timeline:** Creates a continuous timeline of consultations, tests, reports, follow-ups, and other care events.
- **Expected vs Actual Care Comparison:** Compares the expected sequence of care events with the events actually recorded.
- **Care-Gap and Dependency Detection:** Identifies missing, delayed, or incomplete care-process steps and their dependencies.
- **Explainable Care-Gap Alerts:** Explains where the care journey broke and why the system detected a gap.
- **AI-Powered Journey Insights:** Provides an understandable summary of the patient's care journey, open gaps, and expected next steps.


---

## 🛠️ Tech Stack

| Category | Technologies |
|---|---|
| **Languages** | [Python, JavaScript, HTML, CSS] |
| **Frameworks** | [Flask] |
| **IBM Technologies** | [IBM Bob , IBM watsonx.ai, IBM Granite]|
| **Databases** | [SQLite] |
| **Other** | [GitHub, REST API, PDF document upload, Windows Batch] |

---

## 📁 Repository Structure

```text
├── src/                       # All source code
│   ├── backend/               # Flask backend and care-gap detection
│   ├── frontend/              # HTML, CSS and JavaScript frontend
│   ├── requirements.txt       # Python dependencies
│   ├── start.bat              # Windows startup script
│   └── .env.example           # Environment variable template
├── docs/                      # Written documentation
│   ├── problem-statement.md
│   ├── solution-overview.md
│   ├── architecture.md
│   └── setup-guide.md
├── demo/                      # Demo artifacts
│   ├── screenshots/            # App screenshots
│   ├── sample_care_report.txt
│   ├── demo-video-link.txt     # Link to demo video
│   └── live-demo-url.txt       # Live demo information
├── presentation/              # Slide deck
├── submission.yaml            # Structured submission metadata
├── README.md                  # Project overview
└── CONTRIBUTING.md             # Submission guidelines

---

## ⚡ How to Run

> **Copy these exact steps from your [`docs/setup-guide.md`](docs/setup-guide.md)**

```bash
# 1. Clone the repository
git clone https://github.com/Bhavya09cyber/bob-ai-hackathon--NeuraLops-.git
cd bob-ai-hackathon--NeuraLops-

# 2. Install backend dependencies
[pip install -r requirements.txt
]



# 4. Run the project
[python backend/app.py]
```

---

## 🖥️ Demo

| Artifact | Link |
|---|---|
| 📹 Demo Video | [https://drive.google.com/file/d/1ePm5mMIitHJbL92ZgSGSmUVkyriwLKx2/view?usp=sharing](demo/demo-video-link.txt) |
| 🌐 Live Demo | [See demo/live-demo-url.txt](demo/live-demo-url.txt) |
| 🖼️ Screenshots | [demo/screenshots/landing page.jpg](demo/screenshots/) |
| 📊 Presentation | [https://drive.google.com/file/d/1s1Hf-J734_YKDZmLmER3SwUygozKQaSo/view?usp=sharing](presentation/) |

---

## ⚠️ Known Limitations

> Be honest — judges appreciate transparency over overclaiming.

[AI runtime analysis requires IBM watsonx.ai credentials for live IBM Granite inference.]
[The current prototype focuses on healthcare care-process monitoring rather than medical diagnosis or treatment decisions.]
[The demonstration uses synthetic patient data.]
[The current prototype is intended as a hackathon demonstration and is not a production healthcare system.]

---

## 🏅 What We're Most Proud Of

[CareSentinel goes beyond conventional reminder systems by understanding the expected sequence of healthcare events rather than treating each task as an isolated reminder. It detects where the care journey breaks, identifies the missing or delayed care-process step, and provides an explainable alert showing how the gap was detected. This makes the system focused on care continuity and gap detection, not just task reminders.]

---
