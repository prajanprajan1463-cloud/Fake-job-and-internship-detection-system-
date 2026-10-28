# JobSafeGuard — Fake Job & Internship Detection System

[![Python](https://img.shields.io/badge/Python-3.11%20%7C%203.12%20%7C%203.13-blue.svg)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-3.1.3-green.svg)](https://flask.palletsprojects.com/)
[![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-1.9.1-orange.svg)](https://scikit-learn.org/)
[![Tests](https://img.shields.io/badge/pytest-36%20passed-brightgreen.svg)](https://docs.pytest.org/)
[![License](https://img.shields.io/badge/License-MIT-lightgrey.svg)](LICENSE)

A full-stack cybersecurity web application designed to help students, recent graduates, and job seekers identify potentially fraudulent employment advertisements and internship scams. The platform employs natural language processing (NLP), rule-based threat heuristics, and explainable multi-factor risk scoring to safeguard applicants from advance-fee fraud, cashier's check schemes, and identity harvesting.

Developed as a **BCA Final Year Capstone Project**.

---

## Table of Contents
1. [Core Features](#core-features)
2. [Technology Stack](#technology-stack)
3. [Architecture & Machine Learning Methodology](#architecture--machine-learning-methodology)
4. [Project Structure](#project-structure)
5. [User Interface & Screenshots](#user-interface--screenshots)
6. [Windows Installation & Setup (PowerShell)](#windows-installation--setup-powershell)
7. [Running the Application](#running-the-application)
8. [Administrator Account Setup](#administrator-account-setup)
9. [Running Automated Tests](#running-automated-tests)
10. [Production Deployment Guide](#production-deployment-guide)
11. [Privacy & Security Best Practices](#privacy--security-best-practices)
12. [System Limitations & Ethical Notice](#system-limitations--ethical-notice)

---

## Core Features

- **Multi-Factor Threat Detection Engine:**
  - Evaluates job postings across NLP feature vectors and heuristic threat rules.
  - Detects upfront fee solicitations (training, laptop deposits, registration fees).
  - Flags sensitive credential harvesting (online banking passwords, OTPs, CVV, identity cards).
  - Identifies equipment check overpayment schemes.
  - Recognizes chat-only recruitment channels (Telegram, WhatsApp, Signal).
  - Flags impersonation via free webmail addresses (`@gmail.com`, `@yahoo.com`).
  - Spots unrealistic salary claims and artificial urgency tactics.

- **Explainable Risk Scoring (0–100):**
  - **0–29:** *Lower Risk Indicators*
  - **30–59:** *Needs Further Verification*
  - **60–100:** *Potentially Suspicious*
  - Provides exact text evidence quotes explaining *why* each indicator was triggered.
  - Recognizes verified positive signals (live video interview, structured job scope, corporate domain).

- **Candidate Dashboard & Telemetry:**
  - Dynamic Chart.js donut chart reflecting user's historical threat evaluations.
  - Summary metric cards (Total Screenings, High Risk, Needs Verification, Lower Risk).
  - Searchable, filterable, and paginated personal analysis history.

- **Enterprise-Grade Authentication & Security:**
  - Role-Based Access Control (RBAC): Candidate and Administrator roles.
  - Werkzeug password hashing (scrypt / pbkdf2).
  - Form-level CSRF token validation with Flask-WTF.
  - Open-redirect defense with host validation.
  - User data isolation (strict server-side ownership checks on all records).
  - Password visibility toggles and double-submission protection.

- **Administrator Command Center:**
  - Global telemetry: total registered candidates, total analyses, and threat distribution.
  - User account management (role elevation/demotion, account deletion with cascade).
  - Audit trail of system-wide job screening records.
  - Dedicated CLI tool for first-time administrator creation.

---

## Technology Stack

| Layer | Technology | Details |
| :--- | :--- | :--- |
| **Backend** | Python 3.11+ / Flask | Application factory pattern, modular Blueprints |
| **Database ORM** | Flask-SQLAlchemy / SQLite | Easily migrates to PostgreSQL via `DATABASE_URL` |
| **Authentication** | Flask-Login & Werkzeug | Session cookies (`HttpOnly`, `SameSite=Lax`), scrypt hashing |
| **Forms & CSRF** | Flask-WTF / WTForms | Server-side validation, password policy, CSRF protection |
| **Machine Learning** | Scikit-learn, Pandas, NumPy | TF-IDF Vectorizer + Logistic Regression + Rule-based heuristics |
| **Artifacts** | Joblib | Pre-trained model & TF-IDF vectorizer serialization |
| **Frontend** | HTML5, CSS3, Vanilla JavaScript | Cybersecurity SaaS design system (Navy, Teal, Off-white) |
| **Components** | Bootstrap 5, Bootstrap Icons | Responsive grid, accessible modals, tooltips |
| **Visualizations** | Chart.js 4.4 | Doughnut risk telemetry charts |
| **Testing** | Pytest 9.2+ | 36 unit and integration test suites |

---

## Architecture & Machine Learning Methodology

```
+-------------------------------------------------------------+
|                     User Submits Posting                    |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
|       1. Text Preprocessing (ml/text_preprocessor.py)       |
|   - Strips HTML injection & non-breaking unicode spaces     |
|   - Tokenizes and extracts URLs, emails, phone numbers      |
+-------------------------------------------------------------+
           |                                       |
           v                                       v
+-----------------------+              +-----------------------+
|  2. Heuristic Rules   |              |  3. ML Classifier     |
|  (ml/detector.py)     |              |  (ml/train_model.py)  |
| - Upfront fee check   |              | - TF-IDF n-grams      |
| - Banking/OTP check   |              | - Logistic Regression |
| - Check scam check    |              | - Probability of fake |
| - Chat recruiter check|              +-----------------------+
+-----------------------+                          |
           |                                       |
           +-------------------+-------------------+
                               |
                               v
+-------------------------------------------------------------+
|           4. Risk Scoring Engine (ml/risk_scoring.py)       |
|   - Combined score: (60% Heuristic + 40% ML Probability)    |
|   - Mitigation discount for positive signals (up to -10)    |
|   - Safety floor: Score >= 60 if critical red flag present  |
|   - Bounded strictly within [0.0, 100.0]                    |
+-------------------------------------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|    5. Result Report: Score, Categorization & Evidence       |
+-------------------------------------------------------------+
```

### Machine Learning Details:
- **Feature Extraction:** TF-IDF n-grams (unigrams and bigrams), sublinear term frequency scaling, English stop-word filtering.
- **Classifier:** Calibrated Logistic Regression with balanced class weighting.
- **Training Pipeline (`ml/train_model.py`):** Train/test split with stratification and reproducible seed (`random_state=42`).
- **Graceful Fallback:** If the serialized ML model artifact is missing or disabled, the application continues operating seamlessly using high-precision heuristic rule scoring without failing.

---

## Project Structure

```
fake-job-internship-detection/
│
├── app/
│   ├── __init__.py               # Application factory & error handlers
│   ├── extensions.py             # SQLAlchemy, LoginManager, CSRF instances
│   ├── models.py                 # User & JobAnalysis database models
│   ├── admin/                    # Admin console blueprint
│   │   ├── __init__.py
│   │   └── routes.py
│   ├── analysis/                 # Job analysis & dashboard blueprint
│   │   ├── __init__.py
│   │   ├── forms.py
│   │   └── routes.py
│   ├── auth/                     # Authentication & profile blueprint
│   │   ├── __init__.py
│   │   ├── forms.py
│   │   └── routes.py
│   ├── main/                     # Public marketing & informational blueprint
│   │   ├── __init__.py
│   │   └── routes.py
│   ├── services/                 # Business logic service layer
│   │   ├── __init__.py
│   │   └── analyzer_service.py
│   ├── static/                   # Static design assets
│   │   ├── css/style.css         # Cybersecurity SaaS design system
│   │   ├── js/main.js            # Password toggles, spinner, Chart.js
│   │   └── img/logo.svg          # Vector shield logo
│   └── templates/                # Jinja2 HTML templates
│       ├── base.html             # Common layout & navigation
│       ├── admin/                # Admin dashboards & management views
│       ├── analysis/             # Dashboard, analyze form, result, history
│       ├── auth/                 # Sign in, register, profile views
│       ├── errors/               # 400, 403, 404, 500 pages
│       └── main/                 # Landing page, How It Works, Safety Tips
│
├── ml/
│   ├── __init__.py
│   ├── text_preprocessor.py      # Unicode normalizer & entity extractor
│   ├── detector.py               # Rule-based heuristics & evidence extractor
│   ├── risk_scoring.py           # Multi-factor score fusion & calibration
│   ├── dataset.py                # Reference benchmark dataset loader
│   ├── train_model.py            # Reproducible model training pipeline
│   └── artifacts/                # Serialized model, vectorizer & report
│       ├── job_classifier.joblib
│       ├── tfidf_vectorizer.joblib
│       └── training_report.json
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py               # Fixtures, test database, client setup
│   ├── test_admin.py             # RBAC & administrative operation tests
│   ├── test_analysis.py          # Form submission, isolation, deletion tests
│   ├── test_auth.py              # Registration, login, password policy tests
│   ├── test_detector.py          # Heuristic pattern matching & evidence tests
│   ├── test_scoring.py           # Score boundaries, fallback, threshold tests
│   └── test_security.py          # CSRF, open redirects, hash verification
│
├── instance/                     # Local SQLite database (git-ignored)
├── config.py                     # Development, Testing, Production configs
├── run.py                        # Entrypoint & CLI command suite
├── requirements.txt              # Production & development dependencies
├── .env.example                  # Environment configuration template
├── .gitignore                    # Git exclusions
├── Procfile                      # Cloud deployment configuration (Heroku/Render)
└── README.md                     # Comprehensive project documentation
```

---

## User Interface & Screenshots

The user interface follows a modern **Cybersecurity SaaS design language**:
- **Palette:** Deep Navy (`#0f172a`), Cyan Accent (`#0ea5e9`), Off-White (`#f8fafc`).
- **Semantic Indicators:** Emerald Green (Low Risk), Amber (Needs Verification), Crimson (High Risk).
- **Components:** Metric tiles, elevated cards, interactive doughnut chart, risk progress meter, collapsible evidence quotes.

```
+-----------------------------------------------------------------------+
|  [Screenshot Placeholder: Landing Page - Hero & Feature Cards]        |
|  Caption: Modern SaaS landing page explaining employment fraud defense|
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|  [Screenshot Placeholder: Candidate Dashboard & Chart]                |
|  Caption: Real-time risk distribution chart and metric overview       |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|  [Screenshot Placeholder: Risk Assessment Report]                     |
|  Caption: Numerical risk meter, detected red flags with quotes        |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|  [Screenshot Placeholder: Admin Console & User Management]            |
|  Caption: Global telemetry and role governance console                |
+-----------------------------------------------------------------------+
```

---

## Windows Installation & Setup (PowerShell)

Follow these exact steps to set up and run the project locally on Windows:

### Step 1: Open PowerShell and Navigate to the Project Folder
```powershell
cd "c:\Users\praja\Downloads\Fake job detection"
```

### Step 2: Create a Virtual Environment
```powershell
python -m venv .venv
```

### Step 3: Activate the Virtual Environment
```powershell
.\.venv\Scripts\Activate.ps1
```

> **PowerShell ExecutionPolicy Note:**
> If PowerShell displays an error stating that script execution is disabled (`PSSecurityException`), run this safe command in your current session without modifying system-wide security policies:
> ```powershell
> Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
> .\.venv\Scripts\Activate.ps1
> ```

### Step 4: Install Dependencies
```powershell
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### Step 5: Configure Environment Variables
Copy the example configuration to create your local `.env`:
```powershell
Copy-Item .env.example .env
```
*(Optional)* You can open `.env` and set a custom `SECRET_KEY`.

### Step 6: Initialize Database & Train Model
```powershell
python run.py init-db
python ml/train_model.py
```

---

## Running the Application

Launch the local development server:
```powershell
python run.py
```

The application will start immediately. Open your browser and navigate to:
```
http://127.0.0.1:5000
```

---

## Administrator Account Setup

For security, the application **never** ships with a default or public administrator password. Create your first administrator account using the secure CLI command:

### Interactive Prompt Method:
```powershell
python run.py create-admin
```
You will be prompted to enter:
- Admin email address: `admin@example.com`
- Admin full name: `System Administrator`
- Password (hidden input with confirmation prompt)

### Direct Command-Line Method:
```powershell
python run.py create-admin --email admin@example.com --name "System Administrator" --password "YourSecurePassword123"
```

Once created, log in at `http://127.0.0.1:5000/auth/login`. You will see the **Admin Panel** link in the navigation bar.

---

## Running Automated Tests

Run the full pytest suite with verbose output:
```powershell
python -m pytest -v
```

### Verified Test Results:
```text
============================= test session starts =============================
platform win32 -- Python 3.13.16, pytest-9.2.0, pluggy-1.7.0
rootdir: C:\Users\praja\Downloads\Fake job detection
collected 36 items

tests/test_admin.py::test_admin_route_protection_unauthenticated PASSED  [  2%]
tests/test_admin.py::test_admin_route_protection_normal_user PASSED      [  5%]
tests/test_admin.py::test_admin_dashboard_access PASSED                  [  8%]
tests/test_admin.py::test_admin_cannot_toggle_self PASSED                [ 11%]
tests/test_admin.py::test_admin_toggle_other_user_role PASSED            [ 13%]
tests/test_admin.py::test_admin_delete_user PASSED                       [ 16%]
tests/test_analysis.py::test_job_analysis_workflow PASSED                [ 19%]
tests/test_analysis.py::test_analysis_empty_submission_rejected PASSED   [ 22%]
tests/test_analysis.py::test_analysis_invalid_url_rejected PASSED        [ 25%]
tests/test_analysis.py::test_user_data_isolation PASSED                  [ 27%]
tests/test_analysis.py::test_delete_analysis_by_owner PASSED             [ 30%]
tests/test_analysis.py::test_nonexistent_analysis_404 PASSED             [ 33%]
tests/test_auth.py::test_user_registration PASSED                        [ 36%]
tests/test_auth.py::test_duplicate_email_rejection PASSED                [ 38%]
tests/test_auth.py::test_password_policy_enforcement PASSED              [ 41%]
tests/test_auth.py::test_successful_login PASSED                         [ 44%]
tests/test_auth.py::test_failed_login_invalid_password PASSED            [ 47%]
tests/test_auth.py::test_failed_login_nonexistent_email PASSED           [ 50%]
tests/test_auth.py::test_logout PASSED                                   [ 52%]
tests/test_auth.py::test_protected_routes_redirect_unauthenticated PASSED [ 55%]
tests/test_detector.py::test_clean_text_normalizer PASSED                [ 58%]
tests/test_detector.py::test_upfront_fee_detection PASSED                [ 61%]
tests/test_detector.py::test_banking_credentials_detection PASSED        [ 63%]
tests/test_detector.py::test_equipment_check_scam_detection PASSED       [ 66%]
tests/test_detector.py::test_chat_app_recruitment_detection PASSED       [ 69%]
tests/test_detector.py::test_free_webmail_detection PASSED               [ 72%]
tests/test_detector.py::test_positive_indicators_detection PASSED        [ 75%]
tests/test_detector.py::test_recommended_actions_generation PASSED       [ 77%]
tests/test_scoring.py::test_score_boundaries_clean_posting PASSED        [ 80%]
tests/test_scoring.py::test_score_critical_red_flag_safety_floor PASSED  [ 83%]
tests/test_scoring.py::test_graceful_handling_missing_ml_artifacts PASSED [ 86%]
tests/test_scoring.py::test_custom_threshold_configuration PASSED        [ 88%]
tests/test_security.py::test_password_hash_strength PASSED               [ 91%]
tests/test_security.py::test_open_redirect_protection PASSED             [ 94%]
tests/test_security.py::test_is_safe_url_logic PASSED                    [ 97%]
tests/test_security.py::test_csrf_protection_when_active PASSED          [100%]

======================= 36 passed, 6 warnings in 13.69s =======================
```

---

## Production Deployment Guide

### Deploying to Render / Railway / Heroku

1. **Push your code to GitHub:**
   ```powershell
   git init
   git add .
   git commit -m "Initial commit: Fake Job and Internship Detection System"
   git branch -M main
   git remote add origin https://github.com/your-username/fake-job-internship-detection.git
   git push -u origin main
   ```

2. **Web Service Configuration:**
   - **Build Command:** `pip install -r requirements.txt && python ml/train_model.py`
   - **Start Command:** `gunicorn "run:create_app('production')" --bind 0.0.0.0:$PORT`

3. **Required Environment Variables:**
   - `FLASK_ENV=production`
   - `DEBUG=False`
   - `SECRET_KEY`: Generate a random 64-character hex string using `python -c "import secrets; print(secrets.token_hex(32))"`.
   - `DATABASE_URL`: Attach a managed PostgreSQL database (e.g. `postgresql://user:password@hostname:5432/dbname`).
   - `SESSION_COOKIE_SECURE=True`

---

## Privacy & Security Best Practices

- **Zero-Storage of Plaintext Credentials:** Passwords are salted and hashed using Werkzeug scrypt/pbkdf2.
- **SSRF Prevention:** The backend does **not** fetch or scrape arbitrary user-submitted URLs. URLs are validated purely by structure without outbound requests.
- **CSRF Token Protection:** All POST requests (login, registration, job submission, deletions) are protected by synchronized cryptographic tokens.
- **User Record Isolation:** Database queries enforce `user_id == current_user.id` on all individual records. Unauthorized requests are rejected with HTTP 403.
- **Strict Cookie Flags:** Session cookies are configured with `HttpOnly` and `SameSite=Lax`.

---

## System Limitations & Ethical Notice

1. **Automated Screening vs. Formal Fraud Proof:** This system is an automated statistical screening aid. A high risk score indicates the presence of recognized fraud indicators; it does not constitute legal proof that an organization is illegitimate.
2. **Evolving Scammer Tactics:** Adversaries continuously adapt phrasing and communication handles. The absence of detected keywords does not guarantee that a job posting is 100% genuine.
3. **Synthetic Dataset Notice:** The embedded benchmark dataset (`ml/dataset.py`) is explicitly curated for academic demonstration and reproducible pipeline testing. For production deployments, models should be trained on expanded real-world corpora such as the University of the Aegean Employment Scam Aegean Dataset (EMScAD).

---

## License

This project is licensed under the MIT License — feel free to use and adapt it for educational and academic research.

