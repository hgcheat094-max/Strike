# Ember: streak tracker

A single-page streak tracker. Vanilla HTML/CSS/JS in one `index.html`, no build step. Firebase Auth (Google) and Firestore store each user's data. Guest mode works without an account and uses `localStorage`.

Files: `index.html`, `netlify.toml`, `README.md`.

## 1. Firebase setup

1. Open the [Firebase console](https://console.firebase.google.com) and create a project (Analytics is optional).
2. **Add a web app:** Project settings, General, "Your apps", then the `</>` icon. Register the app and copy the `firebaseConfig` object.
3. **Enable Google sign-in:** Build, Authentication, Get started, Sign-in method, Google, Enable. Choose a support email and save.
4. **Create Firestore:** Build, Firestore Database, Create database. Pick a region and start in production mode.
5. **Set security rules:** Firestore, Rules tab. Paste this and publish:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{userId} {
         allow read, write: if request.auth != null && request.auth.uid == userId;
       }
     }
   }
   ```
6. **Add your config:** open `index.html` and replace the values in the `FIREBASE_CONFIG` block near the top of the `<script type="module">`. Firebase web config values are identifiers, not secrets. Your security rules protect the data.
7. **Authorize your domain:** Authentication, Settings, Authorized domains. Add your Netlify domain (for example `your-site.netlify.app`) and any custom domain. `localhost` is allowed by default.

Until the config is filled in, the app runs in guest mode and the sign-in button explains what's missing.

### Using environment variables instead

A site with no build step can't read Netlify environment variables at runtime. If you prefer env vars, add a build command that writes the values into `index.html` with `sed` before publishing. Keep in mind they end up in the public page either way.

## 2. Run locally

ES modules and Google sign-in need an http origin, so don't open the file directly.

```
npx serve .
```

Then open the printed `http://localhost:...` address.

## 3. Deploy to Netlify

**Drag and drop:** go to [app.netlify.com/drop](https://app.netlify.com/drop) and drop the folder containing these files.

**Git:**
1. Push the folder to a GitHub, GitLab, or Bitbucket repo.
2. In Netlify choose Add new site, Import an existing project, and pick the repo.
3. Leave the build command empty. The publish directory comes from `netlify.toml` (`.`). Deploy.

**CLI:**
```
npm i -g netlify-cli
netlify deploy --prod --dir .
```

After the first deploy, add the Netlify URL to Firebase Authorized domains (step 7 above).

## How it works

- **Streaks:** a check-in stores the user's local date (`YYYY-MM-DD`). The current streak counts consecutive local days ending today, or yesterday if today isn't done yet. A missed day resets it to 0. The best streak is the longest run in the log.
- **Flame:** grows at 1, 7, 30, and 100 days. It dims to an ember when today isn't checked in and turns grey when the streak has ended.
- **Guest migration:** on first sign-in, guest tasks are merged into the account (logs combine for matching tasks), then the local copy is removed.
- **Data:** one Firestore document per user at `users/{uid}` holding a `tasks` array.
- **Reordering:** use the up and down buttons on each card.
- **Accessibility:** keyboard operable, visible focus, `prefers-reduced-motion` respected, and light and dark themes.
