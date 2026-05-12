# Complete Firestore Setup & Backend Integration Guide

## Overview
This guide walks you through setting up Firestore rules, enabling Google Sign-In, and testing the full frontend-backend pipeline.

---

## Phase 1: Deploy Firestore Security Rules

### Step 1.1: Access Firestore Rules
1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Select your project: **medhome**
3. In the left sidebar, click **Firestore Database**
4. Click the **Rules** tab at the top
5. You should see the current rules with the timestamp "Today • 12:13 AM"

### Step 1.2: Replace the Rules
1. Delete ALL existing code in the rules editor
2. Copy the code from `firestore.rules` in your project:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       function isAuthenticated() {
         return request.auth != null;
       }

       match /users/{userId} {
         allow read, write: if isAuthenticated() && request.auth.uid == userId;
       }

       match /medicines/{medicineId} {
         allow read, write: if isAuthenticated();
       }

       match /members/{memberId} {
         allow read, write: if isAuthenticated();
       }

       match /reminders/{reminderId} {
         allow read, write: if isAuthenticated();
       }

       match /caregivers/{caregiverId} {
         allow read, write: if isAuthenticated();
       }
     }
   }
   ```

3. Click **Publish** button (blue button on the right)
4. Wait for "Rules updated successfully" message
5. ✅ Rules are now live

**Key Changes:**
- `isAuthenticated()` function checks if user is logged in
- Guest mode is excluded (guests can't write to Firestore)
- Google-authenticated users can read/write all collections
- Only authenticated users from your app can access data (security)

---

## Phase 2: Verify Google Sign-In Setup

### Step 2.1: Check Authentication Providers
1. In Firebase Console, go to **Authentication**
2. Click **Sign-in method** tab
3. Verify **Google** is enabled (should show a green checkmark)
4. If not enabled:
   - Click **Google**
   - Toggle **Enable** on
   - Select a project support email
   - Click **Save**

### Step 2.2: Add Your Domains
1. Still in Authentication → Sign-in method
2. Scroll down to **Authorized domains**
3. Add these domains:
   - `localhost:3000` (for local testing)
   - `localhost:3001` (for alternative port)
   - Your Vercel production domain (e.g., `medhome.vercel.app`)

---

## Phase 3: Environment Variables Setup

### Step 3.1: Get Firebase Credentials
1. In Firebase Console, go to **Project Settings** (gear icon)
2. Click **Your apps** section
3. Find your web app (or create one if missing)
4. Copy these 6 values:
   - **apiKey** → `NEXT_PUBLIC_FIREBASE_API_KEY`
   - **authDomain** → `NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN`
   - **projectId** → `NEXT_PUBLIC_FIREBASE_PROJECT_ID`
   - **storageBucket** → `NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET`
   - **messagingSenderId** → `NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID`
   - **appId** → `NEXT_PUBLIC_FIREBASE_APP_ID`

### Step 3.2: Set Local Environment Variables
1. Create `.env.local` in project root:
   ```bash
   NEXT_PUBLIC_FIREBASE_API_KEY=AIza...
   NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=medhome-e2dd2.firebaseapp.com
   NEXT_PUBLIC_FIREBASE_PROJECT_ID=medhome-e2dd2
   NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=medhome-e2dd2.appspot.com
   NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=123...
   NEXT_PUBLIC_FIREBASE_APP_ID=1:123...
   ```

2. Save file and **restart dev server**:
   ```bash
   npm run dev
   ```

---

## Phase 4: Test the Backend Integration

### Step 4.1: Test with Guest Mode (Local Only)
1. Open app: `http://localhost:3000`
2. Click **Continue as Guest**
3. You'll see the dashboard with dummy data
4. **Try adding a medicine:**
   - Go to Inventory
   - Click **Add Medicine**
   - Fill in details
   - Click **Save**
5. **Check if it persists:**
   - Refresh the page (F5)
   - If data is gone → Firestore write failed (expected for guest mode)
   - If data stays → Success! (unexpected, guest mode shouldn't persist)

### Step 4.2: Test with Google Sign-In (Real Persistence)
1. Open app: `http://localhost:3000`
2. Click **Sign in with Google**
3. Complete Google login
4. You're now in authenticated mode
5. **Add a test medicine:**
   - Go to Inventory → Add Medicine
   - Fill: Name: "Aspirin", Strength: "500mg", etc.
   - Click **Save**
6. **Verify in Firebase Console:**
   - Go to Firestore Database
   - Click **medicines** collection
   - You should see your new medicine document
7. **Test persistence:**
   - Refresh page
   - Medicine should still be there (from Firestore, not local state)
8. **Test from another browser/incognito:**
   - Open new private window
   - Go to `http://localhost:3000`
   - Sign in with same Google account
   - Go to Inventory
   - Your medicine should be visible (proving backend sync works)

### Step 4.3: Verify Firestore Operations

Check these actions work and persist to Firestore:

| Action | Expected Result |
|--------|-----------------|
| Add Medicine | Appears in Firestore medicines collection |
| Edit Medicine | Changes reflected in Firestore |
| Delete Medicine | Removed from Firestore |
| Add Family Member | Appears in Firestore members collection |
| Create Reminder | Appears in Firestore reminders collection |
| Mark Reminder as Taken | Status updated in Firestore |
| Refresh Page | All data loads from Firestore |
| Different Browser | Same data visible (synced via backend) |

---

## Phase 5: Troubleshooting

### Issue: "Missing or insufficient permissions"
**Cause:** Firestore rules aren't updated or user isn't authenticated
**Fix:**
1. Verify rules are published (check Firebase Console Rules tab)
2. Ensure user is signed in with Google (not guest mode)
3. Check browser console for errors: F12 → Console tab
4. Restart dev server: `npm run dev`

### Issue: Data shows while guest, but disappears on refresh
**This is normal.** Guest mode uses local state only, not Firestore.
**Solution:** Test with Google Sign-In instead.

### Issue: Google Sign-In button doesn't work
**Cause:** Firebase credentials not loaded
**Fix:**
1. Verify `.env.local` has all 6 Firebase variables
2. Restart dev server: `npm run dev`
3. Check browser console (F12) for errors

### Issue: Can see data in Firebase Console, but app shows empty
**Cause:** Real-time listener not working
**Fix:**
1. Check network tab (F12 → Network)
2. Verify no CORS errors
3. Check that Firestore rules allow authenticated reads
4. Restart app

---

## Phase 6: Deployment Checklist

Before deploying to Vercel:

- [ ] Firestore rules published (Step 1.2)
- [ ] Google Sign-In enabled (Step 2.1)
- [ ] Authorized domains include your Vercel domain (Step 2.2)
- [ ] All 6 Firebase env vars in `.env.local` (Step 3.2)
- [ ] Tested with Google Sign-In locally (Step 4.2)
- [ ] Data persists on page refresh (Step 4.3)
- [ ] Can see data in Firestore Console

### Deployment Steps:
1. Push to GitHub
2. Go to Vercel Dashboard
3. Add environment variables:
   - Same 6 Firebase variables
   - Set `NEXT_PUBLIC_APP_URL="https://medhome.vercel.app"` (update with your domain)
4. Deploy
5. Test on production URL with Google Sign-In
6. Verify data syncs in real-time

---

## Phase 7: Production Monitoring

Once deployed:
1. Monitor Firestore usage: Firebase Console → Firestore Stats
   - Free tier: 50k reads, 20k writes, 1GB storage per day
2. Set up billing alerts (optional)
3. Monitor authentication: Firebase Console → Authentication → Users
4. Check Cloud Logging for errors

---

## Success Metrics

Your backend is working flawlessly when:
1. ✅ Google Sign-In works
2. ✅ Add/edit/delete operations appear in Firestore within seconds
3. ✅ Data persists after page refresh
4. ✅ Same data visible across different browsers (when logged in with same account)
5. ✅ Zero "permission denied" errors
6. ✅ Real-time listeners sync changes instantly

If all 6 metrics pass, you're ready for production deployment.
