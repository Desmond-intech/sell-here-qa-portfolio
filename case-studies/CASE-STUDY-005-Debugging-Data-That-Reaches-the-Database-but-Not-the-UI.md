# QA Lesson: Debugging Data That Reaches the Database but Not the UI

## Problem

Firestore reviews were successfully being created and visible in the Firestore console, but they were not appearing in the Reviews UI.

MongoDB reviews displayed correctly.

At first, this made the problem appear to be related to the Firestore query, React state, or the Reviews component. The actual problem was earlier in the data flow: the Firestore API was returning the wrong data shape.

## Expected Data Flow

The review system follows this flow:

**Database → API route → fetch hook → React state → product filter → UI**

For reviews:

```text
Firestore
   ↓
/api/fetch-reviews
   ↓
useFetchReviews
   ↓
Reviews.tsx
   ↓
filter by productId
   ↓
render review
```

MongoDB follows the same general path.

## The Actual Firestore Structure

Each Firestore document represents **one review**.

For example:

```text
{
  productId: 121,
  rating: 5,
  name: "But they",
  title: "Hi them",
  review: "Done with those",
  helpful: 0,
  like: [],
  media: null,
  source: "firestore"
}
```

There was no `reviews` property containing an array.

## The Bug

The Firestore API was effectively doing:

```ts
const data = doc.data();

return data.reviews;
```

This assumed the document had this structure:

```text
document
└── reviews
    ├── review
    ├── review
    └── review
```

But the actual structure was:

```text
document
├── productId
├── rating
├── title
├── review
├── name
└── ...
```

Therefore `data.reviews` was `undefined`.

The API consequently returned an empty Firestore review collection.

## How QA Exposed the Problem

The important console output was:

```text
PRODUCT ID: 121

ALL FIRESTORE: []
PRODUCT FIRESTORE: []

ALL MONGO: Array(4)
PRODUCT MONGO: Array(0)
```

This immediately established an important fact:

**The Reviews component was not receiving any Firestore reviews.**

The next question therefore was not:

> "Why isn't React displaying the review?"

It was:

> "At which point between Firestore and React did the review disappear?"

Tracing the data backwards led to `/api/fetch-reviews`.

## Fix

The API was changed from extracting a nonexistent property:

```ts
return data.reviews;
```

to returning the actual document data:

```ts
return data;
```

Now the Firestore review reaches `useFetchReviews`, which passes it into `Reviews.tsx`.

`Reviews.tsx` can then correctly filter the complete collection:

```ts
const productFirestoreReviews = firestoreReviews.filter(
  (review) => review.productId === productId,
);
```

## QA Debugging Principle

When data exists in a database but does not appear in the UI, **trace the data through every boundary**.

Do not immediately assume the problem is:

- React rendering
- component state
- `useEffect`
- the database
- the database query
- the UI

Instead, identify the **first boundary where the expected data becomes empty, incorrect, or changes shape**.

### Recommended debugging sequence

```text
1. Is the data actually in the database?
        ↓
2. Does the API retrieve it?
        ↓
3. What exact shape does the API return?
        ↓
4. Does the fetch hook receive that shape?
        ↓
5. Does React state contain the data?
        ↓
6. Does productId filtering find it?
        ↓
7. Does the component render it?
```

## Key Developer Lesson

**Data shape is part of the contract between system layers.**

A database document can be perfectly valid while the application still fails because the next layer expects a different structure.

In this case:

```text
Actual:
document = review

Code assumed:
document.reviews = reviews[]
```

The database was working.

The review submission was working.

The API request was working.

The React component was working.

**The contract between the database layer and API layer was wrong.**

That distinction is important when debugging backend-driven UI problems.

## QA Test to Retain

Whenever implementing or modifying a data-fetching API, verify both:

### 1. Data existence

```text
Does the record exist in the database?
```

### 2. Data shape

```text
Does the API return the exact structure that the consuming code expects?
```

A successful HTTP request does **not** necessarily mean successful data retrieval.

Likewise, seeing data in a database does **not** mean the frontend is receiving that data.

The most useful QA question is:

> **"What does the data look like at this exact point in the system?"**

That question can quickly locate where data is being lost.
