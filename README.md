# Money Minded

A personal tracker for income, expenses and savings. It's one page (`index.html`) plus a few images, so it runs on GitHub Pages as-is.

```
index.html             the whole app
manifest.webmanifest   lets you "Add to Home Screen" on your phone
assets/                piggy bank, app icons, fonts (Poppins, bundled so it works offline)
```

---

## 1. Put it on GitHub Pages (5 minutes)

1. Create a new repository on GitHub, e.g. `money-minded`. It can be public or private (private repos need a paid plan for Pages).
2. Upload everything in this folder to the repository, keeping the `assets/` folder intact. You can drag and drop on github.com, or run:
   ```bash
   git init && git add . && git commit -m "Money Minded"
   git branch -M main
   git remote add origin https://github.com/<you>/money-minded.git
   git push -u origin main
   ```
3. In the repository, go to **Settings → Pages**. Under *Build and deployment*, choose **Deploy from a branch**, pick `main` and `/ (root)`, then **Save**.
4. After a minute your site is live at `https://<you>.github.io/money-minded/`.

**On your phone:** open the link, then use Share → *Add to Home Screen* (iPhone) or ⋮ → *Add to Home screen* (Android). It opens full-screen like an app.

Until you do step 2 below, entries are saved **in that browser on that device only**. Use *Download backup* on the home screen now and then.

---

## 2. Turn on Google sign-in and sync (about 10 minutes, once)

GitHub Pages only hosts files, so your entries need a place to live. Firebase (Google's free service) provides both the Google login and a small database. The free plan is far more than one person needs.

### a. Create the Firebase project
1. Go to **https://console.firebase.google.com** and click **Create a project**. Name it `money-minded`. You can switch off Google Analytics.

### b. Switch on Google sign-in
2. In the left menu, open **Build → Authentication → Get started**.
3. Under **Sign-in method**, choose **Google**, toggle **Enable**, pick your email as the support email, then **Save**.
4. Still in Authentication, open **Settings → Authorized domains → Add domain**, and add `<you>.github.io`. (`localhost` is already on the list for testing.)

### c. Create the database
5. Open **Build → Firestore Database → Create database**. Pick a location close to you (for India, `asia-south1 (Mumbai)`) and start in **production mode**.
6. Open the **Rules** tab, replace everything with the rules below, and click **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Each signed-in person can only read and write their own data.
    match /users/{uid}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

**Optional: only you can use the database.** Your site is public, so someone else could sign in and keep their *own* entries in your free quota (they can never see yours). To block that, use this line instead of the `allow` line above, with your Gmail address:

```
      allow read, write: if request.auth != null && request.auth.uid == uid
                         && request.auth.token.email == "you@gmail.com";
```

### d. Connect the site
7. Click the gear icon → **Project settings**. Under **Your apps**, click the web icon **`</>`**, name it `Money Minded`, leave *Firebase Hosting* unticked, and click **Register app**.
8. Firebase shows a `firebaseConfig = { ... }` block. Copy just the `{ ... }` part.
9. Open `index.html` and find this line near the top of the `<script>`:
   ```js
   const FIREBASE_CONFIG = null;
   ```
   Replace `null` with what you copied:
   ```js
   const FIREBASE_CONFIG = {
     apiKey: "AIza...",
     authDomain: "money-minded-xxxx.firebaseapp.com",
     projectId: "money-minded-xxxx",
     storageBucket: "money-minded-xxxx.appspot.com",
     messagingSenderId: "1234567890",
     appId: "1:1234567890:web:abc123"
   };
   ```
   These values are meant to be public, so committing them to GitHub is fine. The rules from step 6 are what protect your data.
10. Commit and push. Open your site and tap **Start saving**. You'll now see a sign-in page: tap **Continue with Google**. After that, the site remembers you on that device and skips straight to Home. (You can also sign in or out from the bottom of the home screen.)

Anything you logged on a device before signing in is uploaded to your account the first time you sign in there. After that, every device you sign in on stays in sync, and entries made offline sync once you're back online.

**Testing on your laptop:** Google sign-in doesn't work when you open `index.html` straight from your file browser. Run `python3 -m http.server 8000` in this folder and open `http://localhost:8000` instead.

**If sign-in shows "Add this site to Authorized domains":** repeat step 4 for the exact address in your browser bar.

---

## How the money works

- **Left to spend** this month = income + money taken out of savings − expenses paid from income − money moved to savings.
- **In savings** = money added − money taken out − expenses paid from savings + month-end leftovers.
- **Month-end leftovers:** when a month ends, whatever you didn't spend moves into savings automatically. If you overspent, the shortfall comes out of savings. Logs shows these lines under *Savings → Month-end leftovers*. They're calculated rather than stored, so editing or backdating an old entry keeps them correct.
- **Taking money out of savings** isn't counted as income. It shows under *Savings → Taken out*.
- **Expenses paid from savings** still count in your spending totals and charts. They come out of savings rather than this month's money.
- **Not enough left to spend?** If an expense this month is bigger than what's left, the app offers to take the rest from savings. The entry then shows how much came from where.
- Dates can be backdated (rolling wheel), but not set in the future.

## Backups

**Download backup** (home screen, bottom) saves everything as a `.json` file. **Restore backup** adds a backup's entries back in: entries you already have stay, and matching ones are replaced by the backup copy.
