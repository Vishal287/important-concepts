# MongoDB Indexing: Performance, Best Practices & Optimization

Indexes in MongoDB are essential for optimizing query performance. However, they also introduce write overhead. This guide covers various indexing strategies, best practices, and trade-offs to help you optimize MongoDB performance.

---

## **Table of Contents**
1. [Single vs. Compound Index](#1-single-vs-compound-index)
2. [Compound Index Best Practices](#2-compound-index-best-practices)
3. [Covered Queries & Projection Optimization](#3-covered-queries--projection-optimization)
4. [Sorting & Indexes](#4-sorting--indexes)
5. [TTL Indexes for Automatic Data Expiry](#5-ttl-indexes-for-automatic-data-expiry)
6. [Compound vs. Multi-Key Indexes](#6-compound-vs-multi-key-indexes)
7. [Write Performance vs. Indexing Overhead](#7-write-performance-vs-indexing-overhead)

---

## **1. Single vs. Compound Index**

### **Single Index**
- An index on a single field.
- Improves search performance for queries filtering by that field.
- Example:
  ```js
  db.users.createIndex({ email: 1 });
  ```
- Used when queries frequently filter using a single field.

### **Compound Index**
- An index on multiple fields.
- Improves performance when filtering or sorting on multiple fields.
- Example:
  ```js
  db.users.createIndex({ firstName: 1, lastName: 1 });
  ```
- **Example Query Optimization:**
  ```js
  db.users.find({ firstName: "John", lastName: "Doe" }).explain("executionStats");
  ```
- Follows the **leftmost prefix rule**, meaning it works efficiently only when querying from left to right in the index.

---

## **2. Compound Index Best Practices**
- **Order Matters:** Place the most frequently queried field first.
- **Avoid Redundant Indexes:** A compound index `{ A, B }` can serve queries on `{ A }`, so you don’t need a separate index on `{ A }`.
- **Match Query Patterns:** Ensure indexes match the fields used in `find()`, `sort()`, and `aggregate()`.
- **Monitor Performance:** Use `explain()` to analyze index usage.
- **Example:**
  ```js
  db.orders.createIndex({ customerId: 1, orderDate: -1 });
  db.orders.find({ customerId: "12345" }).sort({ orderDate: -1 });
  ```

---

## **3. Covered Queries & Projection Optimization**

### **Covered Queries**
- A query is "covered" when all required fields are in the index.
- Covered queries do **not** require fetching documents, improving performance.
- Example:
  ```js
  db.users.createIndex({ email: 1, age: 1 });
  db.users.find({ email: "john@example.com" }, { age: 1, _id: 0 }).explain("executionStats");
  ```

### **Projection Optimization**
- Only retrieve necessary fields to reduce disk I/O.
- Example:
  ```js
  db.users.find({}, { firstName: 1, lastName: 1, _id: 0 });
  ```

---

## **4. Sorting & Indexes**
- Sorting is efficient **only** if it matches the index order.
- Example:
  ```js
  db.products.createIndex({ price: 1 });
  db.products.find().sort({ price: 1 }).explain("executionStats");
  ```
- If sorting on multiple fields, create a compound index:
  ```js
  db.orders.createIndex({ customerId: 1, orderDate: -1 });
  ```
- **Avoid Sorting Without Indexes:** Sorting large datasets without an index leads to performance issues.

---

## **5. TTL Indexes for Automatic Data Expiry**
- TTL (Time-To-Live) Index automatically removes documents after a specified time.
- Useful for **logs, sessions, and temporary data**.
- Example:
  ```js
  db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 86400 });
  ```
- **Checking Expired Documents:**
  ```js
  db.sessions.find({ createdAt: { $lt: new Date() }});
  ```
- **Limitations:**
  - Works only on `Date` fields.
  - Cannot be part of a compound index.
  - Deletion is **not instant**; MongoDB removes expired documents periodically.

---

## **6. Compound vs. Multi-Key Indexes**

### **Compound Index**
- Used for multiple **scalar** fields.
- Example:
  ```js
  db.users.createIndex({ firstName: 1, lastName: 1 });
  ```
- Cannot index **array fields efficiently**.

### **Multi-Key Index**
- Automatically created for **array fields**.
- Example:
  ```js
  db.orders.createIndex({ items: 1 });
  ```
- **Example Query:**
  ```js
  db.orders.find({ items: "Laptop" }).explain("executionStats");
  ```
- **Cannot be used with compound indexes on multiple arrays.**

---

## **7. Write Performance vs. Indexing Overhead**

### **How Indexing Affects Write Performance**
- Every `insert`, `update`, and `delete` operation must also update indexes.
- **More indexes = more disk writes**, slowing down inserts/updates.

### **Optimizing Write Performance**
- **Use only necessary indexes** to avoid slow writes.
- **Avoid indexing frequently updated fields**.
- **Monitor index usage** with:
  ```js
  db.collection.stats();
  ```
- **Batch inserts/updates** to reduce index updates.
- **Example of Bulk Insert:**
  ```js
  let bulk = db.users.initializeUnorderedBulkOp();
  for (let i = 0; i < 1000; i++) {
    bulk.insert({ name: "User" + i, email: "user" + i + "@example.com" });
  }
  bulk.execute();
  ```

---

## **Final Thoughts**
✅ Indexing is crucial for read performance but should be used carefully to avoid write slowdowns.
✅ Use `explain()` to analyze index efficiency.
✅ Monitor MongoDB performance to balance read/write operations.

Would you like to contribute or suggest improvements? Feel free to submit a PR! 🚀
