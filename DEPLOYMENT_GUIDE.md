# MedHome Production Deployment Guide

## Overview
This guide provides detailed, step-by-step instructions to deploy MedHome as a public website on Vercel with Firebase backend integration.

---

## Phase 1: Firebase Setup (10-15 minutes)

### Step 1.1: Set Up Firebase Firestore Security Rules

Your Firestore database currently has no security rules, which means it's open to the world. Before deploying, configure proper security rules.

**Instructions:**
1. Go to https://console.firebase.google.com/
2. Select your `medhome-e2dd2` project
3. Click **Firestore Database** in the left sidebar
4. Click the **Rules** tab at the top
5. Replace the default rules with this security rule set:

```firestore
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users collection - users can only read/write their own documents
    match /users/{userId} {
      allow read, write: if request.auth.uid == userId;
    }
    
    // Medicines collection - users can access medicines in their household
    match /medicines/{medicineId} {
      allow read, write: if request.auth != null && 
        resource.data.household == get(/databases/$(database)/documents/users/$(request.auth.uid)).data.household;
      allow create: if request.auth != null && 
        request.resource.data.household == get(/databases/$(database)/documents/users/$(request.auth.uid)).data.household;
    }
    
    // Members collection - same household access
    match /members/{memberId} {
      allow read, write: if request.auth != null && 
        resource.data.household == get(/databases/$(database)/documents/users/$(request.auth.uid)).data.household;
      allow create: if request.auth != null && 
        request.resource.data.household == get(/databases/$(database)/documents/users/$(request.auth.uid)).data.household;
    }
    
    // Reminders collection - same household access
    match /reminders/{reminderId} {
      allow read, write: if request.auth != null && 
        resource.data.household == get(/databases/$(database)/documents/users/$(request.auth.uid)).data.household;
      allow create: if request.auth != null && 
        request.resource.data.household == get(/databases/$(database)/documents/users/$(request.auth.uid)).data.household;
    }
    
    // Caregivers collection - same household access
    match /caregivers/{caregiverId} {
      allow read, write: if request.auth != null && 
        resource.data.household == get(/databases/$(database)/documents/users/$(request.auth.uid)).data.household;
      allow create: if request.auth != null && 
        request.resource.data.household == get(/databases/$(database)/documents/users/$(request.auth.uid)).data.household;
    }
  }
}
```

6. Click **Publish** to save and activate the rules

### Step 1.2: Enable Google Sign-In

1. In Firebase Console, go to **Authentication** in the left sidebar
2. Click the **Sign-in method** tab
3. Click **Google** to enable it
4. Select a project support email and click **Save**

### Step 1.3: Add Your Production Domain to OAuth Authorized Domains

1. In **Authentication** → **Settings** tab
2. Scroll down to **Authorized domains**
3. Your Vercel domain will be added automatically after first deployment, but you can pre-add it:
   - Add `medhome.vercel.app` (or your custom domain)
   - Add `localhost:3000` for local testing

---

## Phase 2: Vercel Deployment Setup (5-10 minutes)

### Step 2.1: Connect GitHub Repository to Vercel

1. Go to https://vercel.com/new
2. Click **Continue with GitHub** and authorize Vercel
3. Search for and select the `Samikshadhanya/medhome` repository
4. Click **Import**

### Step 2.2: Configure Environment Variables

On the Vercel import screen, you'll see an "Environment Variables" section:

Add these variables (copy from your Firebase Console):

```
NEXT_PUBLIC_FIREBASE_API_KEY = [Your Firebase API Key]
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN = medhome-e2dd2.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID = medhome-e2dd2
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET = medhome-e2dd2.appspot.com
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID = [Your Firebase Messaging Sender ID]
NEXT_PUBLIC_FIREBASE_APP_ID = [Your Firebase App ID]
```

**Where to find these values:**
- Go to Firebase Console → Project Settings → Web Apps
- Click your app and copy the firebaseConfig object
- Map each field to the corresponding environment variable above

### Step 2.3: Complete Deployment

1. Click **Deploy**
2. Wait for the deployment to complete (2-3 minutes)
3. You'll get a production URL like: `https://medhome.vercel.app`

---

## Phase 3: Post-Deployment Testing (10-15 minutes)

### Step 3.1: Test Authentication Flow

1. Go to your Vercel URL
2. Click **Sign in with Google**
3. Complete the Google authentication flow
4. Verify you're signed in and can see the dashboard

### Step 3.2: Test Data Persistence

1. Click **Inventory**
2. Click **Add New Medicine**
3. Fill in the form with test data
4. Click **Save**
5. Wait 2-3 seconds
6. Refresh the page (F5)
7. **Expected result:** The medicine should still be there (saved in Firestore)

### Step 3.3: Test Guest Mode (Optional - for testing without login)

For development/testing, the app has a guest mode:
- In the login page, there's a "Continue as Guest" option
- This allows testing without Google Sign-In
- Data will be stored locally and NOT synced to Firestore
- **Don't rely on this for production**

---

## Phase 4: Custom Domain Setup (Optional - 10 minutes)

If you want to use a custom domain instead of `medhome.vercel.app`:

### Step 4.1: Add Domain in Vercel

1. Go to Vercel Dashboard → Your Project → Settings → Domains
2. Click **Add**
3. Enter your custom domain (e.g., `medhome.com`)
4. Follow the DNS configuration instructions for your domain registrar
5. Typically you'll add:
   - A record pointing to Vercel's IP, OR
   - CNAME record pointing to your Vercel domain

### Step 4.2: Update Firebase Authorized Domains

1. Go to Firebase Console → Authentication → Settings
2. Add your custom domain to **Authorized domains**

---

## Phase 5: Monitoring & Maintenance (Ongoing)

### Monitor Firestore Usage

1. Firebase Console → Firestore → Stats
2. Monitor read/write operations and storage
3. Firestore has a free tier: 50k reads, 20k writes, 20k deletes per day
4. For a family app, you'll likely stay well within the free tier

### Enable Firestore Backups (Optional)

1. Firebase Console → Firestore Database
2. Click **Backups** (if available in your region)
3. Set up automatic daily backups

### Monitor Performance

1. Vercel Dashboard → Monitoring → Analytics
2. Check response times and error rates
3. Vercel provides real-time analytics for free

---

## Troubleshooting

### Issue: "Authentication Error" on Vercel

**Solution:**
- Verify all 6 environment variables are set correctly in Vercel
- Ensure the domain is added to Firebase Authorized Domains
- Clear browser cookies and try again

### Issue: Data Not Saving to Firestore

**Solution:**
1. Check Firestore security rules are published (should show green checkmark)
2. Verify user is properly authenticated (not in guest mode)
3. Check browser console for errors (F12 → Console tab)
4. Verify your Firestore database location matches the rules

### Issue: "Permission denied" errors in Firestore

**Solution:**
- Check that security rules are properly deployed
- Verify the `household` field exists on all documents
- Ensure user is authenticated with Firebase Auth

---

## Deployment Checklist

- [ ] Firebase Firestore security rules deployed
- [ ] Google Sign-In enabled in Firebase
- [ ] Production domain added to Firebase Authorized Domains
- [ ] GitHub repository connected to Vercel
- [ ] All 6 Firebase environment variables added to Vercel
- [ ] Initial deployment completed successfully
- [ ] Can log in with Google on production
- [ ] Can add/update/delete data and it persists after refresh
- [ ] Browser console shows no critical errors
- [ ] Firestore shows new collections being created as data is added

---

## Next Steps After Deployment

1. **Share with Family:** Send the production URL to family members
2. **Onboard Users:** Have them sign up with Google accounts
3. **Populate Initial Data:** Add family members and medicines
4. **Set Up Reminders:** Configure daily medication reminders
5. **Test Full Workflow:** Verify the complete feature set works
6. **Gather Feedback:** Collect feedback from users and iterate

---

## Costs

- **Vercel:** Free tier (generous limits for small apps)
- **Firebase:**
  - Authentication: Free
  - Firestore: Free tier (50k reads/day, 20k writes/day)
  - Storage: Free tier (1GB)
  - Total estimated cost for a family app: **$0/month** (free tier)

