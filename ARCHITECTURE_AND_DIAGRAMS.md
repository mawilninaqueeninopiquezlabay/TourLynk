# TourLynk: System Integration, Architecture & Security Specification

**System Title:** TourLynk - Smart Tourism & Event Management System with Geolocation & Security Assurance  
**Course Codes:** IT/CS-APSDEV311 (Application Development & Emerging Technologies) & IAS 1 (Information Assurance and Security 1)  
**Instructor / Panelist Reference:** Mr. Roel Daniot (Information Assurance and Security) & Application Development Faculty  
**System Version:** 4th Increment & Final Production Release  

---

## Executive Summary

TourLynk is an integrated, role-based tourism management and community engagement platform built to empower tourists, tourism staff (organizers), and system administrators. The system features real-time GPS distance calculation (km) to heritage and eco-tourism spots, turn-by-turn route navigation, interactive 5-star visitor reviews with staff response mechanisms, an administrative management hub, and a complete implementation of the Information Assurance and Security 1 (IAS) specification (multi-factor OTP, progressive lockout, CAPTCHA, dynamic password complexity validation, tamper-evident security audit logging, multi-tab session management, 15-minute inactivity auto-logout, and database backup/restore).

---

## 1. System Integration & Architecture Diagrams

### Diagram 1: UML Use Case Diagram (System Users)

```mermaid
graph LR
    subgraph SystemActors["System Actors"]
        Tourist(("👤 Tourist / Visitor"))
        Staff(("👔 Staff / Organizer"))
        Admin(("🛡️ System Admin"))
    end

    subgraph Boundary["TourLynk System Boundary"]
        direction TB
        subgraph ModTourist["Visitor & Tourist Services"]
            direction TB
            UC1["Browse Spots & Live GPS Distance"]
            UC2["Interactive Map & Directions"]
            UC3["Apply for Tourist Passes"]
            UC4["Submit 5-Star Feedback"]
            UC5["Account Security: OTP & Password"]
        end
        subgraph ModStaff["Staff & Organizer Operations"]
            direction TB
            UC6["Review & Decide Ticket Passes"]
            UC7["Propose New Tourism Spot"]
            UC8["Broadcast Announcements"]
            UC9["Reply to Visitor Reviews"]
        end
        subgraph ModAdmin["Administrative Governance"]
            direction TB
            UC10["User Management & RBAC"]
            UC11["Spot Decision Approval"]
            UC12["Security Audit Logs & CSV Export"]
            UC13["Database Backup & Recovery Hub"]
            UC14["Architecture & Diagrams Hub"]
        end
    end

    subgraph ExternalServices["External Systems"]
        GPS["External GPS & Map API"]
        DB[("Supabase Cloud DB & Storage")]
    end

    Tourist --> UC1
    Tourist --> UC2
    Tourist --> UC3
    Tourist --> UC4
    Tourist --> UC5

    Staff --> UC1
    Staff --> UC6
    Staff --> UC7
    Staff --> UC8
    Staff --> UC9
    Staff --> UC5

    Admin --> UC10
    Admin --> UC11
    Admin --> UC12
    Admin --> UC13
    Admin --> UC14
    Admin --> UC5

    UC1 -.-> GPS
    UC2 -.-> GPS
    UC3 -.-> DB
    UC4 -.-> DB
    UC6 -.-> DB
    UC7 -.-> DB
    UC8 -.-> DB
    UC10 -.-> DB
    UC11 -.-> DB
    UC12 -.-> DB
    UC13 -.-> DB
    UC14 -.-> DB
```

#### Actor Identification & Description
- **Tourist / Visitor (Primary Mobile/Web Actor):** Registers an account with IAS password complexity; accesses live GPS geolocation to measure distances in kilometers to local spots; applies for event passes; bookmarks favorite spots; submits ratings and comments.
- **Tourism Staff / Organizer (Primary Operational Actor):** Proposes new cultural/eco-tourism destinations to the Admin; broadcasts official notices; evaluates and approves/declines visitor ticket requests; posts official replies to visitor inquiries.
- **System Administrator / Super Admin (Primary Governance Actor):** Oversees user management (role assignment, activation, locking); reviews and approves proposed spots; monitors tamper-evident audit logs; conducts automated/manual database backups and restores.
- **External GPS / OpenStreetMap Tile Server (Secondary System Actor):** Delivers device coordinates and map tiles for distance calculations via the Haversine algorithm.
- **Supabase Cloud REST API & Local Storage Engine (Secondary Data Actor):** Provides dual-layer persistence, session synchronization, and audit log storage.

---

### Diagram 2: System Flow Diagram (Swimlanes & Pools)

```mermaid
sequenceDiagram
    autonumber
    participant V as 👤 Tourist / Visitor
    participant S as 👔 Staff / Organizer
    participant A as 🛡️ System Admin
    participant C as 💻 TourLynk Client (UI & GPS Engine)
    participant B as ☁️ Core Backend / Supabase DB

    %% Registration & Authentication Swimlane
    Note over V,B: Phase 1: Authentication & IAS Verification
    V->>C: Enter Credentials / Register with Strong Password
    C->>C: Validate Password Rules (8+ chars, upper, lower, digit, symbol)
    C->>B: Query User & Check Lockout State
    alt Account is Locked (3+ Failed Attempts)
        B-->>C: Return Progressive Lockout Countdown
        C-->>V: Display Lockout Timer & Require CAPTCHA
    else Valid Credentials
        B-->>C: Issue Session Token & Role Payload
        C->>B: Record Audit Log (LOGIN_SUCCESS, IP, Timestamp)
        C-->>V: Redirect to Role-Specific Dashboard
    end

    %% GPS Tracking & Spot Exploration
    Note over V,B: Phase 2: GPS Geolocation & Tourist Pass Application
    V->>C: Request Spot List & Enable GPS Location
    C->>C: Calculate Haversine Distance (km) from User GPS to Spots
    C-->>V: Display Spots Sorted with Distance Badges & Map Pins
    V->>C: Submit Ticket Pass Application (Qty: N)
    C->>B: Insert Application (Status: Pending Staff Review)
    C->>B: Record Audit Log (SUBMIT_APPLICATION)

    %% Staff Approval & Response
    Note over S,B: Phase 3: Staff Processing & Review Interaction
    S->>C: Open Staff Console
    C->>B: Fetch Pending Ticket Applications & Feedbacks
    B-->>C: Return Application Records
    S->>C: Click Approve Ticket Request
    C->>B: Deduct Available Slots & Update Application Status
    C->>B: Record Audit Log (STAFF_APPROVE_TICKET)
    V->>C: Open Spot Details & Submit 5-Star Rating + Comment
    C->>B: Store Feedback & Recalculate Aggregate Rating
    S->>C: Post Official Staff Response to Visitor Review
    C->>B: Update Feedback Record with Official Reply

    %% Admin Governance & Backup
    Note over A,B: Phase 4: Administrative Governance & Disaster Recovery
    A->>C: Access Admin Dashboard
    A->>C: Manage Users (Deactivate, Lock, Change Role)
    C->>B: Persist User State & Write Audit Log
    A->>C: Trigger Database Backup Snapshot
    C->>B: Serialize All Entities to JSON Snapshot
    B-->>C: Backup Saved (Timestamp, Size, Hash)
    A->>C: Verify Audit Logs & Export Security Report (CSV)
```

---

### Diagram 3: Entity Relationship Diagram (ERD)

```mermaid
erDiagram
    USERS ||--o{ APPLICATIONS_BOOKINGS : "submits"
    USERS ||--o{ FEEDBACKS_REVIEWS : "writes"
    USERS ||--o{ ANNOUNCEMENTS : "broadcasts"
    USERS ||--o{ SECURITY_AUDIT_LOGS : "triggers"
    USERS ||--o{ DATABASE_BACKUPS : "creates"
    
    TOURISM_SPOTS ||--o{ APPLICATIONS_BOOKINGS : "contains"
    TOURISM_SPOTS ||--o{ FEEDBACKS_REVIEWS : "receives"

    USERS {
        bigint id PK
        string username "Unique login handle (UK)"
        string email "Unique email address (UK)"
        string password_hash "Bcrypt hashed password"
        string full_name "User legal name"
        string role "visitor | organizer | admin | superadmin"
        boolean is_active "Account status active/inactive"
        boolean is_locked "Account security lock state"
        int failed_attempts "Consecutive failed login counter"
        bigint locked_until "Epoch timestamp of lockout expiration"
        datetime created_at
        datetime updated_at
    }

    TOURISM_SPOTS {
        string id PK
        string title "Name of destination / event"
        string category "Culture | Heritage | Eco Tourism | Food & Crafts"
        string location "Physical address or landmark"
        float latitude "GPS decimal coordinates"
        float longitude "GPS decimal coordinates"
        date event_date "Date of festival or schedule"
        text description "Spot overview & tourist guidelines"
        int available_slots "Remaining ticket capacity"
        string status "Approved | Pending Admin Approval | Rejected"
        string image_url "High-res destination photograph"
        string proposed_by FK "Username of proposing staff"
        datetime created_at
    }

    APPLICATIONS_BOOKINGS {
        bigint id PK
        string event_id FK "References TOURISM_SPOTS(id)"
        string applicant_username FK "References USERS(username)"
        string applicant_name "Full name of applicant"
        string title "Cached spot title"
        date date "Scheduled tour date"
        string location "Cached location text"
        int tickets "Requested ticket quantity"
        string status "Pending Staff Review | Approved | Declined by Staff"
        datetime created_at
    }

    FEEDBACKS_REVIEWS {
        bigint id PK
        string event_id FK "References TOURISM_SPOTS(id)"
        string event_title "Spot title for display"
        string visitor FK "References USERS(username)"
        string visitor_name "Display name of reviewer"
        int rating "1 to 5 Star numerical rating"
        text comment "Visitor testimonial or inquiry"
        text staff_reply "Official response from staff"
        string replied_by FK "Staff username who replied"
        datetime created_at
        datetime replied_at
    }

    ANNOUNCEMENTS {
        bigint id PK
        string title "Headline of official announcement"
        text content "Full body text of announcement"
        string posted_by FK "References USERS(username)"
        date date "Broadcast date"
        datetime created_at
    }

    SECURITY_AUDIT_LOGS {
        bigint log_id PK
        string username "User initiating action"
        string action "Standardized audit action code"
        text details "Contextual non-sensitive event details"
        string ip_address "Client IP address (IPv4/IPv6)"
        string affected_module "Auth | Users | Spots | Bookings | Security"
        datetime timestamp
    }

    DATABASE_BACKUPS {
        string backup_id PK
        string backup_name "Descriptive backup name"
        datetime created_at
        string created_by FK "Admin username"
        int total_records "Record count across all tables"
        int size_bytes "JSON payload byte size"
        json snapshot_data "Full serialized database state"
    }
```

---

### Diagram 4: System Flowchart

```mermaid
flowchart TD
    Start(["User Visits TourLynk"]) --> CheckSession{"Active Session in Storage?"}
    
    %% Session Branch
    CheckSession -- "Yes" --> VerifyInactivity{"Inactivity > 15 Mins?"}
    VerifyInactivity -- "Yes" --> AutoLogoutAction["Trigger Auto-Logout & Record Log"] --> LoginPage["Display Login Screen"]
    VerifyInactivity -- "No" --> RouteRole["Route by Role: Visitor / Staff / Admin"]

    %% Authentication Branch
    CheckSession -- "No" --> LoginPage
    LoginPage --> AuthChoice{"Action Choice"}
    
    AuthChoice -- "Forgot Password" --> OtpRecovery["Enter Email -> Generate 6-Digit OTP"]
    OtpRecovery --> VerifyOtp{"OTP Valid & Not Expired (< 3 min)?"}
    VerifyOtp -- "No" --> OtpErr["Increment OTP Failures & Invalidate after 3 tries"] --> LoginPage
    VerifyOtp -- "Yes" --> ResetPwd["Enter New Password with IAS Rules"] --> UpdatePwd["Update Password & Invalidate Sessions"] --> LoginPage

    AuthChoice -- "Register Account" --> InputReg["Input Full Name, Email, Username, Role, Password"]
    InputReg --> CheckPwdRules{"Meets 8+ Chars, Upper, Lower, Num, Special?"}
    CheckPwdRules -- "No" --> ShowPwdErrors["Show Dynamic Rule Violations"] --> InputReg
    CheckPwdRules -- "Yes" --> SaveUser["Save User & Hash Password"] --> LoginPage

    AuthChoice -- "Sign In" --> InputCreds["Enter Username/Email & Password"]
    InputCreds --> CheckLockout{"Is User Currently Locked Out?"}
    CheckLockout -- "Yes" --> ShowLockTimer["Display Lockout Countdown & Require CAPTCHA"] --> LoginPage
    CheckLockout -- "No" --> VerifyCredentials{"Credentials Match Database?"}
    
    VerifyCredentials -- "No" --> IncFailures["Increment Failed Attempts"]
    IncFailures --> CheckLimit{"Failed Attempts >= 3?"}
    CheckLimit -- "Yes" --> ApplyLockout["Lock Account: 1m -> 5m -> 15m & Record Audit Log"] --> ShowLockTimer
    CheckLimit -- "No" --> ShowRemaining["Show Remaining Attempts"] --> LoginPage

    VerifyCredentials -- "Yes" --> ResetFailCount["Reset Failed Attempts to 0"]
    ResetFailCount --> CreateSession["Issue Session Token & Start 15m Inactivity Timer"]
    CreateSession --> RecordLoginLog["Record LOGIN_SUCCESS in Audit Log"]
    RecordLoginLog --> RouteRole

    %% Role Routing
    RouteRole -- "Visitor" --> TouristDashboard["Tourist App: Browse Spots, GPS Map, Bookings"]
    RouteRole -- "Organizer / Staff" --> StaffConsole["Staff Console: Review Passes, Propose Spots, Post Notices"]
    RouteRole -- "Admin / Super Admin" --> AdminDashboard["Desktop Admin: Users, Catalog, Audit Logs, Backup/Restore"]

    %% Tourist Operations
    TouristDashboard --> GPSReq["Acquire Live GPS Coordinates"]
    GPSReq --> CalcDist["Run Haversine Formula for All Spots: Dist in km"]
    CalcDist --> MapView["Render Interactive Leaflet Map & Distance Badges"]
    MapView --> BookingAction["Apply for Passes / Submit 5-Star Feedback"]
    BookingAction --> WriteDB[("Save to Supabase / Local DB")]

    %% Admin Operations
    AdminDashboard --> AdminOps{"Admin Tool Selected"}
    AdminOps -- "User Management" --> UserCRUD["Activate, Deactivate, Lock, Unlock, Assign Roles"]
    AdminOps -- "Spot Approvals" --> SpotDecision["Approve / Reject Proposed Spots"]
    AdminOps -- "Audit Logs" --> LogViewer["Search, Filter & Export CSV Logs"]
    AdminOps -- "Backup & Recovery" --> BackupEngine["Generate JSON Snapshot / Restore Data"]
    UserCRUD --> WriteDB
    SpotDecision --> WriteDB
    BackupEngine --> WriteDB

    %% Global Exit
    TouristDashboard --> LogoutBtn["User Clicks Sign Out"]
    StaffConsole --> LogoutBtn
    AdminDashboard --> LogoutBtn
    LogoutBtn --> DestroySession["Destroy Session, Notify Open Tabs via Storage Event"]
    DestroySession --> RecordLogoutLog["Record LOGOUT in Audit Log"]
    RecordLogoutLog --> EndNode(["Session Terminated"])
```

---

### Diagram 5: Network Diagram (Network Flow)

```mermaid
graph LR
    subgraph ClientTier["Client Tier: Presentation Layer"]
        MobileClient["📱 Tourist Smartphone<br/>(iOS / Android Mobile Browsers)<br/>GPS Satellites Geolocation"]
        StaffPC["💻 Tourism Staff Workstation<br/>(Edge / Chrome / Safari)"]
        AdminPC["🖥️ Executive Admin Desktop<br/>(Secured Browser Terminal)"]
    end

    subgraph PerimeterSecurity["Perimeter & Transport Security"]
        Internet(("Encrypted Public Internet<br/>TLS 1.3 / HTTPS / WSS"))
        Firewall["🛡️ Network Firewall & WAF<br/>(Rate Limiting & DDoS Shield)"]
    end

    subgraph AppTier["Application & Server Tier"]
        WebServer["🌐 Web & Application Server<br/>Apache 2.4 / XAMPP Stack<br/>Port 80/443 (HTTP/HTTPS)<br/>TourLynk Client Engine & REST Gateway"]
    end

    subgraph CloudTier["Cloud Infrastructure & External Services Tier"]
        SupabaseCloud["☁️ Supabase Cloud Platform<br/>PostgreSQL 15 Database Cluster<br/>Auto-scaling REST API & RLS Engine<br/>Port 443 (HTTPS REST Endpoints)"]
        OSMServers["🗺️ OpenStreetMap & Leaflet CDN<br/>Tile Rendering Servers<br/>GeoJSON & Cartographic Assets"]
        SMTPServer["✉️ Cloud Notification & OTP Service<br/>Cryptographic Token Dispatch"]
    end

    MobileClient -- "HTTPS / TLS 1.3" --> Internet
    StaffPC -- "HTTPS / TLS 1.3" --> Internet
    AdminPC -- "HTTPS / TLS 1.3" --> Internet

    Internet --> Firewall
    Firewall --> WebServer

    WebServer -- "REST API (Bearer JWT)" --> SupabaseCloud
    WebServer -- "Map Tile Fetch" --> OSMServers
    WebServer -- "OTP Dispatch" --> SMTPServer
```

#### Network Device and Communication Path Specifications
1. **Client Tier:** Connects over TLS 1.3 via modern web viewports. Captures geolocation data natively via W3C Geolocation API from onboard GPS / Wi-Fi trilateration.
2. **Perimeter Layer:** All traffic passes through standard boundary firewalls with Cross-Origin Resource Sharing (CORS) enforcement and request rate limiting.
3. **Application Tier (Apache/XAMPP):** Delivers TourLynk assets, verifies session tokens, performs client/server-side sanitization, and facilitates offline fallback mechanisms.
4. **Cloud Tier (Supabase REST API):** High-availability PostgreSQL cluster enforcing database integrity, foreign key constraints, and RESTful query processing.
5. **External Services Tier:** OpenStreetMap tile distribution networks and CDN asset delivery (Tailwind CSS, Font Awesome 6, Leaflet 1.9.4).

---

## 2. Information Assurance and Security 1 (IAS) Implementation Matrix

| IAS Requirement | Implementation in TourLynk | Verification Method |
| :--- | :--- | :--- |
| **A. Core Access & Data Model** | Strict relational schema (`users`, `spots`, `applications`, `feedbacks`, `audit_logs`, `backups`). Unique emails and usernames. Full Admin User Management module (create, search, edit, activate/deactivate, lock/unlock, role assignment). | Verified in Admin User Management tab; duplicate usernames/emails rejected. |
| **B. Multi-Factor OTP Verification** | 6-digit cryptographic OTP for First Login, Account Verification, Forgot Password, and Changing Email Address. Features a live 3-minute expiration timer, 60s resend cooldown, and invalidation after 3 failed attempts. | Tested via OTP Modal, Forgot Password, and Profile Email Change flows. |
| **C. Password Security Suite** | 8+ characters, uppercase, lowercase, number, special character regex validation. Real-time dynamic visual rules checklist. Passwords securely hashed; previous password required for changes; active sessions invalidated upon reset. | Verified dynamically on registration and profile password change. |
| **D. Progressive Lockout & CAPTCHA** | 3 failed attempts allowed with remaining attempts displayed. Progressive lockouts (1 min, 5 min, 15 min) with live countdown timer persisting across browser reload (`localStorage`). Visual alphanumeric canvas CAPTCHA required. | Tested with invalid logins; timer persists on page refresh; CAPTCHA validates input. |
| **E. Session Management** | Cryptographic session tokens, complete session destruction on logout, multi-tab logout synchronization via window `storage` events, and 15-minute inactivity auto-logout with a 60s warning modal. | Tested across multiple tabs and via inactivity timer trigger. |
| **F. Tamper-Evident Audit Trail** | Comprehensive event logging (`LOGIN_SUCCESS`, `ACCOUNT_LOCKED`, `PASSWORD_CHANGED`, `USER_DEACTIVATED`, `ROLE_CHANGED`, etc.) with user, IP, timestamp, module, and details. Read-only Admin table with search, filter, and CSV export. | Verified via Audit Logs tab in Admin dashboard; CSV export functional. |
| **G. Input Sanitization & Safety** | XSS sanitization, HTML entity escaping, confirmation modals prior to sensitive actions (user deactivation, locking, role change, backup restore, logout). | Tested with special characters and HTML script tags. |
| **H. Database Backup & Disaster Recovery**| Full JSON snapshot generation of all entities (manual trigger & automated snapshots). Secure local/cloud storage with 1-click restore functionality and audit log recording. | Tested backup creation, download, and snapshot restoration. |

---

## 3. GPS Distance & Route Navigation Technical Specification

### Haversine Mathematical Model
The distance $d$ between the user's GPS coordinates $(\phi_1, \lambda_1)$ and a destination spot $(\phi_2, \lambda_2)$ on a spherical Earth of radius $R = 6,371\text{ km}$ is calculated as:

$$\Delta\phi = \phi_2 - \phi_1$$
$$\Delta\lambda = \lambda_2 - \lambda_1$$
$$a = \sin^2\left(\frac{\Delta\phi}{2}\right) + \cos(\phi_1)\cos(\phi_2)\sin^2\left(\frac{\Delta\lambda}{2}\right)$$
$$c = 2 \cdot \text{atan2}\left(\sqrt{a}, \sqrt{1-a}\right)$$
$$d = R \cdot c$$

In TourLynk:
- Displays exact distance in kilometers on every spot card (e.g. `📍 4.2 km away`).
- "Nearest First" sorting option dynamically re-orders destinations based on live GPS position.
- Interactive Leaflet map renders destination pins, user position with radar pulse, and polyline route lines.
- One-click "Start Navigation" opens Google Maps turn-by-turn directions (`https://www.google.com/maps/dir/?api=1&origin=...&destination=...`).

---

## 4. System Demonstration & Defense Evaluation Guide

1. **Quick Demo Accounts:**
   - **Admin / Super Admin:** Username: `admin` | Password: `Admin@123`
   - **Tourism Staff / Organizer:** Username: `organizer` | Password: `Org@1234`
   - **Tourist / Visitor:** Username: `tourlynk_user` | Password: `User@1234`
2. **GPS Geolocation Demonstration:**
   - Click "Interactive GPS Map" or tap on any spot to view live calculated distance in km.
   - Click "Get Directions" to preview the route line on the Leaflet map and launch navigation.
3. **IAS Security Demonstration:**
   - Attempt 3 incorrect passwords to trigger the progressive lockout countdown and canvas CAPTCHA.
   - Open two browser tabs; log out from Tab 1 to observe Tab 2 immediately synchronizing and logging out.
   - Test password creation to view the dynamic real-time rule checklist.
   - Trigger the Inactivity Warning to observe the auto-logout countdown.
4. **Admin Features Demonstration:**
   - Navigate to **User Management** to search, deactivate, unlock, or reassign user roles.
   - Navigate to **Security Audit Logs** to filter events by category or export the audit trail to CSV.
   - Navigate to **Backup & Disaster Recovery** to capture a fresh system snapshot or restore prior data.
   - Navigate to **Architecture & Diagrams** to view the live embedded UML diagrams!
