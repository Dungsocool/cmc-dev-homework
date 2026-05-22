# 🚀 CMC Intern API Project - Asset Management

This is a project to build a RESTful API system for managing IT assets (Domain, IP, Service), developed using Go (Golang) and a PostgreSQL database. The project strictly adheres to the **Clean Architecture** pattern to optimize maintenance and ensure a clear separation of concerns.

## 📑 Table of Contents
* [🛠️ Technologies & Libraries Used](#technologies-and-libraries)
* [🛡️ Security Spotlight: Complete Prevention of SQL Injection](#security-spotlight)
* [✅ Completed Features](#completed-features)
* [📂 Directory Structure (Project Structure)](#project-structure)
* [📡 API Endpoints List](#api-endpoints)
* [💻 Installation & Running Guide (How to Run)](#how-to-run)
* [🧪 API Testing Guide (Testing Instructions)](#testing-guide)
* [🛡️ OTHER Ways to Prevent SQL Injection](#other-prevention-methods)

---
<a id="technologies-and-libraries"></a>
## 🛠️ Technologies & Libraries Used
* **Language:** Go (Golang)
* **Database:** PostgreSQL
* **Router:** `gorilla/mux` (combined with native `net/http`)
* **Infrastructure:** Docker & Docker Compose
* **Architecture:** Clean Architecture (Handler -> Service -> Storage)

<a id="security-spotlight"></a>
## 🛡️ Security Spotlight: Complete Prevention of SQL Injection (SQLi)

In this project, the risk of SQL Injection has been 100% eliminated in the Storage layer by strictly applying the **Parameterized Queries** technique via Go's native `database/sql` driver.

### How it works (Why is it secure?)
Instead of using dangerous String Concatenation to construct SQL statements, the system completely separates the **"Statement Structure" (Code)** from the **"User Inputs" (Data)**. 

When a query is executed, the process goes through 2 secure steps:

**Step 1 (Prepare)**: The database receives the SQL statement structure with virtual placeholders (like `$1, $2`). It compiles this command structure first.

**Step 2 (Execute)**: The user data is then sent to fill these placeholders. At this point, the database treats the data purely as **literal values (plain text)**, never executing or interpreting it as SQL commands.

### Real-world Illustrations
📍 Part 3 (Batch Delete)

<img width="786" height="523" alt="image" src="https://github.com/user-attachments/assets/20bbf63d-7889-4736-99ff-152d3204a86b" />

Dynamically building Parameterized Queries using the IN operator to secure the batch delete feature, avoiding direct string concatenation of IDs.

📍 Part 7 (Search by Name)
<img width="1074" height="337" alt="image" src="https://github.com/user-attachments/assets/cea5d9fd-febf-4d38-9c47-79840fde173a" />

Using the `$1` placeholder combined with the `ILIKE` operator. The `%` wildcard character is appended in Go code, ensuring the SQL structure remains intact regardless of user input.

#### Conclusion: Thanks to the placeholder parameterization mechanism of PostgreSQL and Go's database driver, all special characters like single quotes ('), semicolons (;), or malicious nested SQL queries injected by hackers are completely neutralized before ever interacting with the database.

<a id="completed-features"></a>
## ✅ Completed Features
The project has successfully implemented all core and advanced requirements:
1. **Part 1 - Statistics APIs:** Provides APIs to get overall asset statistics and count assets with dynamic filters (type, status).
2. **Part 2 - Batch Create:** Safely create multiple assets (up to 100 items/request) using a **Database Transaction** (All-or-nothing).
3. **Part 3 - Batch Delete:** Efficiently delete multiple assets simultaneously using a list of IDs.
4. **Part 4 - Connection Retry:** Implements an **Exponential Backoff** retry algorithm to automatically reconnect (up to 5 times) if the database drops or starts slowly.
5. **Part 5 - Health Check:** Provides a system monitoring endpoint that returns database connectivity health and Connection Pool statistics.
6. **Part 6 - Pagination & Filtering:** Implements server-side pagination and dynamic filtering in SQL to optimize response performance for large asset lists.
7. **Part 7 - Search by Name:** Supports partial and case-insensitive name searches.

<a id="project-structure"></a>
## 📂 Directory Structure (Project Structure)
The project is structured into packages following Clean Architecture principles:

```text
├── cmd/
│   └── server/
│       └── main.go                 # Entry point: Server startup point
├── homeworks/
│   └── submissions/
│       ├── SUBMISSION.md           # Assignment submission checklist file
│       └── (Supporting screenshots/evidence)
├── internal/
│   ├── handler/                    # HTTP Presentation Layer
│   │   └── asset_handler.go        # Handles HTTP Request/Response for Assets
│   ├── model/                      # Domain Model Layer / Struct definitions
│   │   ├── bonus.go
│   │   └── stats.go
│   ├── service/                    # Business Logic Layer
│   │   └── asset_service.go
│   └── storage/                    # Database Infrastructure Layer
│       ├── storage.go              # Storage Interface
│       └── postgres/               # PostgreSQL Specific Implementation
│           ├── asset_storage.go    # Basic CRUD Queries
│           ├── bonus_storage.go    # Queries for Pagination and Sorting/Searching
│           ├── delete_storage.go   # Queries for Batch Delete
│           ├── postgres.go         # Database Connection setup and Retry mechanism
│           └── stats_storage.go    # Queries for Statistics
├── docker-compose.yml              # PostgreSQL Database initialization file
└── README.md
```

<a id="api-endpoints"></a>
## 📡 API Endpoints List

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/health` | Check server and database connectivity status |
| `GET` | `/assets/stats` | Get asset statistics grouped by type and status |
| `GET` | `/assets/count` | Count assets (supports query: `?type=...&status=...`) |
| `POST` | `/assets/batch` | Create multiple assets at once |
| `DELETE` | `/assets/batch` | Delete multiple assets (Query: `?ids=id1,id2...`) |
| `GET` | `/assets` | Get asset list with pagination (Query: `?page=1&limit=20`) |
| `GET` | `/assets/search` | Search assets by name (Query: `?q=keyword`) |

<a id="how-to-run"></a>
## 💻 Installation & Running Guide (How to Run)
1. **Start the Database:**
Make sure you have Docker installed. Open a terminal and run the following command to spin up the database:

```bash
docker-compose up -d
```

2. **Start the API Server:**
Run the following command to start the Go application:

```bash
go run cmd/server/main.go
```

*Note: Thanks to the Connection Retry mechanism (Part 4), the API Server will automatically wait and attempt reconnection if PostgreSQL is still starting up.*

The server will listen to requests at: `http://localhost:8080`

<a id="testing-guide"></a>
## 🧪 API Testing Guide (Testing Instructions)

*(Note: Ensure the server is running at `http://localhost:8080` before running any commands. If you are using Windows PowerShell, type `curl.exe` instead of `curl`, or use Git Bash).*

### [Part 1] Statistics & Count

1. Get the overview statistics report (Total count grouped by Type and Status):

```bash
curl.exe -X GET http://localhost:8080/assets/stats
```
<img width="697" height="67" alt="image" src="https://github.com/user-attachments/assets/50f12b73-ebc5-4092-9122-a6e68b4cf119" />

2. Count assets with a dynamic filter combination (Example: Count active IP assets):

```bash
curl.exe -X GET "http://localhost:8080/assets/count?type=ip&status=active"
```
<img width="894" height="71" alt="image" src="https://github.com/user-attachments/assets/0bbee3b1-f2a5-4b55-b928-2b613bd3ae07" />

### [Part 2] Batch Create

Uses a Database Transaction to ensure data integrity (Maximum of 100 assets per request).

```bash
curl.exe -X POST http://localhost:8080/assets/batch \
-H "Content-Type: application/json" \
-d '{
  "assets": [
    {"name": "Firewall-Core", "type": "ip"},
    {"name": "cmc.com.vn", "type": "domain"},
    {"name": "DB-Server-01", "type": "service"}
  ]
}'
```
<img width="1578" height="96" alt="image" src="https://github.com/user-attachments/assets/c5e77937-ee30-4bb3-a455-bc212485a99a" />

### [Part 3] Batch Delete

Delete multiple assets simultaneously using the `IN` operator.

*(Please replace the ID string below with actual IDs generated from Part 2)*

```bash
curl.exe -X DELETE "http://localhost:8080/assets/batch?ids=id-1,id-2,id-3"
```
<img width="1069" height="56" alt="image" src="https://github.com/user-attachments/assets/d874ad13-2cc2-46b4-b765-3a69e54f903e" />

### [Parts 4 & 5] Connection Retry & Health Check

Check connectivity health of the API Server and PostgreSQL Database (Part 5):

```bash
curl.exe -X GET http://localhost:8080/health
```
<img width="799" height="81" alt="image" src="https://github.com/user-attachments/assets/783af0ba-404c-4da2-b732-c45142297ef2" />

Testing Part 4 (Retry): Try stopping the db container in Docker, then start the server using `go run cmd/server/main.go`. You will see system logs trigger Exponential Backoff, automatically increasing wait times and trying to reconnect up to 5 times instead of causing a panic.

```bash
docker-compose stop db
```
<img width="1529" height="103" alt="image" src="https://github.com/user-attachments/assets/e956bb7e-eb24-4a99-8a63-e6165282b939" />
<img width="1255" height="239" alt="image" src="https://github.com/user-attachments/assets/9293e874-cdd8-48b5-9d19-d81723387dd4" />

### [Part 6] Pagination

Get the list of assets on page 1, limiting to a maximum of 5 records per page:

```bash
curl.exe -X GET "http://localhost:8080/assets?page=1&limit=5"
```
<img width="1600" height="130" alt="image" src="https://github.com/user-attachments/assets/acfc191b-572d-4ddd-be39-31c782561090" />

### [Part 7] Search by Name

Performs a case-insensitive partial match search (using `ILIKE`).

Example: Find assets containing the term "firewall":

```bash
curl.exe -X GET "http://localhost:8080/assets/search?q=firewall"
```
<img width="1607" height="106" alt="image" src="https://github.com/user-attachments/assets/038183b5-74ec-494d-9214-3a329ba97322" />

<a id="other-prevention-methods"></a>
### 🛡️ OTHER Ways to Prevent SQL Injection (For Future Implementation)

```text
🛡️ Method 1: Apply the Principle of Least Privilege (PoLP)
Implementation: Instead of letting the Go API connect to the Database using the superuser postgres account (which has full DB dropping privileges), create a dedicated user (e.g., api_user).

Effect: Grant api_user only SELECT, INSERT, UPDATE, and DELETE privileges specifically on the assets table. Even if an attacker finds a SQLi vulnerability and executes a DROP TABLE assets command, PostgreSQL will immediately block it and raise a "Permission denied" error.

🛡️ Method 2: Use an ORM or Query Builder
Implementation: In larger real-world Go projects, instead of writing raw SQL (database/sql) manually, developers typically use ORM libraries (like GORM) or Query Builders (like Squirrel).

Effect: These libraries automatically sanitize inputs and perform parameter wrapping. You only need to write code like db.Where("name = ?", userInput), and the library takes care of SQL Injection prevention internally.

🛡️ Method 3: Infrastructure Layer Protection - Web Application Firewall (WAF)
Implementation: Deploy a WAF system (such as Cloudflare, AWS WAF, or ModSecurity) directly in front of your API Server (Port 8080).

Effect: WAF uses pre-configured rulesets to detect malicious patterns. If a request contains sensitive keywords combined together such as UNION SELECT, 1=1, or DROP TABLE, the WAF automatically blocks the hacker's IP and returns a 403 Forbidden error before the request even reaches your Go server.
```
