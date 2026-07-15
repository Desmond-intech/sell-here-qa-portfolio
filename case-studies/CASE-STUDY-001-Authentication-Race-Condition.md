# CASE STUDY 001 – Authentication Race Condition Investigation

## Project

Sell Here – Next.js E-commerce Application

---

## Case Study

Authentication Race Condition During Sign-In

---

## Category

Authentication • State Management • Asynchronous Logic

---

## Severity

High

---

## Priority

High

---

## Technologies

- Next.js (App Router)
- React
- TypeScript
- Firebase Authentication
- Firebase Firestore
- Redux Toolkit

---

## Environment

| Item             | Value                                     |
| ---------------- | ----------------------------------------- |
| Application      | Sell Here                                 |
| Browser          | Google Chrome                             |
| Operating System | Windows 10                                |
| Tester           | Desmond-intech                            |
| Test Type        | Manual Investigation & Regression Testing |

---

# Executive Summary

During manual testing of the authentication workflow, intermittent inconsistencies were observed immediately after successful user sign-in.

Although Firebase Authentication completed successfully, parts of the application occasionally behaved as though authentication had not yet finished. The issue manifested as inconsistent UI behaviour, delayed state synchronization, and timing-dependent application behaviour that could not be reproduced consistently.

Rather than originating from a single defect, the investigation identified multiple asynchronous synchronization issues occurring during the sign-in process. These included redundant state updates, timing dependencies, unnecessary operations, and configuration-related failures.

This case study documents the investigation process, the findings, the implemented resolutions, and the regression strategy used to verify a stable authentication workflow.

---

# Investigation Timeline (summary)

## Investigation Timeline

| Step | Investigation Milestone                          |
| :--: | ------------------------------------------------ |
|  1   | Intermittent authentication behaviour observed.  |
|  2   | Redundant listener dispatches identified.        |
|  3   | Unnecessary sign-in API processing discovered.   |
|  4   | Authentication timing dependency isolated.       |
|  5   | Email verification configuration corrected.      |
|  6   | Authentication workflow stabilized and verified. |

---

# System Architecture

The authentication workflow coordinates multiple asynchronous systems before the application reaches a stable authenticated state.

The intended sequence is illustrated below.

```text
User
   │
   ▼
Sign In Request
   │
   ▼
Firebase Authentication
   │
   ▼
onAuthStateChanged()
   │
   ▼
syncUserProfile() [syncing in background]
   │
   ▼
Redux Store Updated
   │
   ▼
Reactive Loaders
(CartLoader / FavoritesLoader)
   │
   ▼
Protected Components Render
   │
   ▼
Verification Banner
   │
   ▼
Authenticated User Experience
```

## Component Responsibilities

| Component                   | Responsibility                                                                 |
| --------------------------- | ------------------------------------------------------------------------------ |
| **Firebase Authentication** | Authenticates the user and establishes the authentication session.             |
| **onAuthStateChanged()**    | Detects authentication state changes and triggers application synchronization. |
| **syncUserProfile()**       | Retrieves the user's Firestore profile and synchronizes application state.     |
| **Redux Store**             | Maintains the authenticated user's global application state.                   |
| **CartLoader**              | Subscribes to the authenticated user's cart data after state synchronization.  |
| **FavoritesLoader**         | Subscribes to the authenticated user's favourites after state synchronization. |
| **Protected Components**    | Render authenticated content using synchronized application state.             |
| **Verification Banner**     | Displays account verification status based on the synchronized user profile.   |

## Expected Behaviour

The authentication workflow should progress sequentially, with each stage completing before dependent components begin execution. Once the application state has been synchronized, reactive loaders initialize, protected components render, and the authenticated user experience becomes available.

---

# Observed Symptoms

During repeated manual testing of the sign-in workflow, the application exhibited intermittent behaviour immediately after successful authentication.

Although users were authenticated successfully, the application did not always reach a stable authenticated state before dependent components began executing.

The following symptoms were observed:

- Delayed navigation after successful sign-in.
- Protected components briefly rendering with incomplete authentication state.
- User interface temporarily behaving as though the user was unauthenticated.
- Verification Banner appearing inconsistently.
- Cart and Favorites reactive loaders initializing before authenticated user data had fully synchronized.
- Intermittent console warnings related to application state timing.
- Temporary inconsistencies between Firebase Authentication and the Redux store immediately after login.

## Characteristics of the Issue

The behaviour was intermittent rather than consistently reproducible.

Under identical test conditions:

- Some sign-in attempts completed successfully without visible issues.
- Other sign-in attempts produced one or more of the symptoms described above.

Repeated testing indicated that the issue was dependent on the timing of asynchronous operations rather than invalid authentication credentials or user input.

This behaviour suggested a potential race condition within the application's authentication workflow.

---

# Investigation Timeline

The authentication workflow was investigated by repeatedly reproducing the sign-in process, observing application behaviour, and isolating individual stages of the authentication lifecycle.

Rather than identifying a single defect, the investigation uncovered several independent findings that collectively contributed to the inconsistent application state.

## Finding 1 – Redundant State Updates

### Observation

Cart and Favorites state were updated from multiple locations during the authentication process.

### Investigation

The authentication flow was reviewed to determine where application state was being synchronized after sign-in.

### Finding

`CartListener` and `FavoritesListener` each dispatched state updates independently, resulting in unnecessary synchronization work during application initialization.

### Action Taken

Redundant state dispatches were removed, allowing each listener to focus solely on its responsibility while preventing duplicate updates.

---

## Finding 2 – Unnecessary API Processing

### Observation

The sign-in API route completed successfully, yet part of the response processing was not used by the application.

### Investigation

The sign-in request was reviewed to determine whether all returned values contributed to the authentication workflow.

### Finding

The response body was parsed using:

`const data = await response.json()`

Although the request completed successfully, the parsed response was never consumed.

### Action Taken

The unused response parsing was removed, simplifying the authentication flow and eliminating unnecessary asynchronous work.

---

## Finding 3 – Authentication Timing Dependency

### Observation

Successful authentication did not always result in an immediately stable application state.

### Investigation

The authentication sequence was observed during repeated sign-in attempts to determine when dependent components began executing.

### Finding

Some logic continued execution before the authentication workflow had fully synchronized application state, creating a timing dependency between authentication, profile synchronization, Redux updates, and component rendering.

### Action Taken

The authentication flow was updated to ensure dependent operations completed before subsequent logic continued.

---

## Finding 4 – Configuration Failure

### Observation

The email verification workflow occasionally failed despite successful authentication.

### Investigation

The verification API was reviewed to identify configuration differences between expected and runtime values.

### Finding

A required URL was undefined because of an environment variable configuration change.

### Action Taken

The environment configuration was corrected, restoring the email verification workflow.

# Root Cause Analysis

The investigation concluded that the intermittent authentication behaviour was not caused by a single defective component. Instead, it resulted from multiple asynchronous operations executing without guaranteed synchronization during the sign-in workflow.

While Firebase Authentication successfully authenticated the user, several dependent processes—including profile synchronization, Redux state updates, reactive data subscriptions, and protected component rendering—were able to proceed before the application reached a fully synchronized state.

As a result, components occasionally attempted to consume authentication-dependent data before it was available.

## Authentication Sequence

The intended authentication workflow is illustrated below.

```text
User Signs In
      │
      ▼
Firebase Authentication
      │
      ▼
onAuthStateChanged()
      │
      ▼
syncUserProfile() [syncing in background]
      │
      ▼
Redux Store Updated
      │
      ▼
Reactive Loaders Initialize
      │
      ▼
Protected Components Render
      │
      ▼
Authenticated User Experience
```

During the investigation, it became evident that portions of the application could continue executing before earlier stages of this sequence had completed.

Because these operations depended on asynchronous execution, variations in network latency, rendering order, and Firestore response times affected when application state became available.

This timing dependency resulted in non-deterministic behaviour, where identical user actions could produce different outcomes across separate sign-in attempts.

## Root Cause Summary

The investigation identified several contributing factors:

- Redundant state synchronization within reactive listeners.
- Unnecessary asynchronous processing during sign-in.
- Dependent logic executing before authentication state had fully synchronized.
- Configuration inconsistencies affecting the email verification workflow.

Collectively, these issues introduced race conditions that prevented the authentication workflow from consistently reaching a stable application state before dependent components executed.

# Implemented Resolutions

The authentication workflow was refined by addressing each contributing factor identified during the investigation. Rather than applying a single fix, multiple improvements were introduced to stabilize the sign-in sequence and eliminate timing dependencies.

---

## Resolution 1 – Removed Redundant State Synchronization

State updates were consolidated by removing redundant dispatches from `CartListener` and `FavoritesListener`.

This reduced unnecessary synchronization work during application initialization and ensured that each component maintained a single, well-defined responsibility.

---

## Resolution 2 – Simplified the Sign-In API Workflow

The sign-in API route was reviewed and unnecessary response processing was removed.

Eliminating unused asynchronous work simplified the authentication flow and reduced unnecessary operations during sign-in.

---

## Resolution 3 – Synchronized Authentication Flow

The authentication sequence was updated so that dependent operations execute only after the required application state has been established.

The revised workflow ensures that:

- Authentication completes successfully.
- User profile synchronization finishes.
- Redux state is updated.
- Reactive loaders initialize.
- Protected components render using synchronized application state.

This eliminated the timing dependency responsible for the intermittent authentication behaviour.

---

## Resolution 4 – Restored Email Verification Configuration

The email verification workflow was corrected by resolving the environment variable configuration responsible for generating an undefined verification URL.

After the configuration was restored, verification links were generated successfully and the verification process completed as expected.

---

## Authentication Workflow After Improvements

```text id="yb9ygh"
User Signs In
      │
      ▼
Firebase Authentication
      │
      ▼
onAuthStateChanged()
      │
      ▼
syncUserProfile() [syncing in background]
      │
      ▼
Redux Store Updated
      │
      ▼
Reactive Loaders Initialize
      │
      ▼
Protected Components Render
      │
      ▼
Verification Banner
      │
      ▼
Stable Authenticated User Experience
```

## Resolution Outcome

The combined improvements produced a predictable and repeatable authentication workflow.

Application state is now synchronized before dependent components begin execution, resulting in consistent rendering, reliable reactive subscriptions, and a stable sign-in experience across repeated authentication scenarios.

# Regression Testing Strategy

Following implementation of the authentication improvements, manual regression testing was performed to verify that the updated sign-in workflow behaved consistently across common authentication scenarios.

Testing focused on confirming that application state was fully synchronized before dependent components executed, while ensuring that existing authentication functionality remained unaffected.

## Test Scenarios

| Scenario                      | Expected Outcome                                                                 | Result  |
| ----------------------------- | -------------------------------------------------------------------------------- | :-----: |
| Standard user sign-in         | User authenticated and application state synchronized successfully               | ✅ Pass |
| Fresh browser session         | Authentication initializes correctly                                             | ✅ Pass |
| Returning authenticated user  | Existing session restored successfully                                           | ✅ Pass |
| Anonymous user upgrade        | Guest account transitions correctly to authenticated account                     | ✅ Pass |
| Cart loading                  | Cart data loads after authentication state is synchronized                       | ✅ Pass |
| Favorites loading             | Favorites data loads after authentication state is synchronized                  | ✅ Pass |
| Email verification flow       | Verification banner and verification workflow operate correctly                  | ✅ Pass |
| Redirect after sign-in        | Navigation occurs only after required application state is available             | ✅ Pass |
| Redux synchronization         | Global authentication state updates correctly before dependent components render | ✅ Pass |
| Logout and subsequent sign-in | Authentication lifecycle remains stable across repeated sessions                 | ✅ Pass |

## Regression Scope

Regression testing verified the following areas of the application:

- Authentication workflow
- User profile synchronization
- Redux authentication state
- Reactive cart subscriptions
- Reactive favorites subscriptions
- Protected component rendering
- Verification banner behaviour
- Navigation after authentication
- Session restoration
- Logout and repeated sign-in cycles

## Regression Outcome

All regression scenarios completed successfully.

Repeated testing confirmed that the authentication workflow consistently reached a stable application state before dependent components executed. No intermittent authentication behaviour, synchronization issues, or regressions were observed during the verification process.

# Lessons Learned

This investigation reinforced several practical software engineering and quality assurance principles.

## Asynchronous Operations Require Explicit Coordination

Successful authentication does not guarantee that the application is ready to consume authenticated state.

Dependent operations such as profile synchronization, global state updates, and reactive data subscriptions must complete before authenticated components begin execution.

---

## Race Conditions Are Often Intermittent

Timing-related defects may appear and disappear under identical user actions.

Consistent reproduction required repeated testing across multiple sign-in attempts rather than relying on a single successful or failed execution.

---

## State Synchronization Is Part of Authentication

Authentication extends beyond validating user credentials.

A reliable sign-in experience also depends on synchronizing application state before allowing dependent features to initialize.

---

## Shared Application State Requires Clear Responsibilities

Removing redundant state updates simplified the authentication workflow and reduced unnecessary synchronization between application components.

Clearly defined responsibilities improved both maintainability and application stability.

---

## Configuration Should Be Verified During Investigation

Not every authentication issue originates from application logic.

The investigation confirmed that environment configuration can directly affect authentication-related functionality and should always be included in the investigation process.

---

## Investigation Before Modification

The most effective improvements resulted from understanding the complete authentication lifecycle before making implementation changes.

Observing application behaviour, isolating contributing factors, and confirming findings before modifying the code reduced unnecessary changes and produced a more reliable solution.

---

## Key Takeaway

A significant outcome of this investigation was understanding that **`await` is more than a mechanism for waiting on asynchronous operations—it establishes a synchronization point between dependent stages of an application workflow.**

Applying `await` strategically ensured that critical authentication tasks completed before subsequent logic executed, resulting in a predictable and stable sign-in experience.

# Case Study Outcome

The authentication investigation successfully identified and resolved multiple independent issues affecting the application's sign-in workflow.

Rather than originating from a single defect, the intermittent behaviour resulted from a combination of asynchronous timing dependencies, redundant state synchronization, unnecessary processing, and configuration inconsistencies. Addressing these issues collectively produced a stable and predictable authentication sequence.

Following the implemented improvements and regression testing, the application consistently:

- Authenticates users successfully.
- Synchronizes user profile data before dependent components execute.
- Updates the Redux store before protected features consume authenticated state.
- Initializes reactive loaders at the appropriate stage of the authentication lifecycle.
- Restores cart and favorites data consistently.
- Displays the verification banner reliably.
- Navigates users only after the application reaches a stable authenticated state.

## Investigation Summary

| Area                    | Outcome         |
| ----------------------- | --------------- |
| Authentication workflow | ✅ Stabilized   |
| State synchronization   | ✅ Improved     |
| Redux initialization    | ✅ Verified     |
| Reactive loaders        | ✅ Synchronized |
| Cart persistence        | ✅ Verified     |
| Favorites persistence   | ✅ Verified     |
| Verification workflow   | ✅ Restored     |
| Navigation timing       | ✅ Corrected    |
| Regression testing      | ✅ Passed       |

## Conclusion

This case study demonstrates a structured investigation into a complex, timing-dependent authentication issue. By combining repeated observation, systematic analysis, targeted implementation changes, and comprehensive regression testing, the authentication workflow was transformed into a predictable and reliable process.

The investigation highlights the importance of understanding asynchronous application behaviour, verifying assumptions through testing, and documenting findings in a manner that supports both software quality assurance and collaborative software engineering.
