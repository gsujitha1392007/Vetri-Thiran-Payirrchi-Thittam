# ComicCraft — AI Comic Story Creator

ComicCraft is a FastAPI web application based on the supplied project documentation. It accepts a story idea and preferences, generates a structured multi-panel outline, expands it into narration/dialogue, generates an illustration for every panel, displays the result, and exports the comic as a PDF.

## Architecture

Browser → FastAPI/Jinja2 → Gemini outline → Gemini story → Hugging Face image generation → layout builder → FPDF PDF export.

The source document describes the same core modules and routes:
- `gemini_flash.py` → structured panel outline
- `gemini_pro.py` → narration/dialogue
- `image_generator.py` → panel illustrations
- `layout_builder.py` → combines text + images
- `exporters.py` → PDF
- `routes.py` → FastAPI workflow

## Important modernization

The supplied document names Gemini 1.5 Flash/Pro and Stable Diffusion v1.5. Those are kept as architectural concepts, but this implementation uses configurable current model IDs in `.env`. The default Gemini IDs are current stable/preview model IDs, and image generation is performed through Hugging Face Inference Providers so a local GPU is not required.

## Folder structure

```text
ComicCraft/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── schemas.py
│   ├── routes.py
│   └── services/
│       ├── __init__.py
│       ├── gemini_flash.py
│       ├── gemini_pro.py
│       ├── image_generator.py
│       ├── layout_builder.py
│       └── exporters.py
├── static/
│   ├── css/style.css
│   ├── js/app.js
│   ├── panels/
│   └── exports/
├── templates/
│   ├── base.html
│   ├── index.html
│   ├── comic_preview.html
│   ├── export_success.html
│   └── error.html
├── .env.example
├── .gitignore
├── requirements.txt
└── run.bat
```

## VS Code setup — Windows

### 1. Open the project

Extract the ZIP and open the `ComicCraft` folder in VS Code.

### 2. Create a virtual environment

Open VS Code Terminal:

```powershell
py -m venv .venv
```

Activate:

```powershell
.venv\Scripts\Activate.ps1
```

If PowerShell blocks activation, use:

```powershell
.venv\Scripts\python.exe -m pip install -r requirements.txt
```

### 3. Install packages

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Create `.env`

Copy `.env.example` and rename the copy to:

```text
.env
```

Add:
- `GEMINI_API_KEY`
- `HF_TOKEN`

Do not share or upload `.env`.

### 5. Select the Python interpreter

In VS Code:
`Ctrl + Shift + P` → `Python: Select Interpreter` → choose:

```text
.venv\Scripts\python.exe
```

### 6. Start the app

```powershell
python -m uvicorn app.main:app --reload
```

Open:

```text
http://127.0.0.1:8000
```

API docs:

```text
http://127.0.0.1:8000/docs
```

Health check:

```text
http://127.0.0.1:8000/health
```

## First test

Use:
- Story Prompt: `A brave fox discovers a glowing map in an enchanted forest.`
- Character: `Luna`
- Setting: `Enchanted forest`
- Tone: `Adventure`
- Art Style: `Comic book`
- Panels: `5`

Click **Create My Comic**.

The expected flow is:

1. Gemini creates 5 structured panels.
2. Gemini expands the panels into captions/narration/dialogue.
3. Hugging Face generates one image per panel.
4. The preview page displays all panels.
5. A PDF is created in `static/exports/`.
6. **Download PDF** downloads the generated comic.

## Testing image generation separately

Open:

```text
http://127.0.0.1:8000/test-image?prompt=A%20brave%20fox%20in%20an%20enchanted%20forest
```

A successful response looks like:

```json
{
  "success": true,
  "image_path": "/static/panels/panel_0_xxxxxxxx.png"
}
```

## Testing the JSON API

POST to:

```text
http://127.0.0.1:8000/generate-comic/json
```

Example JSON:

```json
{
  "story_prompt": "A brave fox discovers a glowing map in an enchanted forest.",
  "character_name": "Luna",
  "setting": "Enchanted forest",
  "tone": "Adventure",
  "art_style": "Comic book",
  "panels": 5
}
```

## Offline/UI testing without AI keys

For testing only, set:

```text
DEMO_MODE=true
```

Then run the server and submit the form. Gemini still needs a key because it generates the story, but the image step uses generated placeholder images.

If you want a completely offline UI/PDF test, replace the AI functions with mocked data in a separate test environment; do not use fake AI output as the final project demo.

## Common errors

### `GEMINI_API_KEY is missing`

Check that:
- the file is named exactly `.env`
- it is in the project root
- `GEMINI_API_KEY=...` is present
- the terminal was restarted after creating `.env`

### `HF_TOKEN is missing`

Add a Hugging Face token with inference access to `.env`.

### Image model unavailable

Change:

```text
HF_IMAGE_MODEL=...
```

to another text-to-image model available through your selected Hugging Face Inference Provider.

### Port already in use

Run:

```powershell
python -m uvicorn app.main:app --reload --port 8001
```

Then open `http://127.0.0.1:8001`.

### PDF has missing images

Make sure `static/panels/` exists and that the image-generation step completed before PDF export.

## Security

Never commit:
- `.env`
- API keys
- Hugging Face tokens
- generated private files

The included `.gitignore` already excludes these.

## Educational mapping to the supplied documentation

Milestone 1: configurable AI model selection and architecture.

Milestone 2: core AI modules and PDF/layout modules.

Milestone 3: `routes.py` contains the main application workflow.

Milestone 4: Jinja2 templates + responsive CSS.

Milestone 5: local deployment with Uvicorn, `.env`, static files and API docs.
