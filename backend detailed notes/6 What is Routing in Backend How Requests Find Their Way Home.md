# 6 - What is Routing in Backend? How Requests Find Their Way Home

---

## What is Routing? (The Big Picture)

Routing is simply telling the server **WHERE** you want to send your request.

Think of it like mailing a letter:
- **HTTP Method (GET, POST, etc.)** = What you want to do (the ACTION)
- **Route/URL** = Where you want to do it (the ADDRESS)

```
┌─────────────────────────────────────────────────────────┐
│                    YOUR REQUEST                         │
│                                                         │
│   METHOD (What?)        +        ROUTE (Where?)         │
│   ─────────────                 ──────────────          │
│      GET                    /api/users                  │
│      (Fetch data)           (Users resource)            │
│                                                         │
│              ↓              ↓                           │
│         ┌─────────────────────────┐                     │
│         │   SERVER MATCHING       │                     │
│         │                         │                     │
│         │   Method + Route =      │                     │
│         │   Specific Handler      │                     │
│         └─────────────────────────┘                     │
│                      ↓                                  │
│              Returns Response                           │
└─────────────────────────────────────────────────────────┘
```

### Simple Summary

```
 Routing = Mapping URLs → Server-side Logic
 
 "I want to GET data from /api/users"
         ↓                    ↓
    (Action)             (Address)
         ↓                    ↓
    ┌──────────────────────────────┐
    │  Server finds matching code  │
    │  and sends back the data     │
    └──────────────────────────────┘
```

---

## HTTP Methods Recap (The "What")

Before diving into routing, remember what HTTP methods do:

| Method | Purpose | Has Body? | Example |
|--------|---------|-----------|---------|
| **GET** | Fetch/Read data | ❌ No | Get list of users |
| **POST** | Create new data | ✅ Yes | Add a new user |
| **PUT** | Update data | ✅ Yes | Update user details |
| **DELETE** | Remove data | ✅ Yes | Delete a user |
| **PATCH** | Partial update | ✅ Yes | Update user name only |

> **Key Point:** GET requests don't have a body. That's why we use **query parameters** to send extra data with GET requests.

---

## Types of Routing

### 1. Static Routes (Fixed Address)

Static routes **never change**. They are fixed strings.

```
┌─────────────────────────────────────────────┐
│            STATIC ROUTES                    │
│                                             │
│   GET    /api/books    → Get all books      │
│   POST   /api/books    → Create new book    │
│   GET    /api/users    → Get all users      │
│   POST   /api/users    → Create new user    │
│                                             │
│   /api/books  ← This part NEVER changes     │
│   /api/users  ← This part NEVER changes     │
└─────────────────────────────────────────────┘
```

**Why "Static"?** Because the route is always the same string. Nothing varies.

---

### 2. Dynamic Routes (Variable Address)

Dynamic routes have **changeable parts** marked with a colon `:`.

```
┌──────────────────────────────────────────────────┐
│              DYNAMIC ROUTES                      │
│                                                  │
│   GET  /api/users/:id                            │
│                    ↑                             │
│              This is DYNAMIC                     │
│              (can be any value)                   │
│                                                  │
│   Examples:                                      │
│   /api/users/123    → User with ID 123           │
│   /api/users/456    → User with ID 456           │
│   /api/users/abc    → User with ID abc           │
│                                                  │
│   The ":id" part matches ANY string              │
└──────────────────────────────────────────────────┘
```

**How the Server Sees It:**

```
Your Request:    GET /api/users/123
                          ↓
Server Route:    GET /api/users/:id
                          ↓
         ┌─────────────────────────────┐
         │  /api  matches  /api     ✓  │
         │  /users matches /users   ✓  │
         │  /123    matches  :id    ✓  │  ← 123 fills the ":id" slot
         └─────────────────────────────┘
                          ↓
              Handler receives id = "123"
```

**Why use Dynamic Routes?**

```
GET /api/users/123
    │       │   │
    │       │   └──→ "Get data for THIS specific user"
    │       └──────→ "From the users resource"
    └──────────────→ "I want to fetch/read"
    
It's human-readable! You can understand what it does just by reading it.
```

---

### 3. Path Parameters vs Query Parameters

These are two different ways to send extra data:

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   PATH PARAMETERS (Route Parameters)                        │
│   ────────────────────────────────────                      │
│   /api/users/123                                            │
│              ↑                                              │
│        Part of the route/path itself                        │
│        Goes AFTER the forward slash                         │
│        Used for IDENTIFYING a resource                      │
│                                                             │
│   Use when: You need to identify WHICH resource            │
│   Example: Which user? → ID 123                             │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   QUERY PARAMETERS                                          │
│   ────────────────                                          │
│   /api/search?query=some+value&page=2                       │
│                ↑              ↑                             │
│           Starts with ?    Key=value pairs                  │
│           Separated by &   Additional metadata              │
│                                                             │
│   Use when: You need to send FILTER/SORT/OPTIONS data      │
│   Example: Search for "some value" on page 2               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**When to Use Which?**

```
PATH PARAMETERS → WHO/WHAT resource?
─────────────────────────────────────
GET /api/users/123        → "Give me user 123"
GET /api/products/456     → "Give me product 456"
GET /api/orders/789       → "Give me order 789"

QUERY PARAMETERS → HOW to filter/sort/paginate?
───────────────────────────────────────────────
GET /api/books?page=2        → "Give me page 2"
GET /api/books?limit=20      → "Give me 20 books"
GET /api/books?sort=price    → "Sort by price"
GET /api/books?order=desc    → "In descending order"
```

---

## Query Parameters in Detail (Pagination Example)

Query parameters are super useful for **pagination** (splitting data into pages).

```
┌─────────────────────────────────────────────────────────┐
│                  PAGINATION EXAMPLE                     │
│                                                         │
│   FIRST REQUEST (No query params - get page 1)          │
│   GET /api/books?limit=20                               │
│                                                         │
│   Response:                                             │
│   {                                                     │
│     "data": [book1, book2, ... book20],                 │
│     "total": 100,          ← Total books in database    │
│     "currentPage": 1,      ← You're on page 1          │
│     "totalPages": 5        ← 100 books ÷ 20 = 5 pages  │
│   }                                                     │
│                                                         │
│                                                         │
│   SECOND REQUEST (Want page 2?)                         │
│   GET /api/books?page=2&limit=20                        │
│                                                         │
│   Response:                                             │
│   {                                                     │
│     "data": [book21, book22, ... book40],               │
│     "total": 100,                                       │
│     "currentPage": 2,                                   │
│     "totalPages": 5                                     │
│   }                                                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Why not use path parameters for pagination?**

```
BAD:  /api/books/2           ← Is this book ID 2 or page 2? CONFUSING!
GOOD: /api/books?page=2      ← Clear! This is page 2. NO CONFUSION!

BAD:  /api/books/page/2/limit/20/sort/price    ← Ugly and hard to maintain
GOOD: /api/books?page=2&limit=20&sort=price    ← Clean and readable
```

---

## Nested Routes (Putting It All Together)

Nested routes combine static and dynamic parts to express **relationships** between resources.

```
┌─────────────────────────────────────────────────────────────┐
│                    NESTED ROUTES                            │
│                                                             │
│   LEVEL 1: All users                                        │
│   GET /api/users                                            │
│   → Returns: [{user1}, {user2}, {user3}...]                 │
│                                                             │
│   LEVEL 2: Specific user                                    │
│   GET /api/users/123                                        │
│   → Returns: {user123 details}                              │
│                                                             │
│   LEVEL 3: Posts by that user                               │
│   GET /api/users/123/posts                                  │
│   → Returns: [{post1}, {post2}, {post3}...]                 │
│                                                             │
│   LEVEL 4: Specific post by that user                       │
│   GET /api/users/123/posts/456                              │
│   → Returns: {post456 details}                              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Visual Breakdown:**

```
/api/users/123/posts/456
│    │     │   │    │
│    │     │   │    └──→ Specific post (dynamic)
│    │     │   └──────→ Posts resource (static)
│    │     └──────────→ Specific user (dynamic)
│    └────────────────→ Users resource (static)
└─────────────────────→ API base (static)

Each level adds more context:
  /api/users        → "Give me users"
  /api/users/123    → "Give me THIS user"
  /api/users/123/posts       → "Give me THIS user's posts"
  /api/users/123/posts/456   → "Give me THIS user's THIS post"
```

---

## Route Versioning (Handling Changes)

When your API needs to change, versioning lets you keep old and new versions running.

```
┌─────────────────────────────────────────────────────────────┐
│                  ROUTE VERSIONING                           │
│                                                             │
│   VERSION 1 (Old format)                                    │
│   GET /api/v1/products                                      │
│   Response: { id, name, price }                             │
│                                                             │
│   VERSION 2 (New format)                                    │
│   GET /api/v2/products                                      │
│   Response: { id, title, price }  ← "name" changed to      │
│                                      "title"                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**Why Versioning?**

```
WITHOUT VERSIONING (Bad):
─────────────────────────
Before: GET /api/products → { id, name, price }
After:  GET /api/products → { id, title, price }  ← BREAKS old clients!

WITH VERSIONING (Good):
───────────────────────
V1: GET /api/v1/products → { id, name, price }  ← Old clients still work
V2: GET /api/v2/products → { id, title, price }  ← New clients use this

Timeline:
─────────
  ┌──────────┐     ┌──────────┐     ┌──────────┐
  │ V1 only  │────→│ V1 + V2  │────→│ V2 only  │
  │          │     │ (both    │     │ (V1      │
  │          │     │  work)   │     │ deprecated)│
  └──────────┘     └──────────┘     └──────────┘
  
  Old clients have time to migrate to V2!
```

---

## Catch-All Routes (The Safety Net)

Catch-all routes handle requests to **non-existent** paths.

```
┌─────────────────────────────────────────────────────────┐
│                 CATCH-ALL ROUTE                         │
│                                                         │
│   Your defined routes:                                  │
│   GET /api/users     → Works ✓                          │
│   GET /api/products  → Works ✓                          │
│                                                         │
│   User types something WRONG:                           │
│   GET /api/v3/products  → ???                           │
│                                                         │
│   Without catch-all:  Returns empty/null (confusing!)   │
│   With catch-all:     Returns friendly message          │
│                                                         │
│   ┌─────────────────────────────────────┐               │
│   │  {                                 │               │
│   │    "error": "Route not found",     │               │
│   │    "message": "This endpoint       │               │
│   │              does not exist"        │               │
│   │  }                                 │               │
│   └─────────────────────────────────────┘               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**How it works in the server:**

```
Server checks routes in order:
─────────────────────────────
  Request: GET /api/v3/products
  
  1. Does it match /api/users?      → NO  → Skip
  2. Does it match /api/products?   → NO  → Skip
  3. Does it match /api/v1/*?       → NO  → Skip
  4. Does it match /api/v2/*?       → NO  → Skip
  5. Does it match /* (catch-all)?  → YES → Run handler
                                         ↓
                               Send "Route not found" message
```

---

## The Complete Picture

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                    HOW A REQUEST TRAVELS                         │
│                                                                 │
│  CLIENT (Browser/App)                                           │
│       │                                                         │
│       │  1. Sends: GET /api/users/123/posts?page=2              │
│       │                                                         │
│       ▼                                                         │
│  SERVER                                                         │
│       │                                                         │
│       │  2. Parses the request                                   │
│       │     ├── Method: GET                                      │
│       │     ├── Route: /api/users/123/posts                      │
│       │     └── Query: ?page=2                                   │
│       │                                                         │
│       │  3. Matches route to handler                             │
│       │     ├── /api/users/:id/posts  ✓ MATCHED                  │
│       │     └── Extracts: id = "123", page = "2"                 │
│       │                                                         │
│       │  4. Runs business logic                                  │
│       │     ├── Query database for user 123's posts              │
│       │     ├── Apply pagination (page 2)                        │
│       │     └── Format response                                  │
│       │                                                         │
│       ▼                                                         │
│  RESPONSE                                                       │
│       {                                                         │
│         "data": [{post3}, {post4}],                              │
│         "currentPage": 2,                                        │
│         "totalPages": 5                                          │
│       }                                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Quick Reference Cheat Sheet

| Concept | Example | Purpose |
|---------|---------|---------|
| **Static Route** | `/api/books` | Fixed endpoint |
| **Dynamic Route** | `/api/users/:id` | Variable in path |
| **Path Parameter** | `/api/users/123` | Identify resource |
| **Query Parameter** | `/api/books?page=2` | Filter/sort/options |
| **Nested Route** | `/api/users/123/posts` | Resource relationships |
| **Versioned Route** | `/api/v1/products` | API evolution |
| **Catch-All Route** | `/*` | Handle unknown routes |

---

## Key Takeaways

1. **Routing = Mapping URLs to Server Logic**
   - Method tells WHAT you want to do
   - Route tells WHERE to do it

2. **Static Routes** = Fixed strings, never change
   - `/api/books`

3. **Dynamic Routes** = Have variable parts (`:id`)
   - `/api/users/:id` matches `/api/users/123`

4. **Path Parameters** = Part of the route, identify resources
   - `/api/users/123` → "user 123"

5. **Query Parameters** = Extra key-value pairs after `?`
   - `/api/books?page=2&limit=20`

6. **Nested Routes** = Show relationships between resources
   - `/api/users/123/posts/456`

7. **Versioning** = Keep old and new API versions running
   - `/api/v1/products` and `/api/v2/products`

8. **Catch-All** = Handle non-existent routes gracefully
   - Returns friendly "not found" message

---

*This covers all the routing concepts you need to understand backend codebases and work with APIs effectively!*
