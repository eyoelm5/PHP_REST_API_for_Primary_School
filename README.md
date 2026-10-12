# Web-based Portal & REST API backend for a Primary School

Note: This repository showcases my university backend project. This hypothetical scenario is provided by my university. 

## Project Overview

This project delivers a **custom PHP REST API** built for a primary school to manage pupils, parents, teachers, classrooms, and library loans through a decoupled client-server architecture. It avoids the usage of third-party framework to develop a better understanding of the core architecture.

## Technology Stack

* **Backend Core**: PHP 8.x (Object-Oriented, vanilla MVC pattern)
* **Database Engine**: MariaDB (Normalized to 3NF, custom constraint triggers)
* **Authentication**: firebase/php-jwt (Stateless JSON Web Tokens)
* **Environment Management**: vlucas/phpdotenv (.env configuration)
* **Dependency Management**: Composer
* **Frontend**: Vanilla JavaScript (ES6+ Fetch API / AJAX), HTML5, CSS3
* **Local Development Environment**: Apache via XAMPP (Not really the best choice)

## Backend Architecture

```
Client (Browser / Fetch API)
         │
         │  HTTP Request + JWT (Auth Cookie / Bearer)
         ▼
 ┌─────────────────────────────────────────────────────────┐
 │                   Routing & Entry Point                 │
 │             (/api/{resource}/{operation})               │
 └───────────────────────────┬─────────────────────────────┘
                             │
                             ▼
 ┌─────────────────────────────────────────────────────────┐
 │                   Auth Middleware                       │
 │      • JWT Validation (Firebase/php-jwt)                │
 │      • Role-Based Access Control (Admin/Teacher/Parent) │
 └───────────────────────────┬─────────────────────────────┘
                             │
                             ▼
 ┌─────────────────────────────────────────────────────────┐
 │                     Controllers                         │
 │      • Request Parsing & Status Code Handling           │
 │      • Strict Data Sanitization (htmlspecialchars)      │
 │      • Deep PHP Validation (Regex, Types, Boundaries)   │
 └───────────────────────────┬─────────────────────────────┘
                             │
                             ▼
 ┌─────────────────────────────────────────────────────────┐
 │                       Models                            │
 │      • Business Logic & Domain Entities                 │
 │      • Database Operations via PDO Prepared Statements  │
 └───────────────────────────┬─────────────────────────────┘
                             │
                             ▼
 ┌─────────────────────────────────────────────────────────┐
 │                    MariaDB Engine                       │
 │      • 3NF Relational Tables                            │
 │      • Capacity & Relationship Triggers                 │
 └─────────────────────────────────────────────────────────┘

```

### 1. RESTful API Implementation

The backend exposes endpoint collections (`/api/users/`, `/api/pupil/`, `/api/books/`, `/api/admin/`) communicating exclusively over JSON:

* **Stateless HTTP Communication:** Client requests are parsed and handled using standard verbs (`GET`, `POST`, `PUT`, `DELETE`).
* **Standard HTTP Response Codes:**
* `200 OK` / `201 Created` on successful operations.
* `400 Bad Request` on failed field validations.
* `401 Unauthorized` on missing, expired, or invalid JWTs.
* `409 Conflict` on duplicate entries (e.g., existing usernames).


* **Flexible Resource Retrieval:** Uses query parameter routing (e.g., `/read_single?id=1`) to pull targeted individual records while keeping controller interfaces uniform.

### 2. Authentication & Authorization (JWT)

* **Token Mechanism:** Stateless authentication using **Firebase PHP-JWT** (`firebase/php-jwt`).
* **Role-Based Access Control (RBAC):** Every decoded JWT token carries the user's role identifier, enforced by backend middleware:
    * **Admin:** Full CRUD access, user management, and teacher approval pipeline.
    * **Teacher:** Pupil record access, grade updates, and classroom overview.
    * **Parent:** Read-only access restricted strictly to their associated pupils.


* **Session Lifecycle:** Secure token distribution with dedicated logout endpoints to invalidate sessions client-side.

### 3. Data Sanitization & In-Depth PHP Validation

Before data reaches any query, it passes through rigorous server-side validation layers within the controller and model layers:

* **XSS Prevention:** All input passes through `htmlspecialchars()` to escape malicious scripts before processing.
* **Strict RegEx & Type Constraints:**
    * **Usernames:** Must be alphanumeric + underscores, 3–20 characters (`^[a-zA-Z0-9_]{3,20}$`).
    * **Names:** Alphabetic validation via `ctype_alpha()`; optional single-character middle initial.
    * **Phone Numbers:** UK mobile format matching (`^07\d{9}$`).
    * **Passwords:** Minimum 8 characters requiring at least one uppercase letter, one lowercase letter, one number, and one special symbol.
    * **ISBNs:** Regex verification supporting both ISBN-10 and ISBN-13 formats.
    * **Domain-Specific Rules:** Validation checking pupil ages (4–15 years), teacher minimum age (18+), and future-dated employee start dates.

### 4. Database Abstraction via PDO

* All database communications use **PHP Data Objects (PDO)**.
* Queries are strictly executed using parameterized **Prepared Statements** with bound parameters (`$stmt->bindParam()`), ensuring user inputs cannot alter SQL query grammar and completely neutralizing SQL Injection attacks.

## Repository Directory Structure

```text
St_Alphonsus_Primary_School/
├── api/                   # Public REST API route endpoints
│   ├── admin/             # Staff approvals and administrative operations
│   ├── books/             # Library inventory and loan management
│   ├── pupil/             # Pupil profiles, medical info, and enrollments
│   └── users/             # Authentication, registration, and user profiles
├── config/                # Core configurations
│   ├── database.php       # PDO connection factory and credentials loader
│   └── jwt.php            # JWT secret key and token handling configuration
├── public/                # Web server document root (HTML/CSS/JS frontend)
│   ├── assets/
│   │   ├── css/           # Modern styling sheets
│   │   ├── images/        # Assest images used on the portal such as logo
│   │   └── js/            # Client-side Fetch API asynchronous logic
│   └── *.html             # Client-facing portal views
├── src/                   # Backend application core (OOP architecture)
│   ├── Controllers/       # Request handlers, input validation, and HTTP dispatchers
│   ├── Middleware/        # Authentication gates and RBAC permission checks
│   └── Models/            # Database entities, SQL queries, and PDO bindings
├── .env.example           # Template for environment configuration
├── composer.json          # Dependency definitions (PHP-JWT, PHPdotenv)
└── README.md              # Project documentation

```

## Installation & Local Setup

### Prerequisites

* [XAMPP](https://www.google.com/search?q=https://www.apachefriends.org/) (Apache & MariaDB)
* [Composer](https://www.google.com/search?q=https://getcomposer.org/)
* Git installed on your local machine

### Step 1: Clone the Repository

Clone the project into your local machine:

```bash
git clone https://github.com/eyoelm5/PHP_REST_API_for_Primary_School.git
```

### Step 2: Deploy to XAMPP Document Root

Move the project folder into your XAMPP `htdocs` directory and rename it to `St_Alphonsus_Primary_School`:

```text
C:/xampp/htdocs/St_Alphonsus_Primary_School/

```

### Step 3: Install PHP Dependencies

Open your terminal, navigate to the project directory, and install dependencies via Composer:

```bash
cd C:/xampp/htdocs/St_Alphonsus_Primary_School
composer install

```

### Step 4: Configure Environment Variables

Create a `.env` file in the project root directory:

```bash
cp .env.example .env

```

Open `.env` and verify your local database settings:

```ini
DB_HOST=localhost
DB_NAME=st_alphonsus_primary_school
DB_USER=root
DB_PASS=
JWT_SECRET=your_super_secret_jwt_key_here

```

### Step 5: Database Setup

1. Open **phpMyAdmin** (`http://localhost/phpmyadmin`).
2. Create a new database named `st_alphonsus_primary_school`.
3. Import the SQL dump provided in the project root (`st_alphonsus_primary_school.sql`).

### Step 6: Launch Application

Start the **Apache** and **MySQL** modules in the XAMPP Control Panel. Open your browser and navigate to:

```text
http://localhost/St_Alphonsus_Primary_School/public/

```

## Default Demo Accounts

Use these pre-configured accounts to test role-based access control and system features:

| Role | Username | Password | Access Scope |
| --- | --- | --- | --- |
| **Administrator** | `employee001` | `Stephanie@2024!` | Full system control, pending teacher approvals |
| **Teacher** | `employee012` | `William@2024!` | Pupil management, class records, book management |
| **Parent** | `parents001` | `Jonathan@2024!` | Viewing assigned pupil details & loans |


## Security & Quality Assurance

Manual and boundary test cases were carried out to verify backend resilience:

* **SQL Injection Prevention:** Tested login fields using malicious payloads (`' OR 1=1 --`). Queries safely failed with unauthorized error responses due to PDO parameter binding.
* **XSS Mitigation:** Inputs containing `<script>` tags were neutralized using `htmlspecialchars()`.
* **Password Security:** Passwords are never stored in plaintext; all credentials are encrypted using PHP's native `password_hash()` (Bcrypt) and verified via `password_verify()`.
* **Database-Level Business Integrity:** In addition to PHP layer checks, MariaDB triggers actively enforce business rules (e.g., rejecting pupil enrollments if classroom capacity is reached, and restricting a maximum of two parents per child).
