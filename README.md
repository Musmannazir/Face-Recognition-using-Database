# Face-Recognition-using-Database
import openpyxl
import os
import tkinter as tk
from tkinter import messagebox, filedialog
import face_recognition
import cv2
from datetime import datetime
import numpy as np
import hashlib
# Paths
FACE_RECOGNITION_FOLDER = "C:/Users/nazir/OneDrive/Desktop/AI_Project"
STUDENTS_PATH = os.path.join(FACE_RECOGNITION_FOLDER, "student.xlsx")
ATTENDANCE_PATH = os.path.join(FACE_RECOGNITION_FOLDER, "Attendance.xlsx")
LOGIN_DETAILS_PATH = os.path.join(FACE_RECOGNITION_FOLDER, "login_details.xlsx")


# Create login details file if it doesn't exist
def create_login_details_file():
    if not os.path.exists(LOGIN_DETAILS_PATH):
        workbook = openpyxl.Workbook()
        sheet = workbook.active
        sheet.append(["Username", "Password"])  # Column headers
        workbook.save(LOGIN_DETAILS_PATH)
        print("Created login details file.")

# Create the attendance file if it doesn't exist
def create_attendance_file():
    if not os.path.exists(ATTENDANCE_PATH):
        workbook = openpyxl.Workbook()
        sheet = workbook.active
        sheet.append(["Name", "Reg No", "Time"])  # Updated headers to remove subject
        workbook.save(ATTENDANCE_PATH)
        print(f"Created new attendance file at {ATTENDANCE_PATH}")

def register_user(username, password):
    create_login_details_file()  # Ensure the file exists
    workbook = openpyxl.load_workbook(LOGIN_DETAILS_PATH)
    sheet = workbook.active

    # Check if the username already exists
    for row in sheet.iter_rows(min_row=2, values_only=True):
        if row[0] == username:
            messagebox.showwarning("Registration Error", "Username already exists.")
            return

    # Save the new user credentials
    sheet.append([username, password])
    workbook.save(LOGIN_DETAILS_PATH)
    messagebox.showinfo("Registration Success", "User registered successfully!")


# Function to validate login
def login_user(username, password):
    create_login_details_file()  # Ensure the file exists
    workbook = openpyxl.load_workbook(LOGIN_DETAILS_PATH)
    sheet = workbook.active

    # Check if credentials match any stored user
    for row in sheet.iter_rows(min_row=2, values_only=True):
        if row[0] == username and row[1] == password:
            messagebox.showinfo("Login Success", "Logged in successfully!")
            return True
    
    messagebox.showerror("Login Error", "Invalid username or password.")
    return False

# Register Frame Content
def register_frame_content():
    def register():
        username = username_entry.get()
        password = password_entry.get()
        if username and password:
            register_user(username, password)
            show_frame(frames["login"])

    register_frame = tk.Frame(root, bg="#2E4053")
    register_frame.place(relwidth=1, relheight=1)
    frames["register"] = register_frame

    tk.Label(
        register_frame,
        text="Register",
        font=("Arial", 16, "bold"),
        fg="white",
        bg="#5D6D7E",
    ).pack(pady=20)

    tk.Label(register_frame, text="Username", font=("Arial", 12), fg="white", bg="#2E4053").pack(pady=5)
    username_entry = tk.Entry(register_frame, font=("Arial", 12))
    username_entry.pack(pady=5)

    tk.Label(register_frame, text="Password", font=("Arial", 12), fg="white", bg="#2E4053").pack(pady=5)
    password_entry = tk.Entry(register_frame, show="*", font=("Arial", 12))
    password_entry.pack(pady=5)

    tk.Button(
        register_frame,
        text="Register",
        command=register,
        font=("Arial", 12, "bold"),
        bg="#1ABC9C",
        fg="white",
    ).pack(pady=20)

    tk.Button(
        register_frame,
        text="Back to Login",
        command=lambda: show_frame(frames["login"]),
        font=("Arial", 12, "bold"),
        bg="#E74C3C",
        fg="white",
    ).pack(pady=5)

# Modify the login screen button
def login_frame_content():
    def login():
        username = username_entry.get()
        password = password_entry.get()
        if login_user(username, password):
            show_frame(frames["home"])

    login_frame = tk.Frame(root, bg="#2E4053")
    login_frame.place(relwidth=1, relheight=1)
    frames["login"] = login_frame

    tk.Label(
        login_frame,
        text="Login",
        font=("Arial", 16, "bold"),
        fg="white",
        bg="#5D6D7E",
    ).pack(pady=20)

    tk.Label(login_frame, text="Username", font=("Arial", 12), fg="white", bg="#2E4053").pack(pady=5)
    username_entry = tk.Entry(login_frame, font=("Arial", 12))
    username_entry.pack(pady=5)

    tk.Label(login_frame, text="Password", font=("Arial", 12), fg="white", bg="#2E4053").pack(pady=5)
    password_entry = tk.Entry(login_frame, show="*", font=("Arial", 12))
    password_entry.pack(pady=5)

    tk.Button(
        login_frame,
        text="Login",
        command=login,
        font=("Arial", 12, "bold"),
        bg="#1ABC9C",
        fg="white",
    ).pack(pady=20)

    tk.Button(
        login_frame,
        text="Create Account",
        command=lambda: show_frame(frames["register"]),  # This now works
        font=("Arial", 12, "bold"),
        bg="#E74C3C",
        fg="white",
    ).pack(pady=5)


def take_attendance_screen():
    # Ensure the attendance file exists
    create_attendance_file()  # This line ensures the file is created if not already present
    workbook = openpyxl.load_workbook(ATTENDANCE_PATH)
    sheet = workbook.active


# Ensure the folder exists
if not os.path.exists(FACE_RECOGNITION_FOLDER):
    os.makedirs(FACE_RECOGNITION_FOLDER)


# Load or create the students database
def load_students():
    if os.path.exists(STUDENTS_PATH):
        workbook = openpyxl.load_workbook(STUDENTS_PATH)
        sheet = workbook.active
        students = []
        for row in sheet.iter_rows(min_row=2, values_only=True):  # Skip header row
            students.append(row)
        return students
    else:
        # Create the students file if it doesn't exist
        workbook = openpyxl.Workbook()
        sheet = workbook.active
        sheet.title = "Students"
        sheet.append(
            ["Reg No", "Name", "Image Path"]
        )  # Updated headers to remove subject
        workbook.save(STUDENTS_PATH)
        return []


# Save a student record to the Excel sheet
def save_student(reg_no, name, image_path):
    workbook = openpyxl.load_workbook(STUDENTS_PATH)
    sheet = workbook.active
    sheet.append([reg_no, name, image_path])  # Save without subject
    workbook.save(STUDENTS_PATH)
    print(f"Student {name} saved successfully.")


# Delete a student record from the Excel sheet
def delete_student(reg_no):
    workbook = openpyxl.load_workbook(STUDENTS_PATH)
    sheet = workbook.active
    for row in sheet.iter_rows(min_row=2, values_only=True):  # Skip header row
        if row[0] == reg_no:
            for cell in sheet.iter_rows(min_row=2):
                if cell[0].value == reg_no:
                    sheet.delete_rows(cell[0].row)
                    workbook.save(STUDENTS_PATH)
                    messagebox.showinfo(
                        "Success", f"Student with Reg No {reg_no} deleted successfully."
                    )
                    return
    messagebox.showwarning("Not Found", f"Student with Reg No {reg_no} not found.")


# GUI-related functions
def add_student(reg_entry, name_entry):
    reg_no = reg_entry.get()
    name = name_entry.get()
    add_frame = tk.Frame(root, bg="#2E4053")
    add_frame.place(relwidth=1, relheight=1)
    frames["add_student"] = add_frame
    image_path = filedialog.askopenfilename(title="Select Student Image")
    if reg_no and name and image_path:
        save_student(reg_no, name, image_path)  # Save without subject
        messagebox.showinfo("Success", f"Student {name} registered successfully.")
    else:
        messagebox.showwarning("Input Error", "Please fill in all details.")
    

# Function to update and show student details
def update_student_details():
    # Clear the current content in the details_frame
    for widget in details_frame.winfo_children():
        widget.destroy()

    tk.Label(
        details_frame,
        text="Student Details",
        font=("Arial", 16, "bold"),
        fg="white",
        bg="#5D6D7E",
    ).pack(pady=20)

    # Load and display the updated student details
    students = load_students()
    if not students:
        tk.Label(
            details_frame,
            text="No students found.",
            font=("Arial", 12),
            fg="white",
            bg="#5D6D7E",
        ).pack(pady=5)
    else:
        for student in students:
            name = student[1]
            reg_no = student[0]
            tk.Label(
                details_frame,
                text=f"Reg No: {reg_no}, Name: {name}",
                font=("Arial", 12),
                fg="white",
                bg="#5D6D7E",
            ).pack(pady=5)

    tk.Button(
        details_frame,
        text="Back to Home",
        command=lambda: show_frame(frames["home"]),
        font=("Arial", 12, "bold"),
        bg="#1ABC9C",
        fg="white",
    ).pack(pady=20)


def delete_student_screen(reg_entry):
    reg_no = reg_entry.get()
    if reg_no:
        delete_student(reg_no)
    else:
        messagebox.showwarning(
            "Input Error", "Please enter a valid Registration Number."
        )


def take_attendance_screen():
    students = load_students()
    # Load known faces and names
    known_face_encodings = []
    known_face_names = []
    reg_nums = []

    for student in students:
        name = student[1]
        reg_no = student[0]
        image_path = student[2]  # Updated to skip the subject column

        # Load student image and encode it
        try:
            image = face_recognition.load_image_file(image_path)
            encoding = face_recognition.face_encodings(image)[0]
            known_face_encodings.append(encoding)
            known_face_names.append(name)
            reg_nums.append(reg_no)
        except Exception as e:
            print(f"Error loading image for {name}: {e}")

    # Open the video capture and start attendance session
    cap = cv2.VideoCapture(0)
    if not cap.isOpened():
        print("Error: Camera not accessible.")
        return

    print("Taking attendance...")
    while True:
        ret, frame = cap.read()
        if not ret:
            print("Error: Failed to capture frame.")
            break

        small_frame = cv2.resize(frame, (0, 0), fx=0.25, fy=0.25)
        rgb_small_frame = cv2.cvtColor(small_frame, cv2.COLOR_BGR2RGB)

        # Recognize faces in the frame
        face_locations = face_recognition.face_locations(rgb_small_frame)
        face_encodings = face_recognition.face_encodings(
            rgb_small_frame, face_locations
        )

        for face_encoding in face_encodings:
            matches = face_recognition.compare_faces(
                known_face_encodings, face_encoding
            )
            face_distance = face_recognition.face_distance(
                known_face_encodings, face_encoding
            )
            best_match_index = np.argmin(face_distance)

            if matches[best_match_index]:
                name = known_face_names[best_match_index]
                reg_no = reg_nums[best_match_index]

                # Display the name and mark attendance
                font = cv2.FONT_HERSHEY_SIMPLEX
                cv2.putText(
                    frame, f"{name} PRESENT", (10, 100), font, 1.5, (255, 0, 0), 3, 2
                )

                # Log attendance in the Excel file
                current_time = datetime.now().strftime("%H-%M-%S")
                workbook = openpyxl.load_workbook(ATTENDANCE_PATH)
                sheet = workbook.active
                sheet.append([name, reg_no, current_time])  # Updated to remove subject
                workbook.save(ATTENDANCE_PATH)

                print(f"Attendance for {name} recorded at {current_time}")

        cv2.imshow("Attendance System", frame)

        if cv2.waitKey(1) & 0xFF == ord("q"):
            break

    cap.release()
    cv2.destroyAllWindows()


# Setting up the GUI
root = tk.Tk()
root.title("Face Recognition Attendance System")
root.geometry("600x500")
root.configure(bg="#2E4053")

# Function to switch frames
frames = {}

login_frame_content()

def show_frame(frame):
    if frame == frames["details"]:
        update_student_details()
    frame.tkraise()

# Home Frame
home_frame = tk.Frame(root, bg="#2E4053")
home_frame.place(relwidth=1, relheight=1)
frames["home"] = home_frame

tk.Label(
    home_frame,
    text="Welcome to Face Recognition Attendance System",
    font=("Arial", 16, "bold"),
    fg="white",
    bg="#5D6D7E",
).pack(pady=20)
tk.Button(
    home_frame,
    text="Add Student",
    command=lambda: show_frame(frames["add_student"]),
    font=("Arial", 12, "bold"),
    bg="#1ABC9C",
    fg="white",
).pack(pady=5)
tk.Button(
    home_frame,
    text="Remove Student",
    command=lambda: show_frame(frames["delete"]),
    font=("Arial", 12, "bold"),
    bg="#1ABC9C",
    fg="white",
).pack(pady=5)
tk.Button(
    home_frame,
    text="Take Attendance",
    command=lambda: take_attendance_screen(),
    font=("Arial", 12, "bold"),
    bg="#1ABC9C",
    fg="white",
).pack(pady=5)
tk.Button(
    home_frame,
    text="Show Student Details",
    command=lambda: show_frame(frames["details"]),
    font=("Arial", 12, "bold"),
    bg="#1ABC9C",
    fg="white",
).pack(pady=5)
tk.Button(
    home_frame,
    text="Exit",
    command=root.quit,
    font=("Arial", 12, "bold"),
    bg="#E74C3C",
    fg="white",
).pack(pady=20)

# Register Frame
register_frame = tk.Frame(root, bg="#34495E")
register_frame.place(relwidth=1, relheight=1)
frames["register"] = register_frame

tk.Label(
    register_frame, text="Reg No", font=("Arial", 12), fg="white", bg="#34495E"
).pack()
reg_entry = tk.Entry(register_frame, font=("Arial", 12))
reg_entry.pack(pady=5)
tk.Label(
    register_frame, text="Name", font=("Arial", 12), fg="white", bg="#34495E"
).pack()
name_entry = tk.Entry(register_frame, font=("Arial", 12))
name_entry.pack(pady=5)

# Global variable to store the selected image path
image_path = ""

# Function to select an image and store the path
def upload_image():
    global image_path
    image_path = filedialog.askopenfilename(
        title="Select Student Image", filetypes=[("Image Files", "*.jpg;*.jpeg;*.png")]
    )
    if image_path:
        messagebox.showinfo("Image Selected", f"Selected Image: {image_path}")
    else:
        messagebox.showwarning("No Image", "No image selected.")

# Add "Upload Image" label and "Browse" button in the register frame
tk.Label(
    register_frame, text="Upload Image", font=("Arial", 12), fg="white", bg="#34495E"
).pack()
tk.Button(
    register_frame,
    text="Browse",
    command=upload_image,
    font=("Arial", 12),
    bg="#1ABC9C",
    fg="white",
).pack(pady=5)

# Add Student Frame
add_student_frame = tk.Frame(root, bg="#34495E")
add_student_frame.place(relwidth=1, relheight=1)
frames["add_student"] = add_student_frame

tk.Label(
    add_student_frame, text="Reg No", font=("Arial", 12), fg="white", bg="#34495E"
).pack()
reg_entry = tk.Entry(add_student_frame, font=("Arial", 12))
reg_entry.pack(pady=5)
tk.Label(
    add_student_frame, text="Name", font=("Arial", 12), fg="white", bg="#34495E"
).pack()
name_entry = tk.Entry(add_student_frame, font=("Arial", 12))
name_entry.pack(pady=5)

# Global variable to store the selected image path
image_path = ""

# Function to select an image and store the path
def upload_image():
    global image_path
    image_path = filedialog.askopenfilename(
        title="Select Student Image", filetypes=[("Image Files", "*.jpg;*.jpeg;*.png")]
    )
    if image_path:
        messagebox.showinfo("Image Selected", f"Selected Image: {image_path}")
    else:
        messagebox.showwarning("No Image", "No image selected.")

# Add "Upload Image" label and "Browse" button in the add student frame
tk.Label(
    add_student_frame, text="Upload Image", font=("Arial", 12), fg="white", bg="#34495E"
).pack()
tk.Button(
    add_student_frame,
    text="Browse",
    command=upload_image,
    font=("Arial", 12),
    bg="#1ABC9C",
    fg="white",
).pack(pady=5)

# Function to add student using the selected image path
def add_student():
    global image_path  # Access the selected image path
    reg_no = reg_entry.get()
    name = name_entry.get()
    if reg_no and name and image_path:
        save_student(reg_no, name, image_path)  # Use the uploaded image path
        messagebox.showinfo("Success", f"Student {name} registered successfully.")
        image_path = ""  # Reset image path after saving
        show_frame(frames["home"])  # Return to home screen after adding student
    else:
        messagebox.showwarning("Input Error", "Please fill in all details.")

# Add "Add Student" button
tk.Button(
    add_student_frame,
    text="Add Student",
    command=add_student,
    font=("Arial", 12, "bold"),
    bg="#1ABC9C",
    fg="white",
).pack(pady=20)

tk.Button(
    add_student_frame,
    text="Back to Home",
    command=lambda: show_frame(frames["home"]),
    font=("Arial", 12, "bold"),
    bg="#E74C3C",
    fg="white",
).pack()

# Add "Add Student" button
tk.Button(
    
    text="Add Student",
    command=lambda: add_student(reg_entry, name_entry),
    font=("Arial", 12, "bold"),
    bg="#1ABC9C",
    fg="white",
).pack(pady=20)

tk.Button(
    text="Back to Home",
    command=lambda: show_frame(frames["home"]),
    font=("Arial", 12, "bold"),
    bg="#E74C3C",
    fg="white",
).pack()

# Delete Frame
delete_frame = tk.Frame(root, bg="#2C3E50")
delete_frame.place(relwidth=1, relheight=1)
frames["delete"] = delete_frame

tk.Label(
    delete_frame,
    text="Delete Student",
    font=("Arial", 16, "bold"),
    fg="white",
    bg="#2C3E50",
).pack(pady=20)
tk.Label(
    delete_frame, text="Enter Reg No", font=("Arial", 12), fg="white", bg="#2C3E50"
).pack()
delete_reg_entry = tk.Entry(delete_frame, font=("Arial", 12))
delete_reg_entry.pack(pady=5)
tk.Button(
    delete_frame,
    text="Delete Student",
    command=lambda: delete_student_screen(delete_reg_entry),
    font=("Arial", 12, "bold"),
    bg="#E74C3C",
    fg="white",
).pack(pady=20)
tk.Button(
    delete_frame,
    text="Back to Home",
    command=lambda: show_frame(frames["home"]),
    font=("Arial", 12, "bold"),
    bg="#1ABC9C",
    fg="white",
).pack()

# Show Student Details Frame
details_frame = tk.Frame(root, bg="#5D6D7E")
details_frame.place(relwidth=1, relheight=1)
frames["details"] = details_frame

tk.Label(
    details_frame,
    text="Student Details",
    font=("Arial", 16, "bold"),
    fg="white",
    bg="#5D6D7E",
).pack(pady=20)

# Load and display the student details
students = load_students()
for student in students:
    name = student[1]
    reg_no = student[0]
    tk.Label(
        details_frame,
        text=f"Reg No: {reg_no}, Name: {name}",
        font=("Arial", 12),
        fg="white",
        bg="#5D6D7E",
    ).pack(pady=5)

tk.Button(
    details_frame,
    text="Back to Home",
    command=lambda: show_frame(frames["home"]),
    font=("Arial", 12, "bold"),
    bg="#1ABC9C",
    fg="white",
).pack(pady=20)

register_frame_content()
login_frame_content()


# Initialize to home frame
show_frame(frames["login"])

root.mainloop()
