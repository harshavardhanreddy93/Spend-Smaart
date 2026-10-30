# Project Check-in 2: Normalization and Architectural Evolution

**Fall 2026 | Due Week 11 | SpendSmart (Android)**  
**Name:** Kallu Harsha Vardhan Reddy   **Student ID:** Q294V924

---

## 1. Schema Evolution

### From Check-in 1 to the Expanded Schema
Check-in 1 used two tables, `Category` and `Expense`, linked by the foreign key `category_id`. As SpendSmart grows, students need to record how they paid (cash, card, mobile wallet) and to label expenses with flexible tags (for example "campus" or "caffeine") that cut across categories. Neither fits in the original two tables, so the schema expands to five.

| Table | Status | Why it exists |
| :--- | :--- | :--- |
| **Category** | Kept from Check-in 1 | Spending category with a monthly budget |
| **Expense** | Kept; added `method_id` foreign key | One logged purchase |
| **Payment_Method** | New | Lookup of how the expense was paid |
| **Tag** | New | Reusable labels that cut across categories |
| **Expense_Tag** | New (junction table) | Resolves the many-to-many link between Expense and Tag |

### Table Relationships
* `Category` 1 : N `Expense` (one category has many expenses; each expense has exactly one category)
* `Payment_Method` 1 : N `Expense` (each expense is paid with exactly one method)
* `Expense` M : N `Tag`, implemented as `Expense` 1 : N `Expense_Tag` and `Tag` 1 : N `Expense_Tag`

*The Entity-Relationship diagram for this schema is included in Check-in 3.*

---

## 2. Step-by-Step Normalization Demonstration
The same three sample expenses are used at every step. Dates, amounts, notes, categories, methods, and tags never change, so you can verify that splitting tables loses no information and the schema keeps its purpose: tracking expenses against category budgets.

### Step 0: Early Unnormalized Design
My first expanded design put everything into one table. The tags column stores several values in one cell.

| expense_id | expense_date | amount | note | category_id | category_name | monthly_budget | method_id | method_name | tags |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| 1 | 2026-09-02 | 4.50 | Latte | 1 | Food | 200 | 1 | Card | caffeine, campus |
| 2 | 2026-09-03 | 12.00 | Bus top-up | 2 | Transport | 60 | 2 | Cash | commute |
| 3 | 2026-09-05 | 8.25 | Lunch | 1 | Food | 200 | 1 | Card | campus |

> **Violation:** the `tags` column is not atomic, so it breaks First Normal Form[cite: 7].

---

### Step 1: First Normal Form (1NF)
**Rule:** every cell holds one atomic value and there are no repeating groups. Each tag moves to its own row. The primary key becomes the composite key `(expense_id, tag_id)`[cite: 7].

| expense_id | tag_id | tag_name | expense_date | amount | note | category_id | category_name | monthly_budget | method_id | method_name |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| 1 | 1 | caffeine | 2026-09-02 | 4.50 | Latte | 1 | Food | 200 | 1 | Card[cite: 7] |
| 1 | 2 | campus | 2026-09-02 | 4.50 | Latte | 1 | Food | 200 | 1 | Card[cite: 7] |
| 2 | 3 | commute | 2026-09-03 | 12.00 | Bus top-up | 2 | Transport | 60 | 2 | Cash[cite: 7] |
| 3 | 2 | campus | 2026-09-05 | 8.25 | Lunch | 1 | Food | 200 | 1 | Card[cite: 7] |

#### Anomalies in 1NF (caused by partial dependencies)
* **Update anomaly:** Expense 1 appears twice. Correcting the Latte amount to 4.75 requires editing both rows; missing one leaves contradictory data[cite: 7].
* **Insertion anomaly:** I cannot create a new tag such as "weekend" until an expense uses it, because `tag_id` is part of the primary key and `expense_id` cannot be empty[cite: 7].
* **Deletion anomaly:** Expense 2 (Bus top-up) has only one row. Removing its tag "commute" deletes the expense itself, and the tag too[cite: 7].

---

### Step 2: Second Normal Form (2NF)
**Rule:** 1NF plus no partial dependencies. Expense details depend only on `expense_id` and `tag_name` depends only on `tag_id`, not on the whole composite key, so each group gets its own table[cite: 7].

#### Expense (PK: expense_id)
| expense_id | expense_date | amount | note | category_id | category_name | monthly_budget | method_id | method_name |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| 1 | 2026-09-02 | 4.50 | Latte | 1 | Food | 200 | 1 | Card[cite: 7] |
| 2 | 2026-09-03 | 12.00 | Bus top-up | 2 | Transport | 60 | 2 | Cash[cite: 7] |
| 3 | 2026-09-05 | 8.25 | Lunch | 1 | Food | 200 | 1 | Card[cite: 7] |

#### Tag (PK: tag_id)
| tag_id | tag_name |
| :---: | :--- |
| 1 | caffeine[cite: 7] |
| 2 | campus[cite: 7] |
| 3 | commute[cite: 7] |

#### Expense_Tag (PK: expense_id, tag_id)
| expense_id | tag_id |
| :---: | :---: |
| 1 | 1[cite: 7] |
| 1 | 2[cite: 7] |
| 2 | 3[cite: 7] |
| 3 | 2[cite: 7] |

The tag anomalies are resolved: a tag can exist with no expenses, and deleting a link row no longer deletes an expense[cite: 7].

#### Anomalies remaining in Expense (caused by transitive dependencies)
* **Update anomaly:** Raising the Food budget from 200 to 250 means editing every Food expense row (expenses 1 and 3)[cite: 7].
* **Insertion anomaly:** I cannot add a category "Books" with a $40 budget until an expense exists for it[cite: 7].
* **Deletion anomaly:** Deleting the only Transport expense (expense 2) also erases the Transport category and its $60 budget. The same happens to the Cash payment method[cite: 7].

---

### Step 3: Third Normal Form (3NF)
**Rule:** 2NF plus no transitive dependencies. Every non-key attribute must depend on the key, the whole key, and nothing but the key. The transitive chains are `expense_id → category_id → (category_name, monthly_budget)` and `expense_id → method_id → method_name`, so each chain moves to its own table[cite: 7].

#### Category (PK: category_id)
| category_id | name | monthly_budget |
| :---: | :--- | :---: |
| 1 | Food[cite: 7] | 200[cite: 7] |
| 2 | Transport[cite: 7] | 60[cite: 7] |

#### Payment_Method (PK: method_id)
| method_id | method_name |
| :---: | :--- |
| 1 | Card[cite: 7] |
| 2 | Cash[cite: 7] |

#### Expense (PK: expense_id; FK: category_id, method_id)
| expense_id | category_id | method_id | amount | expense_date | note |
| :---: | :---: | :---: | :---: | :---: | :--- |
| 1 | 1 | 1 | 4.50 | 2026-09-02 | Latte[cite: 7] |
| 2 | 2 | 2 | 12.00 | 2026-09-03 | Bus top-up[cite: 7] |
| 3 | 1 | 1 | 8.25 | 2026-09-05 | Lunch[cite: 7] |

*Tag and Expense_Tag are unchanged from 2NF; they already satisfy 3NF[cite: 7].*

---

### Final 3NF schema (SQL)
```sql
CREATE TABLE Category (
  category_id    INTEGER PRIMARY KEY,
  name           TEXT NOT NULL UNIQUE,
  monthly_budget REAL CHECK (monthly_budget >= 0)
);
 
CREATE TABLE Payment_Method (
  method_id   INTEGER PRIMARY KEY,
  method_name TEXT NOT NULL UNIQUE
);
 
CREATE TABLE Expense (
  expense_id   INTEGER PRIMARY KEY,
  category_id  INTEGER NOT NULL,
  method_id    INTEGER NOT NULL,
  amount       REAL NOT NULL CHECK (amount > 0),
  expense_date TEXT NOT NULL,
  note         TEXT,
  FOREIGN KEY (category_id) REFERENCES Category(category_id),
  FOREIGN KEY (method_id)   REFERENCES Payment_Method(method_id)
);
 
CREATE TABLE Tag (
  tag_id   INTEGER PRIMARY KEY,
  tag_name TEXT NOT NULL UNIQUE
);
 
CREATE TABLE Expense_Tag (
## 3. The Problem-Solving Journey

### Functional Dependencies

Functional dependencies found in the data, and the normal form each one violated in the early designs:

| Functional dependency | Type | Problem it caused |
|---|---|---|
| `expense_id → expense_date, amount, note, category_id, method_id` | Partial (on part of key `(expense_id, tag_id)`) | Repeated expense data per tag; update anomaly (1NF → 2NF) |
| `tag_id → tag_name` | Partial | Tag cannot exist without an expense; insertion anomaly (1NF → 2NF) |
| `category_id → name, monthly_budget` | Transitive via `expense_id → category_id` | Budget repeated in every row; cannot add a category alone (2NF → 3NF) |
| `method_id → method_name` | Transitive via `expense_id → method_id` | Method name repeated; lost when last expense deleted (2NF → 3NF) |

Dependencies in the final 3NF tables (left side is always a key):

| Table | Functional dependencies |
|---|---|
| `Category` | `category_id → name, monthly_budget` |
| `Payment_Method` | `method_id → method_name` |
| `Tag` | `tag_id → tag_name` |
| `Expense` | `expense_id → category_id, method_id, amount, expense_date, note` |
| `Expense_Tag` | `(expense_id, tag_id)` is the whole key; no non-key attributes |

### Anomalies Encountered and How 3NF Resolves Them

| Anomaly | Early design | Final 3NF structure |
|---|---|---|
| **Update** | Changing Food budget 200 → 250 edited every Food expense row; changing the Latte amount edited two rows | Budget lives in one `Category` row; amount lives in one `Expense` row |
| **Insertion** | Could not add the category "Books" or the tag "weekend" without an expense | Insert directly into `Category` or `Tag` |
| **Deletion** | Deleting the only Transport expense erased the Transport category and its $60 budget | `Category`, `Payment_Method`, and `Tag` rows exist independently of expenses |

### Reflection on My Troubleshooting Process

My first expanded design put everything into one table because it felt faster to build alongside the app. The first red flag was the `tags` column: one cell held several values, and I could not cleanly query "all expenses tagged campus." Splitting tags into rows fixed that, but then updating one expense's amount meant editing multiple rows. That made me ask what each column actually depends on, and writing out the functional dependencies is what made it click: expense details depend only on `expense_id`, not on the whole composite key. After 2NF I thought I was done, until I imagined adding a new category before any spending existed and saw I could not. That exposed the transitive dependency through `category_id`. The hardest part was not the mechanics but learning to ask "what determines this value?" for every column. Next time I would write the functional dependencies before drawing any tables.
EOF_DOC

git config user.name "$GH_USER"
git config user.email "khvr220033@gmail.com"
git add "Project check in-2.md"
git commit -q -m "Update Project check in-2 with code formatting" || echo "No changes to commit"
git push -q origin main
git remote set-url origin "https://github.com/${GH_USER}/${GH_REPO}.git"

echo "Done: https://github.com/${GH_USER}/${GH_REPO}/blob/main/Project%20check%20in-2.md"
  expense_id INTEGER NOT NULL,
  tag_id     INTEGER NOT NULL,
  PRIMARY KEY (expense_id, tag_id),
  FOREIGN KEY (expense_id) REFERENCES Expense(expense_id) ON DELETE CASCADE,
  FOREIGN KEY (tag_id)     REFERENCES Tag(tag_id) ON DELETE CASCADE
);
