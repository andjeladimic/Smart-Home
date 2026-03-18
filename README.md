# Smart Home App

A RESTful backend API for managing smart home devices, built with **FastAPI** and **MySQL**. The application supports user authentication via JWT tokens, role-based access control, and management of multiple device types across different rooms.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [API Endpoints](#api-endpoints)
- [Role System](#role-system)
- [Device Types](#device-types)
- [Database Schema](#database-schema)
- [Project Structure](#project-structure)
- [Technologies](#technologies)
- [Setup & Installation](#setup--installation)

---

## Overview

Smart Home App is a backend web application that simulates a smart home management system. Multiple users can register, log in, and interact with smart devices installed in different rooms of a home. Each user is assigned a role that determines which rooms and actions they have access to. The system uses JWT-based authentication and bcrypt password hashing to ensure secure access.

---

## Features

- **JWT Authentication** — secure login with access tokens (HS256, 30-minute expiry)
- **bcrypt password hashing** — passwords are never stored in plain text
- **Role-based access control** — three user roles with different permission levels
- **Location-based device management** — devices are organized by room
- **Polymorphic device model** — single table inheritance for multiple device types
- **Pagination, filtering & sorting** — flexible device queries via query parameters
- **File upload** — endpoint for uploading files to the server
- **Alembic migrations** — versioned database schema management

---

## API Endpoints

### Users — `/users`

| Method | Endpoint | Description | Auth required |
|--------|----------|-------------|---------------|
| `POST` | `/users/register` | Register a new user | ❌ |
| `POST` | `/users/login` | Login and receive JWT token | ❌ |
| `GET` | `/users/{id}` | Get user by ID | ❌ |

> New users are automatically assigned the **Regular** role and granted access to the Living Room and Kitchen.

### Devices — `/devices`

| Method | Endpoint | Description | Auth required |
|--------|----------|-------------|---------------|
| `GET` | `/devices/` | List devices with pagination, filtering & sorting | ❌ |

Query parameters for `/devices/`:
- `page` — page number (default: 1)
- `page_size` — results per page (default: 10, max: 100)
- `location_id` — filter by room
- `device_type` — filter by device type (`lightbulb`, `thermostat`, `oven`, `doorlock`)
- `sort` — sort by device type (`asc` or `desc`)

### Files — `/files`

| Method | Endpoint | Description | Auth required |
|--------|----------|-------------|---------------|
| `POST` | `/files/upload/` | Upload a file to the server | ❌ |

---

## Role System

The application implements three user roles:

| Role | ID | Permissions |
|------|----|-------------|
| **Admin** | 1 | Assign roles, add new devices |
| **Regular** | 2 | View and update devices in assigned rooms |
| **Elevated** | 3 | View, update, and add devices in assigned rooms |

Users are linked to specific locations via a many-to-many `user_location` association table — each user can only access rooms they have been assigned to.

---

## Device Types

The app uses **single table inheritance (STI)** via SQLAlchemy's polymorphic mapping. All devices share one database table, with type-specific attributes stored in the same row:

| Type | Identifier | Specific Attributes |
|------|-----------|---------------------|
| **LightBulb** | `lightbulb` | `brightness` (float), `color` (string) |
| **Thermostat** | `thermostat` | `temperature` (float, must be ≥ 0) |
| **Oven** | `oven` | `temperature` (float, must be ≥ 0) |
| **DoorLock** | `doorlock` | — |

All devices share: `device_id`, `location_id`, `status`, `device_type`.

---

## Database Schema

```
users ──────────── user_location ──────────── locations
  │                                                │
  └── role_id → roles                    location_id → devices
                                                        │
                                              device_type (polymorphic)
                                         ┌──────────────────────────┐
                                         │  LightBulb │ Thermostat  │
                                         │  DoorLock  │    Oven     │
                                         └──────────────────────────┘
```

---

## Project Structure

```
smart_home/
│
├── app/
│   ├── main.py               # FastAPI app entry point, router registration
│   ├── models.py             # SQLAlchemy ORM models (User, Role, Device, Location, ...)
│   ├── schemas.py            # Pydantic schemas for request/response validation
│   ├── db.py                 # Database engine and session setup
│   ├── dependencies.py       # JWT token validation, get_current_user dependency
│   ├── utils.py              # Password hashing and JWT token generation
│   ├── settings.py           # App configuration (DB credentials, secret key, token expiry)
│   └── routers/
│       ├── users.py          # User registration, login
│       ├── devices.py        # Device listing with pagination, filtering, sorting
│       └── files.py          # File upload
│
├── migrations/               # Alembic migration scripts
├── smart_home_db.sql         # MySQL database dump
└── .venv/                    # Python virtual environment
```

---

## Technologies

- **Python 3.10+**
- **FastAPI** — web framework
- **SQLAlchemy** — ORM and database interaction
- **Alembic** — database migrations
- **MySQL** (via **PyMySQL**) — relational database
- **Pydantic** — data validation and serialization
- **python-jose** — JWT token generation and validation
- **passlib + bcrypt** — password hashing
- **Uvicorn** — ASGI server

---

## Setup & Installation

### Prerequisites
- Python 3.10+
- MySQL server running locally

### Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/andjeladimic/Smart-Home.git
   cd Smart-Home/smart_home
   ```

2. **Create and activate a virtual environment**
   ```bash
   python -m venv .venv
   # Windows:
   .venv\Scripts\activate
   # Linux/macOS:
   source .venv/bin/activate
   ```

3. **Install dependencies**
   ```bash
   pip install fastapi uvicorn sqlalchemy pymysql alembic python-jose passlib bcrypt pydantic[email]
   ```

4. **Set up the database**

   Import the provided SQL dump into MySQL:
   ```bash
   mysql -u root -p < smart_home_db.sql
   ```

   Or apply Alembic migrations from scratch:
   ```bash
   alembic upgrade head
   ```

5. **Configure credentials**

   Edit `app/settings.py` to match your local MySQL setup:
   ```python
   DB_USERNAME = "root"
   DB_PASSWORD = "your_password"
   DB_HOSTNAME = "localhost"
   DB_PORT = "3306"
   DATABASE_NAME = "smart_home_app"
   ```

6. **Run the application**
   ```bash
   cd app
   uvicorn main:app --reload
   ```

7. **Access the interactive API docs**

   Open your browser at: [http://localhost:8000/docs](http://localhost:8000/docs)

---

## Author

**Anđela Dimić**
