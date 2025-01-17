import sqlite3
import tkinter as tk
from tkinter import messagebox
from tkcalendar import DateEntry
from tkcalendar import Calendar
from datetime import datetime
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from datetime import datetime
import threading

# Connect to SQLite database (or create it)
connection = sqlite3.connect('users.db')

# Create users.db
cursor = connection.cursor()

cursor.execute('''
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL
)
''')

connection.commit()
connection.close()

# Create events.db
connection = sqlite3.connect('events.db')

cursor = connection.cursor()

cursor.execute('''
CREATE TABLE IF NOT EXISTS events (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL,
    shared_users TEXT NOT NULL,
    event_name TEXT NOT NULL,
    event_date TEXT NOT NULL,
    event_time TEXT NOT NULL
)
''')

connection.commit()
connection.close()

# Create emails.db
connection = sqlite3.connect('emails.db')
cursor = connection.cursor()

cursor.execute('''
CREATE TABLE IF NOT EXISTS emails (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    thread_id INTEGER NOT NULL,  -- Links emails in the same thread
    sender TEXT NOT NULL,
    recipient TEXT NOT NULL,
    subject TEXT NOT NULL,
    body TEXT NOT NULL,
    timestamp TEXT NOT NULL
)
''')

cursor.execute('''
INSERT OR IGNORE INTO emails (id, thread_id, sender, recipient, subject, body, timestamp) VALUES
(1, 101, 'alice@example.com', 'bob@example.com', 'Meeting Update', 'Hi Bob, just a reminder about our meeting tomorrow at 10 AM.', '2025-01-14 09:30:00'),
(2, 101, 'bob@example.com', 'alice@example.com', 'Re: Meeting Update', 'Thanks, Alice. See you then!', '2025-01-14 09:45:00'),
(3, 102, 'carol@example.com', 'team@example.com', 'Weekly Report', 'Hello Team, attached is the weekly report. Let me know if you have questions.', '2025-01-13 15:00:00'),
(4, 103, 'dave@example.com', 'eve@example.com', 'Lunch Plans?', 'Hey Eve, are you free for lunch tomorrow?', '2025-01-14 12:00:00'),
(5, 103, 'eve@example.com', 'dave@example.com', 'Re: Lunch Plans?', 'Hi Dave, sure! Where should we meet?', '2025-01-14 12:15:00'),
(6, 104, 'support@example.com', 'frank@example.com', 'Ticket #12345 Resolved', 'Dear Frank, your issue has been resolved. Let us know if you need further assistance.', '2025-01-12 10:20:00'),
(7, 105, 'grace@example.com', 'harry@example.com', 'Happy Birthday!', 'Happy Birthday, Harry! Hope you have a fantastic day!', '2025-01-14 08:00:00'),
(8, 106, 'admin@example.com', 'all@example.com', 'System Maintenance', 'Dear All, the system will be undergoing maintenance on Friday at 6 PM.', '2025-01-10 17:00:00'),
(9, 107, 'ivan@example.com', 'jack@example.com', 'Project Update', 'Hi Jack, the project is on track for completion by the end of the month.', '2025-01-13 11:45:00'),
(10, 107, 'jack@example.com', 'ivan@example.com', 'Re: Project Update', 'Thanks for the update, Ivan. Let me know if anything changes.', '2025-01-13 12:00:00')
'''
)

connection.commit()
connection.close()

# Create reminders.db
connection = sqlite3.connect('reminders.db')

cursor = connection.cursor()

cursor.execute('''
    CREATE TABLE IF NOT EXISTS reminders (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        owner TEXT,
        shared_users TEXT,
        reminder_name TEXT,
        date TEXT,
        time TEXT,
        email INTEGER
    )
''')

connection.commit()
connection.close()

# Function to handle user login
def login():
    username = username_entry.get()
    password = password_entry.get()

    # Connect to the database
    connection = sqlite3.connect('users.db')
    cursor = connection.cursor()

    # Check if the user exists
    cursor.execute('SELECT * FROM users WHERE username = ? AND password = ?', (username, password))
    user = cursor.fetchone()

    if user:
        # Fetch the list of other users for sharing events
        cursor.execute('SELECT username FROM users WHERE username != ?', (username,))
        global other_users  # Store other users in a global variable
        other_users = [row[0] for row in cursor.fetchall()]

        messagebox.showinfo('Login Success', f'Welcome, {username}!')
        start_page(username)
    else:
        messagebox.showerror('Login Failed', 'Invalid username or password.')
        
    connection.commit()
    connection.close()

def toggle_password():
    if password_entry.cget('show') == '*':
        password_entry.config(show='')  # Show password
        show_password_button.config(text='Hide Password')
    else:
        password_entry.config(show='*')  # Mask password
        show_password_button.config(text='Show Password')

# Function to handle user registration
def register():
    username = username_entry.get()
    password = password_entry.get()

    if not username or not password:
        messagebox.showwarning('Input Error', 'Both fields are required!')
        return

    if len(password) < 6:
        messagebox.showwarning('Input Error', 'Password must be at least 6 characters long!')
        return

    # Connect to the database
    connection = sqlite3.connect('users.db')
    cursor = connection.cursor()

    try:
        cursor.execute('INSERT INTO users (username, password) VALUES (?, ?)', (username, password))
        messagebox.showinfo('Registration Success', 'User registered successfully!')
    except sqlite3.IntegrityError:
        messagebox.showerror('Registration Failed', 'Username already exists.')
    connection.commit()
    connection.close()

# Gets a list of all users other than the current user
def get_other_users(current_user):
    try:
        connection = sqlite3.connect("users.db")
        cursor = connection.cursor()

        # Query to fetch all users except the current user
        cursor.execute("SELECT username FROM users WHERE username != ?", (current_user,))
        users = [row[0] for row in cursor.fetchall()]
        connection.close()
        return users
    except sqlite3.Error as db_error:
        messagebox.showerror("Database Error", f"Failed to fetch users: {db_error}")
        return []


# Function to display a calendar with events
def open_calendar(username):
    def show_events(event_date):
        #Fetch and display events for the selected date.
        for widget in events_frame.winfo_children():
            widget.destroy()  # Clear the previous events

        # Connect to the events database
        connection = sqlite3.connect('events.db')
        cursor = connection.cursor()

        cursor.execute('''
        SELECT event_name, event_time, username 
        FROM events 
        WHERE (username = ? OR shared_users LIKE ?) AND event_date = ?
        ''', (username, f"%{username}%", event_date))
        events = cursor.fetchall()

        if events:
            for event in events:
                event_label = f"Event: {event[0]} | Time: {event[1]} | Created by: {event[2]}"
                tk.Label(events_frame, text=event_label, font=("Helvetica", 12)).pack(pady=2)
                    # Add a "Delete" button
                delete_button = tk.Button(
                    events_frame,
                    text="Delete",
                    command=lambda e_name=event[0], e_date=event_date, e_time=event[1]: 
                        delete_event(username, e_name, e_date, e_time, events_frame)
                )
                delete_button.pack(side=tk.RIGHT)  
                
        else:
            tk.Label(events_frame, text="No events for this date.", font=("Helvetica", 12)).pack()
        connection.commit()
        connection.close()
    
    def on_date_select(event):
        """Handle the date selection from the calendar."""
        selected_date = cal.get_date()
        selected_date = datetime.strptime(selected_date, "%Y-%m-%d").strftime("%Y-%m-%d")
        show_events(selected_date)

    calendar_window = tk.Toplevel()
    calendar_window.title(f"{username}'s Calendar")
    calendar_window.geometry("800x600")

    tk.Label(calendar_window, text=f"{username}'s Calendar", font=("Helvetica", 16)).pack(pady=10)

    # Calendar widget
    cal = Calendar(calendar_window, selectmode="day", date_pattern="yyyy-mm-dd")
    cal.pack(pady=10)
    cal.bind("<<CalendarSelected>>", on_date_select)

    # Events display frame
    events_frame = tk.Frame(calendar_window, padx=10, pady=10)
    events_frame.pack(fill=tk.BOTH, expand=True)

    # Add event button
    tk.Button(calendar_window, text="Add Event", command=lambda: add_event(username, events_frame)).pack(pady=10)

    # Display today's events by default
    today = cal.get_date()
    show_events(today)

    # Close button
    tk.Button(calendar_window, text="Close", command=calendar_window.destroy).pack(pady=10)

# Function to add a new event
def add_event(username, frame):
    def save_event():
        # Retrieve values from input fields
        event_name = event_name_entry.get()
        event_date = event_date_entry.get_date()
        hour = hour_spinbox.get()
        minute = minute_spinbox.get()

        # Retrieve selected users from the Listbox
        try:
            selected_indices = shared_users_menu.curselection()
            selected_users = [shared_users_menu.get(i) for i in selected_indices]
            shared_users = ", ".join(selected_users)  # Convert the list of selected users to a string
        except Exception as e:
            messagebox.showerror("Input Error", f"Error selecting shared users: {e}")
            return

        # Combine hour and minute for event time
        try:
            event_time = f"{int(hour):02}:{int(minute):02}"
        except ValueError:
            messagebox.showerror("Input Error", "Invalid time. Please enter valid hours and minutes.")
            return

        # Validate required fields
        if not event_name or not event_date or not event_time:
            messagebox.showwarning("Input Error", "All fields are required!")
            return

        # Insert the event into the database
        try:
            connection = sqlite3.connect("events.db")
            cursor = connection.cursor()

            cursor.execute(
                '''
                INSERT INTO events (username, shared_users, event_name, event_date, event_time)
                VALUES (?, ?, ?, ?, ?)
                ''',
                (username, shared_users, event_name, event_date, event_time),
            )
            connection.commit()
            connection.close()

            # Show success message and refresh the UI
            messagebox.showinfo("Success", "Event added successfully!")
            event_window.destroy()
            refresh_events(username, frame)
        except sqlite3.Error as db_error:
            messagebox.showerror("Database Error", f"Failed to save event: {db_error}")

    def validate_hour(input_value):
        """Validate that the hour is within 0-23."""
        return input_value.isdigit() and 0 <= int(input_value) <= 23

    def validate_minute(input_value):
        """Validate that the minute is within 0-59."""
        return input_value.isdigit() and 0 <= int(input_value) <= 59
    
    # Create a new window to input event details
    event_window = tk.Toplevel(frame)
    event_window.title("Add New Event")

    # Event Name
    tk.Label(event_window, text="Event Name:").pack(pady=5)
    event_name_entry = tk.Entry(event_window)
    event_name_entry.pack(pady=5)

    # Event Date
    tk.Label(event_window, text="Event Date (YYYY-MM-DD):").pack(pady=5)
    event_date_entry = DateEntry(
        event_window, date_pattern="yyyy-mm-dd", width=12, background="darkblue", foreground="white", borderwidth=2
    )
    event_date_entry.pack(pady=5)

    # Event Time
    tk.Label(event_window, text="Event Time (HH:MM):").pack(pady=5)
    time_frame = tk.Frame(event_window)
    time_frame.pack(pady=5)

    validate_hour_cmd = event_window.register(validate_hour)
    validate_minute_cmd = event_window.register(validate_minute)

    hour_spinbox = tk.Spinbox(
        time_frame,
        from_=0,
        to=23,
        width=3,
        font=("Arial", 12),
        validate="key",
        validatecommand=(validate_hour_cmd, "%P"),
    )
    hour_spinbox.pack(side=tk.LEFT, padx=2)

    tk.Label(time_frame, text=":", font=("Arial", 12)).pack(side=tk.LEFT)

    minute_spinbox = tk.Spinbox(
        time_frame,
        from_=0,
        to=59,
        width=3,
        font=("Arial", 12),
        validate="key",
        validatecommand=(validate_minute_cmd, "%P"),
    )
    minute_spinbox.pack(side=tk.LEFT, padx=2)

    # Shared Users
    tk.Label(event_window, text="Share Event With:").pack(pady=5)

    # Replace `other_users` with your actual list of usernames
    other_users = get_other_users(username)
    shared_users_var = tk.StringVar(value=other_users)

    shared_users_menu = tk.Listbox(
        event_window,
        listvariable=shared_users_var,
        selectmode=tk.MULTIPLE,
        height=5,
        font=("Arial", 12),
    )
    shared_users_menu.pack(pady=5)

    # Buttons
    tk.Button(event_window, text="Save Event", command=save_event).pack(pady=10)
    tk.Button(event_window, text="Cancel", command=event_window.destroy).pack(pady=10)

def delete_event(username, event_name, event_date, event_time, events_frame):
    """Deletes an event from the events database."""
    
    # Connect to the events database
    connection = sqlite3.connect('events.db')
    cursor = connection.cursor()

    try:
        # Execute the delete query
        cursor.execute('''
            DELETE FROM events 
            WHERE username = ? AND event_name = ? AND event_date = ? AND event_time = ?
        ''', (username, event_name, event_date, event_time))

        connection.commit()  # Commit the changes to the database
        messagebox.showinfo("Success", "Event deleted successfully!")
        # Trigger the initial refresh
        refresh_events(username, events_frame)  # Assuming 'events_frame' is accessible here

    except Exception as e:
        messagebox.showerror("Error", f"Failed to delete event: {str(e)}")

    finally:
        connection.close()  # Always close the connection

# Function to refresh events in the calendar window
def refresh_events(username, frame):
    # Clear the frame
    for widget in frame.winfo_children():
        widget.destroy()

    tk.Label(frame, text=f"{username}'s Calendar Events", font=("Helvetica", 16)).pack(pady=10)

    # Connect to the events database
    connection = sqlite3.connect('events.db')
    cursor = connection.cursor()

    # Retrieve events created by the user or shared with the user
    cursor.execute('''
    SELECT event_name, event_date, event_time, username 
    FROM events 
    WHERE username = ? OR shared_users LIKE ?
    ''', (username, f"%{username}%"))
    events = cursor.fetchall()

    # Display the events
    if events:
        for event in events:
            event_label = f"Event: {event[0]} | Date: {event[1]} | Time: {event[2]} | Created by: {event[3]}"
            tk.Label(frame, text=event_label, font=("Helvetica", 12)).pack(pady=5)
    else:
        tk.Label(frame, text="No events found.", font=("Helvetica", 12)).pack(pady=5)
    connection.commit()
    connection.close()

# Function to handle search and display a pop-up
def search_user():
    # Create a new window for user selection
    search_window = tk.Toplevel(window)
    search_window.title("Search User")
    search_window.geometry("400x300")

    tk.Label(search_window, text="Select a user:", font=("Arial", 14)).pack(pady=10)

    # Frame to list users
    user_list_frame = tk.Frame(search_window)
    user_list_frame.pack(fill=tk.BOTH, expand=True)

    # Add a scrollbar
    scrollbar = tk.Scrollbar(user_list_frame, orient=tk.VERTICAL)
    scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

    user_listbox = tk.Listbox(user_list_frame, yscrollcommand=scrollbar.set, font=("Arial", 12), height=15)
    user_listbox.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
    scrollbar.config(command=user_listbox.yview)

    # Fetch all users from the database
    connection = sqlite3.connect("users.db")
    cursor = connection.cursor()
    cursor.execute("SELECT username FROM users")
    users = cursor.fetchall()
    connection.commit()
    connection.close()

    # Populate the listbox with usernames
    for user in users:
        user_listbox.insert(tk.END, user[0])

    # Function to handle user selection
    def select_user():
        selected_user = user_listbox.get(tk.ACTIVE)
        if selected_user:
            messagebox.showinfo("User Selected", f"User '{selected_user}' has been selected.")
            search_window.destroy()
        else:
            messagebox.showwarning("No Selection", "Please select a user from the list.")

    # Add a Select button
    tk.Button(search_window, text="Select User", command=select_user).pack(pady=10)

    # Add a Close button
    tk.Button(search_window, text="Close", command=search_window.destroy).pack(pady=5)

def open_all_users_calendar():
    def show_all_events(event_date):
        # Fetch and display events for the selected date for all users.
        for widget in events_frame.winfo_children():
            widget.destroy()  # Clear the previous events

        # Connect to the events database
        connection = sqlite3.connect('events.db')
        cursor = connection.cursor()

        cursor.execute('''
        SELECT event_name, event_time, username 
        FROM events 
        WHERE event_date = ?
        ''', (event_date,))
        events = cursor.fetchall()

        if events:
            for event in events:
                event_label = f"Event: {event[0]} | Time: {event[1]} | Created by: {event[2]}"
                tk.Label(events_frame, text=event_label, font=("Helvetica", 12)).pack(pady=2)
        else:
            tk.Label(events_frame, text="No events for this date.", font=("Helvetica", 12)).pack()
        connection.commit()
        connection.close()

    def on_date_select(event):
        """Handle the date selection from the calendar."""
        selected_date = cal.get_date()
        selected_date = datetime.strptime(selected_date, "%Y-%m-%d").strftime("%Y-%m-%d")
        show_all_events(selected_date)

    calendar_window = tk.Toplevel()
    calendar_window.title("All Users' Calendar")
    calendar_window.geometry("800x600")

    tk.Label(calendar_window, text="All Users' Calendar", font=("Helvetica", 16)).pack(pady=10)

    # Calendar widget
    cal = Calendar(calendar_window, selectmode="day", date_pattern="yyyy-mm-dd")
    cal.pack(pady=10)
    cal.bind("<<CalendarSelected>>", on_date_select)

    # Events display frame
    events_frame = tk.Frame(calendar_window, padx=10, pady=10)
    events_frame.pack(fill=tk.BOTH, expand=True)

    # Display today's events by default
    today = cal.get_date()
    show_all_events(today)

    # Close button
    tk.Button(calendar_window, text="Close", command=calendar_window.destroy).pack(pady=10)

def open_email_page(username):
    email_window = tk.Toplevel(window)
    email_window.title(f"{username}'s Emails")
    email_window.geometry("800x600")

    tk.Label(email_window, text="Email Threads", font=("Helvetica", 16)).pack(pady=10)

    # Frame for the table
    table_frame = tk.Frame(email_window)
    table_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

    # Table headers
    headers = ["Thread ID", "Sender", "Recipient", "Subject", "Body", "Timestamp"]
    for col, header in enumerate(headers):
        tk.Label(table_frame, text=header, font=("Helvetica", 12), borderwidth=2, relief="solid").grid(row=0, column=col, sticky="nsew")

    # Retrieve email threads
    connection = sqlite3.connect('emails.db')
    cursor = connection.cursor()
    cursor.execute('SELECT thread_id, sender, recipient, subject, body, timestamp FROM emails ORDER BY timestamp ASC')
    emails = cursor.fetchall()
    connection.commit()
    connection.close()

    # Populate the table
    for row, email in enumerate(emails, start=1):
        for col, value in enumerate(email):
            tk.Label(table_frame, text=value, font=("Arial", 10), borderwidth=1, relief="solid").grid(row=row, column=col, sticky="nsew")

    # Adjust column weights
    for col in range(len(headers)):
        table_frame.grid_columnconfigure(col, weight=1)

    # Sorting buttons
    sort_frame = tk.Frame(email_window)
    sort_frame.pack(pady=10)

    tk.Label(sort_frame, text="Sort By:", font=("Arial", 12)).pack(side=tk.LEFT, padx=5)

    for field in headers[1:]:  # Skip Thread ID
        tk.Button(sort_frame, text=field, command=lambda f=field: sort_emails(table_frame, f)).pack(side=tk.LEFT, padx=5)

    # Summarise button
    tk.Label(email_window, text="Enter Thread ID:", font=("Arial", 12)).pack(pady=10)
    global thread_id_entry
    thread_id_entry = tk.Entry(email_window, font=("Arial", 12))
    thread_id_entry.pack(pady=5)

    tk.Button(email_window, text="Summarise Thread", font=("Arial", 12), command=summarise_thread).pack(pady=10)

def sort_emails(table_frame, field):
    # Map field names to column indexes
    field_map = {
        "Sender": 1,
        "Recipient": 2,
        "Subject": 3,
        "Body": 4,
        "Timestamp": 5,
    }
    col_idx = field_map[field]

    # Fetch emails from the database and sort by the selected field
    connection = sqlite3.connect('emails.db')
    cursor = connection.cursor()
    cursor.execute(f'SELECT * FROM emails ORDER BY {field.lower()} ASC')
    emails = cursor.fetchall()
    connection.commit()
    connection.close()

    # Clear the existing table
    for widget in table_frame.winfo_children():
        widget.destroy()

    # Rebuild the table
    headers = ["Thread ID", "Sender", "Recipient", "Subject", "Body", "Timestamp"]
    for col, header in enumerate(headers):
        tk.Label(table_frame, text=header, font=("Helvetica", 12), borderwidth=2, relief="solid").grid(row=0, column=col, sticky="nsew")

    for row, email in enumerate(emails, start=1):
        for col, value in enumerate(email):
            tk.Label(table_frame, text=value, font=("Arial", 10), borderwidth=1, relief="solid").grid(row=row, column=col, sticky="nsew")

    # Adjust column weights
    for col in range(len(headers)):
        table_frame.grid_columnconfigure(col, weight=1)

def get_email_thread(thread_id):
    connection = sqlite3.connect('emails.db')
    cursor = connection.cursor()

    cursor.execute('''
    SELECT sender, recipient, subject, body, timestamp 
    FROM emails 
    WHERE thread_id = ? 
    ORDER BY timestamp ASC
    ''', (thread_id,))
    emails = cursor.fetchall()
    connection.commit()
    connection.close()

    return emails

from transformers import pipeline
import nltk
nltk.download('punkt')

from nltk.tokenize import word_tokenize

summariser = pipeline("summarization")  

def generate_email_summary(thread_id):
       emails = get_email_thread(thread_id)
       if not emails:
           return "No emails found in this thread."

       # Prepare the email thread text for summarisation
       thread_text = "\n".join([
           f"From: {email[0]} | To: {email[1]} | Subject: {email[2]} | Sent: {email[4]}\n{email[3]}\n"
           for email in emails
       ])

       # Calculate word count of the email body
       body_word_count = len(word_tokenize(thread_text))  

       # Dynamically adjust max_length based on word count
       max_len = int(body_word_count * 0.3)  # Example: 30% of body word count
       max_len = max(max_len, 0)  # Ensure a minimum max_length of 25
       max_len = min(max_len, 100)  # Ensure a maximum max_length of 100

       try:
           summarised_text = summariser(thread_text, max_length=max_len, min_length=10, do_sample=False)[0]["summary_text"]
       except Exception as e:
           print(f"Error during summarisation: {str(e)}")
           return "Failed to generate summary."

       return summarised_text

def summarise_thread():
    thread_id = thread_id_entry.get()
    if not thread_id:
        messagebox.showerror("Error", "Please enter a valid thread ID.")
        return

    try:
        thread_id = int(thread_id)
        summary = generate_email_summary(thread_id)
        messagebox.showinfo("Thread Summary", summary)
    except Exception as e:
        print(f"Error: {str(e)}")
        messagebox.showerror("Error", f"Failed to generate summary: {str(e)}")


def open_reminder(username):
    def schedule_email(reminder_name, date, time, shared_users):
        """
        Schedule an email to be sent at the specified date and time.
        """
        # Convert date and time to a datetime object
        reminder_datetime = datetime.strptime(f"{date} {time}", "%Y-%m-%d %H:%M")

        # Calculate delay in seconds
        now = datetime.now()
        delay = (reminder_datetime - now).total_seconds()

        if delay < 0:
            print(f"Cannot schedule email for the past: {reminder_datetime}")
            return

        # Function to send the email
        def send_email():
            try:
                # Email configuration
                sender_email = "thenovelnotices@gmail.com"
                sender_password = "bifz ioqd kwyw nhtq"  # Use an app-specific password if required
                print(f"Email: {sender_email}")
                print(f"Password: {sender_password}")  # For debugging only, remove later!

                smtp_server = "smtp.gmail.com"
                smtp_port = 587

                # Email content
                subject = f"Reminder: {reminder_name}"
                body = f"This is a reminder for '{reminder_name}' scheduled on {date} at {time}."

                for user_email in shared_users.split(", "):
                    # Set up the email
                    msg = MIMEMultipart()
                    msg["From"] = sender_email
                    msg["To"] = user_email
                    msg["Subject"] = subject

                    msg.attach(MIMEText(body, "plain"))

                    # Connect to the SMTP server and send the email
                    with smtplib.SMTP(smtp_server, smtp_port) as server:
                        server.starttls()
                        server.login(sender_email, sender_password)
                        server.sendmail(sender_email, user_email, msg.as_string())

                    print(f"Email sent to {user_email} for reminder '{reminder_name}'.")

            except Exception as e:
                print(f"Failed to send email: {e}")

        # Schedule the email using a timer
        threading.Timer(delay, send_email).start()
        print(f"Email scheduled for '{reminder_name}' on {reminder_datetime} for users: {shared_users}")
    def remove_reminder(reminder_id):
        try:
            connection = sqlite3.connect("reminders.db")
            cursor = connection.cursor()
            cursor.execute("DELETE FROM reminders WHERE rowid = ?", (reminder_id,))
            connection.commit()
            connection.close()

            # Refresh the reminders list
            refresh_reminders()
            messagebox.showinfo("Success", "Reminder removed successfully!")
        except sqlite3.Error as db_error:
            messagebox.showerror("Database Error", f"Failed to remove reminder: {db_error}")
    def refresh_reminders(sort_by=None):
        """
        Fetch and display reminders, optionally sorted by a specific column.
        """
        for widget in reminders_frame.winfo_children():
            widget.destroy()  # Clear the current list

        # Connect to the database
        connection = sqlite3.connect("reminders.db")
        cursor = connection.cursor()

        # Build the query with optional sorting
        query = """
            SELECT rowid, reminder_name, date, time, shared_users, email, owner
            FROM reminders
            WHERE owner = ? OR shared_users LIKE ?
        """
        if sort_by:
            query += f" ORDER BY {sort_by}"

        # Fetch reminders where the user is either the owner or a shared user
        cursor.execute(query, (username, f"%{username}%"))
        reminders = cursor.fetchall()
        connection.close()

        # Display the reminders
        if reminders:
            # Table Headers
            headers = ["Reminder Name", "Date", "Time", "Shared Users", "Email", "Owner"]
            for col, header in enumerate(headers):
                tk.Button(
                    reminders_frame,
                    text=header,
                    font=("Arial", 12, "bold"),
                    command=lambda h=header: refresh_reminders(h.lower().replace(" ", "_"))
                ).grid(row=0, column=col, padx=5, pady=5, sticky="nsew")

            # Reminder Rows
            for row, reminder in enumerate(reminders, start=1):
                reminder_id = reminder[0]
                for col, value in enumerate(reminder[1:]):  # Exclude rowid
                    if col == 4:  # For the email column
                        value = "Yes" if value == 1 else "No"
                    tk.Label(reminders_frame, text=value, font=("Arial", 12), anchor="w").grid(row=row, column=col, padx=5, pady=5, sticky="nsew")

                # Add a Remove button for each reminder in its own column
                tk.Button(
                    reminders_frame,
                    text="Remove",
                    font=("Arial", 12),
                    command=lambda rid=reminder_id: remove_reminder(rid)).grid(row=row, column=len(headers), padx=5, pady=5, sticky="nsew")
        else:
            tk.Label(reminders_frame, text="No reminders found.", font=("Arial", 12)).pack(pady=10)
    def open_add_reminder_page():
        """
        Open the Add Reminder page.
        """
        def toggle_email_field():
            """Show or hide the email address field based on the checkbox state."""
            if email_reminder_var.get():
                email_address_label.pack(pady=5, before=save_button)
                email_address_entry.pack(pady=5, before=save_button)
            else:
                email_address_label.pack_forget()
                email_address_entry.pack_forget()

        def save_reminder():
            # Retrieve input values
            reminder_name = reminder_name_entry.get()
            date = date_entry.get_date()
            hour = hour_spinbox.get()
            minute = minute_spinbox.get()
            email_address = email_address_entry.get() if email_reminder_var.get() else ""

            # Retrieve selected users
            selected_indices = shared_users_menu.curselection()
            selected_users = [shared_users_menu.get(i) for i in selected_indices]
            shared_users = ", ".join(selected_users)

            # Check if email reminder is enabled
            email_enabled = email_reminder_var.get()

            # Combine hour and minute for reminder time
            try:
                time = f"{int(hour):02}:{int(minute):02}"
                reminder_datetime = datetime.strptime(f"{date} {time}", "%Y-%m-%d %H:%M")
            except ValueError:
                messagebox.showerror("Input Error", "Invalid date or time. Please enter valid values.")
                return

            # Validate that the reminder date and time is in the future
            now = datetime.now()
            if reminder_datetime <= now:
                messagebox.showerror("Input Error", "Reminder date and time must be in the future.")
                return

            # Check if email is mandatory and validate it
            if email_enabled and not email_address.strip():
                messagebox.showerror("Input Error", "Email address is required when email reminder is enabled.")
                return

            # Validate required fields
            if not reminder_name or not date or not time:
                messagebox.showwarning("Input Error", "All fields are required!")
                return
            else:
                try:
                    connection = sqlite3.connect("reminders.db")
                    cursor = connection.cursor()
                    # Insert the reminder, setting the owner to the current user
                    cursor.execute(
                        '''
                        INSERT INTO reminders (owner, shared_users, reminder_name, date, time, email)
                        VALUES (?, ?, ?, ?, ?, ?)
                        ''',
                        (username, shared_users, reminder_name, date, time, int(email_enabled)),
                    )
                    connection.commit()
                    connection.close()

                    # If email reminder is enabled, schedule the email
                    if email_enabled:
                        schedule_email(reminder_name, date, time, email_address)

                    # Show success message
                    messagebox.showinfo("Success", "Reminder added successfully!")
                    
                    # Refresh the reminders page and close the Add Reminder window
                    reminder_window.destroy()  # Only close the Add Reminder page
                    refresh_reminders()  # Ensure reminders page is updated

                except sqlite3.Error as db_error:
                    messagebox.showerror("Database Error", f"Failed to save reminder: {db_error}")

        # Create the reminder addition window
        reminder_window = tk.Toplevel()
        reminder_window.title("Add Reminder")
        reminder_window.geometry("400x450")

        # Reminder Name
        tk.Label(reminder_window, text="Reminder Name:").pack(pady=5)
        reminder_name_entry = tk.Entry(reminder_window)
        reminder_name_entry.pack(pady=5)

        # Reminder Date
        tk.Label(reminder_window, text="Reminder Date (YYYY-MM-DD):").pack(pady=5)
        date_entry = DateEntry(
            reminder_window, date_pattern="yyyy-mm-dd", width=12, background="darkblue", foreground="white", borderwidth=2
        )
        date_entry.pack(pady=5)

        # Reminder Time
        tk.Label(reminder_window, text="Reminder Time (HH:MM):").pack(pady=5)
        time_frame = tk.Frame(reminder_window)
        time_frame.pack(pady=5)

        hour_spinbox = tk.Spinbox(
            time_frame, from_=0, to=23, width=3, font=("Arial", 12)
        )
        hour_spinbox.pack(side=tk.LEFT, padx=2)

        tk.Label(time_frame, text=":", font=("Arial", 12)).pack(side=tk.LEFT)

        minute_spinbox = tk.Spinbox(
            time_frame, from_=0, to=59, width=3, font=("Arial", 12)
        )
        minute_spinbox.pack(side=tk.LEFT, padx=2)

        # Shared Users
        tk.Label(reminder_window, text="Share Reminder With:").pack(pady=5)

        other_users = get_other_users(username)
        shared_users_var = tk.StringVar(value=other_users)

        shared_users_menu = tk.Listbox(
            reminder_window,
            listvariable=shared_users_var,
            selectmode=tk.MULTIPLE,
            height=5,
            font=("Arial", 12),
        )
        shared_users_menu.pack(pady=5)

        # Email Reminder Checkbox
        email_reminder_var = tk.IntVar()
        email_reminder_checkbox = tk.Checkbutton(
            reminder_window, text="Send Reminder to Email", variable=email_reminder_var, command=toggle_email_field
        )
        email_reminder_checkbox.pack(pady=10)

        # Email Address Field (initially hidden)
        email_address_label = tk.Label(reminder_window, text="Email Address:")
        email_address_entry = tk.Entry(reminder_window)

        # Save Reminder Button
        save_button = tk.Button(reminder_window, text="Save Reminder", command=save_reminder)
        save_button.pack(pady=10)

        tk.Button(reminder_window, text="Cancel", command=reminder_window.destroy).pack(pady=10)

    # Create the reminders page window
    reminders_window = tk.Toplevel()
    reminders_window.title(f"{username}'s Reminders")
    reminders_window.geometry("800x600")

    tk.Label(reminders_window, text=f"{username}'s Reminders", font=("Helvetica", 16)).pack(pady=10)

    # Frame to display the reminders list
    reminders_frame = tk.Frame(reminders_window)
    reminders_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

    # Refresh and display reminders
    refresh_reminders()

    # Add Reminder Button
    tk.Button(reminders_window, text="Add Reminder", font=("Arial", 14), command=open_add_reminder_page).pack(pady=10)

    # Close Button
    tk.Button(reminders_window, text="Close", font=("Arial", 14), command=reminders_window.destroy).pack(pady=10)

# Start page
def start_page(username):
    start_window = tk.Toplevel(window)
    start_window.title("Start Page")
    start_window.geometry("400x300")

    tk.Label(start_window, text=f"Welcome, {username}!", font=("Helvetica", 16)).pack(pady=20)

    # Calendar Button
    tk.Button(start_window, text="Calendar", font=("Arial", 14), command=lambda: open_calendar(username)).pack(pady=10)
    
    # Shared Calendar Button
    tk.Button(start_window, text="Shared Calendar", font=("Arial", 14), command=open_all_users_calendar).pack(pady=10)
    
    # Email Button
    tk.Button(start_window, text="Email", font=("Arial", 14), command=lambda: open_email_page(username)).pack(pady=10)

    # Reminders Button
    tk.Button(start_window, text="Reminders", font=("Arial", 14), command=lambda: open_reminder(username)).pack(pady=10)

    # Logout Button
    tk.Button(start_window, text="Logout", font=("Arial", 12), command=start_window.destroy).pack(pady=20)

# Create the main window
window = tk.Tk()
window.title('Login System')

# Fix background to ensure visibility
window.configure(bg="white")

# Create a frame for inputs and buttons
main_frame = tk.Frame(window, padx=10, pady=10, bg="white")
main_frame.pack()

# Username label and entry
tk.Label(main_frame, text='Username:', bg="white", font=("Arial", 12)).grid(row=0, column=0, padx=10, pady=10, sticky="e")
username_entry = tk.Entry(main_frame, font=("Arial", 12))
username_entry.grid(row=0, column=1, padx=10, pady=10)

# Password label and entry
tk.Label(main_frame, text='Password:', bg="white", font=("Arial", 12)).grid(row=1, column=0, padx=10, pady=10, sticky="e")
password_entry = tk.Entry(main_frame, show='*', font=("Arial", 12))
password_entry.grid(row=1, column=1, padx=10, pady=10)

# Show Password button
show_password_button = tk.Button(main_frame, text="Show Password", command=toggle_password)
show_password_button.grid(row=1, column=2, padx=10, pady=10)

# Login and Register buttons
tk.Button(main_frame, text='Login', command=login, font=("Arial", 12)).grid(row=2, column=0, padx=10, pady=10)
tk.Button(main_frame, text='Register', command=register, font=("Arial", 12)).grid(row=2, column=1, padx=10, pady=10)

window.mainloop()
