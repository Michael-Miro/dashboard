# Auth Design

**Goal:** Add a shared-password login screen using Firebase Email/Password Auth, gating the entire dashboard behind authentication.

## Login Screen
Full-page Hebrew RTL login screen matching the blue gradient header style. Contains a password field and "כניסה" button. The `<main>` dashboard element is hidden (`display:none`) until auth succeeds. Inline Hebrew error message on wrong credentials.

The email is hardcoded as a constant (`SHARED_EMAIL`) — the user only types the password.

## Firebase Auth
Add `firebase-auth-compat.js` SDK script tag alongside the existing Firebase scripts.

In `initFirebase()`, after `firebase.initializeApp()`:
- Call `firebase.auth().onAuthStateChanged(user => ...)`
- If `user` exists → hide login screen, show dashboard, proceed with existing Firebase listeners
- If `user` is null → show login screen, hide dashboard

Login button calls `firebase.auth().signInWithEmailAndPassword(SHARED_EMAIL, enteredPassword)`.

## Session Persistence
Firebase Auth defaults to `LOCAL` persistence — session survives page refreshes and browser restarts until explicit sign-out.

## Logout
Small "יציאה" button in the header (top-right). Calls `firebase.auth().signOut()`, which triggers `onAuthStateChanged` with `null`, bringing back the login screen.

## Firebase DB Rules
```json
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null"
  }
}
```

## One-Time Setup
Manually create the shared user in Firebase Console → Authentication → Users → Add user.
