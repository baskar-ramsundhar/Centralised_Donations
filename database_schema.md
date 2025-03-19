# Database Schema Design

## Overview

This document outlines the database schema for the donation platform. We'll be using MongoDB as our database, which provides flexibility for document-based data structures.

## Collections

### 1. Users

```json
{
  "_id": "ObjectId",
  "email": "string",
  "passwordHash": "string",
  "firstName": "string",
  "lastName": "string",
  "role": "enum['donor', 'organization_admin', 'platform_admin']",
  "profileImage": "string (URL)",
  "phoneNumber": "string",
  "address": {
    "street": "string",
    "city": "string",
    "state": "string",
    "postalCode": "string",
    "country": "string"
  },
  "preferences": {
    "emailNotifications": "boolean",
    "newsletterSubscribed": "boolean",
    "donationReceipts": "boolean"
  },
  "stripeCustomerId": "string",
  "savedPaymentMethods": [
    {
      "id": "string",
      "type": "string",
      "last4": "string",
      "expiryMonth": "number",
      "expiryYear": "number",
      "isDefault": "boolean"
    }
  ],
  "createdAt": "date",
  "updatedAt": "date",
  "lastLoginAt": "date"
}
```

### 2. Organizations

```json
{
  "_id": "ObjectId",
  "name": "string",
  "slug": "string",
  "description": "string",
  "mission": "string",
  "logo": "string (URL)",
  "coverImage": "string (URL)",
  "website": "string (URL)",
  "email": "string",
  "phoneNumber": "string",
  "address": {
    "street": "string",
    "city": "string",
    "state": "string",
    "postalCode": "string",
    "country": "string"
  },
  "socialMedia": {
    "facebook": "string (URL)",
    "twitter": "string (URL)",
    "instagram": "string (URL)",
    "linkedin": "string (URL)"
  },
  "taxId": "string",
  "verificationStatus": "enum['pending', 'verified', 'rejected']",
  "verificationDocuments": [
    {
      "type": "string",
      "url": "string",
      "uploadedAt": "date"
    }
  ],
  "stripeConnectedAccountId": "string",
  "payoutSchedule": "enum['daily', 'weekly', 'monthly']",
  "categories": ["ObjectId (ref: Categories)"],
  "featuredRank": "number",
  "metrics": {
    "totalDonations": "number",
    "totalDonors": "number",
    "recurringDonors": "number"
  },
  "admins": ["ObjectId (ref: Users)"],
  "createdAt": "date",
  "updatedAt": "date"
}
```

### 3. Categories

```json
{
  "_id": "ObjectId",
  "name": "string",
  "slug": "string",
  "description": "string",
  "icon": "string",
  "parentCategory": "ObjectId (ref: Categories)",
  "isActive": "boolean",
  "displayOrder": "number",
  "createdAt": "date",
  "updatedAt": "date"
}
```

### 4. Donations

```json
{
  "_id": "ObjectId",
  "donor": "ObjectId (ref: Users)",
  "organization": "ObjectId (ref: Organizations)",
  "amount": "number",
  "currency": "string",
  "status": "enum['pending', 'completed', 'failed', 'refunded']",
  "paymentMethod": {
    "type": "string",
    "last4": "string"
  },
  "stripePaymentIntentId": "string",
  "stripeTransferId": "string",
  "platformFee": "number",
  "stripeFee": "number",
  "coverFees": "boolean",
  "isAnonymous": "boolean",
  "isRecurring": "boolean",
  "subscriptionId": "ObjectId (ref: Subscriptions)",
  "note": "string",
  "receiptSent": "boolean",
  "receiptUrl": "string",
  "metadata": "object",
  "createdAt": "date",
  "updatedAt": "date"
}
```

### 5. Subscriptions

```json
{
  "_id": "ObjectId",
  "donor": "ObjectId (ref: Users)",
  "organization": "ObjectId (ref: Organizations)",
  "status": "enum['active', 'paused', 'canceled', 'failed']",
  "amount": "number",
  "currency": "string",
  "interval": "enum['monthly', 'quarterly', 'annually']",
  "startDate": "date",
  "nextBillingDate": "date",
  "canceledAt": "date",
  "stripeSubscriptionId": "string",
  "paymentMethod": {
    "type": "string",
    "last4": "string"
  },
  "platformFee": "number",
  "stripeFee": "number",
  "coverFees": "boolean",
  "isAnonymous": "boolean",
  "failedPayments": "number",
  "metadata": "object",
  "createdAt": "date",
  "updatedAt": "date"
}
```

### 6. Payouts

```json
{
  "_id": "ObjectId",
  "organization": "ObjectId (ref: Organizations)",
  "amount": "number",
  "currency": "string",
  "status": "enum['pending', 'completed', 'failed']",
  "stripePayoutId": "string",
  "donations": ["ObjectId (ref: Donations)"],
  "periodStart": "date",
  "periodEnd": "date",
  "metadata": "object",
  "createdAt": "date",
  "updatedAt": "date"
}
```

### 7. Campaigns

```json
{
  "_id": "ObjectId",
  "organization": "ObjectId (ref: Organizations)",
  "name": "string",
  "slug": "string",
  "description": "string",
  "goal": "number",
  "currency": "string",
  "startDate": "date",
  "endDate": "date",
  "isActive": "boolean",
  "coverImage": "string (URL)",
  "media": [
    {
      "type": "enum['image', 'video']",
      "url": "string",
      "caption": "string"
    }
  ],
  "metrics": {
    "totalRaised": "number",
    "totalDonors": "number",
    "percentComplete": "number"
  },
  "createdAt": "date",
  "updatedAt": "date"
}
```

### 8. Webhooks

```json
{
  "_id": "ObjectId",
  "source": "enum['stripe', 'system', 'external']",
  "event": "string",
  "payload": "object",
  "processed": "boolean",
  "processingErrors": ["string"],
  "createdAt": "date",
  "processedAt": "date"
}
```

### 9. ActivityLogs

```json
{
  "_id": "ObjectId",
  "user": "ObjectId (ref: Users)",
  "action": "string",
  "entity": {
    "type": "string",
    "id": "ObjectId"
  },
  "details": "object",
  "ipAddress": "string",
  "userAgent": "string",
  "createdAt": "date"
}
```

## Relationships

1. **Users to Donations**: One-to-many (a user can make multiple donations)
2. **Users to Subscriptions**: One-to-many (a user can have multiple recurring donations)
3. **Organizations to Donations**: One-to-many (an organization can receive multiple donations)
4. **Organizations to Campaigns**: One-to-many (an organization can have multiple campaigns)
5. **Organizations to Categories**: Many-to-many (an organization can belong to multiple categories)
6. **Organizations to Users**: Many-to-many (an organization can have multiple admin users)
7. **Subscriptions to Donations**: One-to-many (a subscription generates multiple donations over time)
8. **Organizations to Payouts**: One-to-many (an organization receives multiple payouts)

## Indexes

1. Users: email (unique), stripeCustomerId
2. Organizations: slug (unique), name, categories, verificationStatus
3. Donations: donor, organization, createdAt, status
4. Subscriptions: donor, organization, status, nextBillingDate
5. Campaigns: organization, slug (unique within organization), isActive
6. Categories: slug (unique), parentCategory
7. Payouts: organization, status, createdAt
8. Webhooks: source, event, processed, createdAt

## Data Validation

- Email addresses must be valid format
- Required fields: user email, organization name, donation amount
- Minimum donation amount: $1.00
- Valid currency codes only
- Valid status enumerations
- Date validations (e.g., endDate must be after startDate for campaigns)
