import tkinter as tk
from tkinter import messagebox, ttk
import pymysql

# Function to establish a connection to MySQL
def connect_to_database():
    return pymysql.connect(host='localhost',
                           user='root',
                           password='**********',
                           database='database_name')

# Function to clear the entry fields
def clear_entries():
    student_email_entry.delete(0, 'end')
    student_password_entry.delete(0, 'end')

# Function to clear the entry fields in registration window
def clear_entries_register():
    for entry in [panther_id_entry, first_name_entry, last_name_entry, address_entry,
                  phone_number_entry, student_email_entry_reg, student_password_entry_reg]:
        entry.delete(0, 'end')

# Function to authenticate a student
def login():
    student_email = student_email_entry.get()
    student_password = student_password_entry.get()
    
    # Connect to the database
    db = connect_to_database()
    cursor = db.cursor()
    try:
        # Check if the student email and password match in the database
        cursor.execute("SELECT * FROM Student WHERE StudentEmail=%s AND StudentPassword=%s", (student_email, student_password))
        student = cursor.fetchone()
        if student:
            messagebox.showinfo("Success", "Login successful!")
            # Show the database window
            show_database_window()
        else:
            messagebox.showerror("Error", "Invalid student email or password")
    except Exception as e:
        messagebox.showerror("Error", str(e))
    finally:
        db.close()

# Function to register a new student
def register_student():
    # Retrieve values from registration fields
    panther_id = panther_id_entry.get()
    first_name = first_name_entry.get()
    last_name = last_name_entry.get()
    address = address_entry.get()
    phone_number = phone_number_entry.get()
    student_email = student_email_entry_reg.get()
    student_password = student_password_entry_reg.get()
    
    # Connect to the database
    db = connect_to_database()
    cursor = db.cursor()
    try:
        # Check if a student with the same email already exists
        cursor.execute("SELECT * FROM Student WHERE StudentEmail = %s", (student_email,))
        existing_student = cursor.fetchone()
        if existing_student:
            messagebox.showerror("Error", "Student with the same PantherID already exists")
        else:
            # Insert the new student record into the database
            cursor.execute("INSERT INTO Student (PantherID, FirstName, LastName, Address, PhoneNumber, StudentEmail, StudentPassword) VALUES (%s, %s, %s, %s, %s, %s, %s)",
                           (panther_id, first_name, last_name, address, phone_number, student_email, student_password))
            db.commit()  # Commit the transaction
            messagebox.showinfo("Success", "Registration successful!")
    except Exception as e:
        messagebox.showerror("Error", str(e))
    finally:
        db.close()

# Function to display the registration window
def register():
    # Show the registration window
    root.withdraw()  # Hide the main window
    register_window.deiconify()  # Show the registration window

# Function to show the database window
def show_database_window():
    # Hide the main window
    root.withdraw()
    
    # Create the database window
    database_window = tk.Toplevel(root)
    database_window.title("Student Database")
    
    # Connect to the database
    db = connect_to_database()
    cursor = db.cursor()
    
    try:
        # Fetch all the student records from the database
        cursor.execute("SELECT * FROM Student")
        students = cursor.fetchall()
        
        # Create a treeview to display the student records
        tree = ttk.Treeview(database_window)
        tree["columns"] = ("1", "2", "3", "4", "5", "6")
        tree.column("#0", width=100)
        tree.column("1", width=100)
        tree.column("2", width=100)
        tree.column("3", width=100)
        tree.column("4", width=100)
        tree.column("5", width=100)
        tree.column("6", width=100)
        tree.heading("#0", text="PantherID")
        tree.heading("1", text="FirstName")
        tree.heading("2", text="LastName")
        tree.heading("3", text="Address")
        tree.heading("4", text="PhoneNumber")
        tree.heading("5", text="StudentEmail")
        tree.heading("6", text="StudentPassword")
        tree.grid(row=0, column=0, padx=10, pady=10)
        
        # Populate the treeview with the student records
        for student in students:
            tree.insert("", "end", text=student[0], values=student[1:])
        
        # Create update and delete buttons
        update_btn = tk.Button(database_window, text="Update", command=lambda: update_student(tree))
        update_btn.grid(row=1, column=0, padx=10, pady=10)
        
        delete_btn = tk.Button(database_window, text="Delete", command=lambda: delete_student(tree))
        delete_btn.grid(row=2, column=0, padx=10, pady=10)
        
        # Add a close button
        close_btn = tk.Button(database_window, text="Close", command=database_window.destroy)
        close_btn.grid(row=3, column=0, padx=10, pady=10)
    except Exception as e:
        messagebox.showerror("Error", str(e))
    finally:
        db.close()

# Function to update a student record
def update_student(tree):
    # Get the selected item from the treeview
    selected_item = tree.focus()
    if selected_item:
        # Get the values of the selected item
        values = tree.item(selected_item, "values")
        panther_id = tree.item(selected_item, "text")
        
        # Create a new window for the update form
        update_window = tk.Toplevel(root)
        update_window.title("Update Student")
        
        # Create and place labels and entry fields for the update form
        tk.Label(update_window, text="Panther ID:").grid(row=0, column=0, sticky='w')
        panther_id_entry = tk.Entry(update_window)
        panther_id_entry.grid(row=0, column=1)
        panther_id_entry.insert(0, panther_id)
        
        tk.Label(update_window, text="First Name:").grid(row=1, column=0, sticky='w')
        first_name_entry = tk.Entry(update_window)
        first_name_entry.grid(row=1, column=1)
        first_name_entry.insert(0, values[0])
        
        tk.Label(update_window, text="Last Name:").grid(row=2, column=0, sticky='w')
        last_name_entry = tk.Entry(update_window)
        last_name_entry.grid(row=2, column=1)
        last_name_entry.insert(0, values[1])
        
        tk.Label(update_window, text="Address:").grid(row=3, column=0, sticky='w')
        address_entry = tk.Entry(update_window)
        address_entry.grid(row=3, column=1)
        address_entry.insert(0, values[2])
        
        tk.Label(update_window, text="Phone Number:").grid(row=4, column=0, sticky='w')
        phone_number_entry = tk.Entry(update_window)
        phone_number_entry.grid(row=4, column=1)
        phone_number_entry.insert(0, values[3])
        
        tk.Label(update_window, text="Student Email:").grid(row=5, column=0, sticky='w')
        student_email_entry = tk.Entry(update_window)
        student_email_entry.grid(row=5, column=1)
        student_email_entry.insert(0, values[4])
        
        tk.Label(update_window, text="Student Password:").grid(row=6, column=0, sticky='w')
        student_password_entry = tk.Entry(update_window)
        student_password_entry.grid(row=6, column=1)
        student_password_entry.insert(0, values[5])
        
        # Create a button to save the updated record
        save_btn = tk.Button(update_window, text="Save", command=lambda: save_updated_record(tree, selected_item,
                                                                                              panther_id_entry.get(),
                                                                                              first_name_entry.get(),
                                                                                              last_name_entry.get(),
                                                                                              address_entry.get(),
                                                                                              phone_number_entry.get(),
                                                                                              student_email_entry.get(),
                                                                                              student_password_entry.get()))
        save_btn.grid(row=7, column=0, padx=10, pady=10)
    else:
        messagebox.showerror("Error", "No item selected.")

# Function to save the updated record
def save_updated_record(tree, selected_item, panther_id, first_name, last_name, address, phone_number, student_email, student_password):
    # Connect to the database
    db = connect_to_database()
    cursor = db.cursor()
    try:
        # Update the student record in the database
        cursor.execute("UPDATE Student SET PantherID=%s, FirstName=%s, LastName=%s, Address=%s, PhoneNumber=%s, StudentEmail=%s, StudentPassword=%s WHERE PantherID=%s",
                       (panther_id, first_name, last_name, address, phone_number, student_email, student_password, panther_id))
        db.commit()
        
        # Update the treeview with the new values
        tree.item(selected_item, text=panther_id, values=(first_name, last_name, address, phone_number, student_email, student_password))
        messagebox.showinfo("Success", "Student record updated successfully.")
    except Exception as e:
        messagebox.showerror("Error", str(e))
    finally:
        db.close()

# Function to delete a student record
def delete_student(tree):
    # Get the selected item from the treeview
    selected_item = tree.focus()
    if selected_item:
        # Get the panther ID of the selected item
        panther_id = tree.item(selected_item, "text")
        
        # Connect to the database
        db = connect_to_database()
        cursor = db.cursor()
        try:
            # Delete the student record from the database
            cursor.execute("DELETE FROM Student WHERE PantherID=%s", (panther_id,))
            db.commit()
            
            # Remove the item from the treeview
            tree.delete(selected_item)
            messagebox.showinfo("Success", "Student record deleted successfully.")
        except Exception as e:
            messagebox.showerror("Error", str(e))
        finally:
            db.close()
    else:
        messagebox.showerror("Error", "No item selected.")

# Create the main window
root = tk.Tk()
root.title("Student Authentication")

# Create and place labels and entry fields for login
tk.Label(root, text="Student Email:").grid(row=0, column=0, sticky='w')
student_email_entry = tk.Entry(root)
student_email_entry.grid(row=0, column=1)

tk.Label(root, text="Student Password:").grid(row=1, column=0, sticky='w')
student_password_entry = tk.Entry(root, show='*')
student_password_entry.grid(row=1, column=1)

# Create and place buttons for login, register, and clear
login_btn = tk.Button(root, text="Login", command=login)
login_btn.grid(row=2, column=0)

register_btn = tk.Button(root, text="Register", command=register)  # Bind to register function
register_btn.grid(row=2, column=1)

clear_btn = tk.Button(root, text="Clear", command=clear_entries)
clear_btn.grid(row=2, column=2)

# Create the registration window
register_window = tk.Toplevel(root)
register_window.title("Student Registration")
register_window.withdraw()  # Hide the registration window initially

# Create and place labels and entry fields for registration
tk.Label(register_window, text="Panther ID:").grid(row=0, column=0, sticky='w')
panther_id_entry = tk.Entry(register_window)
panther_id_entry.grid(row=0, column=1)

tk.Label(register_window, text="First Name:").grid(row=1, column=0, sticky='w')
first_name_entry = tk.Entry(register_window)
first_name_entry.grid(row=1, column=1)

tk.Label(register_window, text="Last Name:").grid(row=2, column=0, sticky='w')
last_name_entry = tk.Entry(register_window)
last_name_entry.grid(row=2, column=1)

tk.Label(register_window, text="Address:").grid(row=3, column=0, sticky='w')
address_entry = tk.Entry(register_window)
address_entry.grid(row=3, column=1)

tk.Label(register_window, text="Phone Number:").grid(row=4, column=0, sticky='w')
phone_number_entry = tk.Entry(register_window)
phone_number_entry.grid(row=4, column=1)

tk.Label(register_window, text="Student Email:").grid(row=5, column=0, sticky='w')
student_email_entry_reg = tk.Entry(register_window)
student_email_entry_reg.grid(row=5, column=1)

tk.Label(register_window, text="Student Password:").grid(row=6, column=0, sticky='w')
student_password_entry_reg = tk.Entry(register_window)
student_password_entry_reg.grid(row=6, column=1)

# Create and place register and clear buttons for registration
register_btn_reg = tk.Button(register_window, text="Register", command=register_student)
register_btn_reg.grid(row=7, column=0)

clear_btn_reg = tk.Button(register_window, text="Clear", command=clear_entries_register)
clear_btn_reg.grid(row=7, column=1)

# Run the Tkinter event loop
root.mainloop()

