SENTIAI
Customer Sentiment Intelligence Platform

Technical Documentation & Project Report
Version 1.0  |  May 2026  |  Full-Stack NLP Application
 
1. Project Overview
SentiAI is a full-stack web application that classifies customer reviews into Positive, Negative, or Neutral sentiment categories using Natural Language Processing (NLP). The platform is designed to help businesses gain actionable insights from unstructured customer feedback at scale.

The system leverages VADER (Valence Aware Dictionary and sEntiment Reasoner) as its primary model, with an optional Logistic Regression classifier (scikit-learn) for domain-specific accuracy improvements.

Core Technologies
Python 3.9+ · Flask 3.0 · NLTK/VADER · scikit-learn · ReportLab · Chart.js · HTML5/CSS3/JavaScript

1.1 Project Objectives
•	Provide real-time sentiment classification for individual and bulk customer reviews
•	Enable CSV batch upload and automated PDF report generation
•	Support optional custom ML model training on labelled domain data
•	Deliver a secure, authenticated multi-user web platform
•	Visualise sentiment trends through interactive Chart.js dashboards

1.2 System Architecture Summary
Layer	Technology	Responsibility
Presentation	HTML5, CSS3, JavaScript	UI, Chart.js visualisations
Application	Flask 3.0 (Python)	REST API, session management, routing
NLP Engine	NLTK VADER + scikit-learn	Text preprocessing, sentiment prediction
Report Layer	ReportLab	PDF report generation
Data Store	JSON flat-file	User credential storage
Email	smtplib / Gmail SMTP	Welcome email on registration

 
2. Software Development Life Cycle (SDLC)
SentiAI was developed following the Agile SDLC model, executed in iterative sprints. The phases below describe the full development journey from conception to deployment.

2.1 Phase 1 — Requirements Gathering
Stakeholder interviews and competitive analysis were conducted to define system requirements.
•	Functional Requirements: user authentication, text input, CSV upload, sentiment classification, PDF export
•	Non-Functional Requirements: response time < 2 seconds, HTTPS-ready, scalable to 1 000+ reviews per batch
•	Constraints: open-source stack only, deployable on standard Linux VPS

2.2 Phase 2 — System Design
•	Architecture design: MVC-inspired separation (Flask routes → SentimentEngine → report_generator)
•	Database design: flat-file JSON for MVP; schema ready for PostgreSQL migration
•	UI wireframes: login, signup, dashboard with sidebar navigation
•	API contract defined (see Section 7)

2.3 Phase 3 — Implementation
•	Sprint 1: Project scaffolding, Flask app, routing, session management
•	Sprint 2: VADER integration, SentimentEngine class, text preprocessing pipeline
•	Sprint 3: Frontend dashboard, Chart.js integration, bulk analysis
•	Sprint 4: CSV upload endpoint, PDF report generation (ReportLab)
•	Sprint 5: User authentication, email welcome flow, login/signup pages
•	Sprint 6: Custom ML training script (train_model.py), model persistence

2.4 Phase 4 — Testing
Test Type	Scope	Tool / Method
Unit Testing	SentimentEngine.predict(), _clean(), _build_summary()	pytest / manual
Integration Testing	Flask route → engine → JSON response	Postman, curl
UI Testing	Login, signup, dashboard flows	Manual + Chrome DevTools
Performance Testing	Bulk analysis (500 reviews)	Locust (load test)
Security Testing	Authentication, session expiry, input sanitisation	Manual OWASP checklist

2.5 Phase 5 — Deployment
•	Containerised with Docker (optional Compose file for production)
•	Environment variables (MAIL_USER, MAIL_PASS) configured via .env file
•	Reverse proxy via Nginx with SSL termination
•	Process management via systemd or Gunicorn + supervisor

2.6 Phase 6 — Maintenance
•	Version control via Git / GitHub with branch-per-feature workflow
•	Logging: Flask DEBUG mode (dev), rotating file handler (prod)
•	VADER lexicon auto-downloaded on first run via NLTK downloader
•	ML model versioned as data/ml_model.pkl; retrained as new labelled data accumulates

 
3. Agile Methodology
SentiAI followed the Scrum framework within an Agile methodology. Development was structured into six one-week sprints with clearly defined ceremonies and artefacts.

3.1 Scrum Team Structure
Role	Responsibilities
Product Owner	Defines user stories, prioritises backlog, accepts deliverables
Scrum Master	Facilitates ceremonies, removes blockers, enforces process
Dev Team	Designs, implements, tests all features across front and back end

3.2 Sprint Backlog Summary
Sprint	Goal	Key Stories	Status
Sprint 1	Project foundation	Scaffold Flask app, routing, templates	Done
Sprint 2	NLP core	VADER engine, text cleaning, prediction API	Done
Sprint 3	Dashboard UI	Charts, bulk analysis, filter list	Done
Sprint 4	Data I/O	CSV upload, PDF report download	Done
Sprint 5	Auth system	Login, signup, session, welcome email	Done
Sprint 6	ML training	train_model.py, model persistence, switching	Done

3.3 Agile Ceremonies
•	Sprint Planning (Monday): team selects stories from backlog, estimates with story points
•	Daily Standups: 15-minute syncs — what was done, what is planned, any blockers
•	Sprint Review (Friday): demo working software to product owner
•	Sprint Retrospective: identify process improvements for next sprint

3.4 Definition of Done
•	Feature implemented and manually tested on Chrome, Firefox, Edge
•	API endpoint returns correct HTTP status codes and JSON schema
•	No regressions in previously working flows
•	Code reviewed and merged to main branch
•	README / documentation updated if applicable

 
4. UML Diagrams
This section provides the key UML models describing the SentiAI system. Diagrams are presented in textual/tabular form for document portability.

4.1 Use Case Diagram — Actors & Use Cases
Actor	Use Cases
Guest (Unauthenticated)	View landing page · Register account · Log in
Authenticated User	Analyse single review · Bulk analyse (paste) · Upload CSV · View charts · Download PDF · Log out
System (VADER/ML Engine)	Preprocess text · Predict sentiment · Return confidence score
Email Service (Gmail SMTP)	Send welcome email with credentials
Admin / Developer	Train custom ML model · Manage users.json · Configure environment

4.2 Class Diagram
Key classes and their relationships:

┌─────────────────────────────────────────────────────────────────────┐
│                         SentimentEngine                             │
├─────────────────────────────────────────────────────────────────────┤
│ - vader : SentimentIntensityAnalyzer                                │
│ - ml_model : Pipeline | None                                        │
├─────────────────────────────────────────────────────────────────────┤
│ + predict(text: str) : dict                                         │
│ + train(texts: list, labels: list) : str                            │
│ - _vader_predict(original, cleaned) : dict                          │
│ - _ml_predict(original, cleaned) : dict                             │
│ - _clean(text: str) : str                                           │
│ - _save_model() : void                                              │
│ - _load_model() : void                                              │
└───────────────────────────┬─────────────────────────────────────────┘
                            │ uses
             ┌──────────────┴──────────────┐
             ▼                              ▼
  ┌──────────────────────┐     ┌──────────────────────────┐
  │   Flask App (app.py) │     │   report_generator.py    │
  ├──────────────────────┤     ├──────────────────────────┤
  │ engine : SentEngine  │     │ + generate_report(       │
  │ USERS_FILE : str     │     │     results, summary)    │
  ├──────────────────────┤     │   : bytes (PDF)          │
  │ + analyse()          │     └──────────────────────────┘
  │ + upload()           │
  │ + report()           │
  │ + api_login()        │
  │ + api_signup()       │
  │ - _build_summary()   │
  │ - _send_welcome_email│
  └──────────────────────┘

4.3 Sequence Diagram — Bulk Analysis Flow
User          Browser          Flask (app.py)     SentimentEngine   Response
 │                │                  │                   │              │
 │──POST reviews──▶                  │                   │              │
 │                │──POST /api/analyse▶                   │              │
 │                │                  │──[for each review]│              │
 │                │                  │──predict(review)──▶              │
 │                │                  │◀──{sentiment, confidence, scores}│
 │                │                  │──_build_summary(results)         │
 │                │◀────JSON {results, summary}──────────────────────── │
 │◀──render charts─                  │                   │              │

4.4 Activity Diagram — Signup Flow
  [Start]
     │
     ▼
  User fills name / email / password
     │
     ▼
  Client-side validation
     │── Invalid ──▶ Show error message (loop back)
     │
     ▼  Valid
  POST /api/signup
     │
     ▼
  Email already exists? ──Yes──▶ Return 409 Conflict
     │ No
     ▼
  Hash password (SHA-256)
     │
     ▼
  Save user to users.json
     │
     ▼
  Send welcome email (async try/catch)
     │
     ▼
  Set session cookie
     │
     ▼
  Redirect to /dashboard
     │
   [End]

4.5 State Diagram — Sentiment Prediction
         ┌─────────────────────────────────────────────────────┐
         │               SentimentEngine.predict()             │
         └─────────────────────────────────────────────────────┘
                                    │
                      Raw text input received
                                    │
                                    ▼
                           [Text Cleaning]
                   Lowercase → Strip URLs, HTML,
                   punctuation → Normalise whitespace
                                    │
                     ┌──────────────┴─────────────┐
                ml_model?                    ml_model is None
                     │                             │
                     ▼                             ▼
             [ML Prediction]              [VADER Prediction]
         TF-IDF → LogReg.predict()    polarity_scores() → compound
         Returns: class + proba        ≥0.05 → Positive
                                       ≤-0.05 → Negative
                                       else → Neutral
                     │                             │
                     └──────────────┬─────────────┘
                                    ▼
                    Return {text, sentiment, confidence,
                            scores, model}

 
5. Use Case Specifications

UC-01: Analyse Single Review
Field	Detail
Use Case ID	UC-01
Name	Analyse Single Review
Actor	Authenticated User
Preconditions	User is logged in; dashboard is loaded
Trigger	User types review text and clicks 'Analyse'
Main Flow	1. User enters review in textarea. 2. Browser POSTs to /api/analyse. 3. Engine preprocesses and predicts sentiment. 4. Dashboard displays badge (Positive/Negative/Neutral) with confidence %.
Alternate Flow	Empty input → client-side validation prevents submission; shows inline error
Postconditions	Result rendered in single-review result area; review added to results list
Business Rule	Confidence ≥ 50% always; compound score displayed for VADER predictions

UC-02: Bulk CSV Upload & Analysis
Field	Detail
Use Case ID	UC-02
Name	Bulk CSV Upload
Actor	Authenticated User
Preconditions	User has a .csv file with one text column
Trigger	User selects file via upload zone or drag-and-drop
Main Flow	1. Browser POSTs file to /api/upload. 2. Flask reads CSV, strips header row if detected. 3. Each row passed to engine.predict(). 4. Results + summary returned as JSON. 5. Charts and review list rendered.
Alternate Flow	CSV with no text found → 400 error 'No text found in CSV'
Postconditions	Summary metrics updated; pie, bar, trend charts rendered; 'Download PDF' button shown
Constraint	File must be UTF-8 encoded; single-column CSV recommended

UC-03: Download PDF Report
Field	Detail
Use Case ID	UC-03
Name	Download PDF Report
Actor	Authenticated User
Preconditions	At least one analysis has been run; results exist in browser state
Trigger	User clicks 'Download PDF Report' button
Main Flow	1. Browser POSTs results + summary JSON to /api/report. 2. Flask calls generate_report(). 3. ReportLab builds PDF in memory. 4. File streamed as attachment (sentiment_report_YYYYMMDD_HHMMSS.pdf).
Alternate Flow	No results in payload → 400 error 'No results provided'
Postconditions	PDF saved to user's downloads folder; named with timestamp

UC-04: User Registration
Field	Detail
Use Case ID	UC-04
Name	User Registration (Sign Up)
Actor	Guest
Preconditions	Email not already registered in users.json
Trigger	User completes signup form and submits
Main Flow	1. Guest visits /signup. 2. Enters name, email, password. 3. Client validates; password strength shown. 4. POST /api/signup. 5. Password hashed (SHA-256); user saved. 6. Welcome email sent. 7. Session created; redirect to /dashboard.
Alternate Flow	Duplicate email → 409 Conflict. Password < 6 chars → 400 error.
Security Note	Passwords stored as SHA-256 hash only; plain-text sent once in welcome email (recommend removing in production)

UC-05: Train Custom ML Model
Field	Detail
Use Case ID	UC-05
Name	Train Custom ML Model
Actor	Developer / Admin
Preconditions	Labelled CSV file (text, label columns) available; virtual env active
Trigger	Developer runs: python train_model.py --data path/to/data.csv
Main Flow	1. Script loads CSV. 2. If no label column, VADER auto-labels. 3. 80/20 train-test split. 4. TF-IDF + Logistic Regression Pipeline fitted. 5. Classification report printed. 6. Model saved to data/ml_model.pkl. 7. Flask app picks up model on next start.
Postconditions	App switches from VADER to ML model; badge in dashboard shows 'LogisticRegression'

 
6. Data Flow Diagram (DFD)

Level 0 — Context Diagram
                    ┌──────────────────────────────────────────┐
  ┌───────────┐     │                                          │     ┌───────────────┐
  │   User    │────▶│          SentiAI System                  │────▶│  Email Service│
  │ (Browser) │◀────│   (Flask + NLP Engine + ReportLab)       │     │ (Gmail SMTP)  │
  └───────────┘     │                                          │     └───────────────┘
                    └──────────────────────┬───────────────────┘
                                           │
                                    ┌──────▼──────┐
                                    │  users.json │
                                    │ ml_model.pkl│
                                    └─────────────┘

Level 1 — Main Processes
User Input Text/CSV
        │
        ▼
  ┌─────────────────┐
  │  1.0 Preprocess  │  lowercase, strip URLs/HTML/punct
  └────────┬────────┘
           │
           ▼
  ┌─────────────────┐
  │  2.0 Classify   │  VADER compound score or ML proba
  └────────┬────────┘
           │
    ┌──────┴──────┐
    ▼             ▼
 ┌──────┐    ┌──────────┐
 │ 3.0  │    │   4.0    │
 │Summ- │    │ Report   │
 │arise │    │ Generate │
 └──┬───┘    └────┬─────┘
    │              │
    ▼              ▼
 JSON to         PDF bytes
 Browser         to Browser

 
7. API Reference

Method	Endpoint	Auth	Description	Response
GET	/	No	Landing page	HTML
GET	/login	No	Login page	HTML
GET	/signup	No	Signup page	HTML
GET	/dashboard	Yes	Main dashboard	HTML
POST	/api/login	No	Authenticate user	JSON {success, redirect}
POST	/api/signup	No	Register new user	JSON {success, redirect}
POST	/api/analyse	Yes	Analyse JSON review list	JSON {results, summary}
POST	/api/upload	Yes	Upload & analyse CSV	JSON {results, summary}
POST	/api/report	Yes	Generate PDF report	application/pdf
GET	/logout	Yes	Clear session	Redirect → /

7.1 Sample API Request — Analyse
POST /api/analyse
Content-Type: application/json

{
  "reviews": [
    "The product quality exceeded my expectations!",
    "Shipping was extremely slow — very disappointed.",
    "It is okay, nothing special."
  ]
}

── Response 200 OK ──
{
  "results": [
    {"text": "...", "sentiment": "Positive", "confidence": 87, "model": "VADER"},
    {"text": "...", "sentiment": "Negative", "confidence": 79, "model": "VADER"},
    {"text": "...", "sentiment": "Neutral",  "confidence": 63, "model": "VADER"}
  ],
  "summary": {"total": 3, "counts": {"Positive":1,"Negative":1,"Neutral":1},
               "percentages": {"Positive":33.3,...}, "dominant": "Positive"}
}

 
8. Security Considerations

Area	Current Implementation	Recommended Improvement
Password Storage	SHA-256 hash in JSON	Migrate to bcrypt/argon2 + salting
Session Security	Flask session (HMAC-signed cookie)	Add session timeout, secure + httpOnly flags
Email Credentials	Env vars (MAIL_USER, MAIL_PASS)	Use OAuth2 / App Passwords; never hardcode
User Data Storage	Flat JSON file	Migrate to PostgreSQL with parameterised queries
Input Sanitisation	CSV stripped, VADER handles raw text	Add HTML-escape on any user content rendered
HTTPS	Not enforced in dev mode	Enforce via Nginx + Let's Encrypt in production
Welcome Email	Password sent in plain-text email	Remove password from email; use one-time links
CSRF Protection	Not currently implemented	Add Flask-WTF CSRF tokens on all POST forms

 
9. Installation & Deployment

9.1 Local Development Setup
1.	Clone or download the project to your local machine
2.	Create and activate a Python virtual environment
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
3.	Install dependencies
pip install -r requirements.txt
4.	Set environment variables (optional for email functionality)
export MAIL_USER=your@gmail.com
export MAIL_PASS=your_app_password
5.	Run the Flask development server
python app.py
6.	Open browser to http://localhost:5000

9.2 Production Deployment (Gunicorn + Nginx)
# Install Gunicorn
pip install gunicorn

# Start with 4 workers
gunicorn -w 4 -b 127.0.0.1:5000 app:app

# Nginx site config (excerpt)
server {
    listen 443 ssl;
    server_name yourdomain.com;
    location / { proxy_pass http://127.0.0.1:5000; }
}

9.3 Training a Custom Model
# Prepare CSV with columns: text, label
# label must be: Positive | Negative | Neutral

python train_model.py --data data/your_labelled_data.csv

# Model saved to data/ml_model.pkl
# Restart Flask app to activate ML mode

 
10. Project Structure
senti_ai/
├── app.py                  # Flask app, routes, API endpoints
├── sentiment_engine.py     # VADER + Logistic Regression NLP engine
├── report_generator.py     # ReportLab PDF generation
├── train_model.py          # CLI script to train custom ML model
├── requirements.txt        # Python dependencies
├── data/
│   ├── users.json          # User accounts (hashed passwords)
│   ├── sample_reviews.csv  # 15 sample reviews for demo
│   └── ml_model.pkl        # Trained model (created post-training)
├── templates/
│   ├── landing.html        # Public landing page
│   ├── login.html          # Login page
│   ├── signup.html         # Signup page
│   ├── dashboard.html      # Main authenticated dashboard
│   └── index.html          # Dashboard inner layout
└── static/
    ├── css/
    │   └── style.css       # Dashboard stylesheet
    └── js/
        └── app.js          # Chart.js logic, API calls, filtering

 
11. Dependencies & Licences

Package	Version	Purpose	Licence
Flask	>=3.0.0	Web framework, routing, session management	BSD-3
NLTK	>=3.8.1	NLP toolkit — VADER sentiment analyser	Apache 2.0
scikit-learn	>=1.4.0	TF-IDF vectoriser + Logistic Regression	BSD-3
pandas	>=2.0.0	CSV loading, data manipulation	BSD-3
numpy	>=1.26.0	Numerical operations	BSD-3
ReportLab	>=4.0.0	PDF generation	BSD-3
Chart.js (CDN)	4.4.1	Interactive browser charts	MIT
Syne / DM Sans	Google Fonts	UI typography	OFL

 
12. Real-World Application Use Cases

Industry	Use Case	Benefit
Retail / E-commerce	Analyse product reviews from Takealot / Amazon in bulk CSV	Identify top complaints; improve product descriptions
Banking & Finance	Monitor customer complaint emails (Nedbank, Standard Bank)	Flag service failures; reduce churn
Telecom	Track Twitter/X mentions for MTN, Vodacom, Cell C	Real-time brand sentiment; rapid response
Hospitality	Process post-stay survey responses from hotels / Airbnb	Pinpoint service gaps; reward high-performers
Healthcare	Analyse patient satisfaction surveys	Improve care quality; identify systemic issues
Government	Monitor public feedback on service delivery portals	Data-driven policy response

13. Glossary

Term	Definition
VADER	Valence Aware Dictionary and sEntiment Reasoner — rule-based NLP model optimised for social media text
Compound Score	VADER's normalised aggregate score from -1.0 (most negative) to +1.0 (most positive)
TF-IDF	Term Frequency-Inverse Document Frequency — statistical measure for text feature extraction
Logistic Regression	Supervised ML classification algorithm used as the optional custom model
Pipeline (sklearn)	Chained sequence of transformers and estimators; here TF-IDF → LogReg
Session	Flask server-side state stored in a signed browser cookie (HMAC-SHA256)
SHA-256	Cryptographic hash function used to store user passwords (256-bit output)
SMTP SSL	Simple Mail Transfer Protocol over TLS/SSL — used for Gmail outbound email
DFD	Data Flow Diagram — shows how data moves through a system
UML	Unified Modelling Language — standardised notation for software design diagrams
Scrum	Agile framework using fixed-length sprints, ceremonies, and defined roles
SDLC	Software Development Life Cycle — structured process for planning and building software

14. Future Roadmap

•	Migrate user store from JSON to PostgreSQL for concurrency and scalability
•	Add OAuth2 social login (Google, GitHub) via Flask-OAuthlib
•	Implement BERT / Transformer-based sentiment model for higher accuracy
•	Add real-time analysis via WebSocket for live review streams
•	Build REST API key management for third-party integrations
•	Docker Compose with Nginx + Gunicorn for one-command production deployment
•	Multi-language sentiment support (Zulu, Afrikaans, Swahili) via multilingual models
•	Admin panel for user management and model retraining from the web UI
•	Webhook integration for Slack / Microsoft Teams sentiment alerts
•	Exportable charts as PNG/SVG in addition to PDF reports


SentiAI — Customer Sentiment Intelligence Platform
Built with Python · Flask · NLTK · scikit-learn · Chart.js
© 2026 SentiAI  |  MIT Licence  |  Version 1.0
