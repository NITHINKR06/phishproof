# 🛡️ PhishProof — Interactive Phishing Attack Simulator

> **"The best way to learn about phishing is to feel the attack firsthand."**

PhishProof is a **controlled sandbox web application** designed to teach employees and students about phishing attacks through **real simulated experience** — not just theory. As the administrator (attacker), you build realistic fake login pages, generate tracked phishing links, send them to target users, and watch in real-time who clicked and who submitted credentials. As a learner (victim), you receive a convincing phishing email, click the link, land on a pixel-perfect fake login page — and only after submitting your credentials do you see the dramatic **"You Were Phished!"** reveal screen explaining exactly what happened, what data was captured, and how to protect yourself next time.

This is **learning by experience** — the most effective form of security awareness training.

---

## 📋 Table of Contents

1. [What is PhishProof?](#what-is-phishproof)
2. [How It Works — Full Walkthrough](#how-it-works)
3. [Features](#features)
4. [Tech Stack](#tech-stack)
5. [Project Architecture](#project-architecture)
6. [Database Design](#database-design)
7. [Prerequisites](#prerequisites)
8. [Installation & Setup](#installation--setup)
9. [Default Credentials](#default-credentials)
10. [Page-by-Page Guide](#page-by-page-guide)
11. [Phishing Templates](#phishing-templates)
12. [Troubleshooting](#troubleshooting)
13. [Security Notes](#security-notes)
14. [Project Structure](#project-structure)

---

## What is PhishProof?

PhishProof simulates the **entire lifecycle of a real phishing attack** inside a safe, controlled environment. There are two roles:

| Role | What They Do |
|---|---|
| **Admin (Attacker)** | Creates campaigns, picks fake login templates, assigns targets, generates unique tracked links, views real-time reports |
| **Learner (Victim)** | Receives simulated phishing emails in their inbox, clicks links, lands on cloned login pages, submits credentials, then sees the awareness reveal |

The application is built entirely on open-source Java web technology — no cloud services, no external APIs, no data ever leaves your local server.

---

## How It Works

### Step 1 — Admin Creates a Campaign

The administrator logs into the **Admin Dashboard** and clicks **"New Campaign"**. They fill in:
- A campaign name (e.g. `Q1 Finance Phishing Test`)
- A difficulty level (Easy / Medium / Hard)
- A phishing template (Google, Facebook, Microsoft, Bank, Corporate)
- Target users — one or more learners from the user list

When the campaign is saved, the system automatically generates a **unique cryptographically secure URL token** (256-bit SecureRandom) for every selected target. For example:

```
http://localhost:8080/PhishProof/phish?t=a7Kx2mNpQrTs...
```

Each token is different per user so the admin can track exactly who clicked and when.

---

### Step 2 — Admin Launches the Campaign

The admin clicks **"Launch Campaign"** which changes the campaign status from `DRAFT` to `ACTIVE`. The phish links are now live and trackable. The admin can copy any user's unique phish link from the Campaign Detail page.

---

### Step 3 — Learner Receives the Email

The learner logs into their account and opens their **Simulated Inbox**. They see a convincing phishing email — crafted with urgency language like:

> *"We have detected unusual login activity on your account. Verify your identity immediately or your account will be suspended within 24 hours."*

The email appears to come from a spoofed address like `security-alert@accounts-verify.net`. The learner sees a countdown timer, a plausible sender name, and a button that says **"Verify My Account Now"**.

At this point the learner has a choice:
- Click the link (fall for it) → proceeds to Step 4
- Click **"Report as Phishing"** → they correctly identified it and get praise

---

### Step 4 — Link Click is Tracked

The moment the learner clicks the phishing link, the **PhishServlet** intercepts the request:
1. Looks up the token in the database
2. Records the click with a timestamp and IP address
3. Marks the phish_link row as `clicked = 1`
4. Loads the campaign's configured template (e.g. Google login)
5. Renders the fake login page

The admin's dashboard updates — they can see that this specific user clicked the link.

---

### Step 5 — Fake Login Page

The learner lands on a pixel-perfect clone of a real login page. Depending on the template chosen, this could look exactly like:

- **Google Sign-In** — white card, Google logo, blue Next button
- **Facebook Login** — blue header, classic FB layout
- **Microsoft 365** — dark blue gradient background, Segoe UI font
- **Online Banking** — official-looking bank portal with security badges
- **Corporate VPN** — internal IT portal with company branding

The URL in the address bar is the only giveaway — `localhost:8080/PhishProof/phish?t=...` — but most users don't check it.

The page also silently collects:
- Time spent on the page before submitting
- Screen resolution
- Browser User-Agent string

---

### Step 6 — Credentials Are Captured

When the learner types their username and password and hits submit, the **PhishServlet POST handler** captures:
- The entered username / email
- The entered password (stored for educational display only)
- Their IP address
- Their browser and OS details
- Time they spent on the page
- Marks the phish_link as `submitted = 1`

All of this is stored in the `captured_data` table and associated with the phish link and campaign.

---

### Step 7 — The Reveal Page

Instead of a real login success, the learner is immediately redirected to the **Reveal Page** — the most important screen in the app. It shows:

**"🎣 You Were Phished!"**

Then in dramatic detail:
- **Exactly what data was captured** — their username, masked password (with a reveal button), IP address, browser info, time on page
- **An animated awareness score** (0–100) calculated based on how they interacted
- **6 red flags they missed** — suspicious URL, urgency tactics, spoofed sender, misleading links, HTTPS misconception, cloned UI
- **6 protection tips** — how to avoid this next time (password managers, MFA, URL checking etc.)
- **A full attack timeline** — step by step breakdown of exactly what just happened to them

This is the **learning moment** — the shock of seeing their own credentials captured makes the lesson unforgettable.

---

### Step 8 — Admin Views the Report

Back in the Admin Panel, the full campaign report shows:
- Total targets vs clicked vs submitted vs stayed safe
- Click rate and submission rate with progress bars
- A **donut chart** of results breakdown (Chart.js)
- A **bar chart** of user-level stats
- A full table of captured credentials with timestamps

Reports are available per-campaign via `/admin/reports?id=X` and chart data is served as JSON from `/admin/reports/data?id=X`.

---

## Features

### Admin Side
- **Secure login** with BCrypt password hashing
- **Campaign dashboard** with live stats (total campaigns, active campaigns, total clicks, total captures)
- **Campaign builder** — name, description, difficulty, template picker, multi-user target selector
- **Campaign detail view** — per-user phish link table, copy-to-clipboard URLs, click/submit status
- **Campaign status management** — Draft → Active → Completed lifecycle
- **Analytics reports** — Chart.js donut + bar charts, credential capture table, risk level badges
- **User management** — add users, assign roles, activate/deactivate accounts
- **Filter campaigns** by status (All / Draft / Active / Completed)

### Learner Side
- **Simulated inbox** — email client UI with unread/read states, countdown urgency timer
- **"Report as Phishing" button** — learner can correctly identify and report the phish
- **Awareness score page** — personal history of all simulations, animated gauge chart, level badge (Novice / Learning / Aware / Expert)

### Phishing Engine
- **5 realistic fake login templates** — Google, Facebook, Microsoft 365, Banking, Corporate VPN
- **Unique 256-bit tokens** per user per campaign (cryptographically secure, URL-safe)
- **Click tracking** — IP, timestamp, only first click recorded (idempotent)
- **Credential capture** — username, password, IP, User-Agent, screen size, time on page
- **Token expiry** — links expire after 30 days

### Reveal & Education
- **Animated credential display** — reveals exactly what was captured with a password reveal toggle
- **Animated score counter** (0–100 with color transitions)
- **6 red flag cards** — visual explanation of each missed warning sign
- **Attack timeline** — step-by-step breakdown of the attack
- **6 actionable protection tips**

---

## Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Backend | Java 11 | Core application logic |
| Web Framework | Jakarta Servlets 4.0 | HTTP request handling |
| View Layer | JSP + JSTL | Server-side HTML rendering |
| Database | MySQL 8.0 | Persistent data storage |
| DB Access | Plain JDBC | DAO pattern, PreparedStatements |
| Password Security | jBCrypt | BCrypt hashing for user passwords |
| Token Generation | java.security.SecureRandom | 256-bit cryptographic phish tokens |
| JSON | Gson | Report data API responses |
| Frontend | HTML5, CSS3 | Structure and styling |
| UI Framework | Bootstrap 5.3 | Responsive layout components |
| Charts | Chart.js 4.4 | Campaign analytics visualisations |
| Fonts | Google Fonts | Rajdhani, Share Tech Mono |
| Build Tool | Apache Maven 3 | Dependency management, WAR packaging |
| Server | Apache Tomcat 9/10 | Servlet container |

---

## Project Architecture

PhishProof follows a clean **MVC (Model-View-Controller)** architecture:

```
Browser Request
      ↓
[ Servlet (Controller) ]  — handles HTTP GET/POST, session auth, routing
      ↓
[ DAO (Data Layer) ]      — all SQL queries via PreparedStatements
      ↓
[ MySQL Database ]        — persists all application state
      ↑
[ Model (POJO) ]          — plain Java objects passed between layers
      ↑
[ JSP View ]              — renders HTML using JSTL tags + model data
```

### Controller Layer (Servlets)
Each servlet handles one area of the application:

| Servlet | URL Pattern | Responsibility |
|---|---|---|
| `LoginServlet` | `/login` | Authentication, session creation |
| `LogoutServlet` | `/logout` | Session invalidation |
| `AdminDashboardServlet` | `/admin/dashboard` | Admin overview stats |
| `CampaignServlet` | `/admin/campaigns/*` | Campaign CRUD, launch, link generation |
| `PhishServlet` | `/phish` | Click tracking, template rendering, credential capture |
| `RevealServlet` | `/reveal` | Awareness reveal page data |
| `ReportServlet` | `/admin/reports/*` | Analytics HTML + JSON API |
| `UserInboxServlet` | `/user/inbox` | Learner email inbox |
| `UserScoreServlet` | `/user/score` | Learner awareness score |
| `UserManagementServlet` | `/admin/users/*` | User add, activate, deactivate |

### DAO Layer
Each DAO handles all database operations for one entity using `PreparedStatement` (SQL injection safe):

| DAO | Table | Key Operations |
|---|---|---|
| `UserDAO` | `users` | findByEmail, findAll, insert, updateLastLogin, toggleActive |
| `CampaignDAO` | `campaigns` | findAll, findById, insert, updateStatus, delete |
| `PhishLinkDAO` | `phish_links` | insert, findByToken, findByCampaign, markClicked, markSubmitted |
| `CapturedDataDAO` | `captured_data` | insert, findByLinkId, findByCampaign |
| `TemplateDAO` | `templates` | findAll, findById |
| `AwarenessScoreDAO` | `awareness_scores` | insert, findByUser, getAverageScore |

---

## Database Design

Six tables cover the complete data model:

```
users
├── id, name, email, password (BCrypt), role (ADMIN/LEARNER)
├── department, avatar_url, is_active
└── created_at, last_login

templates
├── id, name, category (SOCIAL/BANKING/CORPORATE/CLOUD/ECOMMERCE)
├── brand, description, html_file (JSP filename)
└── preview_img, is_active, created_at

campaigns
├── id, name, description
├── template_id → templates.id
├── created_by  → users.id
├── status (DRAFT/ACTIVE/PAUSED/COMPLETED)
├── difficulty (EASY/MEDIUM/HARD)
└── created_at, launched_at, ended_at

phish_links                          ← One row per user per campaign
├── id, token (unique 256-bit)
├── campaign_id → campaigns.id
├── user_id     → users.id
├── clicked (0/1), submitted (0/1)
└── clicked_at, submitted_at, expires_at

captured_data                        ← Filled when user submits the form
├── id, link_id → phish_links.id
├── fake_username, fake_password
├── ip_address, user_agent
├── screen_size, referrer, time_on_page
└── captured_at

awareness_scores
├── id, user_id → users.id
├── campaign_id → campaigns.id
├── score (0–100), caught_phish (0/1)
├── feedback, red_flags (JSON)
└── evaluated_at
```

---

## Prerequisites

Make sure the following are installed before you begin:

| Tool | Version | Download |
|---|---|---|
| Java JDK | 11 or higher | https://adoptium.net |
| Apache Maven | 3.6 or higher | https://maven.apache.org |
| Apache Tomcat | 9 or 10 | https://tomcat.apache.org |
| MySQL Server | 8.0 or higher | https://dev.mysql.com/downloads |

Verify your installations:
```bash
java -version
mvn -version
mysql --version
```

---

## Installation & Setup

### Step 1 — Extract the Project

Extract the zip file into a folder of your choice:
```
PhishProof/
├── pom.xml
├── sql/
└── src/
```

### Step 2 — Set Up the Database

Open a terminal and run:
```bash
mysql -u root -p < sql/schema.sql
```

Enter your MySQL root password when prompted. This will:
- Create the `phishproof_db` database
- Create all 6 tables
- Insert 1 default admin user
- Insert 4 default learner users
- Insert 5 phishing templates

Verify it worked:
```bash
mysql -u root -p -e "USE phishproof_db; SHOW TABLES;"
```

Expected output:
```
awareness_scores
campaigns
captured_data
inbox_messages
phish_links
templates
users
```

### Step 3 — Configure Database Connection

Open this file in your editor:
```
src/main/java/com/phishproof/util/DBConnection.java
```

Update these three lines:
```java
private static final String DB_URL      = "jdbc:mysql://localhost:3306/phishproof_db?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true";
private static final String DB_USER     = "root";           // ← your MySQL username
private static final String DB_PASSWORD = "your_password";  // ← your MySQL password
```

### Step 4 — Build the Project

From the project root folder (where `pom.xml` is):
```bash
mvn clean package
```

Maven will download all dependencies and create the WAR file at:
```
target/PhishProof.war
```

### Step 5 — Deploy to Tomcat

**Option A — Deploy WAR manually:**
```bash
# Copy WAR to Tomcat's webapps folder
cp target/PhishProof.war /path/to/tomcat/webapps/

# Start Tomcat
# Linux / macOS:
$CATALINA_HOME/bin/startup.sh

# Windows:
%CATALINA_HOME%\bin\startup.bat
```

**Option B — Run with embedded Tomcat (quick dev mode):**
```bash
mvn tomcat7:run
```

### Step 6 — Open the App

```
http://localhost:8080/PhishProof
```

You will be redirected to the login page automatically.

---

## Default Credentials

| Role | Email | Password | Department |
|---|---|---|---|
| Admin | admin@phishproof.local | Admin@123 | IT Security |
| Learner | alice@phishproof.local | Test@1234 | Finance |
| Learner | bob@phishproof.local | Test@1234 | HR |
| Learner | carol@phishproof.local | Test@1234 | Engineering |
| Learner | david@phishproof.local | Test@1234 | Marketing |

> ⚠️ Change all passwords before using in any real training environment.

---

## Page-by-Page Guide

### Login Page — `/login`
Dark cyber-themed login with animated matrix background. Enter email and password. Admins redirect to dashboard; learners go to inbox.

### Admin Dashboard — `/admin/dashboard`
Overview showing: total campaigns, active campaigns, total link clicks, total credential captures. Live campaign table with click/submit rates as animated progress bars.

### Campaigns List — `/admin/campaigns`
Card-based view of all campaigns. Filter by status (All / Draft / Active / Completed). Each card shows targets, clicks, captures and progress bar. Buttons to view detail, view report, launch, or delete.

### Create Campaign — `/admin/campaigns/new`
Two-column form:
- **Left:** Campaign name, description, difficulty level, template picker (radio cards showing each fake login brand)
- **Right:** Target user selector with Select All / Clear buttons

On submit, creates the campaign and generates one unique phish link token per selected user.

### Campaign Detail — `/admin/campaigns/view?id=X`
Per-user phish link table showing who clicked, who submitted, timestamps, and a "Copy URL" button per link. Launch button if status is DRAFT.

### Reports — `/admin/reports?id=X`
Full analytics page:
- **Donut chart** — Safe / Clicked Only / Submitted proportions
- **Bar chart** — Absolute numbers
- **Credential table** — who submitted what, IP, time on page, timestamp

### User Inbox — `/user/inbox`
Email client UI. Clicking an email opens a modal with the full phishing email, the phish link button, and a "Report as Phishing" option with a countdown timer for urgency.

### Reveal Page — `/reveal?lid={linkId}`
Shown after credential submission:
- Blinking red warning banner
- Animated "You Were Phished!" hero section
- Full captured data table (username, masked password with reveal toggle, IP, browser, screen, time on page)
- Animated awareness score counter (0–100)
- 6 red flag explanation cards
- 6 protection tips
- Complete attack timeline

### User Score — `/user/score`
Personal awareness history, half-donut gauge chart, stats (received / clicked / stayed safe), simulation history list, level badge (Novice / Learning / Aware / Expert).

---

## Phishing Templates

Five realistic cloned login pages:

| Template | File | Style |
|---|---|---|
| Google Sign-In | `google-login.jsp` | White card, Google SVG logo, blue Next button |
| Facebook Login | `facebook-login.jsp` | Full FB layout, blue header, green sign-up button |
| Microsoft 365 | `microsoft-login.jsp` | Dark blue gradient, Segoe UI, minimalist underline inputs |
| SecureBank Online | `bank-login.jsp` | Navy blue banking portal with security badges and info panel |
| Corporate VPN Portal | `corporate-login.jsp` | Dark themed internal IT portal, domain dropdown, security footer |

Each template silently collects `screenSize` and `timeOnPage` via hidden fields and posts to `PhishServlet`.

---

## Troubleshooting

**`ClassNotFoundException: com.mysql.cj.jdbc.Driver`**
Run `mvn clean package` again. The MySQL driver must be packaged into the WAR.

**`Access denied for user 'root'@'localhost'`**
Wrong DB credentials in `DBConnection.java`. Check your MySQL username and password.

**`Table 'phishproof_db.users' doesn't exist`**
Schema not imported. Run: `mysql -u root -p < sql/schema.sql`

**Port 8080 already in use**
Change Tomcat's port in `conf/server.xml` to e.g. `8090`, then access at `http://localhost:8090/PhishProof`.

**JSP shows raw `${variable}` text**
JSTL not processing. Ensure `jstl` dependency is in `pom.xml` and taglib declarations are at the top of each JSP.

**Phish link says "expired or invalid"**
Token expired (30-day TTL) or doesn't exist. Create a new campaign to generate fresh tokens.

**`mvn tomcat7:run` fails**
Ensure the `tomcat7-maven-plugin` is in `pom.xml`. If behind a proxy, configure Maven's `settings.xml`.

---

## Security Notes

> ⚠️ **This tool is for authorized training and educational use only.**

- **Sandboxed** — all phishing links resolve to your local server. No real credentials leave the machine.
- **Fake credentials stored for education** — entered passwords are stored in plain text in `captured_data` solely for the reveal page display. Discard or encrypt them after display in any real deployment.
- **Real passwords use BCrypt** — actual user account passwords are hashed with BCrypt (cost factor 12).
- **SQL injection protection** — all queries use `PreparedStatement` with parameterised inputs.
- **Secure token generation** — phish tokens use `java.security.SecureRandom` (256-bit), cryptographically unpredictable.
- **Internal networks only** — never expose this application to the public internet.
- **No real emails** — phishing emails exist only inside the in-app simulated inbox. No SMTP is used.

---

## Project Structure

```
PhishProof/
├── pom.xml                                      ← Maven build + dependencies
├── README.md                                    ← This file
├── sql/
│   └── schema.sql                               ← DB schema + seed data
└── src/main/
    ├── java/com/phishproof/
    │   ├── controller/                          ← Servlets (HTTP handlers)
    │   │   ├── LoginServlet.java                ← GET/POST /login
    │   │   ├── LogoutServlet.java               ← GET /logout
    │   │   ├── AdminDashboardServlet.java        ← GET /admin/dashboard
    │   │   ├── CampaignServlet.java             ← /admin/campaigns/*
    │   │   ├── PhishServlet.java                ← Core phishing engine
    │   │   ├── RevealServlet.java               ← GET /reveal
    │   │   ├── ReportServlet.java               ← /admin/reports/*
    │   │   ├── UserInboxServlet.java            ← GET /user/inbox
    │   │   ├── UserManagementServlet.java        ← /admin/users/*
    │   │   └── UserScoreServlet.java            ← GET /user/score
    │   ├── dao/                                 ← Database access objects
    │   │   ├── UserDAO.java
    │   │   ├── CampaignDAO.java
    │   │   ├── PhishLinkDAO.java
    │   │   ├── CapturedDataDAO.java
    │   │   ├── TemplateDAO.java
    │   │   └── AwarenessScoreDAO.java
    │   ├── model/                               ← Plain Java domain objects
    │   │   ├── User.java
    │   │   ├── Campaign.java
    │   │   ├── PhishLink.java
    │   │   ├── Template.java
    │   │   ├── CapturedData.java
    │   │   └── AwarenessScore.java
    │   └── util/
    │       ├── DBConnection.java                ← JDBC connection manager
    │       ├── PasswordUtil.java                ← BCrypt hash + verify
    │       └── TokenGenerator.java             ← SecureRandom URL tokens
    └── webapp/
        ├── index.jsp                            ← Root redirect
        ├── css/
        │   ├── main.css                         ← Dark cyber theme
        │   ├── admin.css                        ← Admin panel styles
        │   └── inbox.css                        ← Email client styles
        ├── js/
        │   └── main.js                          ← Toast, clipboard, nav utils
        └── WEB-INF/
            ├── web.xml                          ← Servlet config + error pages
            └── views/
                ├── login.jsp
                ├── reveal.jsp                   ← "You were phished!" page
                ├── error.jsp
                ├── partials/
                │   └── admin-nav.jsp
                ├── admin/
                │   ├── dashboard.jsp
                │   ├── campaigns.jsp
                │   ├── create-campaign.jsp
                │   ├── campaign-detail.jsp
                │   ├── reports.jsp
                │   ├── report-detail.jsp
                │   └── users.jsp
                ├── user/
                │   ├── inbox.jsp
                │   └── score.jsp
                └── templates/
                    ├── google-login.jsp
                    ├── facebook-login.jsp
                    ├── bank-login.jsp
                    ├── microsoft-login.jsp
                    └── corporate-login.jsp
```

---

1. railway.app → New Project
2. Deploy from GitHub → select PhishProof repo
3. + New Service → Add MySQL
4. Railway auto-injects: MYSQLHOST, MYSQLPORT, MYSQLDATABASE, MYSQLUSER, MYSQLPASSWORD
5. Open MySQL service → Query tab → paste sql/schema.sql contents → Run
6. Your app deploys at: https://phishproof-xxxx.up.railway.app


## License

This project is intended for **educational and authorized internal training use only**.
Do not use it to conduct phishing attacks against real users without explicit consent and organizational authorization.

---

*Built with ☕ Java + 🛡️ security awareness in mind.*
