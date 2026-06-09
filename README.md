✨ Features

Fully automatic — runs itself, no clicking or babysitting required
Auto-resumes after reload — uses sessionStorage to continue across page reloads without losing progress
Batch processing — unlikes up to 80 posts per batch, then reloads and keeps going
Live status overlay — a green status box appears on Instagram showing exactly what the bot is doing in real time
Smart retry logic — retries button detection up to 6 times before giving up, handles slow page loads gracefully
Auto-stop — detects when no items are left and stops cleanly
Multi-language support — works for both English (Unlike) and Italian (Non mi piace più) Instagram interfaces
No external dependencies — pure vanilla JavaScript, nothing to install beyond Tampermonkey


📋 Requirements

A desktop browser: Chrome, Firefox, or Edge
The Tampermonkey browser extension


🚀 Setup from Scratch
Step 1 — Install Tampermonkey
BrowserLinkChromeChrome Web StoreFirefoxFirefox Add-onsEdgeEdge Add-ons
After installing, you'll see the Tampermonkey icon (dark circle with two dots) in your browser toolbar.

Step 2 — Install the Script

Click the Tampermonkey icon in your toolbar
Select Dashboard
Click the + tab to create a new script
Delete all the default code in the editor
Copy the entire contents of instagram-unlike-bot.user.js and paste it in
Press Ctrl+S (or Cmd+S on Mac) to save
You should see the script appear in your dashboard with a green toggle


Step 3 — Run It

Go to this exact URL in your browser:

   https://www.instagram.com/your_activity/interactions/likes/

The script starts automatically — no button to click
Watch for the green status box in the bottom-right corner of the page
Leave the tab open and let it run — it will reload the page after each batch and keep going on its own


⚠️ Do not close the tab while it's running. You can minimise the window.


⚙️ Configuration
You can tweak these values at the top of the script:
jsconst BATCH_SIZE      = 80;    // how many posts to unlike per batch
const SCROLL_ATTEMPTS = 6;     // how many times to scroll to load more items
const DELAY_SCROLL    = 1200;  // ms to wait between scrolls
const DELAY_CHECKBOX  = 60;    // ms between each checkbox tick
const MAX_SELECT_TRIES = 5;    // retries for finding the Select button
If Instagram is slow on your connection, try increasing DELAY_SCROLL to 2000 and DELAY_CHECKBOX to 100.

🔍 How It Works
Page loads
    │
    ▼
Check sessionStorage → is a run already in progress?
    │
    ▼
Find the "Select" button and click it
    │
    ▼
Scroll the list to load all visible items
    │
    ▼
Tick up to 80 checkboxes
    │
    ├── 0 items found → stop, clear sessionStorage (done!)
    │
    ▼
Click the "Unlike" button → confirm in the popup
    │
    ▼
Wait 3 seconds, then reload the page
    │
    ▼
(sessionStorage flag persists → script auto-resumes from top)
Each full cycle removes one batch of up to 80 likes. Instagram typically shows ~50–80 items per page load, so the number of cycles depends on your total likes.

🟢 Status Overlay
While running, a small overlay appears in the bottom-right corner of Instagram:
MessageMeaningUnlike bot starting…First run, initialisingResuming after reload…Continuing from a previous batchScrolling to load items…Loading more posts into the listSelecting N items…Ticking checkboxesUnlike button foundAbout to open the unlike popupConfirming unlike…Clicking the confirm buttonBatch done — reloading…Batch complete, reloading pageNo items left — done!All likes removed, script stopped

❓ Troubleshooting
The green box doesn't appear

Make sure you're on the exact URL: https://www.instagram.com/your_activity/interactions/likes/
Check Tampermonkey dashboard — the script toggle must be ON (green/blue)
Try a hard reload: Ctrl+Shift+R / Cmd+Shift+R

The script stops mid-way

Open DevTools (F12) → Console tab and look for red errors
Instagram may have updated its UI — open an issue with the error message

It says "Select not found" and keeps reloading

Instagram may be loading slowly. Try increasing DELAY_SCROLL to 2000 in the config

I want to stop it mid-run

Open DevTools console and run: sessionStorage.removeItem('unlikeBot')
Then close or navigate away from the tab


⚠️ Disclaimer
This script interacts with Instagram's UI via simulated user actions. Use it responsibly:

Instagram's terms of service prohibit automated interactions — use at your own risk
Avoid running it at very high speeds; the default delays are intentionally conservative
The author is not responsible for any account restrictions


📄 License
MIT — free to use, modify, and distribute.

🙌 Credits
Original script by maximuspixel. Improved and maintained with auto-resume, status overlay, and error handling.

