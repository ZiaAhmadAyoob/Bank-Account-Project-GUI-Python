# 🏦 Bank Account Management System (Python + MySQL + CustomTkinter)

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python)
![MySQL](https://img.shields.io/badge/Database-MySQL-orange?logo=mysql)
![GUI](https://img.shields.io/badge/GUI-CustomTkinter-green?logo=tkinter)
![Platform](https://img.shields.io/badge/Cross--Platform-Windows%20%7C%20Linux%20%7C%20macOS-lightgrey)
![License](https://img.shields.io/badge/License-MIT-purple)

> A full-featured, secure and interactive **Bank Account Management System** built using Python and MySQL with a modern GUI using CustomTkinter.

---

## ✨ Features

- 🔐 **User Authentication** (Login via CNIC & Password)
- 📝 **Account Creation** with validation
- 💳 **Deposit / Withdraw / Transfer** funds
- 📜 **Transfer History** with CSV export
- 🧾 **Balance Inquiry** in real-time
- 🔁 **Live Balance Refresh** every 10 seconds
- 🌗 **Light / Dark Theme Toggle**
- 🔥 **Account Deletion / Deactivation**
- 📁 **Modern, Responsive UI** with CustomTkinter

---

## 📸 Screenshots

| Welcome Screen | Dashboard | Transfer History |
|----------------|-----------|------------------|
| ![Welcome](assets/screenshot1.png) | ![Dashboard](assets/screenshot2.png) | ![History](assets/screenshot3.png) |

---

## ⚙️ Requirements

- Python 3.10+
- MySQL Server
- `customtkinter`
- `mysql-connector-python`

### 🔧 Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/bank-account-system.git
cd bank-account-system

# Create virtual environment (optional but recommended)
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows

# Install dependencies
pip install -r requirements.txt
