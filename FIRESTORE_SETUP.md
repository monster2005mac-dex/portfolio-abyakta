# Firebase Firestore portfolio sharing

The portfolio is designed to run on a static host. Firestore stores only AES-GCM encrypted portfolio data; recipients use the share password to decrypt it in their browser. Firebase web configuration is public and is not an API secret.

## Create and configure Firebase

1. Open [Firebase Console](https://console.firebase.google.com/) and create a project (or select an existing project).
2. Under **Build > Authentication**, open **Sign-in method**, enable **Email/Password**, and create the owner account under **Users**.
3. Open the user created in the previous step and copy its **User UID**.
4. Under **Build > Firestore Database**, create a database. Choose your region and start in production mode.
5. Under **Project settings > General > Your apps**, register a **Web app** if one does not already exist.
6. Copy the generated web app configuration into `window.FIREBASE_CONFIG` in `index.html`. Replace every `YOUR_*` value. This object is public browser configuration by design; never put service-account keys or admin credentials in it.
7. Set `ownerEmail` to the Authentication account email and `ownerUid` to the account UID.
8. Under **Authentication > Settings > Authorized domains**, add the exact production domain (and `localhost` while testing locally). Firebase Hosting domains are added automatically; custom domains must be listed here.

## Firestore security rules

Replace `OWNER_UID` with the value of the owner's Authentication UID, then publish these rules from **Firestore Database > Rules**:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /portfolios/{id} {
      allow get: if true;
      allow list: if false;
      allow create, update, delete: if request.auth != null
        && request.auth.uid == 'OWNER_UID';
    }
  }
}
```

Anyone with a link can read one document by ID, while writes require the configured owner account. The web app asks visitors for the separate share password; the password is not stored in Firestore.

## Owner password and first publish

1. Deploy the configured site.
2. Open it and choose **Password**, then set the owner password. Enter the same password when Firebase owner sign-in is required; it must also be the Firebase Authentication account password.
3. Choose **Lock**, then use the small owner-login control at the bottom of the page.
4. Choose **Share link**, enter a share password of at least four characters, repeat it, and copy the generated `?portfolio=<id>` URL.
5. To update an existing shared portfolio, keep the same document URL or open the owner login first, load the shared document, enter its share password, edit, and let the page update Firestore.

Changing the share password after publishing creates a new encrypted payload; share the regenerated URL.

## Local testing and deployment

Serve the repository over HTTP rather than opening `index.html` with `file://`, because Web Crypto and Firebase Authentication require a secure browser context or localhost:

```sh
python3 -m http.server 8000
```

Add `localhost` as an authorized domain, then open `http://localhost:8000`.

Deploy the static files with Firebase Hosting, Vercel, Netlify, GitHub Pages, or any HTTPS static host. When deployed, add its custom domain to Firebase Authentication authorized domains and test both public loading and owner sign-in:

```sh
npx firebase-tools@latest deploy --only hosting
```

Firebase Hosting configuration is not required by this repository, but should serve `index.html` at `/`. For Vercel, the included `vercel.json` is sufficient.

## Troubleshooting

- **"Firebase sharing is not configured"**: replace the placeholder values in `window.FIREBASE_CONFIG`.
- **Damaged link or document not found**: check that the `portfolio` query value is complete and that the Firestore document exists.
- **Wrong password**: AES-GCM unlock correctly displays an error; check that the link recipient has the exact share password.
- **Permission denied on save**: verify the owner UID rule, Firebase owner email, owner UID, authorized domain, and that the Email/Password user exists.
