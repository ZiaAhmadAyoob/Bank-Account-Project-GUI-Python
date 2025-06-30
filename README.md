# 🏦 Bank Account Management System

A full-featured desktop banking system developed using **Python**, **CustomTkinter**, and **MySQL**. This application simulates real-world banking operations such as account creation, login authentication, fund management, transaction history tracking, and more — with a modern and responsive GUI interface.

> 💡 Ideal for students learning database integration, GUI development, and object-oriented programming in Python.

---

## 📖 Table of Contents

- [About the Project](#-about-the-project)
- [Key Features](#-key-features)
- [Technologies Used](#-technologies-used)
- [System Architecture](#-system-architecture)
- [Getting Started](#-getting-started)
- [Screenshots](#-screenshots)
- [Future Improvements](#-future-improvements)
- [Contributing](#-contributing)
- [License](#-license)
- [Author](#-author)

---

## 📌 About the Project

The **Bank Account Management System** is a simulation of digital banking operations, designed to mimic real-world use cases such as:

- Account creation with form validation
- CNIC-based secure login system
- Account balance operations (Deposit, Withdraw)
- Fund transfers between users
- Auto-refreshing dashboard with real-time data
- Export of transaction history in CSV format

This project follows a **modular design** approach with clean separation between:
- GUI logic
- Database interaction
- Event-based user actions

---

## ✨ Key Features

- 🌗 **Light/Dark Mode Toggle** for UI customization
- 🔐 **Secure Login** using CNIC & hashed password (SHA-256)
- 🧾 **Account Creation Form** with validation and password toggle
- 💰 **Deposit / Withdraw Money** with validation
- 🔁 **Transfer Funds** between accounts with balance check
- 📜 **View Transaction History** filtered by user
- 📤 **Export Transfer History** to `.csv` format
- 🔄 **Auto-Refresh Dashboard** for real-time balance updates
- 🗑️ **Delete Account** functionality
- 🧩 Fully object-oriented and event-driven programming

---

## 💻 Technologies Used

| Technology       | Description                                  |
|------------------|----------------------------------------------|
| Python 3.9       | Main programming language                    |
| CustomTkinter    | Advanced GUI library for modern interfaces   |
| MySQL            | Relational database system                   |
| mysql-connector-python | Python-MySQL connector library          |
| hashlib          | Used for SHA-256 password hashing            |
| CSV              | Exporting data (transaction history)         |
| threading        | For async message display (flash messages)   |

---

## 🏗️ System Architecture

### 🔹 Flow Diagram:
Start
|
Show Main Menu
|-------|----------
| | |
Create Login Exit
| |
Submit Validate
| |
Store Dashboard


### 🔹 Class Structure:
+--------------+ +--------------+
| BankDB |<------>| BankApp |
+--------------+ +--------------+
| + init_db() | | + GUI setup |
| + connect() | | + Actions |
+--------------+ +--------------+


---

## 🚀 Getting Started

### 📋 Prerequisites

Ensure the following software is installed:

- Python 3.x
- MySQL Server
- pip (Python package manager)

### 📦 Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/yourusername/bank-account-management.git
   cd bank-account-management
   pip install customtkinter mysql-connector-python
## 👤 Author
### Zia
📧 Email: zia63000@gmail.com
