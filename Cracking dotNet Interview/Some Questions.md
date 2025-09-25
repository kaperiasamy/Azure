In a C# / .NET technical interview once I was asked to find the third largest number from the given array of integers, using for loop. In another interview I was asked to find the caracter count from the given string. Tomorrow I am having a screen test in which I may face similar questions and may be asked to write simple C# programs for such problems. Please get me such questions, with C# programs (10 - 15 lines) which are most commonly asked in various technical interviews and other programs which may be of implementing most commonly used / asked algorithms. 

**Prompt:**
I'm preparing for a C#/.NET technical screen test where I might be asked to solve common programming problems using simple for-loops and core C# logic. In previous interviews, I was asked to:

* Find the third largest number in an array using a `for` loop.
* Count the number of occurrences of each character in a string.

To prepare effectively, please provide a curated list of commonly asked C# programming interview questions. Focus on problems that are solvable in about 10–15 lines of code. These should include:

* Frequently asked algorithmic problems in C# interviews.
* Basic string, array, and number manipulation tasks.
* Classic logic problems or patterns commonly seen in technical screens.

For each problem, provide a brief description and a complete C# code solution.

---
Sure! Here's a version of the prompt specifically tailored for **Claude AI**, with phrasing and context that suits Anthropic’s assistant:

---

**Prompt for Claude AI:**

Hi Claude, I'm preparing for a C#/.NET technical screen test where I might be asked to solve simple programming problems using basic constructs like `for` loops and string/array manipulation.

In previous interviews, I was asked to:

* Find the third largest number in an array using a `for` loop.
* Count the frequency of each character in a string.

To prepare, could you generate a list of **commonly asked C# programming interview questions** (70 questions) that:

* Are typically solvable in about 10–15 lines of code.
* Focus on core concepts like arrays, strings, loops, conditionals, and basic algorithms.
* Reflect the types of questions often asked in entry to mid-level .NET interviews.

For each question, please include:

1. A brief description of the problem.
2. A clean, well-structured C# code solution (simple, 10 - 15 lines of code).

Also, include a few additional problems that cover **commonly implemented algorithms**, such as sorting, searching, or basic recursion, if relevant.

Thank you!

---

---

**Prompt for Claude AI:**

Hi Claude, I'm preparing for a C#/.NET technical screen test where I might be asked to find the time-complexity and space complexity values by giving a program. In that case how to calculate the values. 

To prepare, could you generate a list of **C# programs** (70 questions) and explain the approach to calculate the time and space complexity values. 

* Are typically solvable in about 10–15 lines of code.
* Focus on core concepts like arrays, strings, loops, conditionals, and basic algorithms.
* Reflect the types of questions often asked in mid to advanced / very advanced-level .NET interviews.

For each question, please include:

1. A brief description of the problem.
2. A clean, well-structured C# code solution (simple, 10 - 15 lines of code).

Also, include a few additional problems that cover **commonly implemented algorithms**, such as sorting, searching, or basic recursion, if relevant.

Thank you!

---

# 70 Common C#/.NET Interview Questions with Solutions

## Array Manipulation Questions

### 1. Find the Maximum Number in an Array
**Problem:** Find the largest number in an integer array using a for loop.
```csharp
public static int FindMaximum(int[] arr)
{
    int max = arr[0];
    for (int i = 1; i < arr.Length; i++)
    {
        if (arr[i] > max)
            max = arr[i];
    }
    return max;
}
```

### 2. Find the Second Largest Number in an Array
**Problem:** Find the second largest number without sorting the array.
```csharp
public static int FindSecondLargest(int[] arr)
{
    int max = int.MinValue, secondMax = int.MinValue;
    for (int i = 0; i < arr.Length; i++)
    {
        if (arr[i] > max)
        {
            secondMax = max;
            max = arr[i];
        }
        else if (arr[i] > secondMax && arr[i] != max)
            secondMax = arr[i];
    }
    return secondMax;
}
```

### 3. Find the Third Largest Number in an Array
**Problem:** Find the third largest number in an array using basic loops.
```csharp
public static int FindThirdLargest(int[] arr)
{
    int first = int.MinValue, second = int.MinValue, third = int.MinValue;
    for (int i = 0; i < arr.Length; i++)
    {
        if (arr[i] > first)
        {
            third = second; second = first; first = arr[i];
        }
        else if (arr[i] > second && arr[i] != first)
        {
            third = second; second = arr[i];
        }
        else if (arr[i] > third && arr[i] != second && arr[i] != first)
            third = arr[i];
    }
    return third;
}
```

### 4. Reverse an Array
**Problem:** Reverse the elements of an array in-place.
```csharp
public static void ReverseArray(int[] arr)
{
    int left = 0, right = arr.Length - 1;
    while (left < right)
    {
        int temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;
        left++;
        right--;
    }
}
```

### 5. Find Sum of Array Elements
**Problem:** Calculate the sum of all elements in an integer array.
```csharp
public static int SumArray(int[] arr)
{
    int sum = 0;
    for (int i = 0; i < arr.Length; i++)
    {
        sum += arr[i];
    }
    return sum;
}
```

### 6. Find Average of Array Elements
**Problem:** Calculate the average of all elements in an array.
```csharp
public static double FindAverage(int[] arr)
{
    int sum = 0;
    for (int i = 0; i < arr.Length; i++)
    {
        sum += arr[i];
    }
    return (double)sum / arr.Length;
}
```

### 7. Count Even Numbers in Array
**Problem:** Count how many even numbers are present in an array.
```csharp
public static int CountEvenNumbers(int[] arr)
{
    int count = 0;
    for (int i = 0; i < arr.Length; i++)
    {
        if (arr[i] % 2 == 0)
            count++;
    }
    return count;
}
```

### 8. Count Odd Numbers in Array
**Problem:** Count how many odd numbers are present in an array.
```csharp
public static int CountOddNumbers(int[] arr)
{
    int count = 0;
    for (int i = 0; i < arr.Length; i++)
    {
        if (arr[i] % 2 != 0)
            count++;
    }
    return count;
}
```

### 9. Find Missing Number in Array
**Problem:** Find the missing number in an array of 1 to n+1 numbers.
```csharp
public static int FindMissingNumber(int[] arr, int n)
{
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int i = 0; i < arr.Length; i++)
    {
        actualSum += arr[i];
    }
    return expectedSum - actualSum;
}
```

### 10. Remove Duplicates from Sorted Array
**Problem:** Remove duplicates from a sorted array and return the new length.
```csharp
public static int RemoveDuplicates(int[] arr)
{
    if (arr.Length == 0) return 0;
    int j = 0;
    for (int i = 1; i < arr.Length; i++)
    {
        if (arr[i] != arr[j])
        {
            j++;
            arr[j] = arr[i];
        }
    }
    return j + 1;
}
```

## String Manipulation Questions

### 11. Count Character Frequency
**Problem:** Count the frequency of each character in a string.
```csharp
public static Dictionary<char, int> CountCharacterFrequency(string str)
{
    var frequency = new Dictionary<char, int>();
    for (int i = 0; i < str.Length; i++)
    {
        char c = str[i];
        if (frequency.ContainsKey(c))
            frequency[c]++;
        else
            frequency[c] = 1;
    }
    return frequency;
}
```

### 12. Check if String is Palindrome
**Problem:** Check if a given string reads the same forwards and backwards.
```csharp
public static bool IsPalindrome(string str)
{
    int left = 0, right = str.Length - 1;
    while (left < right)
    {
        if (str[left] != str[right])
            return false;
        left++;
        right--;
    }
    return true;
}
```

### 13. Reverse a String
**Problem:** Reverse the characters in a string.
```csharp
public static string ReverseString(string str)
{
    char[] chars = str.ToCharArray();
    int left = 0, right = chars.Length - 1;
    while (left < right)
    {
        char temp = chars[left];
        chars[left] = chars[right];
        chars[right] = temp;
        left++;
        right--;
    }
    return new string(chars);
}
```

### 14. Count Vowels in String
**Problem:** Count the number of vowels in a given string.
```csharp
public static int CountVowels(string str)
{
    int count = 0;
    string vowels = "aeiouAEIOU";
    for (int i = 0; i < str.Length; i++)
    {
        if (vowels.Contains(str[i]))
            count++;
    }
    return count;
}
```

### 15. Count Words in String
**Problem:** Count the number of words in a sentence.
```csharp
public static int CountWords(string str)
{
    if (string.IsNullOrWhiteSpace(str)) return 0;
    int count = 1;
    for (int i = 0; i < str.Length; i++)
    {
        if (str[i] == ' ' && i + 1 < str.Length && str[i + 1] != ' ')
            count++;
    }
    return count;
}
```

### 16. Remove Spaces from String
**Problem:** Remove all spaces from a given string.
```csharp
public static string RemoveSpaces(string str)
{
    string result = "";
    for (int i = 0; i < str.Length; i++)
    {
        if (str[i] != ' ')
            result += str[i];
    }
    return result;
}
```

### 17. Check if Two Strings are Anagrams
**Problem:** Check if two strings contain the same characters with the same frequency.
```csharp
public static bool AreAnagrams(string str1, string str2)
{
    if (str1.Length != str2.Length) return false;
    var count = new int[26];
    for (int i = 0; i < str1.Length; i++)
    {
        count[str1[i] - 'a']++;
        count[str2[i] - 'a']--;
    }
    for (int i = 0; i < 26; i++)
        if (count[i] != 0) return false;
    return true;
}
```

### 18. Find First Non-Repeating Character
**Problem:** Find the first character that appears only once in a string.
```csharp
public static char FindFirstNonRepeating(string str)
{
    var frequency = new Dictionary<char, int>();
    foreach (char c in str)
        frequency[c] = frequency.ContainsKey(c) ? frequency[c] + 1 : 1;
    
    for (int i = 0; i < str.Length; i++)
        if (frequency[str[i]] == 1)
            return str[i];
    
    return '\0';
}
```

### 19. Capitalize First Letter of Each Word
**Problem:** Capitalize the first letter of each word in a sentence.
```csharp
public static string CapitalizeWords(string str)
{
    char[] chars = str.ToCharArray();
    bool newWord = true;
    for (int i = 0; i < chars.Length; i++)
    {
        if (chars[i] == ' ')
            newWord = true;
        else if (newWord)
        {
            chars[i] = char.ToUpper(chars[i]);
            newWord = false;
        }
    }
    return new string(chars);
}
```

### 20. Check if String Contains Only Digits
**Problem:** Check if a string contains only numeric digits.
```csharp
public static bool IsNumericString(string str)
{
    if (string.IsNullOrEmpty(str)) return false;
    for (int i = 0; i < str.Length; i++)
    {
        if (!char.IsDigit(str[i]))
            return false;
    }
    return true;
}
```

## Number and Mathematical Questions

### 21. Check if Number is Prime
**Problem:** Determine if a given number is prime.
```csharp
public static bool IsPrime(int n)
{
    if (n <= 1) return false;
    if (n <= 3) return true;
    if (n % 2 == 0 || n % 3 == 0) return false;
    for (int i = 5; i * i <= n; i += 6)
    {
        if (n % i == 0 || n % (i + 2) == 0)
            return false;
    }
    return true;
}
```

### 22. Calculate Factorial
**Problem:** Calculate the factorial of a given number.
```csharp
public static long Factorial(int n)
{
    if (n <= 1) return 1;
    long result = 1;
    for (int i = 2; i <= n; i++)
    {
        result *= i;
    }
    return result;
}
```

### 23. Calculate Factorial (Recursive)
**Problem:** Calculate factorial using recursion.
```csharp
public static long FactorialRecursive(int n)
{
    if (n <= 1)
        return 1;
    return n * FactorialRecursive(n - 1);
}
```

### 24. Generate Fibonacci Sequence
**Problem:** Generate the first n numbers in the Fibonacci sequence.
```csharp
public static int[] GenerateFibonacci(int n)
{
    if (n <= 0) return new int[0];
    int[] fib = new int[n];
    if (n >= 1) fib[0] = 0;
    if (n >= 2) fib[1] = 1;
    for (int i = 2; i < n; i++)
    {
        fib[i] = fib[i - 1] + fib[i - 2];
    }
    return fib;
}
```

### 25. Check if Number is Perfect Square
**Problem:** Check if a number is a perfect square.
```csharp
public static bool IsPerfectSquare(int n)
{
    if (n < 0) return false;
    int sqrt = (int)Math.Sqrt(n);
    return sqrt * sqrt == n;
}
```

### 26. Find GCD of Two Numbers
**Problem:** Find the Greatest Common Divisor of two numbers.
```csharp
public static int FindGCD(int a, int b)
{
    while (b != 0)
    {
        int temp = b;
        b = a % b;
        a = temp;
    }
    return a;
}
```

### 27. Check if Number is Armstrong Number
**Problem:** Check if a number equals the sum of its digits raised to the power of number of digits.
```csharp
public static bool IsArmstrongNumber(int n)
{
    string str = n.ToString();
    int digits = str.Length;
    int sum = 0, temp = n;
    while (temp > 0)
    {
        int digit = temp % 10;
        sum += (int)Math.Pow(digit, digits);
        temp /= 10;
    }
    return sum == n;
}
```

### 28. Reverse a Number
**Problem:** Reverse the digits of a given number.
```csharp
public static int ReverseNumber(int n)
{
    int reversed = 0;
    while (n > 0)
    {
        reversed = reversed * 10 + n % 10;
        n /= 10;
    }
    return reversed;
}
```

### 29. Check if Number is Palindrome
**Problem:** Check if a number reads the same forwards and backwards.
```csharp
public static bool IsNumberPalindrome(int n)
{
    string str = n.ToString();
    int left = 0, right = str.Length - 1;
    while (left < right)
    {
        if (str[left] != str[right])
            return false;
        left++;
        right--;
    }
    return true;
}
```

### 30. Sum of Digits
**Problem:** Calculate the sum of all digits in a number.
```csharp
public static int SumOfDigits(int n)
{
    int sum = 0;
    while (n > 0)
    {
        sum += n % 10;
        n /= 10;
    }
    return sum;
}
```

## Loop and Control Structure Questions

### 31. Print Multiplication Table
**Problem:** Print the multiplication table for a given number.
```csharp
public static void PrintMultiplicationTable(int n)
{
    for (int i = 1; i <= 10; i++)
    {
        Console.WriteLine($"{n} x {i} = {n * i}");
    }
}
```

### 32. Print Number Pattern
**Problem:** Print a pattern of numbers in pyramid form.
```csharp
public static void PrintNumberPattern(int n)
{
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= i; j++)
        {
            Console.Write(j + " ");
        }
        Console.WriteLine();
    }
}
```

### 33. Print Star Pattern
**Problem:** Print a star pattern in pyramid form.
```csharp
public static void PrintStarPattern(int n)
{
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= n - i; j++)
            Console.Write(" ");
        for (int k = 1; k <= 2 * i - 1; k++)
            Console.Write("*");
        Console.WriteLine();
    }
}
```

### 34. Count Digits in Number
**Problem:** Count the number of digits in a given number.
```csharp
public static int CountDigits(int n)
{
    if (n == 0) return 1;
    int count = 0;
    while (n > 0)
    {
        count++;
        n /= 10;
    }
    return count;
}
```

### 35. Find Power of Number
**Problem:** Calculate x raised to the power of y.
```csharp
public static double Power(int x, int y)
{
    double result = 1;
    for (int i = 0; i < Math.Abs(y); i++)
    {
        result *= x;
    }
    return y < 0 ? 1 / result : result;
}
```

## Search and Sort Questions

### 36. Linear Search
**Problem:** Find the index of an element in an array using linear search.
```csharp
public static int LinearSearch(int[] arr, int target)
{
    for (int i = 0; i < arr.Length; i++)
    {
        if (arr[i] == target)
            return i;
    }
    return -1;
}
```

### 37. Binary Search
**Problem:** Find an element in a sorted array using binary search.
```csharp
public static int BinarySearch(int[] arr, int target)
{
    int left = 0, right = arr.Length - 1;
    while (left <= right)
    {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

### 38. Bubble Sort
**Problem:** Sort an array using bubble sort algorithm.
```csharp
public static void BubbleSort(int[] arr)
{
    int n = arr.Length;
    for (int i = 0; i < n - 1; i++)
    {
        for (int j = 0; j < n - i - 1; j++)
        {
            if (arr[j] > arr[j + 1])
            {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```

### 39. Selection Sort
**Problem:** Sort an array using selection sort algorithm.
```csharp
public static void SelectionSort(int[] arr)
{
    int n = arr.Length;
    for (int i = 0; i < n - 1; i++)
    {
        int minIndex = i;
        for (int j = i + 1; j < n; j++)
            if (arr[j] < arr[minIndex])
                minIndex = j;
        
        int temp = arr[minIndex];
        arr[minIndex] = arr[i];
        arr[i] = temp;
    }
}
```

### 40. Insertion Sort
**Problem:** Sort an array using insertion sort algorithm.
```csharp
public static void InsertionSort(int[] arr)
{
    for (int i = 1; i < arr.Length; i++)
    {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key)
        {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

## Two Pointer Technique Questions

### 41. Two Sum Problem
**Problem:** Find two numbers in a sorted array that add up to a target sum.
```csharp
public static int[] TwoSum(int[] arr, int target)
{
    int left = 0, right = arr.Length - 1;
    while (left < right)
    {
        int sum = arr[left] + arr[right];
        if (sum == target) return new int[] { left, right };
        if (sum < target) left++;
        else right--;
    }
    return new int[] { -1, -1 };
}
```

### 42. Remove Element
**Problem:** Remove all instances of a value from an array in-place.
```csharp
public static int RemoveElement(int[] arr, int val)
{
    int k = 0;
    for (int i = 0; i < arr.Length; i++)
    {
        if (arr[i] != val)
        {
            arr[k] = arr[i];
            k++;
        }
    }
    return k;
}
```

### 43. Move Zeros to End
**Problem:** Move all zeros in an array to the end while maintaining order of non-zeros.
```csharp
public static void MoveZeros(int[] arr)
{
    int writeIndex = 0;
    for (int i = 0; i < arr.Length; i++)
    {
        if (arr[i] != 0)
        {
            arr[writeIndex] = arr[i];
            writeIndex++;
        }
    }
    for (int i = writeIndex; i < arr.Length; i++)
        arr[i] = 0;
}
```

## Matrix Questions

### 44. Matrix Addition
**Problem:** Add two matrices and return the result.
```csharp
public static int[,] AddMatrices(int[,] matrix1, int[,] matrix2)
{
    int rows = matrix1.GetLength(0);
    int cols = matrix1.GetLength(1);
    int[,] result = new int[rows, cols];
    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < cols; j++)
        {
            result[i, j] = matrix1[i, j] + matrix2[i, j];
        }
    }
    return result;
}
```

### 45. Matrix Transpose
**Problem:** Find the transpose of a given matrix.
```csharp
public static int[,] TransposeMatrix(int[,] matrix)
{
    int rows = matrix.GetLength(0);
    int cols = matrix.GetLength(1);
    int[,] transpose = new int[cols, rows];
    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < cols; j++)
        {
            transpose[j, i] = matrix[i, j];
        }
    }
    return transpose;
}
```

### 46. Sum of Matrix Diagonal
**Problem:** Calculate the sum of diagonal elements in a square matrix.
```csharp
public static int SumOfDiagonal(int[,] matrix)
{
    int sum = 0;
    int n = matrix.GetLength(0);
    for (int i = 0; i < n; i++)
    {
        sum += matrix[i, i];
    }
    return sum;
}
```

## Data Structure Questions

### 47. Stack Implementation Using Array
**Problem:** Implement a basic stack using an array.
```csharp
public class Stack
{
    private int[] arr;
    private int top;
    
    public Stack(int size)
    {
        arr = new int[size];
        top = -1;
    }
    
    public void Push(int item) => arr[++top] = item;
    public int Pop() => arr[top--];
    public bool IsEmpty() => top == -1;
}
```

### 48. Queue Implementation Using Array
**Problem:** Implement a basic queue using an array.
```csharp
public class Queue
{
    private int[] arr;
    private int front, rear;
    
    public Queue(int size)
    {
        arr = new int[size];
        front = rear = -1;
    }
    
    public void Enqueue(int item) => arr[++rear] = item;
    public int Dequeue() => arr[++front];
    public bool IsEmpty() => front == rear;
}
```

### 49. Parentheses Matching
**Problem:** Check if parentheses in an expression are balanced.
```csharp
public static bool IsBalanced(string expression)
{
    int count = 0;
    for (int i = 0; i < expression.Length; i++)
    {
        if (expression[i] == '(') count++;
        else if (expression[i] == ')') count--;
        if (count < 0) return false;
    }
    return count == 0;
}
```

## Recursion Questions

### 50. Sum of Array (Recursive)
**Problem:** Calculate sum of array elements using recursion.
```csharp
public static int SumArrayRecursive(int[] arr, int index)
{
    if (index >= arr.Length)
        return 0;
    return arr[index] + SumArrayRecursive(arr, index + 1);
}
```

### 51. Power Calculation (Recursive)
**Problem:** Calculate x^n using recursion.
```csharp
public static int PowerRecursive(int x, int n)
{
    if (n == 0) return 1;
    if (n == 1) return x;
    return x * PowerRecursive(x, n - 1);
}
```

### 52. Count Digits (Recursive)
**Problem:** Count digits in a number using recursion.
```csharp
public static int CountDigitsRecursive(int n)
{
    if (n == 0) return 0;
    return 1 + CountDigitsRecursive(n / 10);
}
```

## Miscellaneous Common Questions

### 53. Swap Two Numbers Without Third Variable
**Problem:** Swap two numbers without using a temporary variable.
```csharp
public static void SwapNumbers(ref int a, ref int b)
{
    a = a + b;
    b = a - b;
    a = a - b;
}
```

### 54. Convert Binary to Decimal
**Problem:** Convert a binary string to decimal number.
```csharp
public static int BinaryToDecimal(string binary)
{
    int result = 0;
    int power = 0;
    for (int i = binary.Length - 1; i >= 0; i--)
    {
        if (binary[i] == '1')
            result += (int)Math.Pow(2, power);
        power++;
    }
    return result;
}
```

### 55. Convert Decimal to Binary
**Problem:** Convert a decimal number to binary string.
```csharp
public static string DecimalToBinary(int n)
{
    if (n == 0) return "0";
    string binary = "";
    while (n > 0)
    {
        binary = (n % 2) + binary;
        n /= 2;
    }
    return binary;
}
```

### 56. Check Leap Year
**Problem:** Check if a given year is a leap year.
```csharp
public static bool IsLeapYear(int year)
{
    return (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);
}
```

### 57. Generate Random Array
**Problem:** Generate an array of random numbers within a range.
```csharp
public static int[] GenerateRandomArray(int size, int min, int max)
{
    Random rand = new Random();
    int[] arr = new int[size];
    for (int i = 0; i < size; i++)
    {
        arr[i] = rand.Next(min, max + 1);
    }
    return arr;
}
```

### 58. Find Intersection of Two Arrays
**Problem:** Find common elements between two arrays.
```csharp
public static List<int> FindIntersection(int[] arr1, int[] arr2)
{
    var result = new List<int>();
    for (int i = 0; i < arr1.Length; i++)
    {
        for (int j = 0; j < arr2.Length; j++)
        {
            if (arr1[i] == arr2[j] && !result.Contains(arr1[i]))
            {
                result.Add(arr1[i]);
                break;
            }
        }
    }
    return result;
}
```

### 59. Check if Array is Sorted
**Problem:** Check if an array is sorted in ascending order.
```csharp
public static bool IsSorted(int[] arr)
{
    for (int i = 1; i < arr.Length; i++)
    {
        if (arr[i] < arr[i - 1])
            return false;
    }
    return true;
}
```

### 60. Rotate Array Left
**Problem:** Rotate array elements to the left by k positions.
```csharp
public static void RotateLeft(int[] arr, int k)
{
    int n = arr.Length;
    k = k % n;
    for (int i = 0; i < k; i++)
    {
        int first = arr[0];
        for (int j = 0; j < n - 1; j++)
            arr[j] = arr[j + 1];
        arr[n - 1] = first;
    }
}
```

### 61. Rotate Array Right
**Problem:** Rotate array elements to the right by k positions.
```csharp
public static void RotateRight(int[] arr, int k)
{
    int n = arr.Length;
    k = k % n;
    for (int i = 0; i < k; i++)
    {
        int last = arr[n - 1];
        for (int j = n - 1; j > 0; j--)
            arr[j] = arr[j - 1];
        arr[0] = last;
    }
}
```

### 62. Merge Two Sorted Arrays
**Problem:** Merge two sorted arrays into one sorted array.
```csharp
public static int[] MergeSortedArrays(int[] arr1, int[] arr2)
{
    int[] merged = new int[arr1.Length + arr2.Length];
    int i = 0, j = 0, k = 0;
    while (i < arr1.Length && j < arr2.Length)
    {
        if (arr1[i] <= arr2[j])
            merged[k++] = arr1[i++];
        else
            merged[k++] = arr2[j++];
    }
    while (i < arr1.Length) merged[k++] = arr1[i++];
    while (j < arr2.Length) merged[k++] = arr2[j++];
    return merged;
}
```

### 63. Find Peak Element
**Problem:** Find a peak element in an array (element greater than its neighbors).
```csharp
public static int FindPeak(int[] arr)
{
    int n = arr.Length;
    if (n == 1) return 0;
    if (arr[0] >= arr[1]) return 0;
    if (arr[n - 1] >= arr[n - 2]) return n - 1;
    
    for (int i = 1; i < n - 1; i++)
    {
        if (arr[i] >= arr[i - 1] && arr[i] >= arr[i + 1])
            return i;
    }
    return -1;
}
```

### 64. Count Inversions in Array
**Problem:** Count number of inversions in an array (i < j but arr[i] > arr[j]).
```csharp
public static int CountInversions(int[] arr)
{
    int count = 0;
    for (int i = 0; i < arr.Length - 1; i++)
    {
        for (int j = i + 1; j < arr.Length; j++)
        {
            if (arr[i] > arr[j])
                count++;
        }
    }
    return count;
}
```

### 65. Find Majority Element
**Problem:** Find the element that appears more than n/2 times in an array.
```csharp
public static int FindMajorityElement(int[] arr)
{
    for (int i = 0; i < arr.Length; i++)
    {
        int count = 0;
        for (int j = 0; j < arr.Length; j++)
        {
            if (arr[j] == arr[i])
                count++;
        }
        if (count > arr.Length / 2)
            return arr[i];
    }
    return -1;
}
```

### 66. Check if Array Contains Duplicate
**Problem:** Check if any value appears at least twice in an array.
```csharp
public static bool ContainsDuplicate(int[] arr)
{
    for (int i = 0; i < arr.Length; i++)
    {
        for (int j = i + 1; j < arr.Length; j++)
        {
            if (arr[i] == arr[j])
                return true;
        }
    }
    return false;
}
```

### 67. Find Single Number
**Problem:** Find the number that appears only once while others appear twice.
```csharp
public static int FindSingleNumber(int[] arr)
{
    int result = 0;
    for (int i = 0; i < arr.Length; i++)
    {
        result ^= arr[i];
    }
    return result;
}
```

### 68. Valid Parentheses (Multiple Types)
**Problem:** Check if string has valid parentheses including (), [], and {}.
```csharp
public static bool IsValidParentheses(string s)
{
    var stack = new Stack<char>();
    for (int i = 0; i < s.Length; i++)
    {
        char c = s[i];
        if (c == '(' || c == '[' || c == '{')
            stack.Push(c);
        else if (c == ')' || c == ']' || c == '}')
        {
            if (stack.Count == 0) return false;
            char top = stack.Pop();
            if ((c == ')' && top != '(') || (c == ']' && top != '[') || (c == '}' && top != '{'))
                return false;
        }
    }
    return stack.Count == 0;
}
```

### 69. Pascal's Triangle Row
**Problem:** Generate the nth row of Pascal's triangle.
```csharp
public static int[] GeneratePascalRow(int rowIndex)
{
    int[] row = new int[rowIndex + 1];
    row[0] = 1;
    for (int i = 1; i <= rowIndex; i++)
    {
        for (int j = i; j > 0; j--)
        {
            row[j] = row[j] + row[j - 1];
        }
    }
    return row;
}
```

### 70. Roman to Integer Conversion
**Problem:** Convert a Roman numeral string to an integer.
```csharp
public static int RomanToInt(string s)
{
    var romanMap = new Dictionary<char, int>
    {
        {'I', 1}, {'V', 5}, {'X', 10}, {'L', 50},
        {'C', 100}, {'D', 500}, {'M', 1000}
    };
    
    int result = 0;
    for (int i = 0; i < s.Length; i++)
    {
        if (i + 1 < s.Length && romanMap[s[i]] < romanMap[s[i + 1]])
            result -= romanMap[s[i]];
        else
            result += romanMap[s[i]];
    }
    return result;
}
```

## Advanced Algorithm Questions

### Bonus: Quick Sort Implementation
**Problem:** Implement the Quick Sort algorithm.
```csharp
public static void QuickSort(int[] arr, int low, int high)
{
    if (low < high)
    {
        int pi = Partition(arr, low, high);
        QuickSort(arr, low, pi - 1);
        QuickSort(arr, pi + 1, high);
    }
}

private static int Partition(int[] arr, int low, int high)
{
    int pivot = arr[high];
    int i = low - 1;
    for (int j = low; j < high; j++)
    {
        if (arr[j] < pivot)
        {
            i++;
            Swap(arr, i, j);
        }
    }
    Swap(arr, i + 1, high);
    return i + 1;
}

private static void Swap(int[] arr, int i, int j)
{
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

### Bonus: Merge Sort Implementation
**Problem:** Implement the Merge Sort algorithm.
```csharp
public static void MergeSort(int[] arr, int left, int right)
{
    if (left < right)
    {
        int mid = left + (right - left) / 2;
        MergeSort(arr, left, mid);
        MergeSort(arr, mid + 1, right);
        Merge(arr, left, mid, right);
    }
}

private static void Merge(int[] arr, int left, int mid, int right)
{
    int[] temp = new int[right - left + 1];
    int i = left, j = mid + 1, k = 0;
    
    while (i <= mid && j <= right)
        temp[k++] = arr[i] <= arr[j] ? arr[i++] : arr[j++];
    
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];
    
    for (i = left; i <= right; i++)
        arr[i] = temp[i - left];
}
```

## Tips for Interview Success

1. **Start with a simple approach**: Even if it's not the most efficient, show you can solve the problem first.
2. **Think out loud**: Explain your thought process as you code.
3. **Test your solution**: Walk through your code with sample inputs.
4. **Consider edge cases**: Empty arrays, null inputs, single elements, etc.
5. **Optimize if asked**: Be ready to improve time/space complexity.
6. **Use meaningful variable names**: Makes your code more readable.
7. **Handle error conditions**: Check for null or invalid inputs when appropriate.

## Common Time Complexities to Know

- **O(1)** - Constant time (accessing array element)
- **O(log n)** - Logarithmic (binary search)
- **O(n)** - Linear (single loop through array)
- **O(n log n)** - Linearithmic (efficient sorting algorithms)
- **O(n²)** - Quadratic (nested loops)

Remember to practice these problems multiple times and understand the underlying concepts rather than just memorizing the solutions!

--- 

Perfect! I've created a comprehensive guide with 70 C# programming questions covering all the key areas you requested. Each problem includes:

1. **Clear problem description**
2. **Clean, concise C# solution** (typically 10-15 lines)
3. **Detailed time and space complexity analysis**
4. **Reasoning behind the complexity calculations**

## Key Areas Covered:

**📊 Arrays & Basic Operations (1-15)**: Essential array manipulations, searching, and basic algorithms

**📝 String Operations (16-30)**: Character processing, pattern matching, and string manipulations  

**🔄 Loops & Iterations (31-45)**: Mathematical computations, number theory, and iterative algorithms

**🔍 Search & Sort Algorithms (46-55)**: Classic algorithms every developer should know

**♻️ Recursion (56-65)**: Recursive thinking and call stack analysis

**🚀 Advanced Problems (66-70)**: Dynamic programming, data structures, and complex algorithms

## How This Helps Your Interview Prep:

1. **Pattern Recognition**: You'll start recognizing common complexity patterns across different problem types
2. **Analysis Skills**: Each explanation teaches you HOW to think about complexity, not just memorize results
3. **Interview Readiness**: These are actual interview-style questions with the level of detail interviewers expect
4. **Progressive Difficulty**: Starts with basics and builds to advanced concepts

The complexity analysis approach I've outlined will serve you well in any technical interview. Focus on understanding the reasoning behind each analysis - that's what separates strong candidates from those who just memorize Big O notation!

--- 
