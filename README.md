# CriminalFaceDB

**Criminal Face Recognition & Crime Database System** — a Java Swing desktop application backed by MySQL for managing criminal records, crimes, cases and police officers, with a face-scanner module and a crime analytics dashboard.

![Java](https://img.shields.io/badge/Java-11%2B-orange) ![MySQL](https://img.shields.io/badge/MySQL-8%2B-blue) ![UI](https://img.shields.io/badge/UI-Swing-informational) ![Status](https://img.shields.io/badge/face%20recognition-demo%20%2F%20simulated-yellow)

---

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Database Schema](#database-schema)
- [Getting Started](#getting-started)
- [Usage Guide](#usage-guide)
- [Face Recognition Status](#face-recognition-status)
- [Roadmap](#roadmap)
- [Responsible Use](#responsible-use)
- [Contributing](#contributing)
- [Author](#author)

---

## Overview

CriminalFaceDB is a police-records management system with a dark-themed desktop interface. It stores detailed criminal profiles (with photos), links them to crimes and investigation cases handled by police officers, and visualizes crime statistics. A Face Scanner screen provides the interface for identifying a person against the database.

The project ships with sample data themed around Punjab Police (Ludhiana, Amritsar, Patiala, Chandigarh, Jalandhar) so it can be demonstrated immediately after setup.

## Features

- **Dashboard** — live stat cards: total criminals, wanted count, total crimes, open cases, solved cases.
- **Criminal management** — add, edit, delete and search criminals by name, address or blood group; each profile stores age, gender, address, nationality, height, weight, blood group and wanted status.
- **Photo handling** — upload a photo per criminal (stored in `resources/images/`) and preview it in the table view.
- **Crime records** — view and filter crimes by type, status and severity (Low / Medium / High / Critical).
- **Case management** — track FIR-numbered cases with assigned officer, status, opened/closed dates, court date and notes.
- **Officer directory** — police officers with badge number, rank, department and station.
- **Analytics** — custom-drawn bar chart (crimes by type) and pie chart (case status breakdown).
- **Face Scanner screen** — camera controls, scan action and an identification result card (see [Face Recognition Status](#face-recognition-status)).

## Tech Stack

| Layer | Technology |
|-------|------------|
| Language | Java 11+ |
| UI | Java Swing / AWT (custom dark theme in `UITheme`) |
| Database | MySQL 8+ |
| DB access | JDBC via `mysql-connector-j`, DAO pattern |
| Build | Plain `javac` scripts (`build.sh`, `build.bat`) |

## Project Structure

```
CriminalFaceDB/
├── build.sh                  # Compile & run (Linux / macOS)
├── build.bat                 # Compile & run (Windows)
├── lib/
│   └── README.txt            # Put mysql-connector-j.jar here
├── sql/
│   └── schema.sql            # Tables, views and sample data
└── src/criminaldb/
    ├── model/
    │   └── Criminal.java
    ├── db/
    │   ├── DBConnection.java # JDBC connection settings
    │   ├── CriminalDAO.java  # Criminal CRUD, search, statistics
    │   └── CaseDAO.java      # Crimes, cases, officers
    ├── utils/
    │   └── UITheme.java      # Colors, fonts, button styling
    └── ui/
        ├── MainFrame.java    # Entry point, sidebar navigation
        ├── DashboardPanel.java
        ├── CriminalPanel.java
        ├── CriminalFormDialog.java
        ├── CrimePanel.java   # Also contains Cases and Officers panels
        ├── AnalyticsPanel.java
        └── FaceRecPanel.java
```

## Database Schema

Database name: `criminal_db`

| Table | Purpose |
|-------|---------|
| `criminals` | Personal details, photo path, wanted flag |
| `crimes` | Crime type, description, date, location, status, severity (FK → criminals) |
| `police_officers` | Badge number, rank, department, station, contact |
| `cases` | Case number, linked criminal and officer, status, dates, notes |
| `face_encodings` | Reserved for face embeddings (`LONGBLOB`) per criminal |

Two views are included: `v_criminal_summary` (crime and case counts per criminal) and `v_case_details` (cases joined with criminal and officer names).

## Getting Started

### Prerequisites
- JDK 11 or newer
- MySQL Server 8+
- [MySQL Connector/J](https://dev.mysql.com/downloads/connector/j/) (`.jar`)

### 1. Clone the repository
```bash
git clone https://github.com/tranumgrover/CriminalFaceDB.git
cd CriminalFaceDB
```

### 2. Create the database
```bash
mysql -u root -p < sql/schema.sql
```
This creates `criminal_db`, all tables, views and the sample data.

### 3. Add the JDBC driver
Download Connector/J and place it in `lib/` named exactly **`mysql-connector-j.jar`** (the build scripts expect this filename).

### 4. Configure the connection
Edit `src/criminaldb/db/DBConnection.java` if your MySQL settings differ:
```java
private static final String DB_URL  = "jdbc:mysql://localhost:3306/criminal_db?useSSL=false&serverTimezone=UTC";
private static final String DB_USER = "root";
private static final String DB_PASS = "";   // set your password
```

### 5. Build and run
```bash
# Linux / macOS
chmod +x build.sh
./build.sh

# Windows
build.bat
```

The scripts compile everything into `bin/` and launch `criminaldb.ui.MainFrame`.

## Usage Guide

1. **Dashboard** — overview of the system at a glance.
2. **Criminals** — click *Add* to create a record, select a row to edit/delete it, use the search box to filter, and *Upload Photo* to attach an image.
3. **Crimes / Cases / Officers** — browse records and add new crimes or cases.
4. **Analytics** — view crime-type and case-status charts.
5. **Face Scanner** — start the camera and click *Scan Face* to run an identification.

## Face Recognition Status

> **Important:** the face scanner is currently a **demonstration prototype**, not a working recognition engine.

- *Start Camera* shows a placeholder; there is no live webcam feed yet.
- *Scan Face* does **not** compare facial features. After a short delay it returns a random wanted criminal from the database to demonstrate the result card.
- *Capture Frame* in the Criminals screen saves a screen capture as a placeholder, not a webcam image.
- The `face_encodings` table exists in the schema but is not yet used.

Integration paths are documented in the header comment of `FaceRecPanel.java`:

1. **OpenCV (Java)** — webcam capture with `VideoCapture`, detection with `CascadeClassifier`, comparison against stored encodings.
2. **Python bridge** — a `face_match.py` script using the `face_recognition` library, called from Java and returning the matched `criminal_id`.
3. **DeepFace** — `DeepFace.find()` against the stored image folder.

## Roadmap

- [ ] Real webcam capture and face detection (OpenCV)
- [ ] Real face matching using stored embeddings in `face_encodings`
- [ ] Confidence score with a configurable match threshold
- [ ] Login and role-based access for officers
- [ ] Edit/delete for crimes and cases
- [ ] Move DB credentials to a config file or environment variables
- [ ] Export reports (PDF / CSV)
- [ ] Maven or Gradle build

## Responsible Use

Facial recognition can produce false matches and carries privacy and bias risks. This project is for **educational and demonstration purposes** only. All sample records are fictional. Any real-world use would require accurate models, human verification, proper authorization and legal compliance, and a match must never be the sole basis for accusing anyone.

## Contributing

Contributions are welcome:

1. Fork the repository
2. Create a branch: `git checkout -b feature/your-feature`
3. Commit your changes and push the branch
4. Open a pull request

## Author

**Tranum Grover** — [@tranumgrover](https://github.com/tranumgrover)
