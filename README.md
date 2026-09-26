 Kigarama Growers SACCO
A production-ready web application and management system for Kigarama Growers SACCO, an agricultural Savings and Credit Cooperative Organization serving coffee and matooke farmers.

Built with:

Backend: Java 17, Spring Boot 3 (Spring Web, Spring Security, Spring Data JPA), Maven
Database: MySQL 8
Frontend: HTML5, CSS3 (agrarian green theme), vanilla modern JavaScript (no build step, no npm required)
Project Structure
Kigarama_Growers_SACCO/
├── run.sh / run.bat                     # One-command start (builds + runs everything)
├── database/
│   └── schema.sql                       # Reference/manual MySQL DDL + seed data
├── frontend/
│   └── public/                          # The entire frontend — HTML, CSS, JS
│       ├── index.html                   # Login page
│       ├── dashboard-superadmin.html
│       ├── dashboard-admin.html
│       ├── dashboard-cashier.html
│       ├── dashboard-member.html
│       ├── dashboard-receptionist.html  # Receptionist overview + prices + unread messages
│       ├── chat.html                    # Universal Messages page (every role)
│       ├── member-directory.html        # Receptionist: searchable member lookup
│       ├── forgot-password.html, reset-password.html, profile.html
│       ├── loans.html, manage-loans.html, mobile-money.html, mobile-money-approvals.html
│       ├── manage-users.html            # Create/manage accounts (Admin/Super Admin)
│       ├── manage-prices.html           # Coffee & matooke pricing (Admin/Super Admin)
│       ├── cashier-teller.html          # Deposit/withdraw
│       ├── cashier-produce.html         # Produce intake logging
│       ├── member-produce.html, transactions.html, produce-records.html
│       ├── css/style.css
│       └── js/ (config.js, api.js, sidebar.js, topbar.js)
└── backend/
    ├── pom.xml                          # Maven build file
    ├── mvnw / mvnw.cmd, .mvn/           # Maven Wrapper — no local Maven install needed
    └── src/main/
        ├── java/com/kigarama/sacco/
        │   ├── SaccoApplication.java    # Spring Boot entry point — RUN THIS CLASS
        │   ├── config/                  # Security, CORS, web, data-bootstrap, error handling
        │   ├── model/                   # JPA entities (User, Account, CoffeeLog, ...)
        │   ├── repository/              # Spring Data JPA repositories
        │   ├── service/                 # Business logic
        │   ├── security/                # Spring Security integration
        │   ├── dto/                     # Request payload classes
        │   └── controller/              # REST controllers (/api/**)
        └── resources/
            ├── application.properties   # DB connection, bootstrap credentials, CORS
            └── static/                  #  A COPY of frontend/public — see note below
Important: how the frontend connects to the backend
The backend/src/main/resources/static/ folder is an exact copy of frontend/public/. Spring Boot automatically serves everything in src/main/resources/static as your website — so when you start the backend, the frontend is already there at http://localhost:8080. You do not need to run a separate web server for the frontend; opening frontend/public/index.html directly as a file:// URL will not work correctly (the browser blocks the API calls), so always access the app through the backend's URL.

The frontend/public/ folder at the top level is provided so the frontend source is easy to find and edit on its own, and so it can be deployed separately (e.g. to Nginx or Netlify) if you ever want the frontend and backend on different servers — see "Deploying frontend and backend separately" below. If you edit anything in frontend/public/, copy your changes into backend/src/main/resources/static/ too (or just always edit the backend/.../static copy — it's the one that's actually served).

All JS files load js/config.js before js/api.js. By default API_BASE_OVERRIDE is empty, meaning the frontend calls the backend using relative paths (/api/...) — this is what makes same-origin deployment (the default) work with zero configuration.

Prerequisites
Java 17+ JDK (not just a JRE — check with javac -version)
MySQL 8.x running locally (or reachable over the network)
Maven is not required — the Maven Wrapper (mvnw/mvnw.cmd) downloads the correct Maven version for you automatically.
Fastest way to run it
# 1. Create/point the database credentials (see below)
# 2. From the project root:
./run.sh          # Mac/Linux
run.bat           # Windows
Then open http://localhost:8080.

Running from an IDE (IntelliJ IDEA / Eclipse / VS Code)
The most common reason a Spring Boot project "won't run" from an IDE is that the IDE opened the folder as a plain folder instead of importing it as a Maven project, so it never downloaded the dependencies or set up the classpath. Follow these steps exactly:

IntelliJ IDEA
File → Open... and select the backend folder specifically (not the top-level Kigarama_Growers_SACCO folder — that one has no pom.xml at its root).
IntelliJ should detect pom.xml and prompt "Load Maven Project" — click it. If it doesn't prompt, right-click pom.xml → Add as Maven Project.
Wait for the Maven import to finish (bottom-right progress bar) — this downloads all dependencies, which requires an internet connection the first time.
Open src/main/java/com/kigarama/sacco/SaccoApplication.java, click the green ▶ run icon next to the main method (or right-click the file → Run 'SaccoApplication.main()').
Make sure your Project SDK is Java 17+ (File → Project Structure → Project).
Eclipse / Spring Tool Suite (STS)
File → Import → Maven → Existing Maven Projects.
Browse to the backend folder and select its pom.xml.
Right-click the imported project → Maven → Update Project (forces a dependency refresh).
Right-click SaccoApplication.java → Run As → Spring Boot App (or Run As → Java Application if the Spring plugin isn't installed).
VS Code
Open the backend folder (code backend).
Install the "Extension Pack for Java" and "Spring Boot Extension Pack" if prompted.
Open SaccoApplication.java and click Run above the main method.
Command line (works everywhere, no IDE needed)
cd backend
./mvnw spring-boot:run        # Mac/Linux
mvnw.cmd spring-boot:run      # Windows
If your IDE still won't run it: check the Problems/Errors panel for red underlines on import statements — that always means the Maven import step above didn't complete. Re-run "Reload Maven Project" / "Maven → Update Project" and wait for it to finish before hitting Run.

Database Setup
Option A — let the app create it automatically (default)
The JDBC URL in application.properties includes createDatabaseIfNotExist=true, so as long as MySQL is running and your credentials can create databases, the app creates kigarama_sacco and all its tables itself on first run.

Option B — create it manually first
mysql -u root -p < database/schema.sql
Configure your credentials
Edit backend/src/main/resources/application.properties:

spring.datasource.username=root
spring.datasource.password=root Default Logins
Created automatically on first launch:

👥 Account Hierarchy & Workflow
Log in as Super Admin (or Admin).
Manage Accounts → create Cashier, Receptionist, and Member accounts (Super Admin can also create Admins). Members automatically get a savings account number + PIN.
Produce Prices → set today's coffee price (per kg) and matooke prices (per bunch, by size).
Log in as Cashier → process deposits/withdrawals and log coffee/matooke intake; proceeds credit the member's account instantly.
Log in as a Member → view live prices, produce history, deposit/withdraw, view balance (PIN required), apply for loans, submit mobile money deposits, and message anyone via Messages.
Log in as Receptionist → check the front-desk Overview, look up members in the Member Directory, and answer messages — from anyone, not just members.
Deploying frontend and backend separately (optional)
If you want to host the frontend on a static host (Nginx, Netlify, Vercel, S3, etc.) and the backend on a different server/domain:

Deploy the backend/ Spring Boot app as usual (mvnw clean package → run the resulting jar).
Deploy the contents of frontend/public/ to your static host.
Edit frontend/public/js/config.js on your static host and set:
window.SACCO_CONFIG = { API_BASE_OVERRIDE: 'https://your-backend-domain.com' };
In the backend's application.properties, add your frontend's origin to sacco.cors.allowed-origins (comma-separated), e.g.:
sacco.cors.allowed-origins=https://your-frontend-domain.com
If both sides are served over HTTPS on different domains, also uncomment in application.properties:
server.servlet.session.cookie.same-site=none
server.servlet.session.cookie.secure=true
This step is entirely optional — the default same-origin setup (backend serves the frontend directly) needs none of this and just works out of the box.

Security Notes
Passwords and PINs are stored using BCrypt hashing — never in plaintext, and never returned in any API response (@JsonIgnore on both fields).
Session-based authentication (HTTP-only cookies) protects all /api/** endpoints.
Every endpoint has Spring Security method-level @PreAuthorize role checks, in addition to the UI's own role-based navigation/redirects.
Chat is open to every role by design — anyone can message anyone (ChatService.sendMessage only blocks messaging yourself or a disabled account). If you ever want to restrict it again (e.g. back to Member ⇄ Receptionist only), that logic lives in one place: ChatService.
Chat attachments are restricted to PDF, Word (.doc/.docx) and image files.
New: Password Reset, Notifications, Profile Pics, Mobile Money & Loans
Forgot password (email reset code)
Every account now has an email (required for new accounts created via Manage Accounts). From the login page, "Forgot your password?" → enter email → a 6-digit code is emailed → enter the code + new password on the reset page. Email sending is OFF by default — until you configure real SMTP credentials, reset codes are printed to the server console log instead, so you can still test the whole flow locally. To send real emails:

sacco.mail.enabled=true
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=your-gmail-address@gmail.com
spring.mail.password=your-16-char-app-password   # Gmail "App Password", not your login password
Notifications
A bell icon in the top-right of every dashboard shows a badge with the number of unread chat messages and a preview dropdown — for Members (messages from the Receptionist) and the Receptionist (messages from Members).

Profile pictures
Click your avatar (top-right, next to the bell) to open My Profile, where anyone — any role — can upload a photo, edit their name/email/phone, and change their password. Once uploaded, the photo replaces the initials-avatar everywhere it appears.

Mobile Money deposits
Members can submit a deposit via Mobile Money Deposit (MTN MoMo / Airtel Money): they enter the phone number, amount, and the SMS confirmation code they received. A Cashier/Admin/Super Admin then verifies it on Mobile Money Approvals and approves it, which credits the member's account immediately (or rejects it with a note). This app does not integrate with a live telecom payment API — it's a verify-and-approve workflow, matching how most SACCOs actually reconcile mobile money today.

Loans
Members apply for a loan (amount + purpose) on the Loans page — this goes to the manager (Admin/Super Admin) for approval on Loan Applications. Once approved, a Cashier disburses the funds (credits the member's savings account) from the same page. Members see their outstanding loan balance and can repay part or all of it from their savings balance at any time; a loan closes automatically once fully repaid.

Responsive hamburger sidebar
On screens narrower than 768px, the sidebar collapses behind a ☰ button in a mobile header bar; tap it to slide the sidebar in over a dark backdrop, tap the backdrop (or any nav link) to close it. Desktop is unaffected — the sidebar stays fixed and visible as before.

Open chat — message anyone, from any role
Chat is no longer limited to Member ⇄ Receptionist. Every account — Super Admin, Admin, Cashier, Receptionist, Member — gets a Messages page (chat.html) listing every other person in the system as a potential contact, searchable by name or username, with an unread-count badge per person. Pick anyone and start typing; attachments (PDF, Word, images) still work the same way.

New member accounts now get their SACCO PIN
Previously, when a Member account was created, its auto-generated (or admin-set) PIN was hashed immediately and never shown anywhere again — nobody, not even the admin who created the account, had any way to find out what it was. Now:

A welcome email is sent automatically to the new user with their username, temporary password, and (for Members) their SACCO PIN. Like the password-reset codes, this prints to the server console instead of sending a real email until you configure SMTP (see above).
The Manage Accounts page also shows the generated PIN once, right after creation, as an on-screen fallback — make a note of it then, since it won't be shown again.
Members can change their own PIN later via My Profile → Change Password, or by asking an Admin to look up their account.
Receptionist dashboard: overview, prices, and a member directory
The Receptionist's landing page is now a proper Overview — total members, unread message count, and today's coffee/matooke prices at a glance, plus a feed of recent unread messages with a one-click reply. A new Member Directory page lets them search members by name/username to pull up contact details or start a chat, without exposing any balance or financial data (that stays strictly PIN-gated to the member themselves).

Design
The UI uses a professional agrarian green palette (#1b4d3e deep forest, #2e8b57 SACCO green, #a3c1ad sage) with earth-tone accents, paired with authentic Ugandan coffee-farm and matooke-plantation imagery across the login screen and dashboard banners.

Troubleshooting
"Unknown database" / connection refused — confirm MySQL is running and reachable at the host/port in application.properties; createDatabaseIfNotExist=true needs a MySQL user with CREATE DATABASE privileges.
"Access denied for user" — double-check spring.datasource.username / password.
Port 8080 already in use — change server.port in application.properties.
IDE shows red/errored imports everywhere — you opened the folder without importing it as a Maven project; see the IDE-specific steps above.
Frontend loads but every page shows a login redirect loop / API errors — you likely opened an HTML file directly via file:// instead of through http://localhost:8080; always start the backend first and browse to its URL.
Chat attachment fails to upload — check that the uploads/chat directory (configurable via sacco.uploads.dir) is writable by the process running the app.
"Forgot password" email never arrives — by default sacco.mail.enabled=false, so codes are printed to the server console instead of emailed; check the terminal running the app for a line like Password reset code for .... Set up real SMTP credentials (see above) to send actual emails.
New user creation fails with "Email is required" — every new account now needs an email address (used for password reset and notifications); this only applies to accounts created going forward, not to the database schema itself.
