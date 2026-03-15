# EmployeeAnalysis

import sqlite3
import random
import pandas as pd

# connect to database
conn = sqlite3.connect("employees.db")
cursor = conn.cursor()

# create table
cursor.execute("""
CREATE TABLE IF NOT EXISTS employees(
    emp_id INTEGER PRIMARY KEY,
    name TEXT,
    department TEXT,
    salary REAL,
    years_experience INTEGER,
    performance_score REAL
)
""")

departments = ["Engineering", "Sales", "Marketing", "HR", "Finance"]

# insert 40 employees
for i in range(1, 41):
    name = f"Employee_{i}"
    dept = random.choice(departments)
    salary = random.randint(50000, 150000)
    exp = random.randint(1, 15)
    perf = round(random.uniform(1.0, 5.0), 2)

    cursor.execute(
        "INSERT INTO employees VALUES (?, ?, ?, ?, ?, ?)",
        (i, name, dept, salary, exp, perf)
    )

conn.commit()

# Query 1
query1 = """
SELECT name, department, salary, performance_score
FROM employees
WHERE performance_score >= 4.0 AND years_experience >= 3
ORDER BY performance_score DESC
LIMIT 15
"""

df_query1 = pd.read_sql_query(query1, conn)
print(df_query1)


# Query 2
query2 = """
SELECT *
FROM employees
WHERE salary BETWEEN 70000 AND 110000
AND (department='Engineering' OR department='Sales')
ORDER BY department, salary DESC
"""

df_query2 = pd.read_sql_query(query2, conn)
print(df_query2)


# Load full table
df = pd.read_sql_query("SELECT * FROM employees", conn)

# Average performance by department
print(df.groupby("department")["performance_score"].mean())


# Same as Query 1 using Pandas
result = df[
    (df["performance_score"] >= 4.0) &
    (df["years_experience"] >= 3)
][["name", "department", "salary", "performance_score"]]

result = result.sort_values(by="performance_score", ascending=False).head(15)

print(result)

conn.close()
