import tkinter as tk
from tkinter import ttk, messagebox, simpledialog
import pandas as pd
import numpy as np

# -----------------------------
# STEP 1: Initialize Flight Data
# -----------------------------
def create_flight_data():
    """Create initial flight data stored in a pandas DataFrame."""
    data = {
        "Flight No": ["AI101", "BA202", "DL303", "UA404", "AF505"],
        "Origin": ["New York", "London", "Delhi", "Tokyo", "Paris"],
        "Destination": ["London", "New York", "Tokyo", "Delhi", "New York"],
        "Departure": ["08:00", "09:30", "13:00", "17:45", "21:15"],
        "Seats": [5, 5, 5, 5, 5],
        "Price ($)": [800, 750, 650, 900, 1000],
    }
    return pd.DataFrame(data)


# -----------------------------
# STEP 2: Main Application Class
# -----------------------------
class FlightBookingApp:
    def __init__(self, root):
        self.root = root
        self.root.title("✈️ Airline Ticket Booking System")
        self.root.geometry("800x500")
        self.root.config(bg="#f5f5f5")

        # Local in-memory DataFrames (no external storage)
        self.flights = create_flight_data()
        self.bookings = pd.DataFrame(columns=[
            "Ticket ID", "Name", "Age", "Flight No", "Origin", "Destination", "Price ($)"
        ])

        self.create_main_menu()

    # -------------------------
    # Main Menu
    # -------------------------
    def create_main_menu(self):
        title = tk.Label(
            self.root,
            text="✈️ Airline Ticket Booking System ✈️",
            font=("Arial", 20, "bold"),
            bg="#f5f5f5"
        )
        title.pack(pady=20)

        buttons = [
            ("View Flights", self.view_flights),
            ("Book Flight", self.book_flight),
            ("View Bookings", self.view_bookings),
            ("Cancel Booking", self.cancel_booking),
            ("Reset All Data", self.reset_data),
            ("Exit", self.root.destroy)
        ]

        for text, cmd in buttons:
            tk.Button(self.root, text=text, font=("Arial", 14), width=20, command=cmd)\
                .pack(pady=10)

    # -------------------------
    # View Flights
    # -------------------------
    def view_flights(self):
        top = tk.Toplevel(self.root)
        top.title("Available Flights")
        top.geometry("750x300")

        tree = ttk.Treeview(top, columns=list(self.flights.columns), show="headings")
        for col in self.flights.columns:
            tree.heading(col, text=col)
            tree.column(col, width=100)
        for _, row in self.flights.iterrows():
            tree.insert("", tk.END, values=list(row))
        tree.pack(fill="both", expand=True)

    # -------------------------
    # Book a Flight
    # -------------------------
    def book_flight(self):
        flight_no = simpledialog.askstring("Book Flight", "Enter Flight Number:")
        if not flight_no:
            return

        flight_no = flight_no.strip().upper()
        if flight_no not in self.flights["Flight No"].values:
            messagebox.showerror("Error", "Invalid Flight Number!")
            return

        flight = self.flights.loc[self.flights["Flight No"] == flight_no].iloc[0]

        if flight["Seats"] <= 0:
            messagebox.showwarning("Unavailable", "No seats available on this flight.")
            return

        name = simpledialog.askstring("Passenger Info", "Enter passenger name:")
        if not name:
            return
        age = simpledialog.askinteger("Passenger Info", "Enter passenger age:")
        if not age:
            return

        ticket_id = "T" + str(np.random.randint(10000, 99999))

        booking = pd.DataFrame([{
            "Ticket ID": ticket_id,
            "Name": name,
            "Age": age,
            "Flight No": flight["Flight No"],
            "Origin": flight["Origin"],
            "Destination": flight["Destination"],
            "Price ($)": flight["Price ($)"]
        }])

        # Add booking to in-memory DataFrame
        self.bookings = pd.concat([self.bookings, booking], ignore_index=True)

        # Update available seats
        self.flights.loc[self.flights["Flight No"] == flight_no, "Seats"] -= 1

        messagebox.showinfo(
            "Success",
            f"✅ Ticket booked!\n\nTicket ID: {ticket_id}\nPassenger: {name}\n"
            f"From {flight['Origin']} → {flight['Destination']}\nPrice: ${flight['Price ($)']}"
        )

    # -------------------------
    # View Bookings
    # -------------------------
    def view_bookings(self):
        if self.bookings.empty:
            messagebox.showinfo("No Bookings", "You have no bookings yet.")
            return

        top = tk.Toplevel(self.root)
        top.title("Your Bookings")
        top.geometry("800x300")

        tree = ttk.Treeview(top, columns=list(self.bookings.columns), show="headings")
        for col in self.bookings.columns:
            tree.heading(col, text=col)
            tree.column(col, width=100)
        for _, row in self.bookings.iterrows():
            tree.insert("", tk.END, values=list(row))
        tree.pack(fill="both", expand=True)

    # -------------------------
    # Cancel Booking
    # -------------------------
    def cancel_booking(self):
        if self.bookings.empty:
            messagebox.showinfo("No Bookings", "No bookings to cancel.")
            return

        ticket_id = simpledialog.askstring("Cancel Booking", "Enter Ticket ID to cancel:")
        if not ticket_id:
            return
        ticket_id = ticket_id.strip().upper()

        if ticket_id not in self.bookings["Ticket ID"].values:
            messagebox.showerror("Error", "Ticket not found.")
            return

        booking = self.bookings.loc[self.bookings["Ticket ID"] == ticket_id].iloc[0]
        flight_no = booking["Flight No"]

        # Restore seat in local flight DataFrame
        self.flights.loc[self.flights["Flight No"] == flight_no, "Seats"] += 1

        # Remove booking from memory
        self.bookings = self.bookings[self.bookings["Ticket ID"] != ticket_id]

        messagebox.showinfo("Cancelled", f"❌ Booking for Ticket {ticket_id} has been cancelled.")

    # -------------------------
    # Reset All Data
    # -------------------------
    def reset_data(self):
        if messagebox.askyesno("Confirm Reset", "Are you sure you want to reset all data?"):
            self.flights = create_flight_data()
            self.bookings = pd.DataFrame(columns=self.bookings.columns)
            messagebox.showinfo("Reset", "✅ All flight and booking data have been reset.")


# -----------------------------
# Run Application
# -----------------------------
if __name__ == "__main__":
    root = tk.Tk()
    app = FlightBookingApp(root)
    root.mainloop()
