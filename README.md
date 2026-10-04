Project Check-in 1: Scope, Schema, and Strategy

Fall 2026 | Due Week 7

Name: Harsha Vardhan Reddy Kallu
ID: Q294V924
Instructor: Coday Farlow
Date: October 4, 2026

1. Problem Definition and Mobile Scope
Problem

College students often lose track of small, everyday expenses such as coffee, snacks, and transportation. At the end of the month, they may not know where their money went.

SpendSmart is a simple Android app that lets a student:

Log expenses in seconds

Sort expenses into categories

Compare spending against a monthly budget for each category

The goal is to provide a simple alternative to finance applications that may be too complex or require users to link a bank account.

Target Platform

Android (native mobile application)

Scope for the Semester
In Scope

Add, edit, and delete expenses

Create categories with a monthly budget

View expense history with category names

See monthly totals per category compared with the budget

2. Initial Database Design and Mechanics
Tables and Keys
Category

Primary Key: category_id

Expense

Primary Key: expense_id

Foreign Key: category_id references Category(category_id)

SQL Schema
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

Logical Relationship

The relationship between Category and Expense is one-to-many.

One Category can have many Expenses, and every Expense belongs to exactly one Category through the foreign key category_id.

This relationship allows the app to:

Show each expense with its category name

Calculate total spending per category

Compare category spending with the monthly budget

Five Example Queries in Relational Algebra
S. No.	Relational Algebra	App Feature	Purpose
1	π note, amount (σ amount > 50 (Expense))	Large Purchases Alert	Lists expenses over $50
2	π expense_date, amount, name (Expense ⋈ Category)	Expense History Screen	Shows each expense with its category name
3	π name (σ monthly_budget > 200 (Category))	Budget Overview	Finds categories with budgets above $200
4	π expense_date, amount (σ name = 'Food' (Expense ⋈ Category))	Category Detail Screen	Shows all Food spending
5	π category_id (Category) − π category_id (Expense)	Unused Category Suggestion	Finds categories with no expenses yet
Explanation of Queries

Queries 2 and 4 use a natural join on category_id, which is how the foreign key connects the two tables.

Query 5 uses set difference to find category_id values that appear in Category but not in Expense.

3. AI Utilization Plan
Agents and Tools

Claude: Explaining concepts, quizzing me, and reviewing my reasoning after I attempt a task myself.

SQLite / DB Fiddle: Running and testing my SQL so I can verify results myself rather than trusting AI output.

Example Prompts

"Explain the difference between selection (σ) and projection (π) in relational algebra using a small example, then quiz me with three questions."

"Here is my relational algebra query and what I think it returns. Do not rewrite it. Tell me whether my logic is correct and explain why."

"My CREATE TABLE statement gives a foreign key error. Explain what the error message means and what to check, but do not give me the corrected code."

"What are the tradeoffs of storing dates as TEXT versus INTEGER in SQLite? Help me decide, but let me make the final choice."

Building Self-Reliance

I will always attempt each task on my own first, then use AI as a tutor to check my understanding instead of as a generator.

I will not ask AI to produce my whole schema or app. After getting feedback, I will test every SQL statement myself, fix mistakes on my own, and keep a short log of what I learned and what I changed.

I will only submit work that I can explain in my own words.
