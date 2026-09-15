Place all CareSentinel project's source code in this folder.

## Structure Guidelines

CareSentinel is a web application with a Flask backend and HTML/CSS/JavaScript frontend.

```text
src/
├── backend/              ← Flask API server and core application logic
│   ├── app.py   

from flask import Flask, request, jsonify, send_from_directory
from flask_cors import CORS
from pathlib import Path
from datetime import datetime
import os

from database import (
    init_db,
    get_all_patients,
    get_patient,
    get_patient_events,
    create_event,
    get_event_types,
    get_alerts,
    get_documents,
    save_document,
    update_patient_profile_photo,
)
from gap_detector import detect_gaps
from models import EVENT_TYPES
from ai_analyzer import analyze_document

BASE_DIR = Path(__file__).resolve().parent
FRONTEND_DIR = BASE_DIR.parent / "frontend"
UPLOAD_DIR = BASE_DIR / "uploads"
PROFILE_PHOTO_DIR = BASE_DIR / "profile_photos"

UPLOAD_DIR.mkdir(exist_ok=True)
PROFILE_PHOTO_DIR.mkdir(exist_ok=True)

app = Flask(__name__, static_folder=str(FRONTEND_DIR))
CORS(app)

init_db()


@app.route("/")
def index():
    return send_from_directory(FRONTEND_DIR, "index.html")


@app.route("/<path:path>")
def frontend_files(path):
    file_path = FRONTEND_DIR / path

    if file_path.exists() and file_path.is_file():
        return send_from_directory(FRONTEND_DIR, path)

    return send_from_directory(FRONTEND_DIR, "index.html")


@app.route("/api/event-types", methods=["GET"])
def event_types():
    return jsonify(EVENT_TYPES)


@app.route("/api/patients", methods=["GET"])
def patients():
    return jsonify(get_all_patients())


@app.route("/api/patients/<int:patient_id>", methods=["GET"])
def patient(patient_id):
    data = get_patient(patient_id)

    if not data:
        return jsonify({"error": "Patient not found"}), 404

    return jsonify(data)


@app.route("/api/patients/<int:patient_id>/events", methods=["GET"])
def patient_events(patient_id):
    return jsonify(get_patient_events(patient_id))


@app.route("/api/events", methods=["POST"])
def add_event():
    data = request.get_json(silent=True) or {}

    patient_id = data.get("patient_id")

    if not patient_id:
        return jsonify({"error": "patient_id is required"}), 400

    try:
        event = create_event(
            patient_id=patient_id,
            event_type=data.get("event_type"),
            event_date=data.get("event_date"),
            title=data.get("title"),
            description=data.get("description", ""),
            source=data.get("source", "Manual"),
        )

        detect_gaps(patient_id)

        return jsonify(event), 201

    except Exception as e:
        return jsonify({"error": str(e)}), 400


@app.route("/api/alerts", methods=["GET"])
def alerts():
    patient_id = request.args.get("patient_id")

    if patient_id:
        return jsonify(get_alerts(int(patient_id)))

    return jsonify(get_alerts())


@app.route("/api/documents", methods=["GET"])
def documents():
    patient_id = request.args.get("patient_id")

    if patient_id:
        return jsonify(get_documents(int(patient_id)))

    return jsonify(get_documents())


@app.route("/api/upload", methods=["POST"])
def upload_document():
    if "file" not in request.files:
        return jsonify({"error": "No file uploaded"}), 400

    file = request.files["file"]
    patient_id = request.form.get("patient_id")

    if not patient_id:
        return jsonify({"error": "patient_id is required"}), 400

    if not file.filename:
        return jsonify({"error": "No filename provided"}), 400

    filename = Path(file.filename).name
    extension = Path(filename).suffix.lower()

    allowed = [".pdf", ".txt", ".docx"]

    if extension not in allowed:
        return jsonify({
            "error": "Only PDF, TXT and DOCX files are supported"
        }), 400

    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    stored_filename = f"{timestamp}_{filename}"
    stored_path = UPLOAD_DIR / stored_filename

    file.save(stored_path)

    try:
        result = analyze_document(
            str(stored_path),
            int(patient_id)
        )

        document = save_document(
            patient_id=int(patient_id),
            filename=filename,
            stored_filename=stored_filename,
            file_type=extension,
            extracted_text=result.get("text", ""),
        )

        created_events = []

        for event in result.get("events", []):
            try:
                new_event = create_event(
                    patient_id=int(patient_id),
                    event_type=event.get("event_type"),
                    event_date=event.get("event_date"),
                    title=event.get("title"),
                    description=event.get("description", ""),
                    source="Document Upload",
                )

                created_events.append(new_event)

            except Exception:
                continue

        detect_gaps(int(patient_id))

        return jsonify({
            "success": True,
            "document": document,
            "auto_created_events": created_events,
            "ai_used": result.get("ai_used", False),
            "message": "Document uploaded and care events processed successfully."
        })

    except Exception as e:
        return jsonify({
            "error": str(e)
        }), 500


@app.route("/uploads/<path:filename>")
def uploaded_file(filename):
    return send_from_directory(UPLOAD_DIR, filename)


@app.route("/api/patients/<int:patient_id>/photo", methods=["POST"])
def upload_profile_photo(patient_id):
    if "photo" not in request.files:
        return jsonify({"error": "No photo uploaded"}), 400

    photo = request.files["photo"]

    if not photo.filename:
        return jsonify({"error": "No filename provided"}), 400

    extension = Path(photo.filename).suffix.lower()

    allowed = [".jpg", ".jpeg", ".png", ".webp"]

    if extension not in allowed:
        return jsonify({
            "error": "Only JPG, JPEG, PNG and WEBP images are supported"
        }), 400

    filename = f"patient_{patient_id}{extension}"
    path = PROFILE_PHOTO_DIR / filename

    photo.save(path)

    update_patient_profile_photo(patient_id, filename)

    return jsonify({
        "success": True,
        "filename": filename
    })


@app.route("/patient-photos/<path:filename>")
def patient_photo(filename):
    return send_from_directory(PROFILE_PHOTO_DIR, filename)


if __name__ == "__main__":
    print("CareSentinel running at http://127.0.0.1:5000")
    app.run(host="127.0.0.1", port=5000, debug=True)         ← Main Flask application and REST APIs
│   ├── database.py       ← SQLite database setup and operations
│   ├── models.py    EVENT_TYPES = [
    {
        "value": "doctor_consultation",
        "label": "Doctor Consultation"
    },
    {
        "value": "test_ordered",
        "label": "Test Ordered"
    },
    {
        "value": "test_completed",
        "label": "Test Completed"
    },
    {
        "value": "report_available",
        "label": "Report Available"
    },
    {
        "value": "follow_up_consultation",
        "label": "Follow-up Consultation"
    },
    {
        "value": "referral_made",
        "label": "Referral Made"
    },
    {
        "value": "specialist_consultation",
        "label": "Specialist Consultation"
    },
    {
        "value": "hospital_discharge",
        "label": "Hospital Discharge"
    }
]     ← Data models
│   ├── gap_detector.py from datetime import datetime, timedelta

from database import (
    get_patient_events,
    create_alert,
    get_existing_alerts,
    resolve_alert
)


RULES = [
    {
        "from": "test_ordered",
        "to": ["test_completed"],
        "days": 14,
        "severity": "high",
        "message": "Test completion is overdue."
    },
    {
        "from": "test_completed",
        "to": ["report_available"],
        "days": 10,
        "severity": "medium",
        "message": "Expected test report has not appeared."
    },
    {
        "from": "report_available",
        "to": [
            "follow_up_consultation",
            "doctor_consultation"
        ],
        "days": 14,
        "severity": "high",
        "message": "Potential care gap: follow-up."
    },
    {
        "from": "referral_made",
        "to": ["specialist_consultation"],
        "days": 28,
        "severity": "medium",
        "message": "Specialist consultation has not been recorded."
    },
    {
        "from": "hospital_discharge",
        "to": [
            "follow_up_consultation",
            "doctor_consultation"
        ],
        "days": 7,
        "severity": "high",
        "message": "Post-discharge follow-up may be overdue."
    }
]


def parse_date(value):
    if not value:
        return None

    try:
        return datetime.strptime(
            value,
            "%Y-%m-%d"
        ).date()
    except ValueError:
        return None


def detect_gaps(patient_id):
    events = get_patient_events(patient_id)

    if not events:
        return []

    existing_alerts = get_existing_alerts(patient_id)

    alerts_created = []

    today = datetime.now().date()

    for rule in RULES:
        source_events = [
            event for event in events
            if event["event_type"] == rule["from"]
        ]

        for source in source_events:
            source_date = parse_date(source["event_date"])

            if not source_date:
                continue

            expected_until = source_date + timedelta(
                days=rule["days"]
            )

            matching_events = []

            for event in events:
                if event["event_type"] not in rule["to"]:
                    continue

                event_date = parse_date(event["event_date"])

                if not event_date:
                    continue

                if source_date <= event_date <= expected_until:
                    matching_events.append(event)

            if matching_events:
                for alert in existing_alerts:
                    if (
                        alert["source_event_id"] == source["id"]
                        and alert["rule_from"] == rule["from"]
                    ):
                        resolve_alert(alert["id"])

                continue

            if today > expected_until:
                duplicate = False

                for alert in existing_alerts:
                    if (
                        alert["source_event_id"] == source["id"]
                        and alert["rule_from"] == rule["from"]
                        and alert["status"] == "open"
                    ):
                        duplicate = True
                        break

                if duplicate:
                    continue

                alert = create_alert(
                    patient_id=patient_id,
                    source_event_id=source["id"],
                    rule_from=rule["from"],
                    severity=rule["severity"],
                    message=rule["message"],
                    expected_by=str(expected_until)
                )

                alerts_created.append(alert)

    return alerts_created  ← Care-gap detection logic
│   ├── ai_analyzer.py   """Optional IBM watsonx.ai care-event extraction."""
import json, os, re, requests

def configured():
    return bool(os.getenv("WATSONX_APIKEY") and os.getenv("WATSONX_PROJECT_ID"))

def _iam_token(api_key):
    r=requests.post("https://iam.cloud.ibm.com/identity/token",headers={"Content-Type":"application/x-www-form-urlencoded"},data={"grant_type":"urn:ibm:params:oauth:grant-type:apikey","apikey":api_key},timeout=20)
    r.raise_for_status(); return r.json()["access_token"]

def _parse(text):
    text=text.strip(); text=re.sub(r"^```(?:json)?\s*","",text,flags=re.I); text=re.sub(r"\s*```$","",text)
    a,b=text.find("["),text.rfind("]")
    if a>=0 and b>a: return json.loads(text[a:b+1])
    a,b=text.find("{"),text.rfind("}")
    if a>=0 and b>a:
        obj=json.loads(text[a:b+1]); return obj.get("events",[]) if isinstance(obj,dict) else []
    return []

def extract_events(text):
    if not configured() or not text.strip(): return []
    base=os.getenv("WATSONX_BASE_URL","https://us-south.ml.cloud.ibm.com").rstrip("/")
    token=_iam_token(os.environ["WATSONX_APIKEY"])
    prompt=("You are CareSentinel, a healthcare care-process monitoring assistant. "
            "Extract ONLY care-process events. Do not diagnose, interpret lab values, prescribe, or infer disease. "
            "Return ONLY a JSON array. Fields: event_type,title,event_date,description,provider,notes. "
            "Allowed event_type values: doctor_consultation,test_ordered,test_completed,report_available,"
            "follow_up_consultation,prescription_issued,medication_review,referral_made,specialist_consultation,"
            "hospital_admission,hospital_discharge,procedure_performed,vaccination,other. "
            "Use YYYY-MM-DD only when explicitly present; otherwise null.\nDOCUMENT:\n"+text[:18000])
    payload={"model_id":os.getenv("WATSONX_MODEL_ID","ibm/granite-3-3-8b-instruct"),"input":prompt,"parameters":{"max_new_tokens":1200,"temperature":0.1},"project_id":os.environ["WATSONX_PROJECT_ID"]}
    r=requests.post(f"{base}/ml/v1/text/generation?version=2023-05-29",headers={"Authorization":f"Bearer {token}","Content-Type":"application/json"},json=payload,timeout=60)
    r.raise_for_status(); generated=r.json().get("results",[{}])[0].get("generated_text","")
    result=_parse(generated); return result if isinstance(result,list) else []


def _generate_json(prompt, fallback):
    """Call watsonx when configured; otherwise return a safe demo fallback."""
    if not configured():
        return fallback
    try:
        base=os.getenv("WATSONX_BASE_URL","https://us-south.ml.cloud.ibm.com").rstrip("/")
        token=_iam_token(os.environ["WATSONX_APIKEY"])
        payload={
            "model_id":os.getenv("WATSONX_MODEL_ID","ibm/granite-3-3-8b-instruct"),
            "input":prompt,
            "parameters":{"max_new_tokens":900,"temperature":0.1},
            "project_id":os.environ["WATSONX_PROJECT_ID"]
        }
        r=requests.post(f"{base}/ml/v1/text/generation?version=2023-05-29",headers={"Authorization":f"Bearer {token}","Content-Type":"application/json"},json=payload,timeout=45)
        r.raise_for_status()
        generated=r.json().get("results",[{}])[0].get("generated_text","")
        cleaned=generated.strip()
        cleaned=re.sub(r"^```(?:json)?\s*", "", cleaned, flags=re.I)
        cleaned=re.sub(r"\s*```$", "", cleaned)
        # For insight endpoints the model returns one JSON object.
        a,b=cleaned.find("{"),cleaned.rfind("}")
        if a>=0 and b>a:
            obj=json.loads(cleaned[a:b+1])
            if isinstance(obj,dict):
                return obj
        parsed=_parse(cleaned)
        if isinstance(parsed, list) and parsed:
            return parsed[0] if isinstance(parsed[0],dict) else fallback
        if isinstance(parsed, dict):
            return parsed
    except Exception as exc:
        print("watsonx.ai insight failed; using safe fallback:", exc)
    return fallback


def analyze_alert(alert, events):
    """Return an explainable, non-diagnostic AI care-process classification."""
    title=(alert or {}).get("title", "Potential care gap")
    description=(alert or {}).get("description", "")
    rule_id=(alert or {}).get("rule_id") or (alert or {}).get("alert_type") or "care_gap"

    # Specific, human-readable fallback suggestions keep the demo useful even
    # when watsonx credentials are not configured. When configured, watsonx
    # can replace this structured fallback with a generated analysis.
    rule_copy = {
        "test_not_completed": {
            "classification": "Test completion care gap",
            "summary": "A test was ordered, but the expected test-completion event was not recorded within the configured 14-day window.",
            "suggestion": "Check with the patient or caregiver whether the test was completed. If it was completed, record the test-completion event and attach the report when available.",
            "reason": "The care sequence contains a test order followed by an overdue period without a matching test-completed event.",
        },
        "report_not_available": {
            "classification": "Report availability care gap",
            "summary": "A test is recorded as completed, but a related report/result was not recorded within the configured 10-day window.",
            "suggestion": "Verify whether the report has been released and record the report-available event or upload the relevant document.",
            "reason": "A completed test is present, but the expected report step is overdue in the recorded journey.",
        },
        "followup_missing": {
            "classification": "Follow-up care gap",
            "summary": "A report is recorded, but the related follow-up consultation was not recorded within the configured 14-day window.",
            "suggestion": "Confirm with the caregiver or care team whether the follow-up occurred. If completed, record the follow-up consultation and its outcome in CareSentinel.",
            "reason": "The journey reaches a report, but the expected follow-up step is missing after its configured time window.",
        },
        "specialist_followup_missing": {
            "classification": "Specialist referral care gap",
            "summary": "A referral is recorded, but a specialist consultation was not recorded within the configured 28-day window.",
            "suggestion": "Check whether the specialist appointment was scheduled or completed. Record the specialist consultation or the updated referral status.",
            "reason": "The journey contains a referral with no matching specialist-visit event within the expected window.",
        },
        "discharge_followup_missing": {
            "classification": "Discharge follow-up care gap",
            "summary": "A hospital discharge is recorded, but the expected follow-up consultation was not recorded within the configured 7-day window.",
            "suggestion": "Verify the discharge follow-up with the caregiver or care team and record the consultation or updated care-plan status.",
            "reason": "The journey contains a discharge event with no matching follow-up consultation within the expected window.",
        },
    }
    copy=rule_copy.get(rule_id, {
        "classification": "Care-process gap",
        "summary": description or "An expected step in the recorded care journey appears to be missing or overdue.",
        "suggestion": "Review the missing step with the caregiver or care team and record the outcome in CareSentinel.",
        "reason": "The alert was triggered because an expected next care-process event was not recorded within its configured time window.",
    })
    fallback={
        "classification": copy["classification"],
        "priority": (alert or {}).get("severity") or (alert or {}).get("priority") or "medium",
        "summary": copy["summary"],
        "suggestion": copy["suggestion"],
        "reason": copy["reason"],
        "confidence": "High",
        "disclaimer": "AI-assisted workflow analysis only; not a medical diagnosis or treatment recommendation."
    }
    prompt=("You are CareSentinel's explainable care-process AI. Do not diagnose, interpret clinical values, "
            "or prescribe. Classify only the workflow gap. Return ONLY a JSON object with keys: classification, "
            "priority, summary, suggestion, reason, confidence, disclaimer. Keep each value concise and actionable. "
            "For the suggestion, explain what the caregiver/care team should verify or record; do not provide medical treatment advice. "
            "Alert:\n"+json.dumps({"title":title,"description":description,"rule_id":rule_id})+
            "\nRecent events:\n"+json.dumps(events[-12:]))
    return _generate_json(prompt, fallback)


def analyze_timeline(events, open_alerts):
    """Summarize the care journey and next workflow steps, never clinical status."""
    types=[(e.get("event_type") or "other") for e in events]
    names={
        "doctor_consultation":"consultation", "test_ordered":"test order", "test_completed":"test completion",
        "report_available":"report", "follow_up_consultation":"follow-up", "referral_made":"referral",
        "specialist_consultation":"specialist visit", "hospital_discharge":"discharge follow-up"
    }
    missing=[]
    for a in open_alerts or []:
        missing.append(a.get("title") or "Potential care gap")
    if missing:
        summary=f"The recorded journey contains {len(events)} event(s) and has {len(missing)} open care-process gap(s)."
        next_step="Review the highest-priority open gap and record the missing or completed care step."
        continuity="Needs review"
    else:
        summary=f"The recorded journey contains {len(events)} event(s) with no open care-process gaps currently detected."
        next_step="Continue recording future care events so the expected sequence stays up to date."
        continuity="On track"
    fallback={
        "journey_summary":summary,
        "continuity_status":continuity,
        "key_steps":[names.get(t,t.replace('_',' ')) for t in types[-6:]],
        "next_step":next_step,
        "gaps":missing,
        "disclaimer":"AI-assisted care-process summary only; not a medical diagnosis."
    }
    prompt=("You are CareSentinel's AI workflow summarizer. Do not diagnose or interpret medical results. "
            "Summarize only the sequence of care-process events and missing workflow steps. Return ONLY JSON with keys "
            "journey_summary, continuity_status, key_steps, next_step, gaps, disclaimer.\nEVENTS:\n"+json.dumps(events[-18:])+"\nOPEN ALERTS:\n"+json.dumps(open_alerts or []))
    result=_generate_json(prompt, fallback)
    if not isinstance(result,dict): return fallback
    return result
  ← IBM watsonx.ai / Granite AI analysis
│   ├── seed_data.py     from datetime import datetime, timedelta

import database
import gap_detector


def date_days_ago(days):
    return (datetime.now() - timedelta(days=days)).strftime("%Y-%m-%d")


def seed():
    database.init_db()

    # Don't create duplicate patients every time
    # the Flask server restarts.
    existing_patients = database.get_all_patients()

    if existing_patients:
        return

    # =====================================================
    # MARGARET THOMPSON
    # Intentionally has a missing follow-up.
    # This should produce a HIGH severity care-gap alert.
    # =====================================================

    margaret_id = database.insert_patient(
        name="Margaret Thompson",
        dob="1948-03-12",
        age=78,
        caregiver="Susan Thompson",
        notes="Synthetic demo patient for CareSentinel."
    )

    database.insert_event(
        patient_id=margaret_id,
        event_type="doctor_consultation",
        title="Doctor consultation",
        event_date=date_days_ago(38),
        provider="Dr. Sarah Wilson",
        notes="Routine consultation."
    )

    database.insert_event(
        patient_id=margaret_id,
        event_type="test_ordered",
        title="Blood test ordered",
        event_date=date_days_ago(36),
        provider="Dr. Sarah Wilson",
        notes="Blood test requested."
    )

    database.insert_event(
        patient_id=margaret_id,
        event_type="test_completed",
        title="Blood test completed",
        event_date=date_days_ago(33),
        provider="City Diagnostics",
        notes="Test completed."
    )

    database.insert_event(
        patient_id=margaret_id,
        event_type="report_available",
        title="Blood test report available",
        event_date=date_days_ago(30),
        provider="City Diagnostics",
        notes="Report recorded in the care journey."
    )

    # No follow-up event is added intentionally.
    # Gap detection should identify it.

    # =====================================================
    # ROBERT DAVIES
    # Complete care journey.
    # Should NOT have an open care-gap alert.
    # =====================================================

    robert_id = database.insert_patient(
        name="Robert Davies",
        dob="1944-07-29",
        age=82,
        caregiver="James Davies",
        notes="Synthetic demo patient with completed follow-up."
    )

    database.insert_event(
        patient_id=robert_id,
        event_type="doctor_consultation",
        title="Doctor consultation",
        event_date=date_days_ago(55),
        provider="Dr. Michael Brown",
        notes="Routine consultation."
    )

    database.insert_event(
        patient_id=robert_id,
        event_type="test_ordered",
        title="Blood test ordered",
        event_date=date_days_ago(54),
        provider="Dr. Michael Brown",
        notes="Blood test requested."
    )

    database.insert_event(
        patient_id=robert_id,
        event_type="test_completed",
        title="Blood test completed",
        event_date=date_days_ago(52),
        provider="City Diagnostics",
        notes="Test completed."
    )

    database.insert_event(
        patient_id=robert_id,
        event_type="report_available",
        title="Blood test report available",
        event_date=date_days_ago(49),
        provider="City Diagnostics",
        notes="Report recorded."
    )

    database.insert_event(
        patient_id=robert_id,
        event_type="follow_up_consultation",
        title="Follow-up consultation",
        event_date=date_days_ago(43),
        provider="Dr. Michael Brown",
        notes="Follow-up completed."
    )

    # =====================================================
    # RUN CARE-GAP DETECTION
    # =====================================================

    gap_detector.run_gap_detection(margaret_id)
    gap_detector.run_gap_detection(robert_id)


if __name__ == "__main__":
    seed()
    print("CareSentinel demo data seeded successfully.") ← Synthetic demonstration data
│   ├── data/           import sqlite3
import json
from pathlib import Path
from datetime import datetime


# ============================================================
# DATABASE SETUP
# ============================================================

BASE_DIR = Path(__file__).resolve().parent
DATA_DIR = BASE_DIR / "data"
DB_PATH = DATA_DIR / "caresentinel.db"

DATA_DIR.mkdir(exist_ok=True)


# ============================================================
# CONNECTION
# ============================================================

def get_connection():
    connection = sqlite3.connect(
        DB_PATH
    )

    connection.row_factory = sqlite3.Row

    return connection


# ============================================================
# INITIALIZE DATABASE
# ============================================================

def init_db():
    connection = get_connection()
    cursor = connection.cursor()

    cursor.execute("""
        CREATE TABLE IF NOT EXISTS patients (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            dob TEXT,
            age INTEGER,
            caregiver TEXT,
            notes TEXT,
            profile_photo TEXT,
            created_at TEXT DEFAULT CURRENT_TIMESTAMP
        )
    """)

    cursor.execute("""
        CREATE TABLE IF NOT EXISTS events (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            patient_id INTEGER NOT NULL,
            event_type TEXT NOT NULL,
            title TEXT,
            event_date TEXT,
            description TEXT,
            provider TEXT,
            notes TEXT,
            created_at TEXT DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (patient_id)
                REFERENCES patients(id)
                ON DELETE CASCADE
        )
    """)

    cursor.execute("""
        CREATE TABLE IF NOT EXISTS documents (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            patient_id INTEGER NOT NULL,
            filename TEXT NOT NULL,
            original_filename TEXT NOT NULL,
            uploaded_at TEXT DEFAULT CURRENT_TIMESTAMP,
            FOREIGN KEY (patient_id)
                REFERENCES patients(id)
                ON DELETE CASCADE
        )
    """)

    cursor.execute("""
        CREATE TABLE IF NOT EXISTS alerts (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            patient_id INTEGER NOT NULL,
            alert_type TEXT,
            rule_id TEXT,
            severity TEXT,
            title TEXT,
            description TEXT,
            evidence_chain TEXT,
            status TEXT DEFAULT 'open',
            resolved_by_event_id INTEGER,
            created_at TEXT DEFAULT CURRENT_TIMESTAMP,
            resolved_at TEXT,
            FOREIGN KEY (patient_id)
                REFERENCES patients(id)
                ON DELETE CASCADE,
            FOREIGN KEY (resolved_by_event_id)
                REFERENCES events(id)
        )
    """)

    alert_columns = {row[1] for row in cursor.execute("PRAGMA table_info(alerts)").fetchall()}
    if "rule_id" not in alert_columns:
        cursor.execute("ALTER TABLE alerts ADD COLUMN rule_id TEXT")

    patient_columns = {row[1] for row in cursor.execute("PRAGMA table_info(patients)").fetchall()}
    if "profile_photo" not in patient_columns:
        cursor.execute("ALTER TABLE patients ADD COLUMN profile_photo TEXT")
    cursor.execute("""CREATE TABLE IF NOT EXISTS documents (
        id INTEGER PRIMARY KEY AUTOINCREMENT, patient_id INTEGER NOT NULL, filename TEXT NOT NULL,
        original_filename TEXT NOT NULL, uploaded_at TEXT DEFAULT CURRENT_TIMESTAMP,
        FOREIGN KEY (patient_id) REFERENCES patients(id) ON DELETE CASCADE)""")
    connection.commit()
    connection.close()


# ============================================================
# HELPER
# ============================================================

def row_to_dict(row):
    if row is None:
        return None

    return dict(row)


def rows_to_dict(rows):
    return [dict(row) for row in rows]


# ============================================================
# PATIENTS
# ============================================================

def get_all_patients():
    connection = get_connection()

    rows = connection.execute("""
        SELECT *
        FROM patients
        ORDER BY name
    """).fetchall()

    connection.close()

    return rows_to_dict(rows)


def get_patient(patient_id):
    connection = get_connection()

    row = connection.execute("""
        SELECT *
        FROM patients
        WHERE id = ?
    """, (patient_id,)).fetchone()

    connection.close()

    return row_to_dict(row)


def insert_patient(
    name,
    dob=None,
    age=None,
    caregiver=None,
    notes=None
):
    connection = get_connection()

    cursor = connection.execute("""
        INSERT INTO patients
        (name, dob, age, caregiver, notes)
        VALUES (?, ?, ?, ?, ?)
    """, (
        name,
        dob,
        age,
        caregiver,
        notes
    ))

    patient_id = cursor.lastrowid

    connection.commit()
    connection.close()

    return patient_id


def create_patient(
    name,
    dob=None,
    age=None,
    caregiver=None,
    notes=None
):
    return insert_patient(
        name=name,
        dob=dob,
        age=age,
        caregiver=caregiver,
        notes=notes
    )


# ============================================================
# PATIENT PROFILE
# ============================================================

def update_patient_profile_photo(patient_id, filename):
    connection=get_connection()
    connection.execute("UPDATE patients SET profile_photo=? WHERE id=?",(filename,patient_id))
    connection.commit(); connection.close()


# ============================================================
# EVENTS
# ============================================================

def get_events_for_patient(patient_id):
    connection = get_connection()

    rows = connection.execute("""
        SELECT *
        FROM events
        WHERE patient_id = ?
        ORDER BY event_date ASC, id ASC
    """, (patient_id,)).fetchall()

    connection.close()

    return rows_to_dict(rows)


def get_patient_events(patient_id):
    return get_events_for_patient(patient_id)


def get_event_by_id(event_id):
    connection = get_connection()

    row = connection.execute("""
        SELECT *
        FROM events
        WHERE id = ?
    """, (event_id,)).fetchone()

    connection.close()

    return row_to_dict(row)


def get_event(event_id):
    return get_event_by_id(event_id)


def insert_event(
    patient_id,
    event_type,
    title=None,
    event_date=None,
    description="",
    provider="",
    notes=""
):
    connection = get_connection()

    cursor = connection.execute("""
        INSERT INTO events
        (
            patient_id,
            event_type,
            title,
            event_date,
            description,
            provider,
            notes
        )
        VALUES (?, ?, ?, ?, ?, ?, ?)
    """, (
        patient_id,
        event_type,
        title,
        event_date,
        description,
        provider,
        notes
    ))

    event_id = cursor.lastrowid

    connection.commit()
    connection.close()

    return event_id


def create_event(
    patient_id,
    event_type,
    title=None,
    event_date=None,
    description="",
    provider="",
    notes=""
):
    return insert_event(
        patient_id=patient_id,
        event_type=event_type,
        title=title,
        event_date=event_date,
        description=description,
        provider=provider,
        notes=notes
    )


# ============================================================
# ALERTS
# ============================================================

def insert_alert(
    patient_id,
    alert_type=None,
    severity=None,
    title=None,
    description=None,
    evidence_chain=None,
    status="open",
    rule_id=None
):
    if isinstance(evidence_chain, (list, dict)):
        evidence_chain = json.dumps(evidence_chain)

    connection = get_connection()

    cursor = connection.execute("""
        INSERT INTO alerts
        (patient_id, alert_type, rule_id, severity, title, description, evidence_chain, status)
        VALUES (?, ?, ?, ?, ?, ?, ?, ?)
    """, (
        patient_id, alert_type, rule_id, severity, title, description, evidence_chain, status
    ))

    alert_id = cursor.lastrowid
    connection.commit()
    connection.close()
    return alert_id


def create_alert(
    patient_id,
    alert_type=None,
    severity=None,
    title=None,
    description=None,
    evidence_chain=None,
    status="open",
    rule_id=None
):
    return insert_alert(
        patient_id=patient_id,
        alert_type=alert_type,
        severity=severity,
        title=title,
        description=description,
        evidence_chain=evidence_chain,
        status=status,
        rule_id=rule_id
    )


def get_alert_by_id(alert_id):
    connection = get_connection()

    row = connection.execute("""
        SELECT *
        FROM alerts
        WHERE id = ?
    """, (alert_id,)).fetchone()

    connection.close()

    return row_to_dict(row)


def get_alert(alert_id):
    return get_alert_by_id(alert_id)


def get_all_alerts(
    status=None,
    patient_id=None
):
    connection = get_connection()

    query = """
        SELECT *
        FROM alerts
        WHERE 1 = 1
    """

    params = []

    if status:
        query += " AND status = ?"
        params.append(status)

    if patient_id:
        query += " AND patient_id = ?"
        params.append(patient_id)

    query += """
        ORDER BY created_at DESC, id DESC
    """

    rows = connection.execute(
        query,
        params
    ).fetchall()

    connection.close()

    return rows_to_dict(rows)


def get_alerts_for_patient(
    patient_id,
    status=None
):
    return get_all_alerts(
        status=status,
        patient_id=patient_id
    )


def get_open_alerts_for_patient(
    patient_id
):
    return get_alerts_for_patient(
        patient_id,
        status="open"
    )


def resolve_alert(
    alert_id,
    resolved_by_event_id=None
):
    connection = get_connection()

    connection.execute("""
        UPDATE alerts
        SET
            status = 'resolved',
            resolved_by_event_id = ?,
            resolved_at = ?
        WHERE id = ?
    """, (
        resolved_by_event_id,
        datetime.now().isoformat(
            timespec="seconds"
        ),
        alert_id
    ))

    connection.commit()
    connection.close()
    return get_alert_by_id(alert_id)


# ============================================================
# DOCUMENTS
# ============================================================

def insert_document(patient_id, filename, original_filename=None):
    connection = get_connection()
    cursor = connection.execute("""
        INSERT INTO documents (patient_id, filename, original_filename)
        VALUES (?, ?, ?)
    """, (patient_id, filename, original_filename or filename))
    connection.commit()
    document_id = cursor.lastrowid
    connection.close()
    return document_id


def get_documents_for_patient(patient_id):
    connection = get_connection()
    rows = connection.execute("""
        SELECT id, patient_id, filename, original_filename, uploaded_at
        FROM documents
        WHERE patient_id = ?
        ORDER BY uploaded_at DESC, id DESC
    """, (patient_id,)).fetchall()
    connection.close()
    return rows_to_dict(rows)


# ============================================================
# PATIENT SUMMARY
# ============================================================

def get_patient_summary(patient_id):
    patient = get_patient(patient_id)

    if not patient:
        return None

    events = get_events_for_patient(
        patient_id
    )

    alerts = get_alerts_for_patient(
        patient_id
    )

    open_alerts = [
        alert
        for alert in alerts
        if alert.get("status") == "open"
    ]

    return {
        "patient": patient,
        "events": events,
        "alerts": alerts,
        "open_alerts": open_alerts,
        "event_count": len(events),
        "alert_count": len(alerts),
        "open_alert_count": len(open_alerts)
    }


def get_dashboard_data():
    patients = get_all_patients()

    result = []

    for patient in patients:
        summary = get_patient_summary(
            patient["id"]
        )

        result.append(summary)

    return result


# ============================================================
# CLEAR DATABASE
# ============================================================

def clear_database():
    connection = get_connection()

    connection.execute(
        "DELETE FROM alerts"
    )

    connection.execute(
        "DELETE FROM events"
    )

    connection.execute(
        "DELETE FROM patients"
    )

    connection.commit()
    connection.close()  ← SQLite database
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
