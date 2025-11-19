
"""
Bright Minds Academy — Classroom Attendance System (CLI)
Author: Ionuț Ionel (ID: 2322721)

Features:
  1) Add Student (duplicate prevention, auto ID)
  2) Record Attendance (YYYY-MM-DD, Present/Absent)
  3) View by Date (tabular)
  4) Search Student (attendance history)
  5) List Students (roster)
  0) Exit

Data model (in-memory):
  students   : dict[int, dict[str, str]] = { sid: {"name": str, "group": str} }
  attendance : dict[str, dict[int, str]] = { "YYYY-MM-DD": { sid: "P"|"A" } }

No external dependencies. Python 3.9+ recommended.
"""

from __future__ import annotations
from datetime import datetime
from typing import Dict, Optional

# ------------------------- Utilities -------------------------

def valid_date(date_str: str) -> bool:
    """True if date_str matches YYYY-MM-DD and is a real calendar date."""
    try:
        datetime.strptime(date_str, "%Y-%m-%d")
        return True
    except ValueError:
        return False


def normalise_status(s: str) -> Optional[str]:
    """Map user input to 'P' (Present) or 'A' (Absent). Return None if invalid."""
    if not s:
        return None
    t = s.strip().lower()
    if t in {"p", "present"}:
        return "P"
    if t in {"a", "absent"}:
        return "A"
    return None


def display_status(s: Optional[str]) -> str:
    """Make status human-friendly for tabular display."""
    if s == "P":
        return "Present"
    if s == "A":
        return "Absent"
    return "-"


def find_id_by_name(students: Dict[int, Dict[str, str]], name: str) -> Optional[int]:
    """Exact, case-insensitive match by name. Returns sid or None."""
    target = name.strip().lower()
    for sid, info in students.items():
        if info["name"].lower() == target:
            return sid
    return None


def print_menu() -> None:
    print("\n===== Bright Minds Academy - Attendance System =====")
    print("1) Add new student")
    print("2) Record attendance")
    print("3) View attendance by date")
    print("4) Search student attendance")
    print("5) List all students")
    print("0) Exit")

# ---------------------- In-memory storage ----------------------

students: Dict[int, Dict[str, str]] = {}
attendance: Dict[str, Dict[int, str]] = {}
next_student_id: int = 1  # auto-increment

def seed_students() -> None:
    """Preload the 10 students you provided with simple groups."""
    global next_student_id
    initial = [
        ["Ramon Nastase",   "Group A"],
        ["Dragos Ionel",    "Group A"],
        ["Vlad Dimitrie",   "Group B"],
        ["Bandon Josh",     "Group B"],
        ["James Hall",      "Group C"],
        ["Kate Mcbon",      "Group C"],
        ["Lukas Michal",    "Group D"],
        ["Sam Jodarby",     "Group D"],
        ["Ali Khan",        "English-C"],
        ["Sophie Lee",      "Maths-A"],
    ]
    for name, group in initial:
        global students
        students[next_student_id] = {"name": name, "group": group}
        next_student_id += 1

# -------------------------- Actions ---------------------------

def add_student() -> None:
    global next_student_id
    print("\n-- Add New Student --")
    name = input("Enter full name: ").strip()
    group = input("Enter class group (e.g., Maths-A): ").strip()

    if not name or not group:
        print("✖ Name and group are required.")
        return

    if find_id_by_name(students, name) is not None:
        print(f"✖ A student named '{name}' already exists.")
        return

    sid = next_student_id
    students[sid] = {"name": name, "group": group}
    next_student_id += 1
    print(f"✓ Student added with ID: {sid}")


def record_attendance() -> None:
    print("\n-- Record Attendance --")
    date_str = input("Enter date (YYYY-MM-DD): ").strip()
    if not valid_date(date_str):
        print("✖ Invalid date format. Please use YYYY-MM-DD.")
        return

    name = input("Enter student name: ").strip()
    sid = find_id_by_name(students, name)
    if sid is None:
        print(f"✖ Student '{name}' not found.")
        return

    status_in = input("Status (Present/P or Absent/A): ").strip()
    status = normalise_status(status_in)
    if status is None:
        print("✖ Invalid status. Use Present/P or Absent/A.")
        return

    if date_str not in attendance:
        attendance[date_str] = {}
    attendance[date_str][sid] = status
    print(f"✓ Recorded: {students[sid]['name']} (ID {sid}) on {date_str} → {display_status(status)}")


def view_by_date() -> None:
    print("\n-- View Attendance by Date --")
    date_str = input("Enter date (YYYY-MM-DD): ").strip()

    if not valid_date(date_str):
        print("✖ Invalid date format. Please use YYYY-MM-DD.")
        return

    day = attendance.get(date_str, {})
    if not day:
        print("ℹ No records for this date.")
        return

    print(f"\nAttendance for {date_str}:")
    print("-" * 68)
    print(f"{'ID':<4} {'Name':<22} {'Group':<14} {'Status':<10}")
    print("-" * 68)

    # show everyone on roster, mark '-' if not set that day
    for sid in sorted(students.keys()):
        info = students[sid]
        status = display_status(day.get(sid))
        print(f"{sid:<4} {info['name']:<22} {info['group']:<14} {status:<10}")

    print("-" * 68)


def search_student() -> None:
    print("\n-- Search Student Attendance --")
    name = input("Enter student name to search: ").strip()
    sid = find_id_by_name(students, name)
    if sid is None:
        print(f"✖ Student '{name}' not found.")
        return

    info = students[sid]
    print(f"\nAttendance for {info['name']} (ID: {sid}, Group: {info['group']}):")
    rows = []
    for date_str in sorted(attendance.keys()):
        status = attendance[date_str].get(sid)
        if status:
            rows.append((date_str, display_status(status)))

    if not rows:
        print("ℹ No attendance records yet.")
        return

    print(f"{'Date':<12} {'Status'}")
    print("-" * 24)
    for d, s in rows:
        print(f"{d:<12} {s}")


def list_students() -> None:
    print("\n-- Student Roster --")
    if not students:
        print("ℹ No students registered.")
        return

    print(f"{'ID':<4} {'Name':<22} {'Group'}")
    print("-" * 44)
    for sid in sorted(students.keys()):
        info = students[sid]
        print(f"{sid:<4} {info['name']:<22} {info['group']}")
    print("-" * 44)

# --------------------------- Main -----------------------------

def main() -> None:

    try:
        while True:
            print_menu()
            choice = input("Choose an option: ").strip()

            if choice == "0":
                print("Goodbye!")
                break
            elif choice == "1":
                add_student()
            elif choice == "2":
                record_attendance()
            elif choice == "3":
                view_by_date()
            elif choice == "4":
                search_student()
            elif choice == "5":
                list_students()
            else:
                print("✖ Invalid option. Please choose 0–5.")
    except (KeyboardInterrupt, EOFError):
        print("\nExited. Bye!")


if __name__ == "__main__":
    main()
