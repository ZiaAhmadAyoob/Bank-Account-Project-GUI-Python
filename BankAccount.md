```python
import mysql.connector
import random
import customtkinter as ctk
from tkinter import messagebox
import csv
from datetime import datetime
import threading
import hashlib

class BankDB:
    def __init__(self):
        self.config = dict(
            host='localhost',
            user='root',
            password='admin',
            database='SQL_Project'
        )
        self.init_database()

    def init_database(self):
        try:
            db = mysql.connector.connect(
                host=self.config['host'],
                user=self.config['user'],
                password=self.config['password']
            )
            cur = db.cursor()
            cur.execute("CREATE DATABASE IF NOT EXISTS SQL_Project")
            cur.execute("USE SQL_Project")
            # cur.execute("DROP TABLE IF EXISTS bank1")
            cur.execute("""
                CREATE TABLE IF NOT EXISTS bank1 (
                    account_number VARCHAR(12) PRIMARY KEY,
                    name VARCHAR(200),
                    username VARCHAR(200),
                    father_name VARCHAR(200),
                    mobile_Number VARCHAR(200),
                    cnic VARCHAR(200),
                    address VARCHAR(200),
                    email VARCHAR(200),
                    password VARCHAR(200),
                    confirm_password VARCHAR(200),
                    balance FLOAT DEFAULT 0,
                    Creation_Time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            """)
            cur.execute("""
                CREATE TABLE IF NOT EXISTS transfer_history (
                    transfer_id INT AUTO_INCREMENT PRIMARY KEY,
                    sender_account VARCHAR(12),
                    receiver_account VARCHAR(12),
                    amount FLOAT,
                    transfer_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                )
            """)
            db.commit()
            print("Database initialized.")
        except mysql.connector.Error as err:
            print(f"Error: {err}")
        finally:
            if db.is_connected():
                cur.close()
                db.close()

    def get_connection(self):
        return mysql.connector.connect(**self.config)


class BankApp:
    def __init__(self):
        self.db = BankDB()
        self.current_mode = "light"
        self.build_main_ui()

    def toggle_theme(self):
        self.current_mode = "dark" if self.current_mode == "light" else "light"
        ctk.set_appearance_mode(self.current_mode)

    def build_main_ui(self):
        ctk.set_appearance_mode(self.current_mode)
        ctk.set_default_color_theme("blue")

        self.app = ctk.CTk()
        self.app.geometry("500x550")
        self.app.title("🏦 Bank Account Management System")

        ctk.CTkLabel(self.app, text="✨ Welcome to Digital Bank ✨", font=("Arial Rounded MT Bold", 28),
                     text_color="#2c3e50").pack(pady=30)

        ctk.CTkButton(self.app, text="🌗 Toggle Dark/Light Mode", font=("Arial", 16), width=300,
                      command=self.toggle_theme).pack(pady=10)

        ctk.CTkButton(self.app, text="📝 Create New Account", font=("Arial", 18), width=300, height=45,
                      corner_radius=20, fg_color="#1abc9c", hover_color="#16a085",
                      command=self.create_account_window).pack(pady=10)

        ctk.CTkButton(self.app, text="🔐 Login to Account", font=("Arial", 18), width=300, height=45,
                      corner_radius=20, fg_color="#3498db", hover_color="#2980b9",
                      command=self.login_account_window).pack(pady=10)

        ctk.CTkButton(self.app, text="🚪 Exit", font=("Arial", 18), width=300, height=45,
                      corner_radius=20, fg_color="#e74c3c", hover_color="#c0392b",
                      command=self.app.destroy).pack(pady=20)

        self.app.mainloop()

    def create_account_window(self):
        win = ctk.CTkToplevel()
        win.title("Create Account")
        win.geometry("500x700")
    
        labels = ["Name", "Username", "Father Name", "Mobile Number", "CNIC", "Address", "Email", "Password", "Confirm Password"]
        entries = {}
    
        for i, label in enumerate(labels):
            l = ctk.CTkLabel(win, text=label)
            l.grid(row=i, column=0, padx=10, pady=5, sticky="w")
            if "Password" in label:
                entry = ctk.CTkEntry(win, show="*")
                def toggle_password(e=entry):
                    e.configure(show="" if e.cget("show") == "*" else "*")
                eye_btn = ctk.CTkButton(win, text="👁", width=40, command=toggle_password)
                eye_btn.grid(row=i, column=2, padx=5)
            else:
                entry = ctk.CTkEntry(win)
    
            entry.grid(row=i, column=1, padx=10, pady=5)
            entry.bind("<Return>", lambda e, next_i=i+1: entries[labels[next_i]].focus() if next_i < len(labels) else None)
            entries[label] = entry

        def submit():
            values = {label: entries[label].get() for label in labels}
            
            if values["Password"] != values["Confirm Password"]:
                messagebox.showerror("Error", "Passwords do not match!")
                return
    
            db = mysql.connector.connect(
                host='localhost', user='root', password='admin', database='SQL_Project')
            cur = db.cursor()
    
            cur.execute("SELECT * FROM bank1 WHERE username = %s OR email = %s",
                        (values["Username"], values["Email"]))
            if cur.fetchone():
                messagebox.showerror("Error", "Username or Email already exists.")
                return
    
            account_number = str(random.randint(100000000000, 999999999999))  # Also fixes duplicate issue
            hashed_pw = hashlib.sha256(values["Password"].encode()).hexdigest()
            values["Password"] = hashed_pw
            values["Confirm Password"] = hashed_pw
            insert_query = """
                INSERT INTO bank1 
                (account_number, name, username, father_name, mobile_Number, cnic, address, email, password, confirm_password)
                VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
            """
            values_tuple = (
                account_number,
                values["Name"],
                values["Username"],
                values["Father Name"],
                values["Mobile Number"],
                values["CNIC"],
                values["Address"],
                values["Email"],
                values["Password"],
                values["Confirm Password"]
            )
            cur.execute(insert_query, values_tuple)
            db.commit()
            messagebox.showinfo("Success", f"Account created. Your Account Number is: {account_number}")
            db.close()
    
        submit_btn = ctk.CTkButton(win, text="Submit", command=submit)
        submit_btn.grid(row=len(labels), columnspan=2, pady=10)

    def show_dashboard(self, account_number, name):
        self.account_number = account_number
        self.dashboard_window = ctk.CTkToplevel()
        self.dashboard_window.title("🏦 Dashboard")
        self.dashboard_window.geometry("500x650")
    
        card = ctk.CTkFrame(self.dashboard_window)
        card.pack(padx=20, pady=20, fill="both", expand=True)
    
        db = self.db.get_connection()
        cur = db.cursor()
        cur.execute("SELECT balance, cnic, Creation_Time FROM bank1 WHERE account_number = %s", (account_number,))
        user = cur.fetchone()
        db.close()
    
        self.balance_label = ctk.CTkLabel(card, text=f"💰 Current Balance: {user[0]} PKR", font=("Arial", 16))
        self.balance_label.pack(pady=5)
    
        ctk.CTkLabel(card, text=f"🆔 Account Number: {account_number}", font=("Arial", 16)).pack(pady=5)
        ctk.CTkLabel(card, text=f"👤 Name: {name}", font=("Arial", 16)).pack(pady=5)
        ctk.CTkLabel(card, text=f"🪪 CNIC: {user[1]}", font=("Arial", 16)).pack(pady=5)
        ctk.CTkLabel(card, text=f"📅 Created: {user[2].strftime('%Y-%m-%d %H:%M:%S')}", font=("Arial", 16)).pack(pady=5)
    
        # Call auto refresh
        self.auto_refresh()
    
        # Buttons
        ctk.CTkButton(self.dashboard_window, text="🔄 Refresh Balance", command=self.refresh_balance).pack(pady=10)
        ctk.CTkButton(self.dashboard_window, text="💳 View Balance", command=lambda: self.view_balance(account_number)).pack(pady=5)
        ctk.CTkButton(self.dashboard_window, text="➕ Deposit Money", command=lambda: self.deposit(account_number)).pack(pady=5)
        ctk.CTkButton(self.dashboard_window, text="➖ Withdraw Money", command=lambda: self.withdraw(account_number)).pack(pady=5)
        ctk.CTkButton(self.dashboard_window, text="🔁 Transfer Funds", command=lambda: self.transfer_funds(account_number)).pack(pady=5)
        ctk.CTkButton(self.dashboard_window, text="📜 View Transfer History", command=lambda: self.view_transfer_history(account_number)).pack(pady=5)
        ctk.CTkButton(self.dashboard_window, text="📂 Export History (CSV)", command=lambda: self.export_history_csv(account_number)).pack(pady=5)
        ctk.CTkButton(self.dashboard_window, text="🔥 Delete Account", fg_color="#e74c3c", hover_color="#c0392b", command=lambda: self.delete_account(account_number)).pack(pady=10)

    def delete_account(self, account_number):
        result = messagebox.askyesno("Confirm", "Are you sure you want to delete your account?")
        if result:
            try:
                db = mysql.connector.connect(host='localhost', user='root', password='admin', database='SQL_Project')
                cur = db.cursor()
                cur.execute("DELETE FROM bank1 WHERE account_number = %s", (account_number,))
                cur.execute("DELETE FROM transfer_history WHERE sender_account = %s OR receiver_account = %s", (account_number, account_number))
                db.commit()
                db.close()
                messagebox.showinfo("Deleted", "Your account has been deleted.")
            except:
                messagebox.showerror("Error", "Account deletion failed.")
    def refresh_balance(self):
        db = self.db.get_connection()
        cur = db.cursor()
        cur.execute("SELECT balance FROM bank1 WHERE account_number = %s", (self.account_number,))
        result = cur.fetchone()
        db.close()
    
        if result:
            self.balance_label.configure(text=f"💰 Current Balance: {result[0]} PKR")
        else:
            self.balance_label.configure(text="⚠️ Balance unavailable.")

    def auto_refresh(self):
        if hasattr(self, 'dashboard_window') and hasattr(self, 'account_number'):
            self.refresh_balance()
            self.dashboard_window.after(10000, self.auto_refresh)

    def login_account_window(self):
        win = ctk.CTkToplevel()
        win.title("Login")
        win.geometry("400x300")
    
        ctk.CTkLabel(win, text="CNIC").pack(pady=10)
        cnic_entry = ctk.CTkEntry(win)
        cnic_entry.pack()
    
        ctk.CTkLabel(win, text="Password").pack(pady=10)
        password_entry = ctk.CTkEntry(win, show="*")
        password_entry.pack()
        def toggle_pw():
            password_entry.configure(show="" if password_entry.cget("show") == "*" else "*")

        eye_btn = ctk.CTkButton(win, text="👁", width=40, command=toggle_pw)
        eye_btn.pack()
    
    
        def login():
            cnic = cnic_entry.get()
            password = password_entry.get()
    
            db = mysql.connector.connect(
                host='localhost', user='root', password='admin', database='SQL_Project')
            cur = db.cursor()
            hashed_pw = hashlib.sha256(password.encode()).hexdigest()
            cur.execute("SELECT * FROM bank1 WHERE cnic = %s AND password = %s", (cnic, hashed_pw))

            user_data = cur.fetchone()
    
            if user_data:
                messagebox.showinfo("Success", f"Welcome {user_data[1]}!")
                self.show_dashboard(user_data[0], user_data[1])

            else:
                messagebox.showerror("Login Failed", "Invalid CNIC or Password")
    
            db.close()
    
        login_btn = ctk.CTkButton(win, text="Login", command=login)
        login_btn.pack(pady=20)
    
    def view_balance(self,account_number):
        db = mysql.connector.connect(host='localhost', user='root', password='admin', database='SQL_Project')
        cur = db.cursor()
        cur.execute("SELECT balance FROM bank1 WHERE account_number = %s", (account_number,))
        bal = cur.fetchone()
        db.close()
        messagebox.showinfo("Balance", f"Current Balance: {bal[0]}") if bal else messagebox.showerror("Error", "Account not found")
    
    def deposit(self,account_number):
        win = ctk.CTkInputDialog(text="Enter amount to deposit:", title="Deposit")
        try:
            amount = float(win.get_input())
            if amount <= 0:
                raise ValueError
            db = mysql.connector.connect(host='localhost', user='root', password='admin', database='SQL_Project')
            cur = db.cursor()
            cur.execute("UPDATE bank1 SET balance = balance + %s WHERE account_number = %s", (amount, account_number))
            db.commit()
            db.close()
            messagebox.showinfo("Success", f"Deposited {amount} successfully")
        except:
            messagebox.showerror("Error", "Invalid input")
    
    
    def withdraw(self,account_number):
        win = ctk.CTkInputDialog(text="Enter amount to withdraw:", title="Withdraw")
        try:
            amount = float(win.get_input())
            db = mysql.connector.connect(host='localhost', user='root', password='admin', database='SQL_Project')
            cur = db.cursor()
            cur.execute("SELECT balance FROM bank1 WHERE account_number = %s", (account_number,))
            balance = cur.fetchone()[0]
            if amount <= 0 or amount > balance:
                raise ValueError
            cur.execute("UPDATE bank1 SET balance = balance - %s WHERE account_number = %s", (amount, account_number))
            db.commit()
            db.close()
            messagebox.showinfo("Success", f"Withdrew {amount} successfully")
        except:
            messagebox.showerror("Error", "Invalid input or Insufficient Balance")
    
    
    def transfer_funds(self,account_number):
        receiver = ctk.CTkInputDialog(text="Enter receiver's account number:", title="Transfer")
        receiver_account = receiver.get_input()
        amount_input = ctk.CTkInputDialog(text="Enter amount to transfer:", title="Transfer")
        try:
            amount = float(amount_input.get_input())
            db = mysql.connector.connect(host='localhost', user='root', password='admin', database='SQL_Project')
            cur = db.cursor()
            cur.execute("SELECT balance FROM bank1 WHERE account_number = %s", (account_number,))
            sender_balance = cur.fetchone()[0]
            if amount <= 0 or amount > sender_balance:
                raise ValueError
            cur.execute("SELECT account_number FROM bank1 WHERE account_number = %s", (receiver_account,))
            if not cur.fetchone():
                raise LookupError
            cur.execute("UPDATE bank1 SET balance = balance - %s WHERE account_number = %s", (amount, account_number))
            cur.execute("UPDATE bank1 SET balance = balance + %s WHERE account_number = %s", (amount, receiver_account))
            cur.execute("INSERT INTO transfer_history (sender_account, receiver_account, amount) VALUES (%s, %s, %s)", (account_number, receiver_account, amount))
            db.commit()
            db.close()
            self.show_flash_message(f"✅ {amount} PKR transferred to {receiver_account}")
        except LookupError:
            messagebox.showerror("Error", "Receiver account not found")
        except:
            messagebox.showerror("Error", "Invalid input or Insufficient Balance")
    
    
    def view_transfer_history(self,account_number):
        db = mysql.connector.connect(host='localhost', user='root', password='admin', database='SQL_Project')
        cur = db.cursor()
        cur.execute("""
            SELECT transfer_id, sender_account, receiver_account, amount, transfer_date
            FROM transfer_history
            WHERE sender_account = %s OR receiver_account = %s
            ORDER BY transfer_date DESC
        """, (account_number, account_number))
        history = cur.fetchall()
        db.close()
    
        history_str = "\n".join([f"ID: {h[0]}, From: {h[1]}, To: {h[2]}, Amount: {h[3]}, Date: {h[4]}" for h in history])
        messagebox.showinfo("Transfer History", history_str if history else "No history found")
    
    
    def export_history_csv(self,account_number):
        db = mysql.connector.connect(
            host='localhost', user='root', password='admin', database='SQL_Project')
        cur = db.cursor()
        cur.execute("""
            SELECT transfer_id, sender_account, receiver_account, amount, transfer_date
            FROM transfer_history
            WHERE sender_account = %s OR receiver_account = %s
            ORDER BY transfer_date DESC
        """, (account_number, account_number))
        history = cur.fetchall()
        db.close()
    
        if not history:
            messagebox.showinfo("No Data", "No transfer history to export.")
            return
    
        filename = f"history_{account_number}.csv"
        with open(filename, mode="w", newline="") as file:
            writer = csv.writer(file)
            writer.writerow(["Transfer ID", "Sender", "Receiver", "Amount", "Date"])
            writer.writerows(history)
    
        messagebox.showinfo("Exported", f"Transfer history exported to {filename}")

    def show_flash_message(self, message):
        flash = ctk.CTkToplevel()
        flash.overrideredirect(True)
        flash.geometry("400x60+100+100")
        flash.configure(fg_color="#27ae60")
        label = ctk.CTkLabel(flash, text=message, font=("Arial", 16), text_color="white")
        label.pack(padx=10, pady=10)
        threading.Thread(target=lambda: flash.after(2000, flash.destroy)).start()

if __name__ == "__main__":
    BankApp()
```
