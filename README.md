# expense2
python code for expense with excel features
import json
import os
from datetime import datetime
from collections import defaultdict
from openpyxl import Workbook

DATA_FILE = "expenses.json"

# Predefined categories
categories = ["Groceries", "Transportation", "Entertainment", "Utilities", "Other"]

# ---------------- DATA HANDLING ---------------- #

def load_expenses():
    if os.path.exists(DATA_FILE):
        with open(DATA_FILE, "r") as file:
            return json.load(file)
    return []

def save_expenses(expenses):
    with open(DATA_FILE, "w") as file:
        json.dump(expenses, file, indent=4)

# ---------------- EXPENSE CRUD ---------------- #

def add_expense(expenses):
    try:
        amount = float(input("Enter amount: "))
        description = input("Enter description: ")
        date = input("Enter date (YYYY-MM-DD) or press Enter for today: ")
        if not date:
            date = datetime.now().strftime("%Y-%m-%d")

        print("\nCategories:")
        for i, cat in enumerate(categories, 1):
            print(f"{i}. {cat}")
        print(f"{len(categories)+1}. Add new category")

        choice = int(input("Choose category: "))
        if choice == len(categories) + 1:
            new_cat = input("Enter new category: ")
            categories.append(new_cat)
            category = new_cat
        else:
            category = categories[choice - 1]

        expenses.append({
            "date": date,
            "description": description,
            "amount": amount,
            "category": category
        })

        save_expenses(expenses)
        print("✅ Expense added successfully.")

    except ValueError:
        print("❌ Invalid input. Try again.")

def edit_expense(expenses):
    view_expenses(expenses)
    try:
        index = int(input("Enter expense number to edit: ")) - 1
        expenses[index]["amount"] = float(input("New amount: "))
        expenses[index]["description"] = input("New description: ")
        save_expenses(expenses)
        print("✅ Expense updated.")
    except:
        print("❌ Error editing expense.")

def delete_expense(expenses):
    view_expenses(expenses)
    try:
        index = int(input("Enter expense number to delete: ")) - 1
        expenses.pop(index)
        save_expenses(expenses)
        print("🗑 Expense deleted.")
    except:
        print("❌ Error deleting expense.")

def view_expenses(expenses):
    if not expenses:
        print("No expenses found.")
        return
    for i, exp in enumerate(expenses, 1):
        print(f"{i}. {exp['date']} | {exp['description']} | ₹{exp['amount']} | {exp['category']}")

# ---------------- SUMMARY & ANALYSIS ---------------- #

def expense_summary(expenses):
    if not expenses:
        print("No data available.")
        return

    total = sum(exp["amount"] for exp in expenses)
    category_totals = defaultdict(float)

    for exp in expenses:
        category_totals[exp["category"]] += exp["amount"]

    print(f"\n💰 Total Spending: ₹{total:.2f}")
    print("\n📂 Category-wise Spending:")
    for cat, amt in category_totals.items():
        print(f"{cat}: ₹{amt:.2f}")

    highest = max(expenses, key=lambda x: x["amount"])
    lowest = min(expenses, key=lambda x: x["amount"])

    print("\n📈 Highest Expense:", highest)
    print("📉 Lowest Expense:", lowest)

# ---------------- EXCEL EXPORT ---------------- #

def export_to_excel(expenses):
    if not expenses:
        print("No expenses to export.")
        return

    wb = Workbook()
    ws = wb.active
    ws.title = "Expenses"

    ws.append(["Date", "Description", "Amount", "Category"])

    for exp in expenses:
        ws.append([exp["date"], exp["description"], exp["amount"], exp["category"]])

    wb.save("expenses.xlsx")
    print("📁 Expenses exported to expenses.xlsx")

# ---------------- SEARCH ---------------- #

def search_expenses(expenses):
    keyword = input("Enter keyword or date: ").lower()
    results = [exp for exp in expenses if keyword in exp["description"].lower() or keyword in exp["date"]]

    if not results:
        print("No matching records found.")
    else:
        for exp in results:
            print(exp)

# ---------------- MAIN MENU ---------------- #

def main():
    expenses = load_expenses()

    while True:
        print("\n====== Advanced Expense Tracker ======")
        print("1. Add Expense")
        print("2. Edit Expense")
        print("3. Delete Expense")
        print("4. View Expenses")
        print("5. Expense Summary")
        print("6. Export to Excel")
        print("7. Search Expenses")
        print("8. Exit")

        choice = input("Choose an option: ")

        if choice == "1":
            add_expense(expenses)
        elif choice == "2":
            edit_expense(expenses)
        elif choice == "3":
            delete_expense(expenses)
        elif choice == "4":
            view_expenses(expenses)
        elif choice == "5":
            expense_summary(expenses)
        elif choice == "6":
            export_to_excel(expenses)
        elif choice == "7":
            search_expenses(expenses)
        elif choice == "8":
            print("👋 Exiting Expense Tracker. Goodbye!")
            break
        else:
            print("❌ Invalid option.")

if __name__ == "__main__":
    main()
