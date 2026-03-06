# Firebase Auth Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Gate the entire dashboard behind a shared-password Firebase Email/Password login screen.

**Architecture:** Add the Firebase Auth compat SDK, show a full-page Hebrew login screen on load, use `onAuthStateChanged` to toggle between login and dashboard, and add a logout button to the header. The email is hardcoded; the user only types the password. Firebase DB rules are tightened to `auth != null`.

**Tech Stack:** Vanilla JS + HTML in `index.html`. Firebase Auth compat SDK v10.7.1. Firebase Realtime Database rules updated manually in Firebase Console.

---

### Task 1: Add Firebase Auth SDK script tag

**Files:**
- Modify: `index.html:10` (after the firebase-database-compat script tag)

**Step 1: Add auth SDK**

After line 10 (`firebase-database-compat.js`), insert:

```html
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-auth-compat.js"></script>
```

**Step 2: Verify**
Open browser DevTools console, reload the page — no errors about `firebase.auth` being undefined.

**Step 3: Commit**
```bash
git add index.html
git commit -m "Add Firebase Auth compat SDK"
```

---

### Task 2: Add login screen HTML + CSS

**Files:**
- Modify: `index.html:751` (before `<body>` content) — add login overlay
- Modify: `index.html` `<style>` block — add login screen styles

**Step 1: Add login screen HTML**

Immediately after `<body>` (line 751), before `<header class="header">`, insert:

```html
<!-- Login Screen -->
<div id="login-screen" style="display:none">
    <div class="login-box">
        <img src="./logo.png" alt="Logo" class="login-logo">
        <h2>פלוגה ב' - מחלקה 3</h2>
        <form id="login-form" onsubmit="handleLogin(event)">
            <input type="password" id="login-password" placeholder="סיסמה" required autocomplete="current-password">
            <p id="login-error" class="login-error"></p>
            <button type="submit" id="login-btn">כניסה</button>
        </form>
    </div>
</div>
```

**Step 2: Add CSS for login screen**

Inside the `<style>` block, append before the closing `</style>`:

```css
#login-screen {
    position: fixed;
    inset: 0;
    background: linear-gradient(135deg, #2C97D1 0%, #1A5F85 100%);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 9999;
}

.login-box {
    background: white;
    border-radius: 16px;
    padding: 40px;
    width: 320px;
    text-align: center;
    box-shadow: 0 8px 32px rgba(0,0,0,0.2);
}

.login-logo {
    height: 70px;
    margin-bottom: 16px;
}

.login-box h2 {
    font-size: 20px;
    color: #1A5F85;
    margin-bottom: 24px;
    font-weight: 700;
}

.login-box input[type="password"] {
    width: 100%;
    padding: 12px 16px;
    border: 1px solid #ccc;
    border-radius: 8px;
    font-size: 16px;
    text-align: center;
    direction: ltr;
    margin-bottom: 8px;
    box-sizing: border-box;
}

.login-error {
    color: #e53e3e;
    font-size: 14px;
    min-height: 20px;
    margin-bottom: 8px;
}

.login-box button[type="submit"] {
    width: 100%;
    padding: 12px;
    background: linear-gradient(135deg, #2C97D1 0%, #1A5F85 100%);
    color: white;
    border: none;
    border-radius: 8px;
    font-size: 16px;
    font-weight: 600;
    cursor: pointer;
    transition: opacity 0.2s;
}

.login-box button[type="submit"]:disabled {
    opacity: 0.6;
    cursor: not-allowed;
}
```

**Step 3: Verify**
Reload — the login screen should NOT be visible yet (it starts with `display:none`; auth logic will show it in Task 4).

---

### Task 3: Add logout button to header

**Files:**
- Modify: `index.html:753-756` (header-content div)
- Modify: `index.html` `<style>` block

**Step 1: Add logout button to header HTML**

Replace the header-content div (lines 753–756):

```html
<div class="header-content">
    <img src="./logo.png" alt="Logo" class="logo">
    <h1>פלוגה ב' - מחלקה 3</h1>
    <button id="logout-btn" class="logout-btn" onclick="handleLogout()" style="display:none">יציאה</button>
</div>
```

**Step 2: Add CSS for logout button**

In the `<style>` block, append:

```css
.header-content {
    justify-content: space-between;
}

.logout-btn {
    padding: 8px 18px;
    background: rgba(255,255,255,0.2);
    color: white;
    border: 1px solid rgba(255,255,255,0.5);
    border-radius: 8px;
    font-size: 14px;
    font-weight: 600;
    cursor: pointer;
    transition: background 0.2s;
    margin-right: auto;
}

.logout-btn:hover {
    background: rgba(255,255,255,0.35);
}
```

**Step 3: Verify**
Reload — header looks unchanged (logout button hidden by default).

---

### Task 4: Implement auth logic (handleLogin, handleLogout, onAuthStateChanged)

**Files:**
- Modify: `index.html:1126-1143` (initFirebase function)
- Modify: `index.html:1202-1209` (DOMContentLoaded block)

**Step 1: Add SHARED_EMAIL constant and auth functions**

Immediately before `function initFirebase()` (line 1126), insert:

```js
const SHARED_EMAIL = 'team@fleet.com';

function showDashboard() {
    document.getElementById('login-screen').style.display = 'none';
    document.getElementById('logout-btn').style.display = '';
    document.querySelector('header.header').style.display = '';
    document.querySelector('nav.tabs').style.display = '';
    document.querySelector('main.container').style.display = '';
}

function showLoginScreen() {
    document.getElementById('login-screen').style.display = 'flex';
    document.getElementById('logout-btn').style.display = 'none';
    document.querySelector('header.header').style.display = 'none';
    document.querySelector('nav.tabs').style.display = 'none';
    document.querySelector('main.container').style.display = 'none';
}

function handleLogin(event) {
    event.preventDefault();
    const password = document.getElementById('login-password').value;
    const loginBtn = document.getElementById('login-btn');
    const loginError = document.getElementById('login-error');

    loginBtn.disabled = true;
    loginBtn.textContent = 'מתחבר...';
    loginError.textContent = '';

    firebase.auth().signInWithEmailAndPassword(SHARED_EMAIL, password)
        .catch(() => {
            loginError.textContent = 'סיסמה שגויה. נסה שנית.';
            loginBtn.disabled = false;
            loginBtn.textContent = 'כניסה';
        });
}

function handleLogout() {
    firebase.auth().signOut();
}
```

**Step 2: Replace initFirebase to use onAuthStateChanged**

Replace the existing `initFirebase()` function (lines 1126–1143) with:

```js
function initFirebase() {
    try {
        if (firebaseConfig.apiKey !== "YOUR_API_KEY") {
            firebase.initializeApp(firebaseConfig);
            database = firebase.database();
            firebaseEnabled = true;
            updateSyncStatus('מחובר', 'green');
            console.log('Firebase initialized successfully');

            firebase.auth().onAuthStateChanged(user => {
                if (user) {
                    showDashboard();
                    setupFirebaseListeners();
                } else {
                    showLoginScreen();
                }
            });
        } else {
            updateSyncStatus('לא מוגדר', 'orange');
            console.log('Firebase not configured - using localStorage only');
        }
    } catch (error) {
        console.error('Firebase initialization error:', error);
        updateSyncStatus('שגיאה', 'red');
    }
}
```

**Step 3: Hide dashboard elements initially via DOMContentLoaded**

Replace the DOMContentLoaded block (lines 1202–1209) with:

```js
document.addEventListener('DOMContentLoaded', function () {
    // Hide dashboard until auth confirmed
    document.querySelector('header.header').style.display = 'none';
    document.querySelector('nav.tabs').style.display = 'none';
    document.querySelector('main.container').style.display = 'none';

    initFirebase();
    renderCars();
    renderPeople();
    updateStats();
    setupTabs();
    setupSearchListeners();
});
```

**Step 4: Verify**
Reload — page shows the blue login screen. Enter a wrong password → Hebrew error appears. (Login with correct credentials in Task 5 after setup.)

**Step 5: Commit**
```bash
git add index.html
git commit -m "Add Firebase Auth login screen and session management"
```

---

### Task 5: One-time Firebase Console setup (manual)

**No code changes — manual steps in Firebase Console:**

1. Go to [https://console.firebase.google.com/](https://console.firebase.google.com/) → project **fleet-dashboard-e1c72**
2. Click **Authentication** in the left sidebar
3. Click **Get started** (if not enabled), then enable **Email/Password** provider
4. Go to the **Users** tab → **Add user**
5. Set email: `team@fleet.com`, set a strong shared password
6. Click **Realtime Database** → **Rules** tab, set:
```json
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null"
  }
}
```
7. Click **Publish**

**Verify:**
Reload dashboard → enter the shared password → dashboard loads and data syncs from Firebase.

---

### Task 6: Final commit

```bash
git add index.html docs/plans/2026-03-06-firebase-auth.md docs/plans/2026-03-06-auth-design.md
git commit -m "Add shared-password Firebase Auth to fleet dashboard"
```
