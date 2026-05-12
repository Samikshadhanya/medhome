# Quick Troubleshooting Checklist

## Before You Start: Verify Basics

Run this checklist if data isn't persisting or Google Sign-In doesn't work.

---

## Checklist 1: Firestore Rules

- [ ] Go to Firebase Console → Firestore → Rules tab
- [ ] Rules should show "Today" timestamp (not older)
- [ ] Do you see this function?
  ```
  function isAuthenticated() {
    return request.auth != null;
  }
  ```
- [ ] Does each collection have this pattern?
  ```
  allow read, write: if isAuthenticated();
  ```
- [ ] Click **Publish** if not already published
- [ ] Wait for "Rules updated successfully" message

**If Rules are wrong:**
1. Delete all code
2. Copy from `firestore.rules` file in your project
3. Paste into Firebase Console Rules editor
4. Click Publish

---

## Checklist 2: Firebase Credentials

- [ ] Open project root
- [ ] Do you have `.env.local` file?
  - If NO: Create it with all 6 Firebase variables (see guide Step 3.2)
  - If YES: Continue

- [ ] Does `.env.local` contain these variables?
  ```
  NEXT_PUBLIC_FIREBASE_API_KEY=
  NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=
  NEXT_PUBLIC_FIREBASE_PROJECT_ID=
  NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=
  NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=
  NEXT_PUBLIC_FIREBASE_APP_ID=
  ```
  - If NO: Add them from Firebase Console (see guide Step 3.1)
  - If YES: Continue

- [ ] Have you restarted dev server after adding `.env.local`?
  - If NO: Run `npm run dev` again
  - If YES: Continue

---

## Checklist 3: Google Sign-In

- [ ] Go to Firebase Console → Authentication → Sign-in method
- [ ] Is **Google** enabled?
  - If NO: Click Google, toggle Enable, click Save
  - If YES: Continue

- [ ] Scroll to **Authorized domains**
- [ ] Does it list your domain? (e.g., localhost:3000)
  - If NO: Add it (see guide Step 2.2)
  - If YES: Continue

- [ ] Open your app: `http://localhost:3000`
- [ ] Click **Sign in with Google**
- [ ] Does Google login popup appear?
  - If NO: Check browser console (F12 → Console), note any errors, restart dev server
  - If YES: Continue

- [ ] After logging in, are you in the dashboard?
  - If NO: Check console for errors
  - If YES: You're authenticated! Proceed to test data persistence.

---

## Checklist 4: Data Persistence Test

### Test 1: Local State (Should disappear on refresh)
1. Click **Continue as Guest**
2. Go to Inventory
3. Add a medicine (any details)
4. Refresh page (F5)
5. Is the medicine gone?
   - If YES: ✅ Correct (guest mode doesn't use Firestore)
   - If NO: Might be using localStorage, not Firestore

### Test 2: Firestore Persistence (Should persist on refresh)
1. Sign out (click Settings → Log out)
2. Click **Sign in with Google**
3. Complete login
4. Go to Inventory
5. Add a different medicine
6. **Check Firestore Console:**
   - Go to Firebase Console → Firestore Database
   - Click **medicines** collection
   - Do you see your medicine listed?
   - If NO: Rules are blocking writes (go back to Checklist 1)
   - If YES: Continue

7. Refresh page (F5)
8. Is the medicine still visible in the app?
   - If YES: ✅ Firestore persistence working!
   - If NO: Real-time listener issue (see Advanced Troubleshooting)

### Test 3: Multi-Device Sync (Real persistence proof)
1. Keep app open in current browser
2. Open new private/incognito window
3. Go to `http://localhost:3000`
4. Sign in with **same Google account**
5. Go to Inventory
6. Do you see the medicine you added in the other browser?
   - If YES: ✅ Backend sync working perfectly!
   - If NO: Rules are restricting based on household (expected if not set up)

---

## Checklist 5: Browser Console Errors

Open browser developer tools: **F12 → Console tab**

Look for any red error messages. Common ones:

### Error: "Missing or insufficient permissions"
- Rules aren't updated or user isn't authenticated
- **Fix:** Go to Checklist 1 and 3

### Error: "Firebase: Error (auth/operation-not-supported-in-this-environment)"
- Firebase app not initialized properly
- **Fix:** Check `.env.local` has all variables, restart dev server

### Error: "Cannot read property 'data' of undefined"
- Real-time listener issue
- **Fix:** Check Firebase Console to ensure collections exist

### Error: "CORS error" or "network error"
- Firestore unreachable
- **Fix:** Check internet connection, verify authorized domains

### No errors but data doesn't appear?
- Add this to browser console to test:
  ```javascript
  firebase.firestore().collection("medicines").get()
    .then(snap => console.log("Medicines:", snap.docs.length))
    .catch(err => console.log("Error:", err.message))
  ```
- If you see "Medicines: 0" = empty collection (add data)
- If you see an error = Firestore rules or connection issue

---

## Checklist 6: Network Activity

Open browser developer tools: **F12 → Network tab**

1. Clear all requests (trash icon)
2. Go to Inventory
3. Add a medicine
4. Look at Network tab
5. Do you see requests to `firestore.googleapis.com`?
   - If NO: Firestore not being called (check code)
   - If YES: Check response status
     - 200: ✅ Success
     - 403: Forbidden (rules blocking)
     - 500: Server error (Firebase issue)

---

## Checklist 7: Reset and Clean Test

If everything seems broken, try a complete reset:

1. Delete `.env.local`
2. Close dev server (Ctrl+C)
3. Clear browser cache: F12 → Application → Clear site data
4. Clear `node_modules` and reinstall:
   ```bash
   rm -rf node_modules
   npm install
   ```
5. Create fresh `.env.local` with Firebase variables
6. Start dev server: `npm run dev`
7. Test again from Checklist 3

---

## Still Not Working?

Create a support ticket with:
1. Screenshot of Firebase Console Rules tab (timestamp visible)
2. Screenshot of browser console (F12)
3. Screenshot of Firestore Database collections
4. Error message you're seeing (if any)
5. What you did and what you expected to happen

This helps diagnose the issue quickly.
