# QA Case: MongoDB Atlas Connection Failure

## Issue

MongoDB Atlas connection intermittently failed when the application used an SRV connection string (`mongodb+srv://...`).

The application returned:

```text
querySrv ECONNREFUSED _mongodb._tcp.sell-here-cluster.4n5mmum.mongodb.net
```

The failure occurred while connecting through Mongoose from a Next.js API route.

## Environment

- Application: Sell-Here
- Framework: Next.js
- Database: MongoDB Atlas
- ODM: Mongoose
- Machine: Older/slower development machine
- Connection method initially tested: MongoDB SRV (`mongodb+srv://...`)
- DNS configured for Node: `8.8.8.8`

## Observed Behaviour

The review submission reached the API successfully, but the database connection failed:

```text
MongoDB connection state: 0
Failed to connect to database
Error: querySrv ECONNREFUSED
```

The request then took a long time before returning HTTP 500:

```text
POST /api/add-review 500 in 75s
```

The browser subsequently reported the review-save failure.

## Investigation

The following tests were performed:

### 1. Windows DNS resolution

Windows successfully resolved the MongoDB Atlas SRV record:

```text
nslookup -type=SRV _mongodb._tcp.sell-here-cluster.4n5mmum.mongodb.net
```

The Atlas cluster returned its three shard hosts.

### 2. Node DNS configuration

Node initially reported:

```text
[ '127.0.0.1' ]
```

even though Windows networking was configured with public DNS servers.

The application therefore explicitly configured Node DNS:

```ts
dns.setServers(["8.8.8.8"]);
```

Node then reported:

```text
Node DNS servers: [ '8.8.8.8' ]
```

### 3. Direct Node SRV lookup

A standalone Node test using `dns.resolveSrv()` successfully resolved the Atlas SRV record when `8.8.8.8` was specified.

### 4. Direct Mongoose connection

A standalone Node process using the same MongoDB URI successfully connected:

```text
Mongoose connected
```

This demonstrated that:

- MongoDB Atlas was reachable.
- The MongoDB credentials were valid.
- The connection string was valid.
- Mongoose itself could connect successfully from Node.
- The problem was not caused by the review data or API route.

### 5. Next.js API connection

The same application continued to fail when the SRV connection was used from the Next.js server runtime:

```text
querySrv ECONNREFUSED
```

The issue persisted without Turbopack, so Turbopack was not identified as the cause.

## Resolution / Workaround

The MongoDB Atlas connection string was changed from the SRV format:

```text
mongodb+srv://...
```

to the non-SRV connection string provided by MongoDB Atlas.

The application then connected successfully:

```text
MongoDB connection state: 0
connected to database
POST /api/add-review 201
```

The review was successfully persisted.

## Final Result

After switching to the non-SRV connection string:

```text
connected to database
POST /api/add-review 201
GET /products/1019 200
```

The MongoDB review workflow passed successfully.

## QA Assessment

The evidence indicates an **environmental/network/DNS-related connection problem associated with SRV resolution on the development machine**.

The older/slower machine may contribute to the long connection times, but hardware performance alone was not proven to be the root cause.

The strongest evidence is that:

1. Windows could resolve the Atlas SRV record.
2. Standalone Node + Mongoose could connect when DNS was explicitly configured.
3. The Next.js application still experienced `querySrv ECONNREFUSED`.
4. The same Atlas database connected successfully using the non-SRV connection string.

Therefore, the issue should be classified as an **environment-specific MongoDB connectivity/DNS issue**, rather than an application review or database-schema defect.

## Regression Test

After changing the connection string, verify:

- [x] Application starts successfully.
- [x] MongoDB connection succeeds.
- [x] Review submission reaches `/api/add-review`.
- [x] Review is persisted in MongoDB.
- [x] API returns HTTP `201`.
- [x] Product page loads successfully after submission.
- [x] No MongoDB `querySrv ECONNREFUSED` error occurs.

## Reusable QA Classification

**Category:** Environment / Database Connectivity
**Severity:** High during affected development sessions
**Failure type:** MongoDB Atlas connection failure
**Primary symptom:** `querySrv ECONNREFUSED`
**Affected connection type:** SRV (`mongodb+srv://...`)
**Workaround:** Use Atlas non-SRV connection string
**Application defect confirmed:** No
**Environment-specific behaviour:** Yes
