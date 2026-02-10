# newlunch
A specialized mobile commerce platform focused on data-driven discovery. Mobikart uses a sophisticated PostgreSQL architecture to allow users to filter through complex technical specifications from processor clock speeds to battery capacities, ensuring a precise and efficient shopping experience.

This **README.md** is designed to meet Principal Engineer standards. It provides clear architectural context, installation steps, and production deployment guidelines for the **E-Commerce Backend API**.

---

# 🚀 E-Commerce Backend API

A high-performance, production-ready e-commerce REST API built with **FastAPI**. This service powers the e-commerce ecosystem, handling everything from user identity and dynamic product filtering to AWS S3 image management.

> ⚠️ **IMPORTANT:** The payment gateway integration is currently deprecated and should NOT be used in production. Payment processing features are disabled.

## 🗃️ Architecture
This project follows the **Service-Oriented Architecture (SOA)** and the **Repository Pattern**. 
- **Routers (API Layer):** Thin controllers handling HTTP requests/responses and validation via Pydantic.
- **Services (Business Logic):** Handles complex logic and S3 file streaming.
- **Repositories (Data Layer):** Dedicated layer for SQLAlchemy queries, ensuring data persistence logic is decoupled from business rules.
- **Models:** Database schema definitions using SQLAlchemy.
- **Schemas:** Data validation and serialization using Pydantic v2.

---

## 🛠️ Tech Stack
- **Framework:** FastAPI
- **Database:** PostgreSQL (Hosted on AWS RDS)
- **Migrations:** Alembic
- **Storage:** AWS S3 (Product & Profile images)
- **Authentication:** JWT (Stateless)
- **Production Server:** Gunicorn with Uvicorn Workers

---

## 📋 Prerequisites
- Python 3.10+
- PostgreSQL instance (AWS RDS recommended)
- AWS IAM Credentials (S3 `PutObject` & `DeleteObject` permissions)

---

## ⚙️ Environment Variables
Create a `.env` file in the root directory:

```env
# Database (AWS RDS)
DB_USER=your_user
DB_PASSWORD=your_password
DB_HOST=your_rds_endpoint
DB_PORT=5432
DB_NAME=flipcart_db

# Security
SECRET_KEY=your_openssl_generated_hex_key
ALGORITHM=HS256

# AWS S3
AWS_ACCESS_KEY_ID=your_key
AWS_SECRET_ACCESS_KEY=your_secret
AWS_REGION=ap-south-1
AWS_S3_BUCKET_NAME=your_bucket_name

# Internal Security
INTERNAL_WEBHOOK_SECRET=your_custom_shared_key
```

---

## 🚀 Local Setup

1. **Clone and Install:**
   ```bash
   git clone https://github.com/your-username/e-commerce-backend.git
   cd e-commerce-backend
   python -m venv venv
   source venv/bin/activate  # Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

2. **Run Database Migrations:**
   ```bash
   alembic upgrade head
   ```

3. **Start the Development Server:**
   ```bash
   uvicorn app.main:app --reload
   ```

4. **API Documentation:**
   Open [http://localhost:8000/docs](http://localhost:8000/docs) for Swagger UI.

---

## 🌐 Production Deployment (AWS EC2)

### 1. Security Group
Ensure Port `8000` is open for Inbound traffic.

### 2. Systemd Service
Create a service file at `/etc/systemd/system/ecommerce-backend.service`:
```ini
[Unit]
Description=Gunicorn instance to serve E-Commerce Backend
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/e-commerce-backend
ExecStart=/usr/bin/python3 -m gunicorn -w 4 -k uvicorn.workers.UvicornWorker app.main:app --bind 0.0.0.0:8000
Restart=always

[Install]
WantedBy=multi-user.target
```

### 3. Management Commands
```bash
sudo systemctl daemon-reload
sudo systemctl start ecommerce-backend
sudo systemctl enable ecommerce-backend
```

---

## 📂 API Endpoints Summary

| Feature | Endpoint | Method | Auth |
| :--- | :--- | :--- | :--- |
| **Auth** | `/api/v1/auth/register` | POST | Public |
| **Auth** | `/api/v1/auth/login` | POST | Public |
| **Products** | `/api/v1/products/search` | GET | Public |
| **Products** | `/api/v1/products/add` | POST | Seller |
| **Cart** | `/api/v1/shop/cart` | GET | Buyer |
| **Orders** | `/api/v1/orders/checkout` | POST | Buyer |

---

## 🛡️ Security Best Practices Implemented
- **Password Hashing:** Argon2/Bcrypt.
- **SQL Injection Protection:** SQLAlchemy ORM / Parameterized queries.
- **Row-Level Locking:** `with_for_update()` used during order processing to prevent stock overselling.
- **Data Leakage Prevention:** Pydantic schemas filter out sensitive fields (passwords, internal IDs).

---
*Maintained by the E-Commerce Engineering Team.*