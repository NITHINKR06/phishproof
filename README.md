# PhishProof — Interactive Phishing Attack Simulator

A controlled-sandbox phishing awareness training application.
Built with Java Servlets, MySQL, HTML5, CSS3, JavaScript, Bootstrap 5.

---


## ⚡ Quick Start

### 1. Prerequisites
| Tool         | Version     |
|---|---|
| Java JDK     | 11+         |
| Apache Maven | 3.6+        |
| Apache Tomcat| 9 or 10     |
| MySQL Server | 8.0+        |

---

### 2. Database Setup

```bash
mysql -u root -p < sql/schema.sql
```

This creates the `phishproof_db` database with all tables and default seed data.

---

### 3. Configure DB Connection

Edit `src/main/java/com/phishproof/util/DBConnection.java`:

```java
private static final String DB_URL      = "jdbc:mysql://localhost:3306/phishproof_db?useSSL=false&serverTimezone=UTC";
private static final String DB_USER     = "root";
private static final String DB_PASSWORD = "your_password_here";
```

---

### 4. Build

```bash
mvn clean package
```

The WAR file is created at `target/PhishProof.war`.

---

### 5. Deploy

Copy `target/PhishProof.war` into your Tomcat `webapps/` folder, then start Tomcat:

```bash
# Linux / macOS
$CATALINA_HOME/bin/startup.sh

# Windows
%CATALINA_HOME%\bin\startup.bat
```

Or run with embedded Tomcat via Maven:

```bash
mvn tomcat7:run
```

---

### 6. Access the App

Open your browser: **http://localhost:8080/PhishProof**

#### Default credentials

| Role  | Email                        | Password    |
|---|---|---|
| Admin | admin@phishproof.local       | Admin@123   |
| User  | alice@phishproof.local       | Test@1234   |
| User  | bob@phishproof.local         | Test@1234   |
| User  | carol@phishproof.local       | Test@1234   |
| User  | david@phishproof.local       | Test@1234   |

---

## 🗂 Project Structure

```
PhishProof/
├── pom.xml
├── sql/
│   └── schema.sql                        ← Full DB schema + seed data
└── src/main/
    ├── java/com/phishproof/
    │   ├── controller/                   ← Java Servlets
    │   │   ├── LoginServlet.java
    │   │   ├── LogoutServlet.java
    │   │   ├── AdminDashboardServlet.java
    │   │   ├── CampaignServlet.java
    │   │   ├── PhishServlet.java         ← Core phishing engine
    │   │   ├── RevealServlet.java        ← Awareness reveal page
    │   │   ├── ReportServlet.java
    │   │   ├── UserInboxServlet.java
    │   │   ├── UserManagementServlet.java
    │   │   └── UserScoreServlet.java
    │   ├── dao/                          ← Data Access Objects
    │   │   ├── UserDAO.java
    │   │   ├── CampaignDAO.java
    │   │   ├── PhishLinkDAO.java
    │   │   ├── CapturedDataDAO.java
    │   │   ├── TemplateDAO.java
    │   │   └── AwarenessScoreDAO.java
    │   ├── model/                        ← POJOs / domain models
    │   │   ├── User.java
    │   │   ├── Campaign.java
    │   │   ├── PhishLink.java
    │   │   ├── Template.java
    │   │   ├── CapturedData.java
    │   │   └── AwarenessScore.java
    │   └── util/
    │       ├── DBConnection.java
    │       ├── PasswordUtil.java         ← BCrypt hashing
    │       └── TokenGenerator.java       ← Secure random tokens
    └── webapp/
        ├── index.jsp                     ← Root redirect
        ├── css/
        │   ├── main.css                  ← Dark cyber theme
        │   ├── admin.css
        │   └── inbox.css
        ├── js/
        │   └── main.js
        └── WEB-INF/
            ├── web.xml
            └── views/
                ├── login.jsp
                ├── reveal.jsp            ← "You were phished!" page
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
                └── templates/            ← Fake login page clones
                    ├── google-login.jsp
                    ├── facebook-login.jsp
                    ├── bank-login.jsp
                    ├── microsoft-login.jsp
                    └── corporate-login.jsp
```

---

## 🔄 User Flow

```
Admin                                 Learner
─────                                 ───────
Login → Admin Dashboard         →     Login → Inbox
Create Campaign (pick template)       See phishing email
Select target learners                Click link (tracked)
Launch Campaign                →     Land on fake login page
                                      Submit credentials (captured)
                                      ↓
                                      REVEAL PAGE:
                                      "You were phished!"
                                      ↓
                                      Red flags explained
                                      Awareness score shown
                                      ↓
View Reports (click/submit rates) ←  Score page
```

---

## 🛡 Security Notes

- This tool is for **authorized training purposes only**
- All phishing links are sandboxed — no real credentials are sent externally
- Captured passwords are stored in plain text **for educational display only**
  (In production, you would hash or discard them after display)
- BCrypt is used for real user account passwords
- Run only on internal / isolated networks
- Do not expose this application to the public internet

---

## 📦 Tech Stack

| Layer      | Technology                          |
|---|---|
| Backend    | Java 11, Jakarta Servlets 4.0       |
| Database   | MySQL 8.0                           |
| Frontend   | HTML5, CSS3, Bootstrap 5.3          |
| Charts     | Chart.js 4.4                        |
| Security   | BCrypt (jBCrypt), SecureRandom      |
| Build      | Maven 3, Tomcat 9/10                |
