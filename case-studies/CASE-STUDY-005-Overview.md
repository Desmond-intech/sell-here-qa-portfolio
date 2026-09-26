# CASE-STUDY-005 — Overview

## Debugging Data That Reaches the Database but Not the UI

### Summary

This case study documents a production-style debugging issue encountered while integrating **Firestore and MongoDB as separate review data sources** in Sell-Here.

Reviews were successfully written to Firestore and visible in the Firebase Console, yet Firestore reviews were not appearing in the application's Reviews UI. MongoDB reviews displayed correctly.

The investigation demonstrated that the database itself was working. The failure occurred at the **API data-shaping boundary** between Firestore and the frontend.

### System Context

The review flow is:

```text
RateProduct
    ↓
ReviewProvider / addReview
    ↓
/api/add-review
    ↓
Firestore or MongoDB
    ↓
/api/fetch-reviews
    ↓
useFetchReviews
    ↓
Reviews.tsx
    ↓
productId filtering
    ↓
Review UI
```

Firestore and MongoDB use separate storage implementations but are normalized into a shared `Review` structure for the frontend.

### The Failure

A Firestore document represented an individual review:

```text
{
  productId: 121,
  rating: 5,
  name: "But they",
  title: "Hi them",
  review: "Done with those",
  source: "firestore"
}
```

However, the Firestore API code treated the document as though it contained a nested `reviews` property:

```ts
return data.reviews;
```

Because `data.reviews` did not exist, the API returned an empty Firestore review collection.

The correct approach was:

```ts
return data;
```

### Evidence

Browser logging established where the data disappeared:

```text
PRODUCT ID: 121

ALL FIRESTORE: []
PRODUCT FIRESTORE: []

ALL MONGO: Array(4)
```

This showed that `Reviews.tsx` was not receiving Firestore reviews at all.

MongoDB was successfully returning its review collection, proving that the frontend rendering path could display reviews.

The investigation then moved backward through the data pipeline until the incorrect Firestore data extraction was identified.

### Root Cause

**Root cause:** a data-shape mismatch between the Firestore document structure and the API's expected structure.

The API assumed:

```text
document
└── reviews[]
```

while the actual database structure was:

```text
document = review
├── productId
├── rating
├── title
├── review
└── ...
```

### Resolution

The Firestore API was changed to return the document's actual data rather than a nonexistent nested property.

After the correction, Firestore reviews could flow through the same frontend pipeline as MongoDB reviews.

### QA Lesson

The key lesson is:

> **When data exists in the database but does not appear in the UI, trace the data through every system boundary and identify the first point where it becomes empty or changes shape.**

The investigation followed:

```text
Database
   ↓
API
   ↓
Fetch hook
   ↓
React state
   ↓
Filtering
   ↓
UI
```

Instead of assuming the problem was in React rendering, the actual response was inspected at each stage.

### Engineering Lesson

A successful database write does not guarantee successful application behavior.

Likewise:

- A database record can exist while the API returns nothing.
- An API request can succeed while returning the wrong data shape.
- React can render correctly while receiving an empty array.
- Two databases can contain equivalent business data while exposing different storage representations.

Therefore, **data shape is part of the contract between system layers**.

### Broader System-Design Insight

This case reinforced the architectural value of keeping database-specific logic out of the UI.

The frontend should receive a predictable `Review` structure regardless of whether the review originated from:

```text
Firestore
   or
MongoDB
```

The API/data layer should handle differences in:

- document structure
- database identifiers
- timestamps
- serialization
- source labels
- database-specific fields

The UI should primarily consume the normalized result.

### Final Takeaway

This was not fundamentally a Firestore problem or a React problem.

It was a **data-contract problem**.

The most useful debugging question was:

> **"What does the data look like at this exact point in the system?"**

That question turned a vague "Firestore reviews aren't displaying" problem into a specific API data-shape defect.

**QA principle:**
**Trace the data, don't guess the layer.**
