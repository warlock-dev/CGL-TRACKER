# SSC CGL Revision Register

A single-page study tracker for the SSC CGL General Studies syllabus (121 topics across 11 subjects), built around spaced-repetition scheduling, shared between two people (Mayank and Nidhi), synced across devices via Firebase.

## What it does

- Tracks every topic from the syllabus: reading status, revision count, and next-due date.
- Schedules revisions along a forgetting-curve interval: 1 → 3 → 7 → 15 → 30 → 60 days after each revision, marking a topic "mastered" after 5 revisions.
- Shows a daily "docket" of exactly what's overdue or due today.
- Asks "Who's revising?" on open — Mayank and Nidhi each get their own separate progress, saved under their own name.
- Syncs progress to the cloud (Firestore), so opening the site on your phone shows the same state as your computer.

## One-time setup: connect it to your own free Firebase project

The tracker needs a place to store progress that both your phone and computer can reach. Firebase's free tier is enough for this. Takes about 10 minutes.

1. **Create a Firebase project**
   Go to [console.firebase.google.com](https://console.firebase.google.com) → **Add project** → give it any name (e.g. `ssc-cgl-tracker`) → you can turn off Google Analytics → **Create project**.

2. **Turn on Firestore (the database)**
   In the left sidebar, click **Build → Firestore Database → Create database**. Choose a region close to you, and start in **production mode**.

3. **Set the security rules**
   Go to the **Rules** tab in Firestore and replace the contents with:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /progress/{personId} {
         allow read, write: if personId == 'Mayank' || personId == 'Nidhi';
       }
     }
   }
   ```
   Click **Publish**. This keeps the database limited to exactly two documents — one per person — and nothing else is readable or writable.

4. **Register a web app to get your config**
   Back on the project's main page, click the **</>** (web) icon → give the app a nickname → **Register app**. It will show you a `firebaseConfig` object that looks like this:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "ssc-cgl-tracker.firebaseapp.com",
     projectId: "ssc-cgl-tracker",
     storageBucket: "ssc-cgl-tracker.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
   Copy this whole block.

5. **Paste it into `index.html`**
   Open `index.html`, search for `firebaseConfig`, and replace the placeholder object (the one with `"REPLACE_ME"` values) with the one you just copied. Save the file.

   Note: these values are meant to be public in client-side apps — Firebase's actual access control comes from the security rules in step 3, not from hiding this config. It's fine for this to be visible in your public GitHub repo.

## Publishing on GitHub Pages

1. Push this folder to a GitHub repo.
2. In the repo's **Settings → Pages**, set the source to your main branch, root folder.
3. Your tracker will be live at `https://<username>.github.io/<repo-name>/` — open that same link on your phone and your computer.

## How the two-person part works

- The first time anyone opens the site (on any device), it asks "Who's revising?" — tap Mayank or Nidhi.
- That device remembers the choice for next time, but you can tap the name pill in the top-right corner any time to switch.
- Each name's progress is stored separately in Firestore under `progress/Mayank` and `progress/Nidhi`, so you don't overwrite each other.
- If you're offline, progress still saves locally on that device and syncs up the next time you're online.
