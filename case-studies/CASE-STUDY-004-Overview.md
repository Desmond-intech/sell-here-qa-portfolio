# CASE-STUDY-004 — MongoDB Atlas SRV Connection Failure

## Overview

During development of the Sell-Here e-commerce application, MongoDB Atlas repeatedly failed to connect when the application used an SRV (`mongodb+srv://`) connection string.

The failure appeared during review submission and produced:

```text
querySrv ECONNREFUSED _mongodb._tcp.sell-here-cluster.4n5mmum.mongodb.net
```

The API request eventually returned HTTP `500` after a prolonged connection attempt.

## What Made the Issue Difficult

The MongoDB database, credentials, review payload, API route, and Mongoose configuration were initially suspected because the failure occurred during review submission.

Testing showed otherwise.

Windows could resolve the MongoDB Atlas SRV record, and a standalone Node.js process using Mongoose could successfully connect when Node was explicitly configured to use Google's DNS server.

However, the same connection continued to fail from the Next.js development server.

This made the issue an important example of why a database connection failure should be investigated through the entire connection chain rather than immediately treated as an application-code defect.

## Investigation Path

The investigation established that:

- Windows DNS could resolve the Atlas SRV record.
- Node initially reported `127.0.0.1` as its DNS server.
- Node was explicitly configured to use `8.8.8.8`.
- Node could then successfully resolve the Atlas SRV record.
- Standalone Node + Mongoose successfully connected to MongoDB Atlas.
- Next.js continued to experience `querySrv ECONNREFUSED` with the SRV URI.
- Running Next.js without explicitly using the project's Turbopack script did not resolve the issue.
- The review API and MongoDB schema were not responsible for the connection failure.

## Resolution

The MongoDB Atlas connection string was changed from the SRV format:

```text
mongodb+srv://...
```

to the non-SRV connection string supplied by MongoDB Atlas.

The application subsequently connected successfully and the review API returned:

```text
POST /api/add-review 201
```

The review was persisted successfully in MongoDB.

## Technical Lesson

The case demonstrates the importance of separating:

**Application logic**

from

**Database connectivity**

from

**Operating-system/network environment**

The review implementation was functioning correctly. The failure occurred at the infrastructure/connectivity layer while resolving the MongoDB Atlas SRV connection.

The older development machine may have contributed to the unusually long connection attempts, but this was not conclusively established as the root cause.

## Developer Lesson

A database error does not automatically mean the database code is wrong.

When troubleshooting connectivity, isolate the layers:

```text
Application
    ↓
API Route
    ↓
Mongoose
    ↓
Node DNS
    ↓
Operating System / Network
    ↓
MongoDB Atlas
```

Testing each layer independently made it possible to distinguish the review implementation from the environmental connection problem.

## Outcome

**Status:** Resolved

**Resolution:** MongoDB Atlas non-SRV connection string

**Primary classification:** Environment / DNS / database connectivity

**Application defect:** Not identified

**Key error:**

```text
querySrv ECONNREFUSED
```

**Successful result:**

```text
connected to database
POST /api/add-review 201
```

## QA Value

This case is retained as a reusable troubleshooting reference for future MongoDB Atlas connection failures, particularly when:

- SRV connections fail unexpectedly.
- DNS resolution behaves differently between Windows and Node.js.
- API requests remain pending for unusually long periods.
- MongoDB works from standalone Node.js but fails inside the application runtime.
- A non-SRV Atlas connection succeeds where an SRV connection does not.
