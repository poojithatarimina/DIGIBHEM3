# DIGIBHEM
#BMI CALCULATOR USING PYTHON
import tkinter as tk
from tkinter import messagebox
import sqlite3
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg

class BMIApp:
    def __init__(self, root):
        self.root = root
        self.root.title("BMI Calculator")
        
        # Create GUI elements
        self.label_weight = tk.Label(root, text="Weight (kg):")
        self.label_height = tk.Label(root, text="Height (m):")
        self.entry_weight = tk.Entry(root)
        self.entry_height = tk.Entry(root)
        self.btn_calculate = tk.Button(root, text="Calculate BMI", command=self.calculate_bmi)
        self.result_label = tk.Label(root, text="")
        
        # Grid layout
        self.label_weight.grid(row=0, column=0)
        self.entry_weight.grid(row=0, column=1)
        self.label_height.grid(row=1, column=0)
        self.entry_height.grid(row=1, column=1)
        self.btn_calculate.grid(row=2, columnspan=2)
        self.result_label.grid(row=3, columnspan=2)
        
        # Initialize database
        self.conn = sqlite3.connect("bmi_data.db")
        self.create_table()
        
    def create_table(self):
        # Create BMI data table if not exists
        cursor = self.conn.cursor()
        cursor.execute('''CREATE TABLE IF NOT EXISTS bmi_records
                          (id INTEGER PRIMARY KEY AUTOINCREMENT,
                           weight REAL,
                           height REAL,
                           bmi REAL,
                           timestamp DATETIME DEFAULT CURRENT_TIMESTAMP)''')
        self.conn.commit()
    
    def calculate_bmi(self):
        try:
            weight = float(self.entry_weight.get())
            height = float(self.entry_height.get())
            if weight <= 0 or height <= 0:
                raise ValueError("Weight and height must be positive numbers.")
            
            bmi = weight / (height ** 2)
            category = self.classify_bmi(bmi)
            
            self.result_label.config(text=f"BMI: {bmi:.2f} - Category: {category}")
            
            # Insert record into database
            self.insert_record(weight, height, bmi)
            
            # Plot BMI trend
            self.plot_bmi_trend()
            
        except ValueError as e:
            messagebox.showerror("Error", str(e))
    
    def classify_bmi(self, bmi):
        # Implement classification logic as in the beginner version
        pass
    
    def insert_record(self, weight, height, bmi):
        cursor = self.conn.cursor()
        cursor.execute("INSERT INTO bmi_records (weight, height, bmi) VALUES (?, ?, ?)", (weight, height, bmi))
        self.conn.commit()
    
    def plot_bmi_trend(self):
        cursor = self.conn.cursor()
        cursor.execute("SELECT timestamp, bmi FROM bmi_records ORDER BY timestamp")
        data = cursor.fetchall()
        
        timestamps = [row[0] for row in data]
        bmis = [row[1] for row in data]
        
        fig, ax = plt.subplots()
        ax.plot(timestamps, bmis)
        ax.set_xlabel("Timestamp")
        ax.set_ylabel("BMI")
        
        canvas = FigureCanvasTkAgg(fig, master=self.root)
        canvas.get_tk_widget().grid(row=4, columnspan=2)
        canvas.draw()

if __name__ == "__main__":
    root = tk.Tk()
    app = BMIApp(root)
    root.mainloop()
