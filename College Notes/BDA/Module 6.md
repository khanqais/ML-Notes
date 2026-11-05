# Module 6: Data Analytics with R — Exam Notes

> Obsidian notes for R data structures, workspace functions, vectors, sorting, subsetting, combining datasets, and data frames.

---

## 1. R Data Structures (10M)

### Vector
- **Definition:** One-dimensional structure storing elements of the same data type.
- **Created using:** `c()`
- **Example:**

```r
marks <- c(56, 75, 48, 69, 84)
marks
```

**Output:**

```r
[1] 56 75 48 69 84
```

- **Use case:** Marks of students, daily sales, prices.

### Factor
- **Definition:** Stores categorical data.
- **Example:**

```r
gender <- factor(c("Male", "Female", "Male", "Female"))
gender
```

**Output:**

```r
[1] Male   Female Male   Female
Levels: Female Male
```

- **Use case:** Gender, grade, product category.

### Matrix
- **Definition:** Two-dimensional structure with rows and columns; all elements must be of the same type.
- **Example:**

```r
m <- matrix(1:6, nrow = 2, ncol = 3)
m
```

**Output:**

```r
     [,1] [,2] [,3]
[1,]    1    3    5
[2,]    2    4    6
```

- **Use case:** Numeric tables, image data.

### Array
- **Definition:** Multi-dimensional extension of a matrix.
- **Example:**

```r
a <- array(1:12, dim = c(2, 3, 2))
a
```

- **Use case:** Data with layers, such as sales across years, months, and regions.

### List
- **Definition:** One-dimensional structure that can store different data types and structures.
- **Example:**

```r
student <- list(
  name = "Qais",
  marks = c(80, 90, 85),
  passed = TRUE
)
student
```

**Output:**

```r
$name
[1] "Qais"

$marks
[1] 80 90 85

$passed
[1] TRUE
```

- **Use case:** Storing mixed student or employee records.

### Data Frame
- **Definition:** Two-dimensional table where each column can have a different data type, but all columns have the same number of rows.
- **Example:**

```r
df <- data.frame(
  name = c("Amit", "Riya", "John"),
  age = c(21, 22, 23),
  marks = c(85, 90, 78)
)
df
```

**Output:**

```r
  name age marks
1 Amit  21    85
2 Riya  22    90
3 John  23    78
```

- **Use case:** Employee, student, sales, and project datasets.

### Comparison Table

| Structure | Dimensions | Data type | Created using |
|---|---:|---|---|
| Vector | 1 | Same type | `c()` |
| Factor | 1 | Categories | `factor()` |
| Matrix | 2 | Same type | `matrix()` |
| Array | 2+ | Same type | `array()` |
| List | 1 | Different types | `list()` |
| Data frame | 2 | Different types per column | `data.frame()` |

---

## 2. Workspace Functions (10M)

The R workspace is the current working environment containing objects such as vectors, matrices, lists, functions, and data frames.

### `ls()`
Lists all objects in the workspace.

```r
a <- 10
b <- 20
ls()
```

**Output:**

```r
[1] "a" "b"
```

### `rm()`
Removes one or more objects.

```r
rm(a)
ls()
```

**Output:**

```r
[1] "b"
```

Remove all objects:

```r
rm(list = ls())
```

### `getwd()`
Returns the current working directory.

```r
getwd()
```

### `setwd()`
Changes the current working directory.

```r
setwd("C:/RFiles")
```

### `save()`
Saves selected objects to an `.RData` file.

```r
save(df, file = "data.RData")
```

### `save.image()`
Saves the entire workspace.

```r
save.image("mywork.RData")
```

### `load()`
Loads saved R objects.

```r
load("data.RData")
```

### `q()`
Quits R.

```r
q()
```

---

## 3. Vector Sorting (10M)

### Question
Sort the vector `23, 45, 10, 34, 89, 20, 67, 99` in ascending and descending order.

### R Script

```r
x <- c(23, 45, 10, 34, 89, 20, 67, 99)

cat("Original vector:\n")
print(x)

cat("\nAscending order:\n")
print(sort(x))

cat("\nDescending order:\n")
print(sort(x, decreasing = TRUE))
```

### Output

```r
[1] 23 45 10 34 89 20 67 99

[1] 10 20 23 34 45 67 89 99

[1] 99 89 67 45 34 23 20 10
```

### Explanation
- `c()` creates a numeric vector.
- `sort(x)` sorts in ascending order by default.
- `sort(x, decreasing = TRUE)` sorts in descending order.
- In R, logical values are written in uppercase: `TRUE`, not `True`.

### Second Example

```r
x <- c(46, 23, 15, 38, 98, 56, 28, 78)

sort(x)
sort(x, decreasing = TRUE)
```

**Output:**

```r
[1] 15 23 28 38 46 56 78 98
[1] 98 78 56 46 38 28 23 15
```

---

## 4. Subsetting Operators (10M)

| Operator | Meaning | Example |
|---|---|---|
| `[]` | Select elements by position or condition | `x[2]` |
| `[[]]` | Select one element from a list | `mylist[[1]]` |
| `$` | Select a column from a data frame/list | `df$marks` |
| `:` | Create a sequence | `1:5` |
| `which()` | Return indexes satisfying a condition | `which(x > 40)` |
| `&` | AND | `df$a > 2 & df$b == 1` |
| `|` | OR | `df$a > 2 | df$b == 1` |
| `!` | NOT | `!(df$class == 2)` |

### Examples

```r
x <- c(10, 20, 30, 40, 50)

x[2]        # 20
x[c(1, 3)]  # 10 30
x[1:3]      # 10 20 30
x[x > 30]   # 40 50
x[-1]       # 20 30 40 50
which(x > 30) # 4 5
```

### Important Rule

```r
x[condition]       # returns values
which(condition)   # returns positions/indexes
```

---

## 5. Combining Datasets (10M)

| Function | Purpose |
|---|---|
| `c()` | Combines vectors into one vector |
| `cbind()` | Combines objects by columns |
| `rbind()` | Combines objects by rows |
| `merge()` | Merges data frames using a common column |
| `data.frame()` | Creates a data frame from vectors |

### Combine A and B using `c()`

```r
A <- c(6, 7, 8, 9)
B <- c(1, 2, 4, 5)

C <- c(A, B)
C
```

**Output:**

```r
[1] 6 7 8 9 1 2 4 5
```

### `cbind()` Example

```r
A <- c(1, 2, 3)
B <- c(4, 5, 6)

cbind(A, B)
```

**Output:**

```r
     A B
[1,] 1 4
[2,] 2 5
[3,] 3 6
```

### `rbind()` Example

```r
A <- c(1, 2, 3)
B <- c(4, 5, 6)

rbind(A, B)
```

**Output:**

```r
  [,1] [,2] [,3]
A    1    2    3
B    4    5    6
```

### Difference

- `cbind()` combines data **column-wise**.
- `rbind()` combines data **row-wise**.

### `merge()` Example

```r
students <- data.frame(id = c(1, 2, 3), name = c("A", "B", "C"))
marks <- data.frame(id = c(1, 2, 3), score = c(85, 90, 78))

merge(students, marks, by = "id")
```

**Output:**

```r
  id name score
1  1    A    85
2  2    B    90
3  3    C    78
```

---

## 6. Create Five Sales Vectors (10M)

```r
Bread <- c(12, 3, 5, 11, 9)
Milk <- c(21, 27, 18, 20, 15)
Cola_Cans <- c(10, 1, 33, 6, 12)
Chocolate_Bars <- c(6, 7, 4, 13, 12)
Detergent <- c(5, 8, 12, 20, 23)
```

### Vector Subsetting Examples

```r
Milk                 # Display Milk vector
Cola_Cans[3]         # Third element
Chocolate_Bars[Chocolate_Bars > 5]  # Values greater than 5
Detergent[c(2, 4)]   # Second and fourth elements
```

**Outputs:**

```r
[1] 21 27 18 20 15

[1] 33

[1]  6  7 13 12

[1]  8 20
```

---

## 7. Data Frame Subsetting: `subset()` and `[]`

### Sample Data Frame

```r
df <- data.frame(
  subject = 1:6,
  class = c(1, 2, 1, 2, 1, 2),
  marks = c(56, 75, 48, 69, 84, 53)
)

df
```

**Output:**

```r
  subject class marks
1       1     1    56
2       2     2    75
3       3     1    48
4       4     2    69
5       5     1    84
6       6     2    53
```

### Q: `subject < 4` using `subset()`

```r
subset1 <- subset(df, subject < 4)
subset1
```

**Output:**

```r
  subject class marks
1       1     1    56
2       2     2    75
3       3     1    48
```

### Q: `subject < 3 AND class == 2` using `[]`

```r
subset2 <- df[df$subject < 3 & df$class == 2, ]
subset2
```

**Output:**

```r
  subject class marks
2       2     2    75
```

### Explanation

- With `subset()`, write column names directly.
- With `[]`, use `df$column_name`.
- The comma after the condition selects all columns.

```r
df[condition, ]
```

### Second Pattern: `course` Data Frame

```r
df <- data.frame(
  course = 1:6,
  id = 11:16,
  class = c(1, 2, 1, 2, 1, 2),
  marks = c(56, 75, 48, 69, 84, 53)
)
```

### `course < 3` using `[]`

```r
df[df$course < 3, ]
```

**Output:**

```r
  course id class marks
1      1 11     1    56
2      2 12     2    75
```

### `course < 3 OR class == 2` using `subset()`

```r
subset(df, course < 3 | class == 2)
```

**Output:**

```r
  course id class marks
1      1 11     1    56
2      2 12     2    75
4      4 14     2    69
6      6 16     2    53
```

### `course < 5` using `[]`

```r
df[df$course < 5, ]
```

**Output:**

```r
  course id class marks
1      1 11     1    56
2      2 12     2    75
3      3 13     1    48
4      4 14     2    69
```

### `course < 4 OR class == 1` using `subset()`

```r
subset(df, course < 4 | class == 1)
```

**Output:**

```r
  course id class marks
1      1 11     1    56
2      2 12     2    75
3      3 13     1    48
5      5 15     1    84
```

---

## 8. Create Data Frame, `str()`, `summary()`, Extract and Filter (10M)

### Question
Create a data frame using employee vectors, display structure and summary, extract columns, and filter employees with salary <= 60000.

### R Script

```r
emp_id <- c(1:5)

emp_name <- c("Rick", "Dan", "Michelle", "Ryan", "Gary")

start_date <- c(
  "2012-01-01",
  "2013-09-23",
  "2014-11-15",
  "2014-05-11",
  "2015-03-27"
)

salary <- c(60000, 45000, 75000, 84000, 20000)

employee <- data.frame(emp_id, emp_name, start_date, salary)

employee
```

**Output:**

```r
  emp_id emp_name start_date salary
1      1     Rick 2012-01-01  60000
2      2      Dan 2013-09-23  45000
3      3 Michelle 2014-11-15  75000
4      4     Ryan 2014-05-11  84000
5      5     Gary 2015-03-27  20000
```

### Structure

```r
str(employee)
```

**Output:**

```r
'data.frame': 5 obs. of  4 variables:
 $ emp_id    : int  1 2 3 4 5
 $ emp_name  : chr  "Rick" "Dan" "Michelle" "Ryan" ...
 $ start_date: chr  "2012-01-01" "2013-09-23" "2014-11-15" ...
 $ salary    : num  60000 45000 75000 84000 20000
```

### Summary

```r
summary(employee)
```

**Output:**

```r
    emp_id        emp_name          start_date            salary      
 Min.   :1     Length:5           Length:5           Min.   :20000  
 1st Qu.:2     Class :character   Class :character   1st Qu.:45000  
 Median :3     Mode  :character   Mode  :character   Median :60000  
 Mean   :3                                           Mean   :56800  
 3rd Qu.:4                                           3rd Qu.:75000  
 Max.   :5                                           Max.   :84000  
```

### Extract `emp_name` and `salary`

```r
employee[, c("emp_name", "salary")]
```

**Output:**

```r
  emp_name salary
1     Rick  60000
2      Dan  45000
3 Michelle  75000
4     Ryan  84000
5     Gary  20000
```

### Filter salary <= 60000 using `subset()`

```r
subset(employee, salary <= 60000)
```

**Output:**

```r
  emp_id emp_name start_date salary
1      1     Rick 2012-01-01  60000
2      2      Dan 2013-09-23  45000
5      5     Gary 2015-03-27  20000
```

### Same using `[]`

```r
employee[employee$salary <= 60000, ]
```

---

## 9. Create Data Frame and Append Rows (10M)

### Employee Table

```r
emp_name <- c(
  "Vivek", "Karan", "James", "Soham", "Renu",
  "Farah", "Hetal", "Mary", "Ganesh", "Krish"
)

salary <- c(
  21000, 55000, 67000, 50000, 54000,
  40000, 30000, 70000, 20000, 15000
)

employees <- data.frame(emp_name, salary)
employees
```

**Output:**

```r
   emp_name salary
1     Vivek  21000
2     Karan  55000
3     James  67000
4     Soham  50000
5      Renu  54000
6     Farah  40000
7     Hetal  30000
8      Mary  70000
9    Ganesh  20000
10    Krish  15000
```

### Append Five New Employees

```r
new_employees <- data.frame(
  emp_name = c("Amit", "Priya", "Rahul", "Neha", "Arjun"),
  salary = c(25000, 45000, 60000, 35000, 50000)
)

employees <- rbind(employees, new_employees)
employees
```

### Explanation

- `data.frame()` creates a table.
- `rbind()` appends new rows below existing rows.
- The new data frame must have the same column names and compatible data types.

---

## 10. Project Data Frame + `str()` + `summary()` (10M)

### ABC Corp

```r
project_id <- c(1, 2, 3, 4, 5)

project_name <- c(
  "Website Redesign",
  "Mobile App Launch",
  "Data Migration",
  "AI Development",
  "Cybersecurity Audit"
)

budget <- c(150000, 100000, 80000, 200000, 50000)

projects <- data.frame(project_id, project_name, budget)

projects
str(projects)
summary(projects)
```

**Output:**

```r
  project_id         project_name budget
1          1    Website Redesign 150000
2          2   Mobile App Launch 100000
3          3       Data Migration  80000
4          4        AI Development 200000
5          5  Cybersecurity Audit  50000
```

### Explanation

- `str()` shows the number of observations, variables, and column data types.
- `summary()` gives statistical summaries for numeric columns.

---

## 11. Project Data Frame + Append New Projects (10M)

### XYZ Ltd.

```r
project_id <- c(1, 2)

project_name <- c("CRM Implementation", "Data Analytics")

budget <- c(120000, 90000)

status <- c("In Progress", "In Progress")

company <- data.frame(project_id, project_name, budget, status)
company
```

**Output:**

```r
  project_id       project_name budget      status
1          1 CRM Implementation 120000 In Progress
2          2     Data Analytics  90000 In Progress
```

### Append Two New Projects

```r
new_projects <- data.frame(
  project_id = c(3, 4),
  project_name = c("UX Research", "Cloud Integration"),
  budget = c(160000, 190000),
  status = c("Not Started", "Not Started")
)

company <- rbind(company, new_projects)
company
```

**Final Output:**

```r
  project_id       project_name budget      status
1          1 CRM Implementation 120000 In Progress
2          2     Data Analytics  90000 In Progress
3          3        UX Research 160000 Not Started
4          4  Cloud Integration 190000 Not Started
```

---

## 12. Students Data Frame Practice (10M)

### Question
Create a student data frame, extract columns, and filter students with marks greater than 75.

### R Script

```r
roll_no <- c(1, 2, 3, 4, 5)
name <- c("Amit", "Riya", "John", "Sara", "Mohit")
marks <- c(85, 72, 90, 60, 78)

students <- data.frame(roll_no, name, marks)
students
```

**Output:**

```r
  roll_no  name marks
1       1  Amit    85
2       2  Riya    72
3       3  John    90
4       4  Sara    60
5       5 Mohit    78
```

### Extract `name` using `$`

```r
students$name
```

**Output:**

```r
[1] "Amit"  "Riya"  "John"  "Sara"  "Mohit"
```

### Extract `roll_no` and `marks`

```r
students[, c("roll_no", "marks")]
```

**Output:**

```r
  roll_no marks
1       1    85
2       2    72
3       3    90
4       4    60
5       5    78
```

### Filter marks > 75 using `subset()`

```r
subset(students, marks > 75)
```

**Output:**

```r
  roll_no  name marks
1       1  Amit    85
3       3  John    90
5       5 Mohit    78
```

### Same using `[]`

```r
students[students$marks > 75, ]
```

---

## 13. Quick Revision

### Vector Sorting

```r
sort(x)
sort(x, decreasing = TRUE)
```

### Combining Data

```r
C <- c(A, B)          # Combine into one vector
cbind(A, B)           # Combine by columns
rbind(A, B)           # Combine by rows
merge(df1, df2, by = "id")  # Merge by common column
```

### Data Frame Filtering

```r
subset(df, condition)
df[df$column condition, ]
```

### Logical Operators

| Operator | Meaning |
|---|---|
| `==` | Equal to |
| `!=` | Not equal to |
| `<` | Less than |
| `>` | Greater than |
| `<=` | Less than or equal to |
| `>=` | Greater than or equal to |
| `&` | AND |
| `|` | OR |
| `!` | NOT |

### Common Mistakes

- Write `TRUE`, not `True`.
- Use `df$column` inside `[]`.
- Do not forget the comma in `df[condition, ]`.
- Use commas between columns inside `data.frame()`.
- Use `rbind()` to append rows.
- Use `c()` to combine vectors into one vector.
