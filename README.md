# LinkClick — Photo‑Slideshow Web App

Turn a set of photos into a 1080p slideshow video with fade transitions and optional background music. Includes a simple auth flow and a Tailwind UI.

## Stack
- **Backend:** Flask, Flask-JWT-Extended, Flask-Bcrypt, Flask-CORS
- **Media:** MoviePy, OpenCV (image resize/pad, video build, audio overlay)
- **DB:** MySQL (via `mysql-connector-python`) — *sample Postgres connector left in `connect.py`*
- **UI:** Jinja2 templates + TailwindCSS (landing, login, signup, dashboard)

## Project Structure
```
slideshow_app/
├─ app.py               # Flask app (routes, video building)
├─ connect.py           # Sample Postgres connector (unused by default)
├─ db.py                # Example MySQL connection helper
├─ requirements.txt     # Python deps
├─ templates/
│  ├─ landing.html      # product page
│  ├─ login.html        # auth
│  ├─ signup.html       # auth
│  └─ home.html         # dashboard (upload, preview, create video, add music)
├─ static/
│  ├─ images/           # UI images (backgrounds, icons)
│  ├─ audio/            # background music tracks
│  └─ video.mp4         # output (overwritten per render)
└─ instance/            # optional local config
```

## Quick Start
1. **Python 3.9+** recommended.  
2. Install deps:
   ```bash
   pip install -r requirements.txt
   ```
3. Configure a database:
   - **MySQL (recommended for this code path):**
     - Create a DB named `slideshow_app` and a user.
     - Update the connector (either modify `connect.py` to use MySQL or create a MySQL connection in `app.py`).
     - If using MySQL, replace `BYTEA` with `BLOB` in table DDL where needed.
   - **Postgres (optional):**
     - Set `DATABASE_URL=postgresql://user:pass@host:5432/dbname` and adapt MySQL‑specific statements like `CREATE DATABASE IF NOT EXISTS` and `USE` (not valid in Postgres).
4. Run the server:
   ```bash
   python app.py
   ```
   App defaults to `http://127.0.0.1:5000`.

## How It Works
- **Auth:** `/register` (hash with Bcrypt), `/login` (JWT in HTTP‑only cookie), `/logout`.
- **Upload:** Drag/drop images on **Dashboard** → client JS posts selected files to `/upload_images`.
- **Build video:** Server resizes to **1920×1080**, pads to fit, stitches frames with fade‑in/out (0.5s), saves to `static/video.mp4`.
- **Watch:** `/video` renders the preview page and lists available audio tracks from `static/audio` (or DB).
- **Background music:** POST `/add_background_music` with chosen file; video is re‑muxed with a clipped audio track.
- **Per‑slide duration:** POST `/update_duration` with an array of seconds to re‑encode.

## Routes (summary)
| Route | Method | Purpose |
|---|---|---|
| `/` | GET | Landing page |
| `/register` | GET/POST | Create account |
| `/login` | GET/POST | Sign in (sets JWT cookie) |
| `/dashboard` | GET | Auth‑gated main UI |
| `/upload_images` | POST | Accepts multiple images (form-data) |
| `/video` | GET | View generated slideshow |
| `/update_duration` | POST | Rebuild with custom durations |
| `/add_background_music` | POST | Merge selected audio |
| `/logout` | GET | Clear auth cookie |

## Notes & Limitations
- Output always writes to `static/video.mp4` (last render wins).  
- Images are stored as BLOBs per‑user table; consider moving to object storage for production.  
- SQL types/DDL in the repo mix MySQL & Postgres — pick one engine and align the DDL before deploying.  
- Ensure FFmpeg codecs are available on the machine (MoviePy depends on them).

## Development Tips
- Tailwind/UI lives in `templates/`; tweak styles directly.  
- Drop new background tracks into `static/audio/` then restart.  
- For deployment, prefer Gunicorn + Nginx; enable HTTPS and secure cookies.

---


