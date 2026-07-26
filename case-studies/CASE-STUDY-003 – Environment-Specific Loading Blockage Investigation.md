# CASE-STUDY-003 – Environment-Specific Loading Blockage Investigation

## Overview

This investigation began while implementing persistent shopping data synchronization between anonymous (guest) users and authenticated users.

The objective was to ensure that products added to the Cart and Favorites before signing in would seamlessly merge with the user's existing Firestore data after authentication. Once the merge completed, both collections were expected to synchronize in real time through Firestore listeners.

During testing, an intermittent loading issue appeared. Initially, it seemed to be directly related to the newly implemented merge functionality because the problem occurred immediately after signing in. However, as the investigation progressed, new evidence completely changed the direction of the investigation.

By following a structured debugging process, each suspected component was examined individually until the root cause was narrowed down. Cross-device testing ultimately demonstrated that the application logic was functioning correctly and that the loading blockage was isolated to the local development environment rather than the application itself.

---

# Background

The application supports two shopping modes:

### Guest Session

Anonymous users can browse products, add items to their Cart, and save Favorites before creating an account or signing in.

### Authenticated Session

When the user signs in, the application performs the following workflow:

1. Authenticate the user with Firebase Authentication.
2. Read the guest Cart and Favorites from Redux.
3. Retrieve the authenticated user's existing Firestore data.
4. Merge guest and Firestore collections.
5. Save the merged result back to Firestore.
6. Update the application's state.
7. Continue synchronizing changes through Firestore real-time listeners.

The goal was to provide a seamless shopping experience without losing guest selections.

---

# Initial Observation

While testing the new merge functionality, the following behavior was observed.

### Guest Session

- Cart loaded normally.
- Favorites loaded normally.
- Products could be added successfully.

### After Signing In

The merge operation appeared to complete successfully.

Firestore contained the correct merged Cart and Favorites.

However:

- Cart remained on an infinite loading spinner.
- Favorites remained on an infinite loading spinner.

Refreshing the browser immediately displayed the correct merged data.

At this stage, the merge implementation became the primary suspect because the issue consistently appeared immediately after authentication.

---

# Investigation Phase 1 – Merge Workflow

Since the behavior appeared after authentication, the investigation focused on the merge implementation.

The following areas were inspected.

## Firestore document structure

The merge originally wrote data using an incorrect document shape.

The write operation was corrected to match the Firestore document structure.

Result:

Firestore documents became correctly structured.

The loading issue remained.

---

## Redux synchronization

The merge function returned merged Cart and Favorites objects.

Redux dispatches were verified to ensure they matched the expected state structure.

Result:

Redux received correctly structured data.

The loading issue remained.

---

## Guest cleanup actions

After merging, guest state was cleared.

The suspicion was that the cleanup actions were immediately removing authenticated data.

The cleanup actions were temporarily removed.

Result:

The loading issue still occurred.

This hypothesis was eliminated.

---

## Middleware

Redux middleware was inspected to determine whether Cart or Favorites actions were creating a synchronization loop.

Relevant listeners were temporarily removed.

Result:

No change.

Middleware was eliminated as the cause.

---

## Firestore synchronization

Attention shifted toward Firestore.

The synchronization flow between:

- Firestore
- onSnapshot()
- Redux

was investigated for possible recursive updates.

Detailed logging was added throughout the synchronization pipeline.

Result:

No recursive synchronization loop was identified.

---

# Investigation Phase 2 – Component Lifecycle

The investigation moved to the application's initialization process.

The following components were inspected individually:

- AuthProvider
- UserBootstrap
- CartLoader
- FavoritesLoader

Extensive logging was added to verify:

- component mounting
- authentication state
- current user UID
- Firestore subscriptions
- snapshot execution
- Redux dispatches
- loading state updates

Several theories were explored, including:

- authentication race conditions
- Firestore listener timing
- router navigation
- component remounting
- Redux loading state

Each hypothesis was tested individually.

No application logic defect could be confirmed.

---

# Investigation Phase 3 – New Evidence

While continuing testing, an unexpected observation completely changed the investigation.

The same infinite loading spinner began appearing during anonymous guest sessions.

No authentication occurred.

No merge occurred.

No Firestore merge logic executed.

Simply opening the Cart or Favorites page occasionally resulted in the same loading blockage.

This was a critical turning point.

Since guest users never execute the merge workflow, the merge implementation could no longer explain the observed behavior.

The investigation shifted away from authentication and toward the execution environment itself.

---

# Cross-Environment Validation

To determine whether the problem existed in the application or in the local development environment, the exact same application was tested on a mobile device using the same Firebase project.

## Development Laptop

Intermittent behavior:

- Guest sessions occasionally remained on the loading spinner.
- Authenticated sessions occasionally remained on the loading spinner.
- Refreshing the browser immediately restored the correct state.

## Mobile Device

Consistent behavior:

- Guest Cart loaded immediately.
- Guest Favorites loaded immediately.
- Authentication completed normally.
- Cart merged successfully.
- Favorites merged successfully.
- Firestore synchronization completed immediately.
- No loading blockage occurred.

The issue could not be reproduced.

---

# Root Cause Analysis

The investigation demonstrated that:

- Firestore merge logic functioned correctly.
- Authentication completed successfully.
- Redux received the expected data.
- Firestore persistence worked correctly.
- Cart and Favorites synchronized correctly.

The most significant finding occurred when the loading blockage began affecting guest users as well as authenticated users.

Because guest users never execute the merge workflow, the merge implementation was eliminated as the root cause.

Testing on a second device further confirmed that the same application behaved correctly using the same Firebase backend.

Combined with previous observations regarding the local development laptop—including slow disk performance, development slowdowns, and intermittent Firebase network warnings—the evidence strongly indicated that the loading blockage was specific to the local development environment rather than an application defect.

---

# QA Methodology

This investigation followed a structured QA process rather than relying on assumptions.

The process included:

- Reproducing the issue.
- Forming evidence-based hypotheses.
- Investigating one subsystem at a time.
- Instrumenting the application with targeted logging.
- Eliminating hypotheses using observed evidence.
- Re-testing after every change.
- Validating behavior across multiple devices.
- Distinguishing an environmental issue from an application defect.

---

# Lessons Learned

This investigation reinforced several important software quality principles.

- The most recent code change is not always responsible for newly observed behavior.
- Every hypothesis should be supported or rejected using evidence rather than assumptions.
- Logging is one of the most effective debugging tools available.
- Testing on multiple environments is essential before concluding that an application contains a defect.
- Eliminating incorrect hypotheses is just as valuable as identifying the correct one.

Perhaps the most important lesson from this investigation was that changing working code without sufficient evidence can introduce unnecessary complexity. By following a disciplined debugging process, the application architecture remained intact while the true source of the problem was narrowed to the local development environment.

# Testing Date

Sunday 26 July, from 15:00 PM to 01:00 AM Monday
