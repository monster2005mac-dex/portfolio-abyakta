# Firestore share-link setup

1. Create a Firebase project at https://console.firebase.google.com.
2. In **Project settings > General**, add a Web app.
3. Copy the Firebase web configuration values from the app.
4. Replace the placeholders in `FIREBASE_CONFIG` near the top of `index.html`.
5. In Firebase Console, open **Firestore Database** and create a database.
6. Open **Firestore Database > Rules** and publish:

```txt
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /portfolios/{portfolioId} {
      allow read: if true;
      allow create, update: if true;
      allow delete: if false;
    }
  }
}
```

7. Deploy the site to Vercel.
8. Open the deployed site, edit the portfolio, click **Share link**, and enter a password twice.
9. Send the generated `?portfolio=...` link. The recipient enters the password to view it.

## Firebase Hosting authorization

If Firebase reports an `auth/unauthorized-domain` or Firestore request is blocked:

1. Open **Firebase Console > Authentication > Settings > Authorized domains**.
2. Add the Vercel domain, for example `portfolio-abyakta.vercel.app`.
3. Add the custom domain too, if one is configured.

## Security note

The portfolio payload is encrypted with AES-GCM before it is stored in Firestore. Firestore rules above allow public reads and writes, so anyone with the site URL and network access can replace documents that have valid document IDs. For stronger owner-only writes, add Firebase Authentication and require `request.auth != null` plus a configured owner UID.
