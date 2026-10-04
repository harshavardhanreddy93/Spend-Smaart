## Project Check-in 1: Scope, Schema, and Strategy

**Fall 2026 | Due Week 7**

**Name:** Harsha Vardhan reddy kallu

**ID:** Q294V924

**Instructor:** Coday Farlow

**Date:** 10/04/2026

---

# 1. Problem Definition and Mobile Scope

## Problem

College students often lose track of small, everyday expenses such as coffee, snacks, and transport, and at the end of the month they do not know where their money went. Existing finance apps are either too complex or depend on linking a bank account. SpendSmart is a simple Android app that lets a student log expenses in seconds, sort them into categories, and compare spending against a monthly budget for each category.

## Target Platform

Android (native mobile application).

## Scope for the Semester

**In scope:**

- Add, edit, and delete expenses
- Create categories with a monthly budget
- View expense history with category names
- See monthly totals per category compared with the budget

---

# 2. Initial Database Design and Mechanics

## Tables and Keys

- **Category** – primary key: category_id
- **Expense** – primary key: expense_id; foreign key: category_id references Category(category_id)

## SQL

```sql
CREATE TABLE Category (
  category_id    INTEGER PRIMARY KEY,
  name           TEXT NOT NULL,
  monthly_budget REAL
);

CREATE TABLE Expense (
  expense_id   INTEGER PRIMARY KEY,
  category_id  INTEGER NOT NULL,
  amount       REAL NOT NULL,
  expense_date TEXT NOT NULL,
  note         TEXT,
  FOREIGN KEY (category_id) REFERENCES Category(category_id)
);
```

## Logical Relationship

The relationship is one-to-many. One Category can have many Expenses, and every Expense belongs to exactly one Category through the foreign key category_id. This link lets the app show each expense with its category name and total spending per category.

## Five Example Queries in Relational Algebra

**Query 1: Large purchases alert**

- Relational algebra: `π note, amount (σ amount > 50 (Expense))`
- Purpose: Lists expenses over 50 dollars.

```sql
SELECT note, amount
FROM Expense
WHERE amount > 50;
```

**Query 2: Expense history screen**

- Relational algebra: `π expense_date, amount, name (Expense ⋈ Category)`
- Purpose: Shows each expense with its category name.

```sql
SELECT e.expense_date, e.amount, c.name
FROM Expense e
JOIN Category c ON e.category_id = c.category_id;
```

**Query 3: Budget overview**

- Relational algebra: `π name (σ monthly_budget > 200 (Category))`
- Purpose: Finds categories with budgets above 200 dollars.

```sql
SELECT name
FROM Category
WHERE monthly_budget > 200;
```

**Query 4: Category detail screen**

- Relational algebra: `π expense_date, amount (σ name = 'Food' (Expense ⋈ Category))`
- Purpose: Shows all Food spending.

```sql
SELECT e.expense_date, e.amount
FROM Expense e
JOIN Category c ON e.category_id = c.category_id
WHERE c.name = 'Food';
```

**Query 5: Unused category suggestion**

- Relational algebra: `π category_id (Category) − π category_id (Expense)`
- Purpose: Finds categories with no expenses yet.

```sql
SELECT category_id
FROM Category
EXCEPT
SELECT category_id
FROM Expense;
```

**Notes:**

- Queries 2 and 4 use a natural join on category_id, which is how the foreign key connects the two tables.
- Query 5 uses set difference (EXCEPT in SQL) to find category_ids that appear in Category but not in Expense.

---

# 3. AI Utilization Plan

## Agents and Tools

- **Claude:** explaining concepts, quizzing me, and reviewing my reasoning after I attempt a task myself.
- **SQLite / DB Fiddle:** running and testing my SQL so I verify results myself rather than trusting AI output.

## Example Prompts

- "Explain the difference between selection (σ) and projection (π) in relational algebra using a small example, then quiz me with three questions."
- "Here is my relational algebra query and what I think it returns. Do not rewrite it. Tell me whether my logic is correct and explain why."
- "My CREATE TABLE statement gives a foreign key error. Explain what the error message means and what to check, but do not give me the corrected code."
- "What are the tradeoffs of storing dates as TEXT versus INTEGER in SQLite? Help me decide, but let me make the final choice."

## Building Self-Reliance

I will always attempt each task on my own first, then use AI as a tutor to check my understanding instead of as a generator. I will not ask AI to produce my whole schema or app. After getting feedback, I will test every SQL statement myself, fix mistakes on my own, and keep a short log of what I learned and what I changed. I will only submit work I can explain in my own words.
