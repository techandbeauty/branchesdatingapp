# Branches — Landing Page

> A new kind of dating. Built for real connection.

The pre-launch waitlist landing page for **Branches**, a voice-first dating app built for military, veterans, DoD personnel, first responders, and anyone tired of swiping. This page collects early access signups to Firebase Firestore.

---

## Current Status

- **Phase 1** (current): Waitlist landing page — captures early access signups before app launch
- **Phase 2** (post-launch): Convert this same page into a full marketing homepage for the app (see [Future Roadmap](#future-roadmap-marketing-homepage))

---

## Live Site

Once deployed: `https://YOUR_GITHUB_USERNAME.github.io/branches/`

Or custom domain: `https://branchesdate.app` (when configured)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Hosting | GitHub Pages (free, HTTPS) |
| Backend / Database | Firebase Firestore |
| Email Automation | Brevo (optional, ready to enable) |
| Video Hosting | Cloudinary (hero video) |
| Fonts | Cormorant Garamond, Dancing Script, Inter (Google Fonts) |
| Build Tools | None — pure HTML/CSS/JS, no build step |

---

## File Structure

```
branches/
├── index.html              ← The entire landing page (single file)
├── firestore.rules         ← Firebase security rules
└── README.md               ← This file
```

That's it. One HTML file. No npm, no build process, no dependencies to install.

---

## Quick Start

### 1. Clone the repo

```bash
git clone https://github.com/YOUR_USERNAME/branches.git
cd branches
```

### 2. Preview locally

Open `index.html` in your browser — or use VS Code's **Live Server** extension for auto-reload. Install it from the Extensions tab, then right-click `index.html` → **Open with Live Server**.

### 3. Push changes

```bash
git add .
git commit -m "Your change"
git push
```

GitHub Pages auto-deploys in ~60 seconds after every push.

---

## Firebase Setup

The landing page writes signups to a Firestore collection called `waitlist`. If you're setting up a new Firebase project:

### 1. Create the project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project** → name it `branches-dating` → create
3. Left sidebar → **Build → Firestore Database** → **Create database**
4. Select **Production mode** → pick `us-east1` region → enable

### 2. Register the web app

1. **Project Settings** (gear icon) → **Your apps** → click the Web icon (`</>`)
2. Nickname: `Branches Landing`
3. Do **not** check "Firebase Hosting"
4. Copy the `firebaseConfig` object

### 3. Paste the config into `index.html`

Open `index.html`, search for `firebaseConfig`, and replace with yours:

```javascript
const firebaseConfig = {
  apiKey: "YOUR_KEY",
  authDomain: "branches-dating.firebaseapp.com",
  projectId: "branches-dating",
  storageBucket: "branches-dating.firebasestorage.app",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

### 4. Deploy security rules

In Firebase Console → **Firestore → Rules tab**, paste:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /waitlist/{docId} {
      allow create: if true;
      allow read, update, delete: if false;
    }
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

Click **Publish**. These rules allow the public to submit new waitlist entries but block anyone from reading or scraping the list.

---

## Adding the Hero Video (Cloudinary)

The hero has a full-bleed video background that's currently a warm dark gradient. To add your Cloudinary video:

### 1. Upload your video to Cloudinary

1. Log into [cloudinary.com](https://cloudinary.com)
2. Go to **Media Library** → **Upload**
3. Upload your video (MP4, 10–30 seconds, looping-friendly, under 10 MB ideal)
4. Click the uploaded video → copy the **Secure URL**

It will look like: `https://res.cloudinary.com/dxxxxx/video/upload/v1234567890/branches_hero.mp4`

### 2. Add it to `index.html`

Search for this block:

```html
<video id="heroVideo" autoplay muted loop playsinline preload="auto">
  <!-- PASTE YOUR CLOUDINARY VIDEO URL HERE -->
  <!-- <source src="https://res.cloudinary.com/YOUR_CLOUD/video/upload/YOUR_VIDEO.mp4" type="video/mp4"> -->
</video>
```

Uncomment the `<source>` line and paste your URL:

```html
<video id="heroVideo" autoplay muted loop playsinline preload="auto">
  <source src="https://res.cloudinary.com/dxxxxx/video/upload/v1234567890/branches_hero.mp4" type="video/mp4">
</video>
```

Commit and push — the video will fade in behind the text on the next page load.

### Cloudinary tips for best performance

- Use Cloudinary's automatic format/quality delivery: add `/f_auto,q_auto/` to the URL:
  `https://res.cloudinary.com/dxxxxx/video/upload/f_auto,q_auto/v1234567890/branches_hero.mp4`
- Keep the video silent — the hero already mutes it
- Keep it under 15 seconds and make sure the first frame blends seamlessly into the last frame so the loop is invisible

---

## Managing the Waitlist

### View signups

**Firebase Console → Firestore Database → Data tab → `waitlist` collection**

You'll see every signup with:
- `firstName`
- `email`
- `interest` (dropdown selection)
- `source` (always `landing_page`)
- `createdAt` (timestamp)
- `invited` (boolean — flip to `true` when you've sent the app link)

### Export to CSV

Run this Node.js script locally to export everything:

```javascript
// export-waitlist.js
const admin = require('firebase-admin');
admin.initializeApp({
  credential: admin.credential.cert(require('./serviceAccountKey.json'))
});
const db = admin.firestore();

(async () => {
  const snap = await db.collection('waitlist').orderBy('createdAt', 'desc').get();
  let csv = 'First Name,Email,Interest,Signed Up\n';
  snap.forEach(doc => {
    const d = doc.data();
    const date = d.createdAt ? d.createdAt.toDate().toISOString() : '';
    csv += `"${d.firstName}","${d.email}","${d.interest || ''}","${date}"\n`;
  });
  require('fs').writeFileSync('branches-waitlist.csv', csv);
  console.log(`Exported ${snap.size} signups.`);
})();
```

To use it: download a service account key from Firebase Console → Project Settings → Service Accounts → Generate new private key → save as `serviceAccountKey.json` (never commit this file).

---

## Optional: Enable Brevo for Automatic Emails

The landing page is pre-wired to send a welcome email via Brevo when someone joins. To activate it:

1. Sign up at [app.brevo.com](https://app.brevo.com) (free tier = 300 emails/day)
2. Verify your sender email (the "from" address)
3. Generate an API key: **Settings → SMTP & API → API Keys**
4. In `index.html`, search for `YOUR_BREVO_API_KEY` and replace with your actual key
5. Update the sender email on the line `sender: { name: 'Branches', email: 'hello@branchesdate.app' }`
6. Commit and push

**Security note:** Putting the API key in client-side code works for a small waitlist but isn't ideal long-term. When you scale, move the email-sending to a Firebase Cloud Function so the key stays server-side.

---

## Customizing Content

All text is in `index.html` — no translation files, no CMS. To change anything:

| Element | How to find it |
|---|---|
| Main headline | Search `The next` |
| Subtitle | Search `new kind of dating` |
| "Coming Soon" text | Search `Coming Soon` |
| Right-side movement text | Search `more than` |
| Form heading | Search `first to know` |
| Privacy line | Search `respect your privacy` |
| Footer tagline | Search `Built on intention` |
| Thank you screen | Search `Welcome in` |

---

## Deploying to GitHub Pages

### First-time setup

1. Create a GitHub repo (public required for free Pages)
2. Push `index.html` to the `main` branch
3. Repo **Settings → Pages**
4. Source: **Deploy from a branch** → `main` → `/ (root)` → Save
5. Wait ~2 minutes
6. Check **Enforce HTTPS**
7. Your site is live at `https://YOUR_USERNAME.github.io/REPO_NAME/`

### Custom domain (optional)

1. Buy a domain (Namecheap, Google Domains, etc.)
2. Repo **Settings → Pages → Custom domain** → enter your domain
3. At your domain registrar, add these DNS records:
   - `A` → `185.199.108.153`
   - `A` → `185.199.109.153`
   - `A` → `185.199.110.153`
   - `A` → `185.199.111.153`
   - `CNAME www` → `YOUR_USERNAME.github.io`
4. Wait for DNS propagation (a few minutes to a few hours)
5. Enable HTTPS once GitHub verifies the domain

---

## Security

This landing page has several safeguards built in:

- **Input sanitization** — strips HTML entities and blocks SQL/script injection patterns
- **Email validation** — strict regex + blocks disposable email domains (Mailinator, Guerrilla Mail, etc.)
- **Honeypot field** — hidden `website` field catches bots
- **Timing check** — rejects submissions under 2.5 seconds (bot behavior)
- **Rate limiting** — max 3 submissions per session with 8-second cooldown
- **Duplicate detection** — checks Firestore before creating a new entry
- **CSP headers** — `X-Content-Type-Options: nosniff` and `X-Frame-Options: DENY`
- **Firestore rules** — public can write but cannot read/update/delete

---

## Future Roadmap: Marketing Homepage

When the Branches app goes live, this landing page converts into the full marketing homepage. The existing hero, waitlist form, and footer will stay — you'll just add new sections between them.

### Sections to add post-launch

Here's the planned structure when the app is live:

```
┌─────────────────────────────────────────┐
│ NAV (keep — add "Sign In" link)         │
│ ─────────────────────────────────────── │
│ HERO (keep — swap "Coming Soon" for     │
│   "Download on App Store / Google Play")│
│ ─────────────────────────────────────── │
│ ⭐ NEW: THE EXPERIENCE                  │
│   4-step breakdown of how Branches works│
│   (Pod → Conversation → Reveal → IRL)   │
│ ─────────────────────────────────────── │
│ ⭐ NEW: APP SCREENSHOTS                 │
│   iPhone mockups showing the real app UI│
│ ─────────────────────────────────────── │
│ ⭐ NEW: WHY BRANCHES                    │
│   Voice first · Verified · Built for    │
│   commitment                            │
│ ─────────────────────────────────────── │
│ ⭐ NEW: SAFETY FEATURES                 │
│   Panic button · Live location · Check- │
│   ins · Verified identity               │
│ ─────────────────────────────────────── │
│ ⭐ NEW: PRICING                         │
│   Free to start · $8.99/mo after first  │
│   date or 30 days                       │
│ ─────────────────────────────────────── │
│ ⭐ NEW: TESTIMONIALS                    │
│   Real success stories from early users │
│ ─────────────────────────────────────── │
│ ⭐ NEW: MANIFESTO / OUR STORY           │
│   Why you built Branches                │
│ ─────────────────────────────────────── │
│ ⭐ NEW: FAQ                             │
│   Common questions about the app        │
│ ─────────────────────────────────────── │
│ ⭐ NEW: DOWNLOAD CTA                    │
│   Big App Store / Play Store buttons    │
│ ─────────────────────────────────────── │
│ FOOTER (keep — add more links: Privacy, │
│   Terms, Press, Support, Careers)       │
└─────────────────────────────────────────┘
```

### What changes in the hero post-launch

- **"Coming Soon"** → replaced with **App Store + Google Play download buttons**
- **"Watch the Film"** → stays, but links to a polished launch trailer
- **Waitlist form** → moves to a "Stay Updated" newsletter box in the footer, replaced in the hero by download CTAs
- **Navigation** → add `Sign In` link that points to the web version of the app (if applicable)
- **Coming Soon script** → replaced with real press mentions or `★★★★★ 4.9 on the App Store`

### How to do the conversion

When you're ready to flip from landing page to marketing homepage:

1. Create a new branch: `git checkout -b marketing-homepage`
2. Build out the new sections in a copy of `index.html`
3. Test locally with Live Server
4. Preserve the `<form>` and all Firebase/validation JavaScript — you'll still want a newsletter signup in the footer
5. Merge to `main` when ready → GitHub Pages auto-deploys

I can help you build this when the app is ready to launch — just let me know when you're approaching that milestone.

---

## Roadmap Checklist

**Phase 1 — Pre-launch (current)**
- [x] Landing page live on GitHub Pages
- [x] Firebase Firestore collecting waitlist
- [x] Security rules deployed
- [x] Form validation + security
- [ ] Cloudinary hero video added
- [ ] Brevo confirmation emails active
- [ ] Custom domain configured (optional)
- [ ] Share link with military/veteran/DoD communities

**Phase 2 — App launch**
- [ ] Replace "Coming Soon" with download buttons
- [ ] Add app screenshot section
- [ ] Add pricing section
- [ ] Add safety features section
- [ ] Add manifesto/story section
- [ ] Add FAQ section
- [ ] Add testimonials from beta users
- [ ] Email waitlist with download link
- [ ] Launch on Product Hunt / social media

**Phase 3 — Post-launch growth**
- [ ] Add /press page
- [ ] Add /manifesto page
- [ ] Add /journal (blog)
- [ ] Add success stories carousel
- [ ] A/B test CTAs

---

## Support

Built by **Nexus Tech Digital Solutions**  
Founder: Meisha Vernell  
Website: [nexustechdigitalsolutions.com](https://nexustechdigitalsolutions.com)

For questions about this landing page, check the code comments in `index.html` — each section is labeled.

---

## License

© 2026 Branches. All rights reserved. A Nexus Tech Digital Solutions product.
