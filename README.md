# Project-Assigner
Python based project
import tkinter as tk
from tkinter import ttk, messagebox
import json


class ColorfulCompanyDataApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Colorful Software Company Data Manager")
        self.root.geometry("900x600")
        self.root.configure(bg="#2E2E2E")  # Dark background

        # Persistent storage for projects
        self.project_file = "projects.json"
        self.projects = []  # Each project is a dictionary
        self.load_projects()

        # Define styles and colors
        self.colors = {
            "background": "#2E2E2E",
            "frame_background": "#424242",
            "text": "#FFFFFF",
            "button": "#4CAF50",
            "button_hover": "#45A049",
            "listbox_background": "#D3D3D3",
            "entry_background": "#FFFFFF",
            "entry_text": "#000000",
        }
        self.create_styles()

        # Create Layout Sections
        self.create_input_section()
        self.create_project_list_section()
        self.create_project_details_section()

    def create_styles(self):
        """Define styles for the application."""
        style = ttk.Style()
        style.configure(
            "TLabel",
            font=("Helvetica", 12),
            background=self.colors["frame_background"],
            foreground=self.colors["text"]
        )
        style.configure(
            "TButton",
            font=("Helvetica", 10, "bold"),
            background=self.colors["button"],
            foreground="#FFFFFF",
        )
        style.map(
            "TButton",
            background=[("active", self.colors["button_hover"])],
            relief=[("pressed", "sunken"), ("!pressed", "raised")],
        )
        style.configure(
            "TLabelframe",
            font=("Helvetica", 12, "bold"),
            background=self.colors["frame_background"],
            foreground=self.colors["text"],
        )
        style.configure(
            "TEntry",
            font=("Helvetica", 11),
            foreground=self.colors["entry_text"],
            fieldbackground=self.colors["entry_background"],
        )

        # Color for Listbox
        self.root.option_add("*Listbox*Background", self.colors["listbox_background"])
        self.root.option_add("*Listbox*Foreground", "black")
        self.root.option_add("*Listbox*Font", "Helvetica 10")

    def create_input_section(self):
        """Create the section for adding new projects."""
        input_frame = ttk.Labelframe(
            self.root, text="Add New Project", padding=(10, 10)
        )
        input_frame.grid(row=0, column=0, sticky="nw", padx=10, pady=10)

        # Project Name
        ttk.Label(input_frame, text="Project Name:").grid(
            row=0, column=0, sticky="w", pady=5
        )
        self.project_name_entry = ttk.Entry(input_frame, width=40)
        self.project_name_entry.grid(row=0, column=1, padx=10)

        # Team Members
        ttk.Label(input_frame, text="Team Members (comma-separated):").grid(
            row=1, column=0, sticky="w", pady=5
        )
        self.team_members_entry = ttk.Entry(input_frame, width=40)
        self.team_members_entry.grid(row=1, column=1, padx=10)

        # Deadline
        ttk.Label(input_frame, text="Deadline (YYYY-MM-DD):").grid(
            row=2, column=0, sticky="w", pady=5
        )
        self.deadline_entry = ttk.Entry(input_frame, width=40)
        self.deadline_entry.grid(row=2, column=1, padx=10)

        # Status
        ttk.Label(input_frame, text="Status (Not Started/In Progress/Completed):").grid(
            row=3, column=0, sticky="w", pady=5
        )
        self.status_entry = ttk.Entry(input_frame, width=40)
        self.status_entry.grid(row=3, column=1, padx=10)

        # Add Project Button
        ttk.Button(input_frame, text="Add Project", command=self.add_project).grid(
            row=4, column=1, sticky="e", pady=10, padx=10
        )

    def create_project_list_section(self):
        """Create the section displaying and searching projects."""
        list_frame = ttk.Labelframe(self.root, text="Projects", padding=(10, 10))
        list_frame.grid(row=0, column=1, sticky="ne", padx=10, pady=10)

        # Search Bar
        ttk.Label(list_frame, text="Search Projects:").grid(
            row=0, column=0, sticky="w", pady=5
        )
        self.search_entry = ttk.Entry(list_frame, width=35)
        self.search_entry.grid(row=0, column=1, padx=10)
        ttk.Button(list_frame, text="Search", command=self.search_projects).grid(
            row=0, column=2, padx=10
        )

        # Project Listbox
        self.project_listbox = tk.Listbox(
            list_frame, height=15, width=60, background=self.colors["listbox_background"]
        )
        self.project_listbox.grid(row=1, column=0, columnspan=3, pady=(10, 0))
        self.project_listbox.bind("<<ListboxSelect>>", self.display_project_details)

        # Scrollbar
        scrollbar = ttk.Scrollbar(
            list_frame, orient=tk.VERTICAL, command=self.project_listbox.yview
        )
        self.project_listbox.config(yscrollcommand=scrollbar.set)
        scrollbar.grid(row=1, column=4, sticky="ns")

        self.update_project_list()

    def create_project_details_section(self):
        """Create the section for displaying project details."""
        details_frame = ttk.Labelframe(self.root, text="Project Details", padding=(10, 10))
        details_frame.grid(row=1, column=0, columnspan=2, sticky="new", padx=10, pady=10)

        # Details Box
        self.details_text = tk.Text(details_frame, height=10, bg="#e6e6e6", fg="black")
        self.details_text.pack(expand=True, fill="both")
        self.details_text.config(state=tk.DISABLED)

        # Delete Project Button
        ttk.Button(details_frame, text="Delete Project", command=self.delete_project).pack(
            pady=(10, 0), padx=10
        )

    def add_project(self):
        """Add a new project from user input."""
        project_name = self.project_name_entry.get().strip()
        team_members = self.team_members_entry.get().strip()
        deadline = self.deadline_entry.get().strip()
        status = self.status_entry.get().strip()

        if project_name and team_members and deadline and status:
            project = {
                "name": project_name,
                "team_members": [member.strip() for member in team_members.split(",")],
                "deadline": deadline,
                "status": status,
            }
            self.projects.append(project)
            self.save_projects()
            self.update_project_list()

            # Clear input values
            self.project_name_entry.delete(0, tk.END)
            self.team_members_entry.delete(0, tk.END)
            self.deadline_entry.delete(0, tk.END)
            self.status_entry.delete(0, tk.END)
        else:
            messagebox.showerror("Input Error", "All fields are required to add a project.")

    def update_project_list(self):
        """Refresh list of projects."""
        self.project_listbox.delete(0, tk.END)
        for project in self.projects:
            display_text = f"{project['name']} | Deadline: {project['deadline']} | Status: {project['status']}"
            self.project_listbox.insert(tk.END, display_text)

    def display_project_details(self, event):
        """Display details for the selected project."""
        selected_index = self.project_listbox.curselection()
        if selected_index:
            index = selected_index[0]
            project = self.projects[index]
            details = (
                f"Project Name: {project['name']}\n"
                f"Team Members: {', '.join(project['team_members'])}\n"
                f"Deadline: {project['deadline']}\n"
                f"Status: {project['status']}\n"
            )
            self.details_text.config(state=tk.NORMAL)
            self.details_text.delete(1.0, tk.END)
            self.details_text.insert(tk.END, details)
            self.details_text.config(state=tk.DISABLED)

    def search_projects(self):
        """Search for projects based on query."""
        query = self.search_entry.get().strip().lower()
        filtered_projects = [
            project
            for project in self.projects
            if query in project["name"].lower()
               or any(query in member.lower() for member in project["team_members"])
        ]
        self.project_listbox.delete(0, tk.END)
        for project in filtered_projects:
            display_text = f"{project['name']} | Deadline: {project['deadline']} | Status: {project['status']}"
            self.project_listbox.insert(tk.END, display_text)

    def delete_project(self):
        """Delete the selected project."""
        selected_index = self.project_listbox.curselection()
        if selected_index:
            index = selected_index[0]
            del self.projects[index]
            self.save_projects()
            self.update_project_list()
        else:
            messagebox.showerror("Delete Error", "No project selected to delete.")

    def load_projects(self):
        """Load stored projects from file."""
        try:
            with open(self.project_file, "r") as file:
                self.projects = json.load(file)
        except (FileNotFoundError, json.JSONDecodeError):
            self.projects = []

    def save_projects(self):
        """Save current projects to file."""
        with open(self.project_file, "w") as file:
            json.dump(self.projects, file, indent=4)


if __name__ == "__main__":
    root = tk.Tk()
    app = ColorfulCompanyDataApp(root)
    root.mainloop()
