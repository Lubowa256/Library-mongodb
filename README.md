# Library Manager — Database 2 (NoSQL) Coursework

| | |
|---|---|
| **Student** | Lubowa Rehoboam |
| **Registration No.** | 25/BIT/BU/R/0009 |
| **University** | Bugema University |
| **Course unit** | Database 2 (NoSQL) |
| **Submitted** | November 2026 |

A web application for managing a library catalogue. **Backend database: MongoDB.**
Front end: plain HTML, CSS and JavaScript. API: Node.js (built-in `http` module) using the official `mongodb` driver.

![Screenshot](docs/screenshot.png)

## Features

| CRUD operation | Where in the app | MongoDB operation |
|---|---|---|
| **Create** | "+ Add book" form | `insertOne` |
| **Read** | Table, search, genre filter, pagination, summary tiles | `find` + `sort/skip/limit`, `countDocuments`, `distinct`, `aggregate` |
| **Update** | "Edit" button on each row | `updateOne` with `$set` |
| **Delete** | "Delete" button (with confirmation) | `deleteOne` |

NoSQL features demonstrated:
- **Document model** with an **embedded document** (`publisher`) and an **array** (`tags`).
- **Unique index** on `isbn` (duplicates are rejected), plus compound and multikey indexes.
- **Aggregation pipeline** (`$group`, `$sort`) for titles and copies per genre.
- **Regex search** across title, author, ISBN and tags (user input is escaped).
- Server-side validation with field-level error messages shown in the form.

## Document structure (`library_db.books`)

```json
{
  "_id": "ObjectId",
  "title": "Practical MongoDB for Beginners",
  "author": "Lubowa Rehoboam",
  "isbn": "9789970101856",
  "genre": "Computer Science",
  "year": 2026,
  "copies": 4,
  "publisher": { "name": "Bugema University Press", "country": "Uganda" },
  "tags": ["mongodb", "nosql", "databases"],
  "createdAt": "08/11/2025",
  "updatedAt": "09/11/2026"
}
```

## How to run

Requirements: **Node.js 18+** and **MongoDB** running locally (default `mongodb://127.0.0.1:27017`), or a MongoDB Atlas connection string.

```bash
npm install
npm start
```

Open **http://localhost:3000**. On first run the database `library_db` is created and filled with 8 sample books (all sample books, authors and publishers are fictional).

Options (environment variables): `MONGODB_URI`, `DB_NAME` (default `library_db`), `PORT` (default `3000`).
Reset to sample data at any time: `npm run seed`.

## REST API

| Method | URL | Purpose |
|---|---|---|
| POST | `/api/books` | Create |
| GET | `/api/books?q=&genre=&page=&limit=` | List / search / filter |
| GET | `/api/books/:id` | Read one |
| PUT | `/api/books/:id` | Update (any subset of fields) |
| DELETE | `/api/books/:id` | Delete |
| GET | `/api/stats` , `/api/genres` | Aggregation summary, genre list |

Status codes: 200/201 success, 400 validation error, 404 not found, 409 duplicate ISBN.

## Equivalent MongoDB shell commands (for demonstration)

```js
use library_db
db.books.insertOne({ title: "Stories from Kisenyi", author: "Nabukenya Gertrude", isbn: "9789970111111",
  genre: "Fiction", year: 2016, copies: 3, publisher: { name: "Kampala Readers Press", country: "Uganda" }, tags: ["uganda"] })
db.books.find({ genre: "Computer Science" }).sort({ title: 1 })
db.books.updateOne({ isbn: "9789970111111" }, { $set: { copies: 5 } })
db.books.deleteOne({ isbn: "9789970111111" })
db.books.aggregate([{ $group: { _id: "$genre", titles: { $sum: 1 }, copies: { $sum: "$copies" } } }, { $sort: { _id: 1 } }])
db.books.getIndexes()
```

## Testing

```bash
npm test                      # 12 API tests (create/read/update/delete, validation, duplicates, search, paging, security)
node test/ui-test.js          # browser end-to-end test (requires Playwright)
MONGODB_URI=mongodb://127.0.0.1:27017 npm test   # run the same API tests against your real MongoDB
```

## Project structure

```
src/store.js        all MongoDB operations + validation
src/server.js       HTTP routing and static file serving
src/start.js        connects to MongoDB, creates indexes, seeds, starts server
src/seed.js         npm run seed
public/             front end (index.html, style.css, app.js)
test/               API tests, browser test, test-only in-memory stand-in
```
