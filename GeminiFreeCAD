import FreeCAD as App
import FreeCADGui as Gui
from PySide import QtGui, QtCore
import json
import traceback
import urllib.request
import urllib.parse
import os

def _send_request_to_gemini(api_endpoint, request_data, output_log=None):
    """Sends a request to the Gemini API."""
    request_body = json.dumps(request_data).encode("utf-8")
    req = urllib.request.Request(
        api_endpoint,
        data=request_body,
        headers={"Content-Type": "application/json"},
    )
    try:
        with urllib.request.urlopen(req) as response:
            response_body = response.read().decode("utf-8")
            return json.loads(response_body)
    except urllib.error.HTTPError as e:
        if output_log: output_log.append(f"HTTP Error: {e.code} {e.reason}\n")
        try:
            if output_log: output_log.append(json.dumps(json.loads(e.read().decode("utf-8")), indent=2) + "\n")
        except:
            if output_log: output_log.append("Could not read error response body.\n")
        traceback.print_exc()
        return None
    except urllib.error.URLError as e:
        if output_log: output_log.append(f"URL Error: {e.reason}\n")
        traceback.print_exc()
        return None
    except Exception as e:
        if output_log: output_log.append(f"An unexpected error occurred: {e}\n")
        traceback.print_exc()
        return None

class LLMToCADDialog(QtGui.QDialog):
    def __init__(self):
        super(LLMToCADDialog, self).__init__()
        self.setWindowTitle("Gemini to CAD Generator")
        self.resize(700, 800)
        self.setup_ui()
        self.code = None  # Store generated/improved code

    def setup_ui(self):
        layout = QtGui.QVBoxLayout(self)

        # API Key
        api_layout = QtGui.QHBoxLayout()
        api_layout.addWidget(QtGui.QLabel("Google Cloud API Key:"))
        self.api_key_input = QtGui.QLineEdit()
        self.api_key_input.setEchoMode(QtGui.QLineEdit.Password)
        api_layout.addWidget(self.api_key_input)
        layout.addLayout(api_layout)

        # Model selection
        model_layout = QtGui.QHBoxLayout()
        model_layout.addWidget(QtGui.QLabel("Generator Gemini Model:"))
        self.model_selector = QtGui.QComboBox()
        self.model_selector.addItems(["gemini-1.5-pro-002", "gemini-1.0-pro", "gemini-1.0-pro-001"])
        model_layout.addWidget(self.model_selector)
        layout.addLayout(model_layout)

        # Description
        layout.addWidget(QtGui.QLabel("CAD Description:"))
        self.description_input = QtGui.QTextEdit()
        layout.addWidget(self.description_input)

        # User Suggestions
        layout.addWidget(QtGui.QLabel("Your Suggestions:"))
        self.suggestions_input = QtGui.QTextEdit()
        layout.addWidget(self.suggestions_input)

        # Buttons - Now separate Generate and Improve
        button_layout = QtGui.QHBoxLayout()
        self.generate_button = QtGui.QPushButton("Generate")
        self.generate_button.clicked.connect(self.generate_code)  # Connect to generate_code
        button_layout.addWidget(self.generate_button)

        self.improve_button = QtGui.QPushButton("Improve")
        self.improve_button.clicked.connect(self.improve_code)  # Connect to improve_code
        button_layout.addWidget(self.improve_button)

        self.close_button = QtGui.QPushButton("Close")
        self.close_button.clicked.connect(self.close)
        button_layout.addWidget(self.close_button)
        layout.addLayout(button_layout)

        # Output log
        layout.addWidget(QtGui.QLabel("Output:"))
        self.output_log = QtGui.QTextEdit()
        self.output_log.setReadOnly(True)
        layout.addWidget(self.output_log)

    def generate_code(self):
        """Generates the initial code based on the description."""
        api_key = self.api_key_input.text().strip()
        description = self.description_input.toPlainText().strip()
        model_name = self.model_selector.currentText()

        if not api_key or not description:
            self.output_log.append("Error: API key and description are required\n")
            return

        self.output_log.append("Sending request to Gemini...\n")
        self.code = self.get_code_from_gemini(api_key, description, model_name)

        if self.code:
            self.execute_and_display()
        else:
            self.output_log.append("Failed to generate code.\n")

    def improve_code(self):
        """Improves the existing code based on user suggestions."""
        if self.code is None:
            self.output_log.append("Error: No code to improve. Generate code first.\n")
            return

        api_key = self.api_key_input.text().strip()  # Get API key
        suggestions = self.suggestions_input.toPlainText().strip()
        description = self.description_input.toPlainText().strip() #Need the original description
        model_name = self.model_selector.currentText()

        if not suggestions:
            self.output_log.append("Error: Please provide suggestions to improve the code.\n")
            return

        self.output_log.append("Applying your suggestions...\n")
        self.code = self.apply_suggestions(self.code, suggestions, description, model_name)
        self.execute_and_display()

    def execute_and_display(self):
        """Executes the code and displays the result/errors."""
        self.output_log.append("Executing code in FreeCAD...\n")
        success, message = self.execute_code(self.code)
        if success:
            self.output_log.append(f"Success: {message}\n")
        else:
            self.output_log.append(f"Error: {message}\n")
            self.output_log.append(f"Current Code:\n{self.code}\n")


    def apply_suggestions(self, code: str, suggestions: str, original_description: str, model_name: str) -> str:
        """Sends code and user suggestions to LLM for improvement."""
        if not suggestions:  # Should not happen, but good to check
            return code

        api_key = self.api_key_input.text().strip()
        api_endpoint = f"https://generativelanguage.googleapis.com/v1beta/models/{model_name}:generateContent?key={api_key}"
        system_prompt = "Refine FreeCAD Python code based on user suggestions.  Return ONLY improved code, no explanations."
        # Include the original description:
        user_prompt = f"""Original Description:\n{original_description}\n\nOriginal Code:\n{code}\n\nUser Suggestions:\n{suggestions}\n\nImproved Code:"""

        request_data = {"contents": [{"role": "user", "parts": [{"text": f"{system_prompt}\n{user_prompt}"}]}],
                        "generationConfig": {"temperature": 0.3, "maxOutputTokens": 8000}}
        response = _send_request_to_gemini(api_endpoint, request_data, self.output_log)

        if response and response.get("candidates"):
            improved_code = response["candidates"][0]["content"]["parts"][0]["text"].replace("```python", "").replace("```", "").strip()
            return improved_code
        self.output_log.append("Failed to apply suggestions. Using previous code.\n")
        return code


    def get_code_from_gemini(self, api_key: str, description: str, model_name: str) -> str:
        """Gets FreeCAD Python code from the Gemini API."""
        api_endpoint = f"https://generativelanguage.googleapis.com/v1beta/models/{model_name}:generateContent?key={api_key}"

        system_prompt = """You are a CAD assistant that converts descriptions into FreeCAD Python code.
Return ONLY executable Python code with no explanations, comments, or markdown formatting (no ```).
The code should create the specified 3D model in FreeCAD.  Be concise but complete."""

        user_prompt = f"""Create FreeCAD Python code for: {description}

The code must:
1. Import necessary FreeCAD modules (e.g., `import FreeCAD as App`, `import Part`).
2. Create a new document if one is not already active (`App.ActiveDocument`).  Use `App.newDocument("ModelName")` if needed.
3. Create the 3D geometry described, using appropriate FreeCAD functions (e.g., `Part.makeBox`, `Part.makeSphere`, `Part.makeCylinder`).
4. Use parametric modeling where appropriate. Define dimensions as variables (e.g., `length = 10`, `width = 5`).
5. If dimensions are not specified, use reasonable default values.  Do *not* leave dimensions undefined.
6. Add the created shape to the active document using `Part.show(shape_name)`.
7. Ensure the code is complete and ready to execute without errors.
8. Recompute the document App.ActiveDocument.recompute()
9. Set the view to fit the object Gui.SendMsgToActiveView("ViewFit")
Return ONLY the Python code. Do not include ANY explanations or extra text.
"""
        request_data = {
            "contents": [
                {"role": "user", "parts": [{"text": system_prompt + '\n' + user_prompt}]},
            ],
            "generationConfig": {
                "temperature": 0.3,
                "maxOutputTokens": 6000,
            }
        }

        response_data = _send_request_to_gemini(api_endpoint, request_data, self.output_log)
        if not response_data:
            return ""

        if response_data and "candidates" in response_data and response_data["candidates"]:
            code = response_data["candidates"][0]["content"]["parts"][0]["text"]
            code = code.replace("```python", "").replace("```", "").strip()  # Remove markdown
            return code
        else:
            self.output_log.append("Failed to get code from Gemini.\n")
            if response_data:
                self.output_log.append(json.dumps(response_data, indent=2))
            return ""

    def execute_code(self, code: str) -> tuple[bool, str]:
        """Executes the given Python code in FreeCAD."""
        try:
            if App.ActiveDocument is None:
                App.newDocument("GeminiGenerated")
            exec(code, globals())
            App.ActiveDocument.recompute()
            Gui.SendMsgToActiveView("ViewFit")
            return True, "Model created successfully"
        except Exception as e:
            return False, f"{type(e).__name__}: {e}\n{traceback.format_exc()}"


# --- Main Execution ---
if App.ActiveDocument is None:
    App.newDocument("GeminiGenerated")

dialog = LLMToCADDialog()
dialog.show()
