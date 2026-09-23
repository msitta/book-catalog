# book-catalog

SQL analysis of a book-service database (PostgreSQL): catalog sizing, engagement,
and publisher/author insights to support a new reading-app product.

> Context: SQL project from the TripleTen Data Analyst bootcamp. The analysis,
> interpretation choices and decision log are my own.

---

## The question

During the pandemic, reading apps multiplied. Given a competitor's database
(books, authors, publishers, ratings and reviews), what does the catalog look
like, and who actually drives engagement?

## Answers at a glance

| # | Business question | Answer |
|---|---|---|
| 1 | How many books were published after 2000-01-01? | **819** of 1,000 (82%) |
| 2 | Reviews and average rating per book | All 1,000 books have ratings; **6** have no written review |
| 3 | Publisher with the most books over 50 pages | **Penguin Books**, 42 titles (then Vintage 31, Grand Central 25) |
| 4 | Highest-rated author (books with ≥ 50 ratings) | **J.K. Rowling / Mary GrandPré**, avg ≈ 4.28 |
| 5 | Avg. reviews among users who rated > 50 books | **≈ 24.3** reviews (6 users) |

## What matters for the product

- **A small, hyper-engaged core.** Only 6 users rated more than 50 books, and
  all six also write reviews (≈ 24 each). They produce structured data and
  written content at the same time.
- **Written reviews are the scarce channel.** Every book has a rating, but not
  every book has a review. Rating and reviewing are separate behaviours.
- **Long tail.** Only 19 of 1,000 books reach 50+ ratings, so any "top rated"
  ranking needs a volume floor or it gets inflated by books with few ratings.

## Analytical decisions

The notebook ends with a **decision log** covering every place where the
brief was ambiguous and I had to choose an interpretation. Examples:

- **`LEFT JOIN` over `INNER JOIN`** for the per-book metrics. With `INNER`, the
  6 books without reviews disappear and the result has 994 rows, not 1,000.
- **`COUNT(DISTINCT review_id)`** to stop fan-out from the two joins inflating
  the review count.
- **Mean of per-book means** for the author ranking (each book weighted
  equally), instead of a mean weighted by number of ratings. This is the
  literal reading of the brief.
- **"After 2000-01-01" read as exclusive (`>`).**

## SQL techniques used

`LEFT JOIN` · `COUNT(DISTINCT …)` · `GROUP BY` / `HAVING` · nested subqueries
(up to three levels) · `IN` with a subquery

## Data model

| Table | Grain |
|---|---|
| `books` | one row per book (title, pages, publication date, author, publisher) |
| `authors` | one row per author (collaborators are concatenated, e.g. "Author/Illustrator") |
| `publishers` | one row per publisher |
| `ratings` | one row per user × book rating (1–5) |
| `reviews` | one row per written review |

## Repository

```
book_catalog.ipynb   analysis, end to end
.env.example         template for the database connection variables
requirements.txt     Python dependencies
```

## Reproducing

The database is hosted by TripleTen and is only available to bootcamp students,
so the notebook is published **with its outputs already rendered**. To run it
against your own copy of the schema:

```bash
cp .env.example .env      # fill in DB_USER, DB_PASSWORD, DB_HOST, DB_NAME
pip install -r requirements.txt
jupyter lab book_catalog.ipynb
```

`.env` is git-ignored, so credentials never reach the repository.

## Stack

PostgreSQL · SQLAlchemy · pandas · Jupyter

---

*Marco Sitta · [LinkedIn](https://www.linkedin.com/in/marcositta)*
