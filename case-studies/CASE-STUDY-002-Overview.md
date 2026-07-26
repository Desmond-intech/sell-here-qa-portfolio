# CASE-STUDY-002 Overview

## Title

**Firestore Data Merge Failure During User Sign-In**

## Category

Integration Defect

## Severity

Medium

## Status

Resolved

## Project

Sell Here E-Commerce

## Components Involved

- Firebase Authentication
- Cloud Firestore
- Redux
- Cart Synchronization
- Favorites Synchronization

## Summary

During the guest-to-authenticated user synchronization process, the application failed after successful authentication with a Firestore runtime error indicating an invalid nested entity. The issue prevented guest cart and favorite items from being merged into the authenticated user's account.

The investigation revealed that the merge algorithm was functioning correctly. The actual defect originated from a mismatch between the Firestore document schema and the application's read logic. The application retrieved the entire Firestore document instead of the nested `items` property, causing the merge process to operate on an unexpected data structure.

Correcting the Firestore read operation restored successful synchronization while maintaining a consistent document schema.

## Skills Demonstrated

- Root cause analysis
- Integration testing
- Firestore schema validation
- Authentication workflow testing
- Data flow analysis
- Defect isolation
- Regression testing
- Technical documentation

## Outcome

The issue was resolved by aligning the Firestore read operation with the document schema. Guest carts and favorites now merge successfully after sign-in, and additional regression tests were identified to prevent similar schema-related defects in future releases.

**Detailed Investigation:** `CASE-STUDY-002-Firestore-Data-Merge-Failure-During-User-Sign-In.md`
