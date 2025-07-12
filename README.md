# Math Worksheet Generator (Web Edition)

A modern, web-based Python app for generating customizable math worksheet PDFs. Supports multiple problem types, per-type term ranges, and a user-friendly web UI. Built with FastAPI and ReportLab. Originally forked from [https://github.com/januschung/math-worksheet-generator/](https://github.com/januschung/math-worksheet-generator/)

Demo site: [https://math-worksheet-generator-xse1.onrender.com](https://math-worksheet-generator-xse1.onrender.com) (takes up to 60 seconds to load from demo host)

![image](https://github.com/user-attachments/assets/e6a245e4-0628-4217-bbbb-2fd76a7868ef)



## Features
- Generate printable math worksheets as PDFs
- Supports multiplication, addition, subtraction, division, missing factor, and fraction comparison
- Per-type term range customization (e.g., 2..12)
- Modern, interactive web UI (no CLI required)
- REST API for programmatic access
- Inline PDF preview and download
- Bookmarkable URLs to recall specific configurations

## Requirements
- Python 3.8+
- `reportlab` and `fastapi` (see `requirements.txt`)

## Quick Start
1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. Run the web app:
   ```bash
   python main.py
   ```
3. Open your browser to [http://localhost:8000](http://localhost:8000)

## Web UI Usage
- Select one or more problem types.
- Optionally adjust the term ranges (defaults are shown and can be reset).
- Ranges use the `min..max` format (e.g. `3..12`).
  Term1 is the first number for the chosen type and term2 is the second
  (for division that means divisor and quotient).
- Examples with default limits:
  - Multiplication: `3 × 19`
  - Addition: `50 + 99`
  - Subtraction: `99 - 10`
  - Division: `45 ÷ 5 (quotient 9)`
  - Missing Factor: `2 × __ = 40`
  - Fraction Compare: `3/8 ? 5/9`
- Set the number of problems per type.
- Choose how many problems appear in each section (defaults to 25).
- Click **Generate** to preview and download the worksheet PDF.
- After generating, the URL updates so you can bookmark or share the settings.

## API Endpoints
### `POST /generate`
Generate a worksheet PDF.
- **Request JSON:**
  ```json
  {
    "problem_types": ["multiplication", "addition"],
    "n": 100,
    "chunk": 25,
    "term1": "2..12",      // optional, single-type only
    "term2": "2..12",      // optional, single-type only
    "defaults": {           // optional per-type overrides
      "multiplication": {"term1": "3..12", "term2": "2..15"}
    }
  }
  ```
- **Response:** `{ "url": "/pdf/worksheet_xxx.pdf" }`

### `GET /pdf/{filename}`
Serve a generated PDF for inline display or download.

### `GET /defaults`
Return the default term1/term2 ranges for each problem type as a JSON object:
```json
{
  "multiplication": {"term1": "3..12", "term2": "2..19"},
  "addition": {"term1": "50..300", "term2": "10..99"},
  ...
}
```

## Customizing Problem Types and Ranges
- The web UI fetches `/defaults` and prepopulates the term range fields.
- You can edit the ranges before generating a worksheet.
- To change the built-in defaults, edit `PROBLEM_DEFAULTS` in `worksheet_core.py`.

## Extending the Core Logic
- All worksheet logic is in `worksheet_core.py`.
- To add a new problem type:
  1. Create a new class inheriting from `MathProblem`.
  2. Add it to `PROBLEM_DEFAULTS` and `PROBLEM_CLASS_MAP` in `main.py`.
  3. Update the web UI HTML if you want it selectable.

## Example: Programmatic API Usage
```python
import requests
payload = {
    "problem_types": ["multiplication"],
    "n": 50,
    "chunk": 25,
    "term1": "2..12",
    "term2": "2..12"
}
r = requests.post("http://localhost:8000/generate", json=payload)
print(r.json())  # {"url": "/pdf/worksheet_xxx.pdf"}
```

## Notes
- There is no CLI interface; all usage is via the web UI or API.
- All worksheet generation is stateless and per-request.

## Deployment

### Recommended: Docker Compose

For LAN or production use, deploy with Docker Compose for easy management, automatic restarts, and persistent storage.

1. **Build and start the app:**
   ```bash
   docker compose up -d
   ```
   This will build the image and start the service in the background.

2. **Access the app:**
   - Open [http://localhost:8000](http://localhost:8000) or use your server's IP on your LAN.

3. **Persist generated worksheets:**
   - The `generated/` directory is mapped to a Docker volume and will persist across restarts and upgrades.

4. **Stop the app:**
   ```bash
   docker compose down
   ```

5. **Restart the app:**
   ```bash
   docker compose up -d
   ```

6. **Update the app:**
   - Pull the latest code, then rebuild:
     ```bash
     git pull
     docker compose build
     docker compose up -d
     ```

7. **Automatic restart:**
   - The service uses `restart: unless-stopped` and will automatically restart after a server reboot or crash.

### Alternative: Quick Test with Docker Run
For quick, one-off tests (not recommended for production):
```bash
docker run --rm -p 8000:8000 \
  -v $(pwd):/app -w /app python:3.12-slim \
  sh -c "pip install -r requirements.txt && uvicorn main:app --host 0.0.0.0"
```
- This does not persist generated files or restart automatically.

---

For more details, see the code and docstrings in `main.py` and `worksheet_core.py`.
