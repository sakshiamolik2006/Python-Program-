import sqlite3
import tkinter as tk
from tkinter import messagebox, ttk

def init_db():
    conn = sqlite3.connect("expenses.db")
    cursor = conn.cursor()
    cursor.execute('''CREATE TABLE IF NOT EXISTS expenses (
                        id INTEGER PRIMARY KEY,
                        amount REAL,
                        category TEXT,
                        description TEXT,
                        date TEXT)''')
    conn.commit()
    conn.close()

def add_expense():
    amount = amount_entry.get()
    category = category_var.get()
    description = description_entry.get()
    date = date_entry.get()
    
    if not amount or not date:
        messagebox.showerror("Error", "Amount and Date are required!")
        return
    
    try:
        conn = sqlite3.connect("expenses.db")
        cursor = conn.cursor()
        cursor.execute("INSERT INTO expenses (amount, category, description, date) VALUES (?, ?, ?, ?)",
                       (float(amount), category, description, date))
        conn.commit()
        conn.close()
        messagebox.showinfo("Success", "Expense added successfully!")
        show_expenses()
    except Exception as e:
        messagebox.showerror("Error", str(e))

def show_expenses():
    for row in expense_table.get_children():
        expense_table.delete(row)
    
    conn = sqlite3.connect("expenses.db")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM expenses")
    rows = cursor.fetchall()
    conn.close()
    
    for row in rows:
        expense_table.insert("", tk.END, values=row)

def analyze_expenses():
    conn = sqlite3.connect("expenses.db")
    cursor = conn.cursor()
    cursor.execute("SELECT category, SUM(amount) FROM expenses GROUP BY category")
    rows = cursor.fetchall()
    conn.close()
    
    analysis_text.set("\n".join([f"{cat}: ${amt:.2f}" for cat, amt in rows]))

init_db()

root = tk.Tk()
root.title("Expense Tracker")
root.geometry("600x500")

tk.Label(root, text="Amount").pack()
amount_entry = tk.Entry(root)
amount_entry.pack()

tk.Label(root, text="Category").pack()
category_var = tk.StringVar(value="Food")
categories = ["Food", "Transport", "Entertainment", "Other"]
category_menu = ttk.Combobox(root, textvariable=category_var, values=categories)
category_menu.pack()

tk.Label(root, text="Description").pack()
description_entry = tk.Entry(root)
description_entry.pack()

tk.Label(root, text="Date (YYYY-MM-DD)").pack()
date_entry = tk.Entry(root)
date_entry.pack()

tk.Button(root, text="Add Expense", command=add_expense).pack()
tk.Button(root, text="Show Expenses", command=show_expenses).pack()
tk.Button(root, text="Analyze Expenses", command=analyze_expenses).pack()

expense_table = ttk.Treeview(root, columns=("ID", "Amount", "Category", "Description", "Date"), show="headings")
for col in ("ID", "Amount", "Category", "Description", "Date"):
    expense_table.heading(col, text=col)
expense_table.pack()

analysis_text = tk.StringVar()
tk.Label(root, textvariable=analysis_text).pack()

show_expenses()
root.mainloop()
