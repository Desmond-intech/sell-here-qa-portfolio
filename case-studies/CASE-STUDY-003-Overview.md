# CASE-STUDY-003 – Environment-Specific Loading Blockage Investigation

## Overview

This case study documents the investigation of an intermittent loading issue affecting the Cart and Favorites pages during local development of an e-commerce application built with Next.js, Redux Toolkit, Firebase Authentication, and Cloud Firestore.

The issue first appeared immediately after implementing guest-to-authenticated data merging. Following a successful sign-in, Cart and Favorites occasionally remained on an infinite loading spinner despite Firestore containing the correct merged data. Because the behavior appeared directly after the new merge implementation, the merge logic initially became the primary suspect.

A systematic investigation was conducted to validate every stage of the workflow, including Firestore document structure, merge algorithms, Redux state management, middleware, authentication, real-time Firestore listeners, component lifecycle, and loading state management. Extensive logging and controlled experiments were used to eliminate hypotheses one by one.

During the investigation, an important observation changed the direction of the analysis: the same loading blockage began occurring for anonymous guest users before any authentication or merge operation took place. This demonstrated that the merge implementation itself could not fully explain the issue.

To verify whether the behavior was caused by the application or the execution environment, the application was tested on a second device using the same Firebase backend and identical codebase. The issue could not be reproduced. Guest shopping, authentication, data merging, Firestore synchronization, and persistent Cart and Favorites all functioned immediately and correctly.

The investigation concluded that the application's merge implementation, authentication workflow, Firestore synchronization, and Redux state management were operating correctly. The loading blockage was isolated to the local development environment, highlighting the importance of evidence-based debugging, cross-environment validation, and avoiding unnecessary code changes before confirming the true source of a problem.

**Key Skills Demonstrated**

- Root Cause Analysis
- Exploratory Testing
- Evidence-Based Debugging
- Hypothesis-Driven Investigation
- Firestore Data Validation
- Redux State Verification
- Authentication Flow Testing
- Component Lifecycle Analysis
- Cross-Device Validation
- Environment-Specific Defect Investigation
- Logging and Diagnostic Instrumentation
- Software Quality Assurance Methodology
