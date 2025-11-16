Excellent — this is a classic **problem-solving and algorithmic thinking** question for architect roles. It tests your ability to think through logic, optimize, and explain your reasoning clearly.

Let's build the solution step-by-step with detailed explanations.

---

## 🎯 **Problem Statement**

Given a string, find all **duplicate characters** (characters that appear more than once) **without using built-in features** like LINQ, `Dictionary`, `HashSet`, `GroupBy`, etc.

---

## 🧠 **Approach**

Since we can't use built-in collections, we'll use:
1. **A simple character array** to track character frequencies.
2. **ASCII assumption** — we'll assume the input contains standard ASCII characters (0-127) or extended ASCII (0-255).

### **Algorithm:**
1. Create an integer array of size 256 (to cover all ASCII characters).
2. Traverse the string, incrementing the count for each character.
3. Traverse the count array and print characters that appear more than once.

---

## 💻 **C# Program**

```csharp
using System;

class Program
{
    static void Main()
    {
        string input = "programming";
        
        Console.WriteLine($"Input String: {input}");
        Console.WriteLine("Duplicate Characters:");
        
        FindDuplicateCharacters(input);
    }

    static void FindDuplicateCharacters(string str)
    {
        // Step 1: Create an array to store character counts
        // Using size 256 to cover all ASCII characters
        int[] charCount = new int[256];

        // Step 2: Traverse the string and count each character
        for (int i = 0; i < str.Length; i++)
        {
            char currentChar = str[i];
            int asciiValue = (int)currentChar;  // Get ASCII value of character
            charCount[asciiValue]++;             // Increment count at that index
        }

        // Step 3: Traverse the count array and find duplicates
        for (int i = 0; i < charCount.Length; i++)
        {
            if (charCount[i] > 1)  // If count > 1, it's a duplicate
            {
                char duplicateChar = (char)i;  // Convert index back to character
                Console.WriteLine($"'{duplicateChar}' appears {charCount[i]} times");
            }
        }
    }
}
```

---

## 📊 **Output**

```
Input String: programming
Duplicate Characters:
'g' appears 2 times
'm' appears 2 times
'r' appears 2 times
```

---

## 🔍 **Detailed Explanation**

### **Step 1: Create the Count Array**

```csharp
int[] charCount = new int[256];
```

- We create an integer array of size **256** to represent all possible ASCII characters.
- Each index corresponds to the **ASCII value** of a character.
- The value at each index will store **how many times that character appears**.

**Example:**
- Index 97 → character `'a'`
- Index 103 → character `'g'`

---

### **Step 2: Count Character Occurrences**

```csharp
for (int i = 0; i < str.Length; i++)
{
    char currentChar = str[i];
    int asciiValue = (int)currentChar;
    charCount[asciiValue]++;
}
```

- We loop through each character in the string.
- Convert the character to its **ASCII value** using `(int)currentChar`.
- Use that ASCII value as an **index** in our array and increment the count.

**Example for "programming":**

| Character | ASCII Value | Count Array Update |
|-----------|-------------|---------------------|
| 'p' | 112 | charCount[112] = 1 |
| 'r' | 114 | charCount[114] = 1 |
| 'o' | 111 | charCount[111] = 1 |
| 'g' | 103 | charCount[103] = 1 |
| 'r' | 114 | charCount[114] = 2 ← duplicate! |
| 'a' | 97 | charCount[97] = 1 |
| 'm' | 109 | charCount[109] = 1 |
| 'm' | 109 | charCount[109] = 2 ← duplicate! |
| 'i' | 105 | charCount[105] = 1 |
| 'n' | 110 | charCount[110] = 1 |
| 'g' | 103 | charCount[103] = 2 ← duplicate! |

---

### **Step 3: Find and Print Duplicates**

```csharp
for (int i = 0; i < charCount.Length; i++)
{
    if (charCount[i] > 1)
    {
        char duplicateChar = (char)i;
        Console.WriteLine($"'{duplicateChar}' appears {charCount[i]} times");
    }
}
```

- Loop through the count array.
- If any index has a count **greater than 1**, it's a duplicate.
- Convert the index back to a character using `(char)i`.
- Print the character and its count.

---

## 🧩 **Complexity Analysis**

| Metric | Value | Explanation |
|--------|-------|-------------|
| **Time Complexity** | O(n + k) | n = string length, k = character set size (256) |
| **Space Complexity** | O(k) | Fixed array of 256 integers |

Since k is constant (256), we can say:
- **Time:** O(n) — linear
- **Space:** O(1) — constant (fixed size array)

---

## 🔄 **Alternative: Case-Insensitive Version**

If you want to treat 'A' and 'a' as the same:

```csharp
static void FindDuplicateCharacters(string str)
{
    int[] charCount = new int[256];

    // Convert to lowercase for case-insensitive comparison
    string lowerStr = str.ToLower();

    for (int i = 0; i < lowerStr.Length; i++)
    {
        int asciiValue = (int)lowerStr[i];
        charCount[asciiValue]++;
    }

    for (int i = 0; i < charCount.Length; i++)
    {
        if (charCount[i] > 1)
        {
            Console.WriteLine($"'{(char)i}' appears {charCount[i]} times");
        }
    }
}
```

---

## 🔄 **Even More Restricted: Without Using Arrays**

If they say "no arrays either," you can use nested loops (brute force):

```csharp
static void FindDuplicateCharacters(string str)
{
    string processed = "";

    for (int i = 0; i < str.Length; i++)
    {
        char currentChar = str[i];
        
        // Skip if already processed
        bool alreadyProcessed = false;
        for (int p = 0; p < processed.Length; p++)
        {
            if (processed[p] == currentChar)
            {
                alreadyProcessed = true;
                break;
            }
        }
        if (alreadyProcessed) continue;

        // Count occurrences
        int count = 0;
        for (int j = 0; j < str.Length; j++)
        {
            if (str[j] == currentChar)
                count++;
        }

        if (count > 1)
        {
            Console.WriteLine($"'{currentChar}' appears {count} times");
        }

        processed += currentChar;  // Mark as processed
    }
}
```

**Complexity:** O(n²) — less efficient, but truly no built-in data structures.

---

## 🧠 **How to Explain in an Interview**

1. **Clarify constraints:**  
   "Can I assume ASCII characters? Should it be case-sensitive?"

2. **Explain the approach:**  
   "I'll use an integer array indexed by ASCII values to count occurrences, then identify duplicates."

3. **Walk through the code:**  
   Show the three steps clearly.

4. **Discuss complexity:**  
   "This is O(n) time and O(1) space since the array size is fixed."

5. **Mention trade-offs:**  
   "If Unicode is required, I'd use a larger array or switch to a manual hash table implementation."

---

## ✅ **Final Answer Summary**

```csharp
// Clean, production-ready version
static void FindDuplicateCharacters(string str)
{
    int[] charCount = new int[256];

    for (int i = 0; i < str.Length; i++)
    {
        charCount[(int)str[i]]++;
    }

    for (int i = 0; i < 256; i++)
    {
        if (charCount[i] > 1)
        {
            Console.WriteLine($"'{(char)i}' appears {charCount[i]} times");
        }
    }
}
```

This demonstrates **clean logic, algorithmic thinking, and the ability to work within constraints** — exactly what architect-level interviewers are looking for.