from tkinter import *
import tkinter as tk
from tkinter import ttk
from tkinter import messagebox
import sqlite3

# Create the main window
root = tk.Tk()
root.title("Personal Training App")

# Create the database
conn = sqlite3.connect('training.db')
c = conn.cursor()

# Create the table if it doesn't exist
c.execute("""CREATE TABLE IF NOT EXISTS clients (
    client_id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT,
    phone TEXT,
    goals TEXT
)""")

# Create the table if it doesn't exist
c.execute("""CREATE TABLE IF NOT EXISTS workouts (
    workout_id INTEGER PRIMARY KEY AUTOINCREMENT,
    client_id INTEGER,
    date TEXT,
    exercise TEXT,
    sets INTEGER,
    reps INTEGER,
    weight REAL,
    FOREIGN KEY (client_id) REFERENCES clients(client_id)
)""")

conn.commit()

# Create the add client function
def add_client():
    def submit_client():
        name = name_entry.get()
        email = email_entry.get()
        phone = phone_entry.get()
        goals = goals_entry.get("1.0", END)
        
        if not name or not goals:
            messagebox.showerror("Error", "Please fill in all required fields")
            return

        # Insert the client data into the database
        c.execute("INSERT INTO clients (name, email, phone, goals) VALUES (?, ?, ?, ?)", (name, email, phone, goals))
        conn.commit()

        # Clear the entry fields
        name_entry.delete(0, END)
        email_entry.delete(0, END)
        phone_entry.delete(0, END)
        goals_entry.delete("1.0", END)

        # Display a success message
        messagebox.showinfo("Success", "Client added successfully")

        # Close the add client window
        add_client_window.destroy()

    # Create the add client window
    add_client_window = tk.Toplevel(root)
    add_client_window.title("Add Client")

    # Create the labels and entry fields
    name_label = tk.Label(add_client_window, text="Name:")
    name_label.grid(row=0, column=0, padx=5, pady=5)
    name_entry = tk.Entry(add_client_window)
    name_entry.grid(row=0, column=1, padx=5, pady=5)

    email_label = tk.Label(add_client_window, text="Email:")
    email_label.grid(row=1, column=0, padx=5, pady=5)
    email_entry = tk.Entry(add_client_window)
    email_entry.grid(row=1, column=1, padx=5, pady=5)

    phone_label = tk.Label(add_client_window, text="Phone:")
    phone_label.grid(row=2, column=0, padx=5, pady=5)
    phone_entry = tk.Entry(add_client_window)
    phone_entry.grid(row=2, column=1, padx=5, pady=5)

    goals_label = tk.Label(add_client_window, text="Goals:")
    goals_label.grid(row=3, column=0, padx=5, pady=5)
    goals_entry = tk.Text(add_client_window, height=5)
    goals_entry.grid(row=3, column=1, padx=5, pady=5)

    # Create the submit button
    submit_button = tk.Button(add_client_window, text="Submit", command=submit_client)
    submit_button.grid(row=4, column=0, columnspan=2, padx=5, pady=5)

# Create the view clients function
def view_clients():
    def view_workouts(client_id):
        # Create the view workouts window
        view_workouts_window = tk.Toplevel(root)
        view_workouts_window.title("View Workouts")

        # Create the treeview
        workouts_tree = ttk.Treeview(view_workouts_window, columns=("date", "exercise", "sets", "reps", "weight"), show="headings")
        workouts_tree.heading("date", text="Date")
        workouts_tree.heading("exercise", text="Exercise")
        workouts_tree.heading("sets", text="Sets")
        workouts_tree.heading("reps", text="Reps")
        workouts_tree.heading("weight", text="Weight")
        workouts_tree.grid(row=0, column=0, padx=5, pady=5)

        # Get the workouts for the selected client
        c.execute("SELECT * FROM workouts WHERE client_id = ?", (client_id,))
        workouts = c.fetchall()

        # Populate the treeview
        for workout in workouts:
            workouts_tree.insert("", END, values=(workout[2], workout[3], workout[4], workout[5], workout[6]))

    # Create the view clients window
    view_clients_window = tk.Toplevel(root)
    view_clients_window.title("View Clients")

    # Create the treeview
    clients_tree = ttk.Treeview(view_clients_window, columns=("name", "email", "phone", "goals"), show="headings")
    clients_tree.heading("name", text="Name")
    clients_tree.heading("email", text="Email")
    clients_tree.heading("phone", text="Phone")
    clients_tree.heading("goals", text="Goals")
    clients_tree.grid(row=0, column=0, padx=5, pady=5)

    # Get the clients from the database
    c.execute("SELECT * FROM clients")
    clients = c.fetchall()

    # Populate the treeview
    for client in clients:
        clients_tree.insert("", END, values=(client[1], client[2], client[3], client[4]), tags=(client[0],))

        # Bind the click event to view workouts
        clients_tree.tag_bind(client[0], "<<TreeviewSelect>>", lambda event, client_id=client[0]: view_workouts(client_id))

# Create the add workout function
def add_workout():
    def submit_workout():
        client_id = client_id_var.get()
        date = date_entry.get()
        exercise = exercise_entry.get()
        sets = sets_entry.get()
        reps = reps_entry.get()
        weight = weight_entry.get()

        if not client_id or not date or not exercise or not sets or not reps:
            messagebox.showerror("Error", "Please fill in all required fields")
            return

        try:
            sets = int(sets)
            reps = int(reps)
            weight = float(weight)
        except ValueError:
            messagebox.showerror("Error", "Sets, reps, and weight must be numbers")
            return

        # Insert the workout data into the database
        c.execute("INSERT INTO workouts (client_id, date, exercise, sets, reps, weight) VALUES (?, ?, ?, ?, ?, ?)", (client_id, date, exercise, sets, reps, weight))
        conn.commit()

        # Clear the entry fields
        date_entry.delete(0, END)
        exercise_entry.delete(0, END)
        sets_entry.delete(0, END)
        reps_entry.delete(0, END)
        weight_entry.delete(0, END)

        # Display a success message
        messagebox.showinfo("Success", "Workout added successfully")

        # Close the add workout window
        add_workout_window.destroy()

    # Create the add workout window
    add_workout_window = tk.Toplevel(root)
    add_workout_window.title("Add Workout")

    # Create the client dropdown
    client_id_var = tk.StringVar(add_workout_window)
    c.execute("SELECT client_id, name FROM clients")
    clients = c.fetchall()
    client_ids = [str(client[0]) for client in clients]
    client_names = [client[1] for client in clients]
    client_id_var.set(client_ids[0])
    client_dropdown = tk.OptionMenu(add_workout_window, client_id_var, *client_ids)
    client_dropdown.grid(row=0, column=0, padx=5, pady=5)

    # Create the labels and entry fields
    date_label = tk.Label(add_workout_window, text="Date:")
    date_label.grid(row=1, column=0, padx=5, pady=5)
    date_entry = tk.Entry(add_workout_window)
    date_entry.grid(row=1, column=1, padx=5, pady=5)

    exercise_label = tk.Label(add_workout_window, text="Exercise:")
    exercise_label.grid(row=2, column=0, padx=5, pady=5)
    exercise_entry = tk.Entry(add_workout_window)
    exercise_entry.grid(row=2, column=1, padx=5, pady=5)

    sets_label = tk.Label(add_workout_window, text="Sets:")
    sets_label.grid(row=3, column=0, padx=5, pady=5)
    sets_entry = tk.Entry(add_workout_window)
    sets_entry.grid(row=3, column=1, padx=5, pady=5)

    reps_label = tk.Label(add_workout_window, text="Reps:")
    reps_label.grid(row=4, column=0, padx=5, pady=5)
    reps_entry = tk.Entry(add_workout_window)
    reps_entry.grid(row=4, column=1, padx=5, pady=5)

    weight_label = tk.Label(add_workout_window, text="Weight:")
    weight_label.grid(row=5, column=0, padx=5, pady=5)
    weight_entry = tk.Entry(add_workout_window)
    weight_entry.grid(row=5, column=1, padx=5, pady=5)

    # Create the submit button
    submit_button = tk.Button(add_workout_window, text="Submit", command=submit_workout)
    submit_button.grid(row=6, column=0, columnspan=2, padx=5, pady=5)

# Create the menu bar
menubar = Menu(root)

# Create the file menu
filemenu = Menu(menubar, tearoff=0)
filemenu.add_command(label="Add Client", command=add_client)
filemenu.add_command(label="View Clients", command=view_clients)
filemenu.add_command(label="Add Workout", command=add_workout)
filemenu.add_separator()
filemenu.add_command(label="Exit", command=root.quit)
menubar.add_cascade(label="File", menu=filemenu)

# Display the menu bar
root.config(menu=menubar)

# Run the main loop
root.mainloop()

# Close the database connection
conn.close()
