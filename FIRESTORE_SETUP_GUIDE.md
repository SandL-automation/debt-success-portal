# Firebase Firestore Setup & Migration Guide

## Overview

This guide will help you set up Firebase Firestore as the primary database for the Debt Success Client Portal and migrate existing contacts from Pipedrive.

---

## 📋 Prerequisites

- Firebase project already created: `debtsuccessportal`
- Access to Pipedrive account (to export contacts)
- Access to Zapier Tables (during transition period)

---

## 🔥 Step 1: Enable Firestore in Firebase Console

### 1.1 Navigate to Firestore

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Select your project: **debtsuccessportal**
3. In the left sidebar, click **Firestore Database**
4. Click **Create database**

### 1.2 Configure Firestore

1. **Select location:**
   - Choose: **europe-west2 (London)** or closest to South Africa
   - Note: This cannot be changed later

2. **Start in production mode:**
   - Select: **Start in production mode**
   - We'll configure security rules next

3. Click **Create**

### 1.3 Set Security Rules

1. Go to **Firestore Database → Rules** tab
2. Replace the default rules with:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Users collection - users can only read/write their own document
    match /users/{email} {
      allow read, write: if request.auth != null && request.auth.token.email == email;
    }

    // Admin-only collections (for future use)
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

3. Click **Publish**

**What these rules do:**
- Users can ONLY access their own document in the `users` collection
- Document ID must match their authenticated email address
- All other collections are locked by default (for future admin features)

---

## 📊 Step 2: Design Firestore Data Structure

### Collection: `users`

**Document ID:** User's email address (e.g., `stiaan@saltandlightcreations.co.za`)

**Document Structure:**

```javascript
{
  // Basic Information
  "fullName": "Stiaan van Zyl",
  "email": "stiaan@saltandlightcreations.co.za",
  "idNumber": "8001015009088",
  "phone": "+27823722425",
  "address": "123 Main Street, Pretoria, Gauteng, 0001",
  "maritalStatus": "Married in Community of Property",

  // Spouse Information (if applicable)
  "spouseName": "Jane Doe",
  "spouseSurname": "van Zyl",
  "spouseRelation": "Spouse",
  "spousePhone": "+27821234567",

  // Financial Wellness Programme (FWP)
  "fwpEnrolled": false,
  "fwpEnrollmentDate": null,
  "fwpPaymentConsent": false,
  "fwpBillingConsent": false,
  "fwpPreferredContact": null,
  "fwpEmailsStatus": "Not Subscribed",

  // Metadata
  "createdAt": "2026-02-05T10:30:00Z",
  "lastUpdated": "2026-02-05T12:45:00Z",
  "source": "Client Portal"
}
```

---

## 📤 Step 3: Export Contacts from Pipedrive

### 3.1 Export Pipedrive Contacts

1. Log in to [Pipedrive](https://www.pipedrive.com/)
2. Go to **Contacts → People**
3. Click **Export**
4. Select **All fields**
5. Choose format: **CSV (UTF-8)**
6. Click **Export**
7. Save file as: `pipedrive_contacts_export.csv`

### 3.2 Review Export File

Open the CSV file and verify these critical fields exist:
- Email (primary identifier)
- Name
- Phone
- Any custom fields (IN_ prefix, FWP_ prefix, etc.)

---

## 🔄 Step 4: Import Pipedrive Contacts to Firestore

### Option A: Manual Import (Small Dataset - Recommended for ~100 contacts)

Use the Firebase Console:

1. Go to **Firestore Database**
2. Click **Start collection**
3. Collection ID: `users`
4. For each contact from your CSV:
   - Click **Add document**
   - Document ID: Contact's email address
   - Add fields manually from CSV data
   - Click **Save**

**Best for:** Testing with 5-10 contacts first

### Option B: Automated Import Script (Large Dataset)

If you have many contacts, I can create a Node.js script that:
1. Reads the Pipedrive CSV export
2. Transforms data to match Firestore structure
3. Bulk imports all contacts to Firestore

**Requirements:**
- Node.js installed
- Firebase Admin SDK credentials

Let me know if you'd like me to create this script!

---

## ✅ Step 5: Test Firestore Integration

### 5.1 Create Test User in Firestore

1. Go to **Firestore Database**
2. Click **Start collection** (if not created yet) or open `users` collection
3. **Add document:**
   - Document ID: `stiaan@saltandlightcreations.co.za`
   - Fields:
     ```
     fullName: "Stiaan van Zyl"
     email: "stiaan@saltandlightcreations.co.za"
     idNumber: "8001015009088"
     phone: "+27823722425"
     address: "123 Main Street, Pretoria, Gauteng, 0001"
     maritalStatus: "Married in Community of Property"
     lastUpdated: "2026-02-05T12:00:00Z"
     ```
4. Click **Save**

### 5.2 Test Edit Profile Page

1. Deploy updated `edit-profile.html` to GitHub Pages
2. Log in to portal with: `stiaan@saltandlightcreations.co.za`
3. Navigate to **Edit Profile** page
4. Verify fields are pre-filled from Firestore
5. Change phone number
6. Click **Save Changes**
7. Verify data updates in Firestore Console

### 5.3 Verify Hybrid Approach Works

After saving profile:
1. ✅ Check Firestore: Data should be updated
2. ✅ Check Zapier Tables: Data should be updated (via webhook)
3. ✅ Check Pipedrive: Contact should be updated (via Zapier workflow)

---

## 🔀 Step 6: Transition Plan

### Phase 1: Hybrid Mode (NOW - 1 Week After Launch)

**Status:** Writing to BOTH Firestore and Zapier Tables

- ✅ Edit Profile form: Saves to Firestore + Zapier webhook
- ✅ Upgrade form: Saves to Firestore + Zapier webhook
- ✅ Intake form (Fillout): Still uses Zapier Tables
- ⚠️ Keep Zapier Tables active (Fillout forms still in use)

### Phase 2: Portal Launch

**Timeline:** When portal is ready for Sulette & Sammy

- Show Sulette the new portal
- Train on Edit Profile and Upgrade form
- Fillout forms still active (writing to Zapier Tables)

### Phase 3: Zapier Tables Phase-Out (1 Week After Launch)

**Timeline:** 1 week after Sulette starts using portal

1. ✅ Verify all portal forms working correctly
2. ✅ Confirm Firestore has all active contacts
3. 🔄 Migrate any remaining Zapier Tables records to Firestore
4. ❌ Disable Fillout forms (redirect to portal)
5. ❌ Archive Zapier Tables (keep as backup, don't delete immediately)
6. ✅ Remove Zapier webhook calls from portal forms (keep only Firestore)

---

## 🚨 Important Reminders

### During Transition Period:

1. **DO NOT delete Zapier Tables yet** - Fillout forms are still in use
2. **Keep webhook calls active** - Ensures Pipedrive stays updated
3. **Monitor both systems** - Firestore AND Zapier Tables should match
4. **Test thoroughly** - Verify data flows to all systems correctly

### After Transition Complete:

1. **Remove webhook calls** from portal forms (only use Firestore)
2. **Archive Zapier Tables** (don't delete immediately - keep as backup)
3. **Update Zaps** to read from Firestore instead of Zapier Tables (if needed)

---

## 📝 Firestore vs Zapier Tables Comparison

| Feature | Firestore | Zapier Tables |
|---------|-----------|---------------|
| **Cost** | Free tier: 50K reads/day | Paid: Starts at $20/month |
| **Speed** | Real-time, instant reads | API calls, slower |
| **Security** | Firebase Auth integrated | API key based |
| **Scalability** | Unlimited (with pricing) | Limited by Zapier plan |
| **Query Power** | Advanced queries, indexes | Basic search only |
| **Form Pre-fill** | ✅ Yes (instant) | ❌ No (webhook can't return data) |

---

## 🎯 Next Steps

1. ✅ **Enable Firestore** in Firebase Console
2. ✅ **Set security rules** as shown above
3. ✅ **Create test user** in Firestore
4. ✅ **Deploy updated `edit-profile.html`** to GitHub Pages
5. ✅ **Test form pre-fill** and save functionality
6. 📤 **Export Pipedrive contacts** (CSV)
7. 📥 **Import contacts to Firestore** (manual or script)
8. ✅ **Update other forms** to use hybrid approach

---

## 🆘 Troubleshooting

### Form fields not pre-filling

**Check:**
1. Firestore document ID matches user's email exactly
2. Browser console for errors (F12 → Console)
3. Security rules allow read access for authenticated user
4. Document exists in Firestore with correct field names

### Data not saving to Firestore

**Check:**
1. User is authenticated (check `auth.currentUser`)
2. Browser console for error messages
3. Security rules allow write access for authenticated user
4. Network tab shows successful Firestore request

### Zapier webhook not receiving data

**Check:**
1. Webhook URL is correct: `https://hooks.zapier.com/hooks/catch/18655142/uljyyxy/`
2. FormData includes all required fields
3. Network tab shows successful POST request
4. Zapier webhook history shows received data

---

**Last Updated:** 2026-02-05
**Version:** 1.0
**Status:** ✅ Ready for implementation
