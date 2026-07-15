# CASE STUDY 001 – Overview

## Authentication Race Condition Investigation

### Project

**Sell Here** – Next.js E-commerce Application

---

## Summary

This case study documents the investigation of an intermittent authentication race condition affecting the application's sign-in workflow. Rather than originating from a single defect, the issue was traced to multiple asynchronous synchronization problems involving authentication, application state management, reactive data loading, and configuration.

The investigation focused on identifying the underlying causes, implementing targeted resolutions, and validating the updated authentication workflow through structured manual regression testing.

---

## Case Study Information

| Field            | Value                                                                                  |
| ---------------- | -------------------------------------------------------------------------------------- |
| **Duration**     | Several investigation sessions                                                         |
| **Category**     | Authentication / Race Condition                                                        |
| **Complexity**   | High                                                                                   |
| **Application**  | Sell Here                                                                              |
| **Technologies** | Next.js, Firebase Authentication, Firebase Firestore, Redux Toolkit, React, TypeScript |
| **Tester**       | Desmond-intech                                                                         |

---

## Primary Skills Demonstrated

- Root cause analysis
- Race condition investigation
- State synchronization
- Asynchronous workflow analysis
- Manual regression testing
- Authentication testing
- State management validation
- Technical documentation

---

## Investigation Outcome

The investigation successfully identified multiple independent synchronization issues affecting the authentication workflow. After implementing targeted improvements and completing regression testing, the application consistently reached a stable authenticated state before dependent components executed, resulting in a reliable and predictable sign-in experience.

---

**Related Document**

- `CASE-STUDY-001-Authentication-Race-Condition.md`
