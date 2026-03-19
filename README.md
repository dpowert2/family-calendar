# Family Calendar — Wall Display

A full-screen family calendar web app designed for a Microsoft Surface wall display. Built as a static site for GitHub Pages with Supabase for event storage and Outlook calendar integration.

## Features

- Monthly and weekly calendar views
- Tap any day to add family events (title, time, color, description)
- Edit and delete events — both Dave and Sarah can manage events
- Outlook work calendar shown as grey "Busy" blocks (no details exposed)
- Current day highlighted, live clock display
- Touch-friendly UI with swipe navigation and large tap targets
- Auto-refreshes on a configurable interval
- Settings panel to configure Supabase and ICS without editing code
- Keyboard shortcuts: Arrow keys to navigate, T for today, Escape to close modals

## Architecture

```
GitHub Pages (static site)
├── index.html          — Single-page calendar app (HTML + CSS + JS)
├── calendar.ics        — Cached Outlook calendar (auto-updated by GitHub Action)
└── .github/workflows/
    └── sync-calendar.yml — Fetches ICS feed every 15 minutes
```

Supabase provides the backend for family event CRUD via its REST API using the anon key.

## Setup Instructions

### 1. Create a GitHub Repository

```bash
# Create a new repo on GitHub named "family-calendar"
# Then push this code:
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/dpowert2/family-calendar.git
git push -u origin main
```

### 2. Set Up Supabase

1. Go to [supabase.com](https://supabase.com) and create a free project.
2. Open the **SQL Editor** and run this schema:

```sql
CREATE TABLE events (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title TEXT NOT NULL,
  start_time TIMESTAMPTZ NOT NULL,
  end_time TIMESTAMPTZ NOT NULL,
  color TEXT DEFAULT '#4A90D9',
  description TEXT,
  created_by TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE events ENABLE ROW LEVEL SECURITY;

-- Allow all operations with anon key (family use, not public)
CREATE POLICY "Allow all" ON events FOR ALL USING (true) WITH CHECK (true);
```

3. Go to **Settings → API** and copy your **Project URL** and **anon public key**.

### 3. Configure GitHub Secrets

In your GitHub repo, go to **Settings → Secrets and variables → Actions** and add:

| Secret Name     | Value                                      |
|-----------------|--------------------------------------------|
| `ICS_FEED_URL`  | Your Outlook ICS feed URL                  |

The ICS feed URL for this project:
```
https://mail-nam.mcld.fmrcloud.com/owa/calendar/df4dbff18ed44c2aa2527ab91d18d63d@fmr.com/2d6a4011dcea4998bb830cf91dfabe9018128091869267833821/calendar.ics
```

### 4. Enable GitHub Pages

1. Go to repo **Settings → Pages**
2. Under "Source", select **Deploy from a branch**
3. Choose **main** branch, **/ (root)** folder
4. Click Save

Your calendar will be live at: `https://dpowert2.github.io/family-calendar/`

### 5. Configure the App

1. Open the calendar in your browser
2. Click the **gear icon** (⚙) in the top right
3. Enter your **Supabase Project URL** and **anon key**
4. Adjust the refresh interval if desired (default: 5 minutes)
5. Click **Save & Reconnect**

These settings are saved in the browser's localStorage, so each device only needs to be configured once.

### 6. Trigger the First ICS Sync

The GitHub Action runs every 15 minutes automatically. To trigger it immediately:

1. Go to repo **Actions** tab
2. Click **Sync Outlook Calendar** workflow
3. Click **Run workflow**

This will fetch your Outlook calendar and commit `calendar.ics` to the repo.

## Usage Tips

- **Add an event:** Tap any day cell (month view) or time slot (week view)
- **Edit an event:** Tap the colored event chip
- **Delete an event:** Open an event and click the red Delete button
- **Navigate:** Swipe left/right on touch, or use arrow keys
- **Switch views:** Use the Month/Week toggle in the header
- **Full-screen kiosk:** On the Surface, open Edge and press F11 for full-screen mode

## Surface Wall Display Setup

For the best wall-display experience on your Surface:

1. Open Microsoft Edge and navigate to your calendar URL
2. Press **F11** for full-screen mode
3. Optionally, set the page as a startup page in Edge settings
4. Disable sleep/screen timeout in Windows Settings → System → Power
5. Consider using Windows Task Scheduler to auto-launch Edge on boot

## Troubleshooting

- **Events not loading:** Check that Supabase URL and anon key are correct in Settings
- **Work calendar not showing:** Verify the GitHub Action ran successfully (check Actions tab). The `calendar.ics` file should exist in your repo root
- **ICS sync failing:** Make sure the `ICS_FEED_URL` secret is set correctly. Some corporate networks may block external access — the ICS URL must be accessible from GitHub's servers
- **CORS errors:** The app fetches `calendar.ics` from the same origin (GitHub Pages), so CORS should not be an issue. Supabase APIs have CORS enabled by default
