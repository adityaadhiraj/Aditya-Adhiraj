# Aditya-Adhiraj
#class project
import tkinter as tk
from tkinter import messagebox
import math

class AdvancedCalculator:
    def __init__(self, root):
        self.root = root
        self.root.title("Advanced Scientific Calculator")
        self.root.geometry("450x600")
        self.root.configure(bg="#1e1e2e")
        self.root.resizable(False, False)

        # Variables to track history and current expression
        self.expression = ""
        self.history_text = ""

        self.create_widgets()

    def create_widgets(self):
        # Frame for displays
        display_frame = tk.Frame(self.root, bg="#1e1e2e", padx=10, pady=10)
        display_frame.pack(fill=tk.BOTH)

        # Upper label for calculations history
        self.history_label = tk.Label(
            display_frame, text="", font=("Arial", 12), 
            anchor="e", fg="#a6adc8", bg="#1e1e2e", height=1
        )
        self.history_label.pack(fill=tk.X)

        # Primary entry widget for active input/output
        self.display = tk.Entry(
            display_frame, font=("Arial", 26, "bold"), 
            justify="right", bd=0, fg="#cdd6f4", bg="#313244",
            insertbackground="#cdd6f4", highlightthickness=2, 
            highlightbackground="#45475a", highlightcolor="#89b4fa"
        )
        self.display.pack(fill=tk.X, ipady=12)
        self.display.focus()

        # Grid Layout Definition
        # Format: (Button Text, Row, Column, Span, Text Color, Background Color)
        buttons = [
            ('C', 0, 0, 1, '#f38ba8', '#45475a'), ('Del', 0, 1, 1, '#f38ba8', '#45475a'), ('(', 0, 2, 1, '#89b4fa', '#45475a'), (')', 0, 3, 1, '#89b4fa', '#45475a'), ('mod', 0, 4, 1, '#89b4fa', '#45475a'),
            ('sin', 1, 0, 1, '#cba6f7', '#313244'), ('cos', 1, 1, 1, '#cba6f7', '#313244'), ('tan', 1, 2, 1, '#cba6f7', '#313244'), ('^', 1, 3, 1, '#89b4fa', '#45475a'), ('/', 1, 4, 1, '#89b4fa', '#45475a'),
            ('log', 2, 0, 1, '#cba6f7', '#313244'), ('ln', 2, 1, 1, '#cba6f7', '#313244'), ('7', 2, 2, 1, '#cdd6f4', '#181825'), ('8', 2, 3, 1, '#cdd6f4', '#181825'), ('9', 2, 4, 1, '#cdd6f4', '#181825'),
            ('sqrt', 3, 0, 1, '#cba6f7', '#313244'), ('x!', 3, 1, 1, '#cba6f7', '#313244'), ('4', 3, 2, 1, '#cdd6f4', '#181825'), ('5', 3, 3, 1, '#cdd6f4', '#181825'), ('6', 3, 4, 1, '#cdd6f4', '#181825'),
            ('pi', 4, 0, 1, '#f9e2af', '#313244'), ('e', 4, 1, 1, '#f9e2af', '#313244'), ('1', 4, 2, 1, '#cdd6f4', '#181825'), ('2', 4, 3, 1, '#cdd6f4', '#181825'), ('3', 4, 4, 1, '#cdd6f4', '#181825'),
            ('.', 5, 0, 1, '#cdd6f4', '#181825'), ('0', 5, 1, 1, '#cdd6f4', '#181825'), ('+', 5, 2, 1, '#89b4fa', '#45475a'), ('-', 5, 3, 1, '#89b4fa', '#45475a'), ('*', 5, 4, 1, '#89b4fa', '#45475a'),
            ('=', 6, 0, 5, '#11111b', '#a6e3a1') # Span all columns for action button
        ]

        # Container for the grid arrangement
        grid_frame = tk.Frame(self.root, bg="#1e1e2e", padx=10, pady=10)
        grid_frame.pack(fill=tk.BOTH, expand=True)

        # Configure weight so elements expand dynamically
        for i in range(5):
            grid_frame.columnconfigure(i, weight=1)
        for i in range(7):
            grid_frame.rowconfigure(i, weight=1)

        # Generate and bind buttons
        for text, row, col, span, fg, bg in buttons:
            btn = tk.Button(
                grid_frame, text=text, font=("Arial", 14, "bold"),
                fg=fg, bg=bg, activebackground=fg, activeforeground=bg,
                bd=0, relief=tk.FLAT, command=lambda t=text: self.on_button_click(t)
            )
            btn.grid(row=row, column=col, columnspan=span, sticky="nsew", padx=4, pady=4)

        # Bind native Enter key to calculate result
        self.root.bind('<Return>', lambda event: self.on_button_click('='))

    def on_button_click(self, char):
        current_val = self.display.get()

        if char == 'C':
            self.display.delete(0, tk.END)
            self.history_label.config(text="")
        elif char == 'Del':
            self.display.delete(len(current_val) - 1, tk.END)
        elif char == '=':
            self.calculate(current_val)
        elif char in ['sin', 'cos', 'tan', 'log', 'ln', 'sqrt']:
            # Automatically include an opening parenthesis for prefix functions
            self.display.insert(tk.END, f"{char}(")
        elif char == 'x!':
            self.display.insert(tk.END, "!")
        elif char == 'pi':
            self.display.insert(tk.END, str(math.pi))
        elif char == 'e':
            self.display.insert(tk.END, str(math.e))
        else:
            self.display.insert(tk.END, char)

    def calculate(self, raw_expression):
        if not raw_expression:
            return

        # Sanitize mathematical formatting to syntax Python's eval engine expects
        parsed_expr = raw_expression.replace('^', '**')
        parsed_expr = parsed_expr.replace('mod', '%')
        parsed_expr = parsed_expr.replace('!', '')  # Handled distinctly below if found

        # Map display labels to their relative math library utilities
        replacements = {
            'sin(': 'math.sin(math.radians(', # Evaluates trigonometry via degree values
            'cos(': 'math.cos(math.radians(',
            'tan(': 'math.tan(math.radians(',
            'log(': 'math.log10(',
            'ln(': 'math.log(',
            'sqrt(': 'math.sqrt('
        }

        for key, value in replacements.items():
            if key in parsed_expr:
                parsed_expr = parsed_expr.replace(key, value)
                # Count instances to patch trailing parenthesis offsets safely
                open_counts = raw_expression.count(key)
                if key in ['sin(', 'cos(', 'tan(']:
                    parsed_expr += ')' * open_counts

        # Intercept and convert factorials (e.g., "5!" -> "math.factorial(5)")
        if '!' in raw_expression:
            try:
                num = int(raw_expression.replace('!', ''))
                parsed_expr = f"math.factorial({num})"
            except ValueError:
                messagebox.showerror("Error", "Factorials apply exclusively to whole integers.")
                return

        try:
            # Safely process the dynamically adjusted algebraic string
            result = eval(parsed_expr, {"__builtins__": None}, {"math": math})
            
            # Format display precision output
            if isinstance(result, float) and result.is_integer():
                result = int(result)
            elif isinstance(result, float):
                result = round(result, 8)

            # Push context upward to populate calculations history panel
            self.history_label.config(text=f"{raw_expression} =")
            self.display.delete(0, tk.END)
            self.display.insert(0, str(result))

        except ZeroDivisionError:
            messagebox.showerror("Math Error", "Calculation aborted: Division by zero is undefined.")
        except Exception:
            messagebox.showerror("Syntax Error", "Invalid syntax format used.")

if __name__ == "__main__":
    root = tk.Tk()
    app = AdvancedCalculator(root)
    root.mainloop()
