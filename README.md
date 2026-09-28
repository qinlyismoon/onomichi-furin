# 尾道 · 風鈴 — Running locally

Open a terminal in this folder and run one of:

    python3 -m http.server 5173
    # or
    npx serve -l 5173

Then open http://localhost:5173 in your browser.

- localhost counts as a secure context, so **Blow into mic** works. The browser asks for microphone permission the first time.
- In the local version, wishes you hang are saved in the browser's localStorage. Only this computer and this browser can see them.
- The scene follows the local time in 尾道 (Asia/Tokyo): morning, day, sunset, and night. To preview a specific time, add `?time=HH:MM`, e.g. http://localhost:5173/?time=18:42 (sunset) or `?time=21:30` (night).
- Double-clicking `index.html` also opens the page, but the mic may not work under `file://`. Use localhost instead.
