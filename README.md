# 📧 Sandesh Email Service - Public Setup

The **Sandesh Email Service** provides a simple and secure way to send, manage, and track emails using **AWS SES** or **SMTP**, all powered by Docker.

This setup lets anyone run the service locally using pre-built Docker images — **no source code required**.

---

## 🚀 Quick Start

### **Prerequisites**
- Docker and Docker Compose installed  
- AWS SES credentials *(or SMTP settings if not using SES)*

---

## ⚙️ Setup Instructions

1. **Clone or create a folder** for the project.  
2. **Add the `.env` file** with your configuration (see example below).  
3. **Run the service:**
   ```bash
   docker compose up -d
