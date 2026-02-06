# Contact Management System
# Functions + Dictionaries + File Handling

import json
import re
import csv
from datetime import datetime
import os

FILE_NAME = "contacts.json"
BACKUP_FILE = "contacts_backup.json"

# ---------------- VALIDATION FUNCTIONS ---------------- #

def validate_phone(phone):
    digits = re.sub(r"\D", "", phone)
    if 10 <= len(digits) <= 15:
        return True, digits
    return False, None

def validate_email(email):
    if not email:
        return True
    pattern = r'^[\w\.-]+@[\w\.-]+\.\w{2,}$'
    return re.match(pattern, email) is not None

# ---------------- FILE OPERATIONS ---------------- #

def load_contacts():
    if os.path.exists(FILE_NAME):
        with open(FILE_NAME, "r") as f:
            return json.load(f)
    return {}

def save_contacts(contacts):
    if os.path.exists(FILE_NAME):
        with open(FILE_NAME, "r") as f:
            with open(BACKUP_FILE, "w") as b:
                b.write(f.read())

    with open(FILE_NAME, "w") as f:
        json.dump(contacts, f, indent=4)

# ---------------- CRUD OPERATIONS ---------------- #

def add_contact(contacts):
    name = input("Enter name: ").strip().title()
    if not name:
        print("❌ Name cannot be empty")
        return

    if name in contacts:
        print("⚠️ Contact already exists!")
        return

    phone = input("Enter phone: ")
    valid, phone = validate_phone(phone)
    if not valid:
        print("❌ Invalid phone number")
        return

    email = input("Enter email (optional): ")
    if not validate_email(email):
        print("❌ Invalid email format")
        return

    address = input("Enter address (optional): ")
    group = input("Enter group (Friends/Work/Family): ") or "Other"

    contacts[name] = {
        "phone": phone,
        "email": email or None,
        "address": address or None,
        "group": group,
        "created_at": datetime.now().isoformat(),
        "updated_at": datetime.now().isoformat()
    }

    save_contacts(contacts)
    print("✅ Contact added successfully")

def search_contact(contacts):
    term = input("Enter name or phone to search: ").lower()
    found = False

    for name, info in contacts.items():
        if term in name.lower() or term in info["phone"]:
            display_contact(name, info)
            found = True

    if not found:
        print("❌ No matching contact found")

def update_contact(contacts):
    name = input("Enter contact name to update: ").title()
    if name not in contacts:
        print("❌ Contact not found")
        return

    print("Leave blank to keep old value")

    phone = input("New phone: ")
    if phone:
        valid, phone = validate_phone(phone)
        if not valid:
            print("❌ Invalid phone")
            return
        contacts[name]["phone"] = phone

    email = input("New email: ")
    if email and not validate_email(email):
        print("❌ Invalid email")
        return
    if email:
        contacts[name]["email"] = email

    address = input("New address: ")
    if address:
        contacts[name]["address"] = address

    group = input("New group: ")
    if group:
        contacts[name]["group"] = group

    contacts[name]["updated_at"] = datetime.now().isoformat()
    save_contacts(contacts)
    print("✅ Contact updated")

def delete_contact(contacts):
    name = input("Enter contact name to delete: ").title()
    if name not in contacts:
        print("❌ Contact not found")
        return

    confirm = input(f"Are you sure to delete {name}? (y/n): ")
    if confirm.lower() == "y":
        del contacts[name]
        save_contacts(contacts)
        print("🗑️ Contact deleted")

def display_all_contacts(contacts):
    if not contacts:
        print("📭 No contacts available")
        return

    for name, info in contacts.items():
        display_contact(name, info)

def display_contact(name, info):
    print("\n----------------------------")
    print(f"👤 Name    : {name}")
    print(f"📞 Phone   : {info['phone']}")
    if info["email"]:
        print(f"📧 Email   : {info['email']}")
    if info["address"]:
        print(f"📍 Address : {info['address']}")
    print(f"👥 Group   : {info['group']}")
    print("----------------------------")

# ---------------- ADVANCED FEATURES ---------------- #

def export_to_csv(contacts):
    with open("contacts.csv", "w", newline="") as f:
        writer = csv.writer(f)
        writer.writerow(["Name", "Phone", "Email", "Address", "Group"])
        for name, info in contacts.items():
            writer.writerow([
                name,
                info["phone"],
                info["email"],
                info["address"],
                info["group"]
            ])
    print("📁 Contacts exported to contacts.csv")

def statistics(contacts):
    print(f"📊 Total Contacts: {len(contacts)}")
    groups = {}
    for info in contacts.values():
        groups[info["group"]] = groups.get(info["group"], 0) + 1
    for g, c in groups.items():
        print(f"  {g}: {c}")

# ---------------- MENU SYSTEM ---------------- #

def menu():
    print("""
========= CONTACT MANAGER =========
1. Add Contact
2. Search Contact
3. Update Contact
4. Delete Contact
5. Display All Contacts
6. Export to CSV
7. Statistics
8. Exit
==================================
""")

def main():
    contacts = load_contacts()

    while True:
        menu()
        choice = input("Choose an option (1-8): ")

        if choice == "1":
            add_contact(contacts)
        elif choice == "2":
            search_contact(contacts)
        elif choice == "3":
            update_contact(contacts)
        elif choice == "4":
            delete_contact(contacts)
        elif choice == "5":
            display_all_contacts(contacts)
        elif choice == "6":
            export_to_csv(contacts)
        elif choice == "7":
            statistics(contacts)
        elif choice == "8":
            print("👋 Exiting... Goodbye!")
            break
        else:
            print("❌ Invalid choice")

if __name__ == "__main__":
    main()
