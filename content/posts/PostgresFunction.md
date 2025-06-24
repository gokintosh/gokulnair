---
title: "Supercharging Your PostgreSQL with Functions & Stored Procedures"
date: 2025-01-01T12:00:00+00:00
draft: false
tags: ["PostgreSQL", "Database"]
categories: ["Programming"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
UseHugoToc: true
---


Picture this: You’re juggling the same SQL queries day in and day out, feeling like Bill Murray in *Groundhog Day*. If only you had a way to *wrap* your repetitive queries into neatly packaged, reusable code blocks… Well, guess what? **PostgreSQL** has got your back with its **Functions** and **Stored Procedures**!

In this post, we’ll explore what these two can do for you, how to build them, and why they might just become your new best friends (no offense to your dog, of course).

---

## 1. Functions vs. Stored Procedures: A Quick Comparison

In PostgreSQL:

- **Functions**:  
  - Must return a value (kinda like a boomerang—you throw something in, it always comes back).  
  - Invoked using `SELECT function_name(args)` or as part of a query.  
  - Great for computations, data retrieval, and a quick *“show me the money!”* scenario.

- **Stored Procedures**:  
  - Don’t necessarily need to return anything (like a magical black box doing all sorts of tasks).  
  - Invoked with `CALL procedure_name(args)`.  
  - Perfect for multi-step operations that might update data, insert logs, or coordinate a disco party in the break room.

### Key Differences

|                  | **Function**                              | **Stored Procedure**                                   |
|------------------|-------------------------------------------|--------------------------------------------------------|
| **Return Value** | **Must** return something (scalar/table)  | Doesn’t have to return anything (but can)             |
| **Usage**        | Ideal for computations/data retrieval      | Ideal for complex business logic & transaction flow   |
| **Call Syntax**  | `SELECT my_function(params);`             | `CALL my_procedure(params);`                          |

---

## 2. Creating Your First PostgreSQL Function

![Bartender Mixing GIF](https://media.giphy.com/media/5GoVLqeAOo6PK/giphy.gif)

Let’s say you have an `employees` table and you want to get the salary of a specific employee. Instead of typing that **same** `SELECT` statement repeatedly, you can create a function:

```sql
CREATE OR REPLACE FUNCTION get_employee_salary(emp_id INT)
RETURNS NUMERIC AS $$
BEGIN
    RETURN (SELECT salary FROM employees WHERE id = emp_id);
END;
$$ LANGUAGE plpgsql;
