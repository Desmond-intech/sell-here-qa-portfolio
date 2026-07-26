# QA Case Study: Firestore Data Merge Failure During User Sign-In

## Title

**Firestore Merge Failure Caused by Document Schema Mismatch During Guest-to-User Data Synchronization**

---

## Overview

While implementing guest-to-authenticated user data synchronization, the application began failing during the sign-in process with a Firebase Firestore error. Initially, the error appeared to indicate that the data being written contained an invalid object. However, the actual root cause was a mismatch between the Firestore document schema and the application's read logic.

This issue required tracing the complete data flow—from authentication, to Firestore reads, through merge logic, and finally to Firestore writes—to identify the defect.

---

# System Behavior

The application stores shopping cart and favorites in Firestore.

Each document follows this schema:

```ts
{
  items: {
    "1": { ... },
    "2": { ... },
    ...
  }
}
```

During sign-in the application performs the following workflow:

1. Authenticate the user.
2. Read the user's Firestore cart.
3. Read the user's Firestore favorites.
4. Merge guest data with Firestore data.
5. Save the merged result back to Firestore.
6. Load the merged data into Redux.

---

# The Problem

Only users who had guest cart or favorite items experienced a failure during sign-in.

Authentication succeeded.

The failure occurred immediately after authentication while synchronizing user data.

Firebase produced the following error:

```
FirebaseError:
Property items contains an invalid nested entity.
```

The application therefore failed before the merge process completed.

---

# Initial Investigation

Because the error specifically referenced **items**, the first assumption was that one of the cart objects contained invalid Firestore data.

Possible suspects included:

- undefined values
- React components
- functions
- Date objects
- nested unsupported objects
- corrupted Redux state

Every item structure was inspected.

No invalid properties were found.

This made the error misleading because the reported property (**items**) was not actually the source of the defect.

---

# Following the Data Flow

Instead of focusing only on the error message, the investigation followed the entire synchronization process.

The merge workflow was examined step by step.

### Step 1

Read Firestore document.

```ts
const firestoreCart = cartSnap.exists() ? cartSnap.data() : {};
```

At first glance this looked correct.

---

### Step 2

Create a merged copy.

```ts
const mergedCart = {
  ...firestoreCart,
};
```

Again, nothing appeared suspicious.

---

### Step 3

Merge guest items.

```ts
mergedCart[itemId] = item;
```

Still appeared correct.

---

### Step 4

Save back to Firestore.

```ts
await setDoc(cartRef, {
  items: mergedCart,
});
```

Everything seemed consistent.

---

# The Turning Point

The breakthrough came by comparing the Firestore document schema with what was actually being read.

The document stored in Firestore looked like this:

```ts
{
    items: {
        "1": {...},
        "2": {...}
    }
}
```

However,

```ts
cartSnap.data();
```

returns the **entire document**, not just the cart items.

So instead of reading:

```ts
{
    "1": {...},
    "2": {...}
}
```

the application was actually reading:

```ts
{
    items: {
        "1": {...},
        "2": {...}
    }
}
```

This subtle difference completely changed what the merge logic was operating on.

---

# What Happened Next

The merge logic assumed that the object itself contained the products.

Therefore it added new entries directly to that object.

Conceptually the object became:

```ts
{
    items: {
        "1": {...},
        "2": {...}
    },

    "3": {...},
    "4": {...}
}
```

When Firestore attempted to save

```ts
{
  items: mergedCart;
}
```

the resulting structure no longer matched the application's intended schema.

The synchronization process attempted to write an incorrectly structured nested object, producing the misleading Firestore error.

---

# Root Cause

The application read the entire Firestore document instead of reading the nested **items** property.

Incorrect:

```ts
const firestoreCart = cartSnap.exists() ? cartSnap.data() : {};
```

Correct:

```ts
const firestoreCart = cartSnap.exists() ? (cartSnap.data().items ?? {}) : {};
```

The same correction was required for the favorites collection.

---

# Why This Was Difficult to Find

Several factors made this bug particularly challenging.

### The error message was misleading.

Firebase reported an invalid nested entity.

The real issue was not an invalid object inside the cart.

The issue was that the wrong level of the Firestore document had been read.

---

### Authentication succeeded.

The sign-in process worked correctly.

Only the synchronization stage failed.

This initially made authentication appear to be the problem even though it wasn't.

---

### The merge logic itself was correct.

The algorithm for combining guest and Firestore items worked exactly as intended.

The algorithm simply received data in an unexpected shape.

---

### The bug was introduced by a schema mismatch.

The write operation always assumed the schema:

```ts
{
    items: { ... }
}
```

The read operation accidentally assumed the schema:

```ts
{ ... }
```

The read and write paths were no longer symmetrical.

---

### The defect appeared only during a specific workflow.

The issue required all of the following:

- Guest cart data
- User authentication
- Firestore synchronization
- Merge execution
- Firestore write

Testing authentication alone would never expose the problem.

---

# Resolution

The Firestore read operation was updated to retrieve the nested **items** property instead of the entire document.

```ts
const firestoreCart = cartSnap.exists() ? (cartSnap.data().items ?? {}) : {};
```

The same fix was applied to favorites.

After the correction:

- Guest carts merged correctly.
- Favorites merged correctly.
- Firestore schema remained consistent.
- User sign-in completed successfully.
- No further synchronization errors occurred.

---

# QA Lessons Learned

This investigation reinforced several important testing principles.

- Never rely solely on an error message.
- Validate the complete data flow, not just the failing function.
- Verify that read and write operations use the same data schema.
- Integration defects often originate from assumptions made between components rather than defects inside a single component.
- Authentication workflows should always be tested together with post-authentication data synchronization.

---

# Regression Test Cases

| Test Case                            | Expected Result                                                 |
| ------------------------------------ | --------------------------------------------------------------- |
| Guest cart + empty Firestore cart    | Cart merged successfully                                        |
| Guest cart + existing Firestore cart | Quantities merged correctly                                     |
| Guest favorites + existing favorites | Favorites combined without duplicates                           |
| User with no guest data              | Existing Firestore data preserved                               |
| Multiple consecutive sign-ins        | Firestore schema remains unchanged                              |
| Verify Firestore document structure  | Documents always contain a single `items` object                |
| Complete authentication flow         | User signs in successfully and data synchronizes without errors |

---

# Outcome

The issue was resolved by correcting a single property access (`.items`), but identifying that correction required tracing the entire synchronization workflow and validating the consistency of the Firestore document schema across both read and write operations.

Although the code change was small, the investigation demonstrated the importance of understanding system behavior, following the data flow across multiple components, and distinguishing between a misleading runtime error and the actual root cause.
