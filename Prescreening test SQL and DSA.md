# SQL Server & DSA Prescreening Questions - 100 MCQs

# Part A: SQL Server (50 Questions)

## Basic SQL Queries & Syntax (15 Questions)

**Question 1:**
```sql
SELECT * FROM Users WHERE Name = NULL
```
What's wrong with this query?
A) Nothing wrong B) Should use IS NULL C) Should use == NULL D) NULL is not a valid value

**Question 2:**
```sql
SELECT TOP 5 * FROM Products ORDER BY Price
```
This query will return:
A) 5 most expensive products B) 5 cheapest products C) Random 5 products D) Syntax error

**Question 3:**
What's the difference between `WHERE` and `HAVING` clauses?
A) No difference B) WHERE filters rows, HAVING filters groups C) HAVING is faster D) WHERE works with aggregates

**Question 4:**
```sql
SELECT COUNT(*) FROM Users WHERE Age > 25
```
If there are NULLs in Age column, what happens?
A) COUNT includes NULLs B) COUNT excludes NULLs C) Error occurs D) COUNT returns NULL

**Question 5:**
```sql
SELECT Name, Age FROM Users
UNION
SELECT Name, Age FROM Customers
```
What does UNION do by default?
A) Includes duplicates B) Removes duplicates C) Causes error D) Sorts results

**Question 6:**
Which is faster for checking existence?
A) `SELECT COUNT(*) > 0` B) `SELECT * WHERE EXISTS()` C) `IF EXISTS()` D) All are same

**Question 7:**
```sql
SELECT * FROM Products WHERE Price BETWEEN 100 AND 200
```
Is the range inclusive or exclusive?
A) Exclusive B) Inclusive C) Left inclusive, right exclusive D) Depends on data type

**Question 8:**
What's the correct syntax for case-insensitive search?
A) `WHERE Name = 'john'` B) `WHERE LOWER(Name) = 'john'` C) `WHERE Name LIKE 'john'` D) Both B and C

**Question 9:**
```sql
SELECT Name, ROW_NUMBER() OVER (ORDER BY Age) as RowNum FROM Users
```
What does ROW_NUMBER() return for duplicate Ages?
A) Same number B) Different sequential numbers C) NULL D) Error

**Question 10:**
Which SQL clause is executed first?
A) SELECT B) WHERE C) FROM D) ORDER BY

**Question 11:**
```sql
SELECT * FROM Users WHERE Name LIKE '%John%'
```
What does this query do?
A) Exact match 'John' B) Names starting with John C) Names containing John D) Names ending with John

**Question 12:**
What's the difference between `INNER JOIN` and `CROSS JOIN`?
A) No difference B) INNER JOIN needs ON clause C) CROSS JOIN is faster D) INNER JOIN returns fewer rows

**Question 13:**
```sql
SELECT DISTINCT Name FROM Users
```
If Name column has NULLs, what happens?
A) NULLs are excluded B) Single NULL is included C) Multiple NULLs included D) Error occurs

**Question 14:**
Which aggregate function ignores NULL values?
A) COUNT(*) B) COUNT(column) C) Both A and B D) Neither

**Question 15:**
```sql
SELECT * FROM Products ORDER BY Price DESC LIMIT 10
```
What's wrong with this query in SQL Server?
A) Nothing wrong B) Should use TOP instead of LIMIT C) DESC is wrong D) * is not allowed

## Joins & Relationships (10 Questions)

**Question 16:**
```sql
SELECT u.Name, o.OrderDate
FROM Users u
LEFT JOIN Orders o ON u.ID = o.UserID
```
What will this return?
A) Only users with orders B) All users, with/without orders C) Only orders D) All users and orders

**Question 17:**
Given: Users(10 records), Orders(20 records). What's the maximum possible result of CROSS JOIN?
A) 30 B) 200 C) 10 D) 20

**Question 18:**
```sql
SELECT *
FROM TableA a
FULL OUTER JOIN TableB b ON a.ID = b.ID
```
This returns:
A) Records in both tables B) All records from both tables C) Only matching records D) No records

**Question 19:**
What's a self-join used for?
A) Joining table to itself B) Recursive queries C) Hierarchical data D) All of the above

**Question 20:**
```sql
SELECT u.Name, COUNT(o.ID) as OrderCount
FROM Users u
LEFT JOIN Orders o ON u.ID = o.UserID
GROUP BY u.Name
```
What happens to users with no orders?
A) Excluded from result B) Show OrderCount = 0 C) Show OrderCount = NULL D) Causes error

**Question 21:**
Which join type is most efficient for large tables?
A) INNER JOIN B) LEFT JOIN C) RIGHT JOIN D) Depends on data distribution

**Question 22:**
```sql
SELECT * 
FROM Products p1, Products p2 
WHERE p1.CategoryID = p2.CategoryID
```
This is equivalent to:
A) INNER JOIN B) CROSS JOIN C) LEFT JOIN D) SELF JOIN

**Question 23:**
What's the difference between `LEFT JOIN` and `LEFT OUTER JOIN`?
A) LEFT JOIN is faster B) Different syntax for same operation C) LEFT OUTER JOIN includes NULLs D) No difference

**Question 24:**
In a many-to-many relationship, what do you need?
A) Foreign key B) Primary key C) Junction table D) Index

**Question 25:**
```sql
SELECT u.Name
FROM Users u
WHERE u.ID IN (SELECT UserID FROM Orders WHERE Amount > 1000)
```
This can be rewritten using:
A) INNER JOIN B) EXISTS C) LEFT JOIN D) Both A and B

## Indexes & Performance (8 Questions)

**Question 26:**
What's the primary purpose of an index?
A) Store data B) Speed up queries C) Ensure uniqueness D) Save space

**Question 27:**
Which type of query benefits most from indexing?
A) INSERT B) UPDATE C) DELETE D) SELECT

**Question 28:**
```sql
CREATE INDEX IX_Users_Name ON Users(Name)
```
What type of index is this?
A) Clustered B) Non-clustered C) Unique D) Primary

**Question 29:**
How many clustered indexes can a table have?
A) 0 B) 1 C) 5 D) Unlimited

**Question 30:**
```sql
SELECT * FROM Users WHERE UPPER(Name) = 'JOHN'
```
Why might this query not use an index on Name?
A) UPPER function makes index unusable B) Syntax error C) No index needed D) * prevents index usage

**Question 31:**
What's a covering index?
A) Index on all columns B) Index that includes all needed columns for a query C) Primary key index D) Clustered index

**Question 32:**
Which scenario might cause index fragmentation?
A) Frequent SELECT B) Frequent INSERT/DELETE C) Large table size D) Multiple indexes

**Question 33:**
What's the downside of having too many indexes?
A) Slower SELECT B) Slower INSERT/UPDATE/DELETE C) More storage D) Both B and C

## Stored Procedures & Functions (7 Questions)

**Question 34:**
```sql
CREATE PROCEDURE GetUsersByAge(@MinAge INT)
AS
BEGIN
    SELECT * FROM Users WHERE Age >= @MinAge
END
```
What's the advantage over dynamic SQL?
A) Better performance B) SQL injection prevention C) Reusability D) All of the above

**Question 35:**
What's the difference between stored procedure and function?
A) No difference B) Function must return value C) Procedure is faster D) Function can modify data

**Question 36:**
```sql
DECLARE @Count INT
EXEC GetUserCount @Count OUTPUT
```
What does OUTPUT parameter do?
A) Input parameter B) Return value from procedure C) Optional parameter D) Error handling

**Question 37:**
Can a function call a stored procedure?
A) Yes, always B) No, never C) Only scalar functions D) Only table-valued functions

**Question 38:**
What's the maximum number of parameters in a stored procedure?
A) 1024 B) 2100 C) Unlimited D) 255

**Question 39:**
```sql
CREATE FUNCTION GetUserAge(@UserID INT) RETURNS INT
AS
BEGIN
    DECLARE @Age INT
    SELECT @Age = Age FROM Users WHERE ID = @UserID
    RETURN @Age
END
```
What type of function is this?
A) Scalar B) Table-valued C) Inline D) System

**Question 40:**
Which statement is true about functions?
A) Can use TRY-CATCH B) Can modify table data C) Cannot call other functions D) Must return a value

## Transactions & Concurrency (10 Questions)

**Question 41:**
```sql
BEGIN TRANSACTION
    UPDATE Users SET Balance = Balance - 100 WHERE ID = 1
    UPDATE Users SET Balance = Balance + 100 WHERE ID = 2
COMMIT TRANSACTION
```
What ACID property does this ensure?
A) Atomicity B) Consistency C) Isolation D) Durability

**Question 42:**
What happens if you don't explicitly commit a transaction?
A) Auto-commit B) Auto-rollback C) Depends on isolation level D) Connection timeout

**Question 43:**
```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED
```
What problem can this cause?
A) Deadlocks B) Dirty reads C) Phantom reads D) Lost updates

**Question 44:**
Which isolation level prevents dirty reads but allows phantom reads?
A) READ UNCOMMITTED B) READ COMMITTED C) REPEATABLE READ D) SERIALIZABLE

**Question 45:**
```sql
UPDATE Users SET Balance = Balance + 100 WHERE ID = 1
```
If this runs concurrently, what problem might occur?
A) Dirty read B) Lost update C) Deadlock D) Phantom read

**Question 46:**
What's a deadlock?
A) Long-running query B) Two transactions blocking each other C) Database crash D) Lock timeout

**Question 47:**
```sql
SELECT * FROM Users WITH (NOLOCK)
```
What does NOLOCK hint do?
A) Prevents locking B) Same as READ UNCOMMITTED C) Improves performance D) All of the above

**Question 48:**
What's the default isolation level in SQL Server?
A) READ UNCOMMITTED B) READ COMMITTED C) REPEATABLE READ D) SERIALIZABLE

**Question 49:**
```sql
BEGIN TRANSACTION
    -- Some operations
    SAVE TRANSACTION SavePoint1
    -- More operations
    ROLLBACK TRANSACTION SavePoint1
```
What happens to operations before SavePoint1?
A) Also rolled back B) Remain committed C) Remain in transaction D) Cause error

**Question 50:**
Which lock type allows multiple readers but exclusive writer?
A) Shared lock B) Exclusive lock C) Update lock D) Intent lock

---

# Part B: Data Structures & Algorithms (50 Questions)

## Time & Space Complexity (10 Questions)

**Question 51:**
What's the time complexity of binary search?
A) O(n) B) O(log n) C) O(n log n) D) O(1)

**Question 52:**
```python
for i in range(n):
    for j in range(n):
        print(i, j)
```
Time complexity?
A) O(n) B) O(n²) C) O(2n) D) O(log n)

**Question 53:**
What's the space complexity of merge sort?
A) O(1) B) O(log n) C) O(n) D) O(n²)

**Question 54:**
```python
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```
Time complexity without memoization?
A) O(n) B) O(2^n) C) O(n²) D) O(log n)

**Question 55:**
Which has better average-case time complexity for search?
A) Array B) Linked List C) Hash Table D) Binary Tree

**Question 56:**
```python
for i in range(n):
    for j in range(i):
        print(i, j)
```
Time complexity?
A) O(n) B) O(n²) C) O(n log n) D) O(log n)

**Question 57:**
What's the time complexity of inserting at the beginning of an array?
A) O(1) B) O(log n) C) O(n) D) O(n²)

**Question 58:**
Which sorting algorithm has O(n) best-case time complexity?
A) Quick Sort B) Merge Sort C) Bubble Sort D) Heap Sort

**Question 59:**
What's the space complexity of depth-first search (DFS)?
A) O(1) B) O(V) C) O(E) D) O(V + E)

**Question 60:**
```python
def mystery(n):
    if n <= 1:
        return 1
    return mystery(n//2) + mystery(n//2)
```
Time complexity?
A) O(n) B) O(log n) C) O(n log n) D) O(2^n)

## Arrays & Strings (8 Questions)

**Question 61:**
Given array [1,2,3,4,5], what's the result of rotating left by 2?
A) [3,4,5,1,2] B) [4,5,1,2,3] C) [2,3,4,5,1] D) [1,2,3,4,5]

**Question 62:**
Best approach to find if array has duplicate elements?
A) Nested loops B) Sort first C) Hash set D) Binary search

**Question 63:**
```python
def reverse_string(s):
    return s[::-1]
```
Time complexity?
A) O(1) B) O(log n) C) O(n) D) O(n²)

**Question 64:**
Two Sum problem: Find two numbers that add up to target. Best approach?
A) Nested loops O(n²) B) Sort + two pointers O(n log n) C) Hash map O(n) D) Binary search O(log n)

**Question 65:**
```python
arr = [1, 2, 3, 4, 5]
arr.insert(0, 0)  # Insert at beginning
```
Time complexity of this operation?
A) O(1) B) O(log n) C) O(n) D) O(n²)

**Question 66:**
Find maximum subarray sum (Kadane's algorithm). Time complexity?
A) O(n³) B) O(n²) C) O(n log n) D) O(n)

**Question 67:**
Given sorted array, best way to remove duplicates in-place?
A) Hash set B) Two pointers C) New array D) Sorting again

**Question 68:**
String matching: Find pattern in text. Which algorithm is most efficient?
A) Brute force B) KMP C) Rabin-Karp D) All are same

## Linked Lists (6 Questions)

**Question 69:**
Advantage of linked list over array?
A) Better cache locality B) Random access C) Dynamic size D) Less memory usage

**Question 70:**
```python
# Detect cycle in linked list
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            return True
    return False
```
This algorithm is called?
A) Two pointers B) Floyd's algorithm C) Tortoise and hare D) All of the above

**Question 71:**
Time complexity to find middle element of linked list?
A) O(1) if we know length B) O(n) single pass with two pointers C) O(n) two passes D) All are correct approaches

**Question 72:**
Reverse a linked list iteratively. Space complexity?
A) O(1) B) O(n) C) O(log n) D) O(n²)

**Question 73:**
Merge two sorted linked lists. Time complexity?
A) O(m + n) B) O(m * n) C) O(max(m,n)) D) O(log(m+n))

**Question 74:**
Remove nth node from end of linked list in one pass. Best approach?
A) Two passes B) Stack C) Two pointers with gap D) Recursion

## Stacks & Queues (6 Questions)

**Question 75:**
Valid parentheses problem: Check if brackets are properly nested. Best data structure?
A) Array B) Stack C) Queue D) Hash table

**Question 76:**
```python
# Implement queue using two stacks
class Queue:
    def __init__(self):
        self.s1 = []
        self.s2 = []
```
Time complexity for enqueue and dequeue operations?
A) Both O(1) B) O(1) and O(n) C) Both O(n) D) O(n) and O(1)

**Question 77:**
Evaluate postfix expression "2 3 + 4 *". What data structure to use?
A) Queue B) Stack C) Array D) Tree

**Question 78:**
Find next greater element for each element in array. Best approach?
A) Nested loops O(n²) B) Stack O(n) C) Sorting O(n log n) D) Hash map O(n²)

**Question 79:**
Implement stack that supports getMin() in O(1). Best approach?
A) Keep minimum variable B) Use auxiliary stack C) Sort elements D) Use heap

**Question 80:**
BFS traversal uses which data structure?
A) Stack B) Queue C) Priority queue D) Hash table

## Trees & Graphs (10 Questions)

**Question 81:**
Maximum number of nodes in binary tree of height h?
A) 2^h B) 2^(h+1) - 1 C) h + 1 D) 2h

**Question 82:**
```python
def height(root):
    if not root:
        return 0
    return 1 + max(height(root.left), height(root.right))
```
Time complexity?
A) O(log n) B) O(n) C) O(n log n) D) O(n²)

**Question 83:**
In BST, inorder traversal gives:
A) Random order B) Sorted order C) Reverse sorted D) Level order

**Question 84:**
Find lowest common ancestor (LCA) in BST. Time complexity?
A) O(n) B) O(log n) C) O(h) where h is height D) Both B and C

**Question 85:**
Check if binary tree is balanced. A tree is balanced if:
A) It's complete B) Height difference of subtrees ≤ 1 C) It's a BST D) All leaves at same level

**Question 86:**
Dijkstra's algorithm finds:
A) Minimum spanning tree B) Shortest path from source C) Strongly connected components D) Topological order

**Question 87:**
```python
# DFS traversal
def dfs(graph, node, visited):
    visited[node] = True
    for neighbor in graph[node]:
        if not visited[neighbor]:
            dfs(graph, neighbor, visited)
```
Space complexity?
A) O(1) B) O(V) C) O(E) D) O(V + E)

**Question 88:**
Detect cycle in undirected graph using DFS. What do you need to track?
A) Visited nodes only B) Visited nodes and parent C) Start and end times D) Color of nodes

**Question 89:**
Convert binary tree to its mirror. Time complexity?
A) O(log n) B) O(n) C) O(n log n) D) O(n²)

**Question 90:**
Topological sorting is possible only for:
A) Connected graphs B) Directed Acyclic Graphs (DAG) C) Binary trees D) Complete graphs

## Dynamic Programming & Recursion (6 Questions)

**Question 91:**
0/1 Knapsack problem. Bottom-up DP time complexity?
A) O(n) B) O(W) where W is capacity C) O(n * W) D) O(2^n)

**Question 92:**
Longest Common Subsequence (LCS). Space complexity can be optimized to:
A) O(m * n) B) O(min(m, n)) C) O(max(m, n)) D) O(1)

**Question 93:**
```python
def climb_stairs(n):
    if n <= 2:
        return n
    dp = [0] * (n + 1)
    dp[1], dp[2] = 1, 2
    for i in range(3, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```
This solves which problem pattern?
A) Fibonacci-like B) Coin change C) Longest increasing subsequence D) Edit distance

**Question 94:**
Edit Distance problem uses which DP approach?
A) 1D array B) 2D matrix C) Tree D) Graph

**Question 95:**
When should you use memoization vs tabulation?
A) Memoization for all subproblems, tabulation for some B) Tabulation for all subproblems, memoization for some C) No difference D) Memoization is always better

**Question 96:**
Maximum sum path in binary tree. What's the key insight?
A) Use level order traversal B) At each node, consider paths through it C) Use postorder traversal D) Convert to graph problem

## Sorting & Searching (4 Questions)

**Question 97:**
Which sorting algorithm is stable and has O(n log n) worst-case?
A) Quick Sort B) Heap Sort C) Merge Sort D) Selection Sort

**Question 98:**
Find kth largest element in unsorted array. Most efficient approach?
A) Sort array O(n log n) B) Min heap of size k O(n log k) C) Quickselect O(n) average D) All are equally good

**Question 99:**
Binary search on rotated sorted array. Time complexity?
A) O(n) B) O(log n) C) O(n log n) D) Not possible

**Question 100:**
Count inversions in array (pairs where i < j but arr[i] > arr[j]). Efficient approach?
A) Nested loops O(n²) B) Modified merge sort O(n log n) C) Hash table O(n) D) Sorting O(n log n)

---

## Quick Answer Keys

### SQL Server (Questions 1-50):
1.B 2.B 3.B 4.B 5.B 6.C 7.B 8.D 9.B 10.C
11.C 12.B 13.B 14.B 15.B 16.B 17.B 18.B 19.D 20.B
21.D 22.A 23.D 24.C 25.D 26.B 27.D 28.B 29.B 30.A
31.B 32.B 33.D 34.D 35.B 36.B 37.B 38.B 39.A 40.D
41.A 42.B 43.B 44.B 45.B 46.B 47.D 48.B 49.C 50.A

### DSA (Questions 51-100):
51.B 52.B 53.C 54.B 55.C 56.B 57.C 58.C 59.B 60.A
61.A 62.C 63.C 64.C 65.C 66.D 67.B 68.B 69.C 70.D
71.D 72.A 73.A 74.C 75.B 76.B 77.B 78.B 79.B 80.B
81.B 82.B 83.B 84.D 85.B 86.B 87.B 88.B 89.B 90.B
91.C 92.B 93.A 94.B 95.B 96.B 97.C 98.C 99.B 100.B

## Time Management Tips:
- **SQL Questions**: 15-20 seconds each (syntax/concept recognition)
- **DSA Questions**: 20-25 seconds each (complexity analysis focus)
- **Pattern Recognition**: Trust your algorithmic knowledge
- **Skip and Return**: Don't get stuck on complex scenarios

--- 

I've created comprehensive MCQ sets for both **SQL Server (50 questions)** and **Data Structures & Algorithms (50 questions)** - 100 total questions designed for prescreening scenarios.

## SQL Server Coverage:
- **Basic Queries (30%)** - SELECT, WHERE, JOIN syntax, common functions
- **Joins & Relationships (20%)** - Different join types, performance implications
- **Indexes & Performance (16%)** - Index types, query optimization
- **Stored Procedures (14%)** - Functions vs procedures, parameters
- **Transactions & Concurrency (20%)** - ACID properties, isolation levels, locking

## DSA Coverage:
- **Complexity Analysis (20%)** - Big O notation, time/space complexity
- **Arrays & Strings (16%)** - Common algorithms, two-sum, rotations
- **Linked Lists (12%)** - Cycle detection, reversal, merging
- **Stacks & Queues (12%)** - Valid parentheses, BFS/DFS applications
- **Trees & Graphs (20%)** - BST operations, traversals, shortest paths
- **Dynamic Programming (12%)** - Classic problems, memoization vs tabulation
- **Sorting & Searching (8%)** - Algorithm comparisons, binary search variants

## Key Patterns for Quick Recognition:

### SQL Red Flags:
- `= NULL` instead of `IS NULL`
- `LIMIT` in SQL Server (should be `TOP`)
- Missing `GROUP BY` with aggregates
- Cartesian products in joins
- Function calls on indexed columns

### DSA Quick Identification:
- **Two pointers**: Sorted arrays, palindromes, cycle detection
- **Hash maps**: Two-sum problems, frequency counting
- **Stack**: Parentheses, next greater element, expression evaluation
- **BFS/DFS**: Tree/graph traversal, shortest path
- **DP**: Overlapping subproblems, optimal substructure

## Speed Strategy:
1. **SQL**: Focus on syntax errors and basic concepts first
2. **DSA**: Recognize algorithm patterns, don't calculate exact complexity
3. **Elimination**: Remove obviously wrong answers
4. **Experience**: Trust your practical knowledge over theoretical edge cases

These questions mirror the difficulty and style of actual prescreening tests used by major tech companies and consulting firms. Practice pattern recognition rather than detailed problem-solving to maximize your speed during the exam.

--- 

