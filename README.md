# GeminiFreeCAD
A Macro for FreeCAD that allows the user to communicate with Gemini API to produce 3D Models from text desctriptions.

## Installation

### Automatic Installation
1. Download and unzip the GeminiFeedbackCAD package
2. Run `install.py` (double-click or run with Python)
3. The script will automatically copy the macro to your FreeCAD Macro directory

### Manual Installation
If automatic installation fails:
1. Locate your FreeCAD Macro directory:
   - Windows: `%APPDATA%\FreeCAD\Macro`
   - macOS: `~/Library/Preferences/FreeCAD/Macro`
   - Linux: `~/.FreeCAD/Macro`
2. Copy `GeminiFreeCAD.FCMacro` to this directory

## Getting a Free Google Gemini API Key

1. Visit the [Google AI Studio](https://makersuite.google.com/app/apikey) (sign in with your Google account)
2. Click on "Get API key" in the left sidebar
3. Either:
   - Select an existing project, or
   - Create a new project (click "+ Create a new project")
4. Click "Create API key"
5. Copy your new API key for use in the GeminiFeedbackCAD macro
6. Note: Google's free tier currently includes up to 60 requests per minute, which is more than sufficient for casual use

## Usage
1. In FreeCAD, go to Macro → Macros...
2. Select "GeminiFreeCAD" and click "Execute"
3. Enter your Google Cloud API key
4. Type a description of what you want to create
5. Click "Generate" to create the model
6. Add suggestions in the feedback field and click "Improve" to refine the model

## Requirements
- FreeCAD 1.0.0 or newer
- Google Cloud API key with Gemini API access

## License
[MIT License](LICENSE)
