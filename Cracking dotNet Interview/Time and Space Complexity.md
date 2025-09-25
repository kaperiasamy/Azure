# C# Time & Space Complexity Interview Guide - 70 Questions

## How to Calculate Time and Space Complexity

### Time Complexity Analysis Approach:
1. **Count the basic operations** (comparisons, assignments, arithmetic)
2. **Identify loops and their iterations**
3. **Consider nested structures** (loops within loops multiply complexity)
4. **Analyze recursive calls** (use recurrence relations)
5. **Focus on the dominant term** for large inputs (Big O notation)

### Space Complexity Analysis Approach:
1. **Input space** - space used by input parameters
2. **Auxiliary space** - extra space used by algorithm
3. **Count variables, arrays, and recursive call stack**
4. **Consider space reused vs. new allocations**

### Common Complexity Classes:
- **O(1)** - Constant time/space
- **O(log n)** - Logarithmic (binary search, balanced trees)
- **O(n)** - Linear (single loop through data)
- **O(n log n)** - Linearithmic (efficient sorting)
- **O(n²)** - Quadratic (nested loops)
- **O(2ⁿ)** - Exponential (recursive without memoization)

---

## 70 C# Programming Questions with Complexity Analysis

### **Arrays & Basic Operations (1-15)**

#### 1. Find Maximum Element in Array
**Problem:** Find the largest element in an integer array.
```csharp
public static int FindMax(int[] arr)
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
**Time Complexity:** O(n) - single pass through array  
**Space Complexity:** O(1) - only using one variable

#### 2. Reverse Array In-Place
**Problem:** Reverse an array without using extra space.
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
**Time Complexity:** O(n) - visit each element once  
**Space Complexity:** O(1) - only using index variables

#### 3. Check if Array is Sorted
**Problem:** Determine if array is sorted in ascending order.
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
**Time Complexity:** O(n) - worst case check all elements  
**Space Complexity:** O(1) - no extra space used

#### 4. Find Second Largest Element
**Problem:** Find the second largest element in array.
```csharp
public static int FindSecondLargest(int[] arr)
{
    int first = int.MinValue, second = int.MinValue;
    foreach (int num in arr)
    {
        if (num > first)
        {
            second = first;
            first = num;
        }
        else if (num > second && num != first)
            second = num;
    }
    return second;
}
```
**Time Complexity:** O(n) - single traversal  
**Space Complexity:** O(1) - two variables only

#### 5. Move Zeros to End
**Problem:** Move all zeros to the end while maintaining relative order.
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
    while (writeIndex < arr.Length)
        arr[writeIndex++] = 0;
}
```
**Time Complexity:** O(n) - two passes through array  
**Space Complexity:** O(1) - in-place modification

#### 6. Find Missing Number in Range 1-N
**Problem:** Find missing number in array containing n-1 numbers from 1 to n.
```csharp
public static int FindMissing(int[] arr, int n)
{
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    foreach (int num in arr)
        actualSum += num;
    return expectedSum - actualSum;
}
```
**Time Complexity:** O(n) - sum all elements  
**Space Complexity:** O(1) - only arithmetic variables

#### 7. Find Duplicate in Array
**Problem:** Find duplicate number in array where each number appears at most twice.
```csharp
public static int FindDuplicate(int[] arr)
{
    var seen = new HashSet<int>();
    foreach (int num in arr)
    {
        if (seen.Contains(num))
            return num;
        seen.Add(num);
    }
    return -1;
}
```
**Time Complexity:** O(n) - single pass with HashSet operations  
**Space Complexity:** O(n) - HashSet storage

#### 8. Rotate Array Left by K Positions
**Problem:** Rotate array elements to the left by k positions.
```csharp
public static void RotateLeft(int[] arr, int k)
{
    int n = arr.Length;
    k = k % n;
    Array.Reverse(arr, 0, k);
    Array.Reverse(arr, k, n - k);
    Array.Reverse(arr, 0, n);
}
```
**Time Complexity:** O(n) - three array reversals  
**Space Complexity:** O(1) - in-place rotation

#### 9. Find Intersection of Two Arrays
**Problem:** Find common elements between two arrays.
```csharp
public static List<int> FindIntersection(int[] arr1, int[] arr2)
{
    var set1 = new HashSet<int>(arr1);
    var result = new List<int>();
    foreach (int num in arr2)
    {
        if (set1.Contains(num))
        {
            result.Add(num);
            set1.Remove(num);
        }
    }
    return result;
}
```
**Time Complexity:** O(n + m) - where n, m are array lengths  
**Space Complexity:** O(min(n, m)) - HashSet and result storage

#### 10. Check if Two Arrays are Equal
**Problem:** Determine if two arrays contain same elements in same order.
```csharp
public static bool AreArraysEqual(int[] arr1, int[] arr2)
{
    if (arr1.Length != arr2.Length)
        return false;
    for (int i = 0; i < arr1.Length; i++)
    {
        if (arr1[i] != arr2[i])
            return false;
    }
    return true;
}
```
**Time Complexity:** O(n) - compare each element  
**Space Complexity:** O(1) - no extra space

#### 11. Find Peak Element
**Problem:** Find an element that is greater than its neighbors.
```csharp
public static int FindPeak(int[] arr)
{
    int n = arr.Length;
    if (n == 1) return 0;
    if (arr[0] > arr[1]) return 0;
    if (arr[n-1] > arr[n-2]) return n-1;
    
    for (int i = 1; i < n-1; i++)
    {
        if (arr[i] > arr[i-1] && arr[i] > arr[i+1])
            return i;
    }
    return -1;
}
```
**Time Complexity:** O(n) - linear search  
**Space Complexity:** O(1) - constant space

#### 12. Calculate Array Sum
**Problem:** Calculate sum of all elements in array.
```csharp
public static int CalculateSum(int[] arr)
{
    int sum = 0;
    foreach (int num in arr)
        sum += num;
    return sum;
}
```
**Time Complexity:** O(n) - visit each element once  
**Space Complexity:** O(1) - only sum variable

#### 13. Find Minimum and Maximum
**Problem:** Find both minimum and maximum in single pass.
```csharp
public static (int min, int max) FindMinMax(int[] arr)
{
    int min = arr[0], max = arr[0];
    for (int i = 1; i < arr.Length; i++)
    {
        if (arr[i] < min) min = arr[i];
        if (arr[i] > max) max = arr[i];
    }
    return (min, max);
}
```
**Time Complexity:** O(n) - single traversal  
**Space Complexity:** O(1) - two variables

#### 14. Count Even and Odd Numbers
**Problem:** Count even and odd numbers in array.
```csharp
public static (int even, int odd) CountEvenOdd(int[] arr)
{
    int evenCount = 0, oddCount = 0;
    foreach (int num in arr)
    {
        if (num % 2 == 0)
            evenCount++;
        else
            oddCount++;
    }
    return (evenCount, oddCount);
}
```
**Time Complexity:** O(n) - examine each element  
**Space Complexity:** O(1) - two counters

#### 15. Merge Two Sorted Arrays
**Problem:** Merge two sorted arrays into one sorted array.
```csharp
public static int[] MergeSorted(int[] arr1, int[] arr2)
{
    int[] result = new int[arr1.Length + arr2.Length];
    int i = 0, j = 0, k = 0;
    
    while (i < arr1.Length && j < arr2.Length)
        result[k++] = arr1[i] <= arr2[j] ? arr1[i++] : arr2[j++];
    
    while (i < arr1.Length) result[k++] = arr1[i++];
    while (j < arr2.Length) result[k++] = arr2[j++];
    return result;
}
```
**Time Complexity:** O(n + m) - visit each element once  
**Space Complexity:** O(n + m) - result array

### **String Operations (16-30)**

#### 16. Check if String is Palindrome
**Problem:** Determine if string reads same forwards and backwards.
```csharp
public static bool IsPalindrome(string s)
{
    int left = 0, right = s.Length - 1;
    while (left < right)
    {
        if (s[left] != s[right])
            return false;
        left++;
        right--;
    }
    return true;
}
```
**Time Complexity:** O(n) - check half the characters  
**Space Complexity:** O(1) - two pointers

#### 17. Reverse String
**Problem:** Reverse a string character by character.
```csharp
public static string ReverseString(string s)
{
    char[] chars = s.ToCharArray();
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
**Time Complexity:** O(n) - swap half the characters  
**Space Complexity:** O(n) - character array copy

#### 18. Count Character Frequency
**Problem:** Count frequency of each character in string.
```csharp
public static Dictionary<char, int> CountChars(string s)
{
    var frequency = new Dictionary<char, int>();
    foreach (char c in s)
    {
        if (frequency.ContainsKey(c))
            frequency[c]++;
        else
            frequency[c] = 1;
    }
    return frequency;
}
```
**Time Complexity:** O(n) - visit each character  
**Space Complexity:** O(k) - where k is unique characters

#### 19. Find First Non-Repeating Character
**Problem:** Find first character that appears exactly once.
```csharp
public static char FirstNonRepeating(string s)
{
    var frequency = new Dictionary<char, int>();
    foreach (char c in s)
        frequency[c] = frequency.GetValueOrDefault(c, 0) + 1;
    
    foreach (char c in s)
    {
        if (frequency[c] == 1)
            return c;
    }
    return '\0';
}
```
**Time Complexity:** O(n) - two passes through string  
**Space Complexity:** O(k) - frequency dictionary

#### 20. Check if Two Strings are Anagrams
**Problem:** Check if two strings contain same characters with same frequency.
```csharp
public static bool AreAnagrams(string s1, string s2)
{
    if (s1.Length != s2.Length) return false;
    
    var count = new Dictionary<char, int>();
    foreach (char c in s1)
        count[c] = count.GetValueOrDefault(c, 0) + 1;
    
    foreach (char c in s2)
    {
        if (!count.ContainsKey(c) || count[c] == 0)
            return false;
        count[c]--;
    }
    return true;
}
```
**Time Complexity:** O(n) - process each string once  
**Space Complexity:** O(k) - character count dictionary

#### 21. Remove Duplicates from String
**Problem:** Remove duplicate characters keeping first occurrence.
```csharp
public static string RemoveDuplicates(string s)
{
    var seen = new HashSet<char>();
    var result = new StringBuilder();
    foreach (char c in s)
    {
        if (!seen.Contains(c))
        {
            seen.Add(c);
            result.Append(c);
        }
    }
    return result.ToString();
}
```
**Time Complexity:** O(n) - single pass through string  
**Space Complexity:** O(n) - HashSet and StringBuilder

#### 22. Find Longest Word in Sentence
**Problem:** Find the longest word in a sentence.
```csharp
public static string FindLongestWord(string sentence)
{
    string[] words = sentence.Split(' ');
    string longest = "";
    foreach (string word in words)
    {
        if (word.Length > longest.Length)
            longest = word;
    }
    return longest;
}
```
**Time Complexity:** O(n) - where n is total characters  
**Space Complexity:** O(m) - where m is number of words

#### 23. Count Words in String
**Problem:** Count number of words in a sentence.
```csharp
public static int CountWords(string sentence)
{
    if (string.IsNullOrWhiteSpace(sentence))
        return 0;
    
    string[] words = sentence.Trim().Split(new char[] { ' ' }, 
        StringSplitOptions.RemoveEmptyEntries);
    return words.Length;
}
```
**Time Complexity:** O(n) - process each character  
**Space Complexity:** O(m) - array of words

#### 24. Check if String Contains Only Digits
**Problem:** Verify if string contains only numeric characters.
```csharp
public static bool IsNumeric(string s)
{
    if (string.IsNullOrEmpty(s))
        return false;
    
    foreach (char c in s)
    {
        if (!char.IsDigit(c))
            return false;
    }
    return true;
}
```
**Time Complexity:** O(n) - check each character  
**Space Complexity:** O(1) - no extra space

#### 25. Replace Characters in String
**Problem:** Replace all occurrences of one character with another.
```csharp
public static string ReplaceChar(string s, char oldChar, char newChar)
{
    var result = new StringBuilder();
    foreach (char c in s)
    {
        result.Append(c == oldChar ? newChar : c);
    }
    return result.ToString();
}
```
**Time Complexity:** O(n) - process each character  
**Space Complexity:** O(n) - StringBuilder storage

#### 26. Find Substring Index
**Problem:** Find first occurrence of substring in main string.
```csharp
public static int FindSubstring(string main, string sub)
{
    for (int i = 0; i <= main.Length - sub.Length; i++)
    {
        int j = 0;
        while (j < sub.Length && main[i + j] == sub[j])
            j++;
        if (j == sub.Length)
            return i;
    }
    return -1;
}
```
**Time Complexity:** O(n*m) - nested character comparison  
**Space Complexity:** O(1) - only index variables

#### 27. Convert String to Title Case
**Problem:** Capitalize first letter of each word.
```csharp
public static string ToTitleCase(string s)
{
    var words = s.ToLower().Split(' ');
    for (int i = 0; i < words.Length; i++)
    {
        if (words[i].Length > 0)
            words[i] = char.ToUpper(words[i][0]) + words[i].Substring(1);
    }
    return string.Join(" ", words);
}
```
**Time Complexity:** O(n) - process each character  
**Space Complexity:** O(n) - word array and result

#### 28. Check Balanced Parentheses
**Problem:** Check if parentheses are properly balanced.
```csharp
public static bool IsBalanced(string s)
{
    int count = 0;
    foreach (char c in s)
    {
        if (c == '(') count++;
        else if (c == ')') count--;
        if (count < 0) return false;
    }
    return count == 0;
}
```
**Time Complexity:** O(n) - single pass through string  
**Space Complexity:** O(1) - only counter variable

#### 29. Remove Spaces from String
**Problem:** Remove all whitespace characters from string.
```csharp
public static string RemoveSpaces(string s)
{
    var result = new StringBuilder();
    foreach (char c in s)
    {
        if (c != ' ')
            result.Append(c);
    }
    return result.ToString();
}
```
**Time Complexity:** O(n) - examine each character  
**Space Complexity:** O(n) - StringBuilder storage

#### 30. Find Common Prefix
**Problem:** Find longest common prefix among array of strings.
```csharp
public static string FindCommonPrefix(string[] strs)
{
    if (strs.Length == 0) return "";
    
    for (int i = 0; i < strs[0].Length; i++)
    {
        char c = strs[0][i];
        for (int j = 1; j < strs.Length; j++)
        {
            if (i >= strs[j].Length || strs[j][i] != c)
                return strs[0].Substring(0, i);
        }
    }
    return strs[0];
}
```
**Time Complexity:** O(S) - where S is sum of all string lengths  
**Space Complexity:** O(1) - only index variables

### **Loops & Iterations (31-45)**

#### 31. Print Multiplication Table
**Problem:** Print multiplication table for given number up to 10.
```csharp
public static void PrintTable(int number)
{
    for (int i = 1; i <= 10; i++)
    {
        Console.WriteLine($"{number} x {i} = {number * i}");
    }
}
```
**Time Complexity:** O(1) - fixed 10 iterations  
**Space Complexity:** O(1) - no extra space

#### 32. Calculate Factorial Iteratively
**Problem:** Calculate factorial of a number using iteration.
```csharp
public static long Factorial(int n)
{
    long result = 1;
    for (int i = 2; i <= n; i++)
    {
        result *= i;
    }
    return result;
}
```
**Time Complexity:** O(n) - loop n times  
**Space Complexity:** O(1) - only result variable

#### 33. Calculate Power of Number
**Problem:** Calculate base raised to power exponent.
```csharp
public static double Power(double baseNum, int exponent)
{
    double result = 1;
    int absExp = Math.Abs(exponent);
    for (int i = 0; i < absExp; i++)
    {
        result *= baseNum;
    }
    return exponent < 0 ? 1.0 / result : result;
}
```
**Time Complexity:** O(n) - where n is exponent  
**Space Complexity:** O(1) - only result variable

#### 34. Generate Fibonacci Series
**Problem:** Generate first n numbers of Fibonacci sequence.
```csharp
public static List<int> GenerateFibonacci(int n)
{
    var result = new List<int>();
    int a = 0, b = 1;
    for (int i = 0; i < n; i++)
    {
        result.Add(a);
        int temp = a + b;
        a = b;
        b = temp;
    }
    return result;
}
```
**Time Complexity:** O(n) - generate n numbers  
**Space Complexity:** O(n) - store n numbers

#### 35. Check if Number is Prime
**Problem:** Determine if given number is prime.
```csharp
public static bool IsPrime(int n)
{
    if (n <= 1) return false;
    if (n == 2) return true;
    if (n % 2 == 0) return false;
    
    for (int i = 3; i <= Math.Sqrt(n); i += 2)
    {
        if (n % i == 0)
            return false;
    }
    return true;
}
```
**Time Complexity:** O(√n) - check up to square root  
**Space Complexity:** O(1) - only loop variable

#### 36. Sum of Digits
**Problem:** Calculate sum of all digits in a number.
```csharp
public static int SumDigits(int number)
{
    int sum = 0;
    number = Math.Abs(number);
    while (number > 0)
    {
        sum += number % 10;
        number /= 10;
    }
    return sum;
}
```
**Time Complexity:** O(log n) - number of digits  
**Space Complexity:** O(1) - only sum variable

#### 37. Reverse a Number
**Problem:** Reverse the digits of an integer.
```csharp
public static int ReverseNumber(int number)
{
    int reversed = 0;
    bool isNegative = number < 0;
    number = Math.Abs(number);
    
    while (number > 0)
    {
        reversed = reversed * 10 + number % 10;
        number /= 10;
    }
    return isNegative ? -reversed : reversed;
}
```
**Time Complexity:** O(log n) - process each digit  
**Space Complexity:** O(1) - only variables

#### 38. Count Digits in Number
**Problem:** Count total number of digits in an integer.
```csharp
public static int CountDigits(int number)
{
    if (number == 0) return 1;
    
    int count = 0;
    number = Math.Abs(number);
    while (number > 0)
    {
        count++;
        number /= 10;
    }
    return count;
}
```
**Time Complexity:** O(log n) - divide by 10 repeatedly  
**Space Complexity:** O(1) - only counter

#### 39. Find GCD of Two Numbers
**Problem:** Find Greatest Common Divisor using Euclidean algorithm.
```csharp
public static int FindGCD(int a, int b)
{
    a = Math.Abs(a);
    b = Math.Abs(b);
    
    while (b != 0)
    {
        int temp = b;
        b = a % b;
        a = temp;
    }
    return a;
}
```
**Time Complexity:** O(log min(a,b)) - Euclidean algorithm  
**Space Complexity:** O(1) - only temp variable

#### 40. Print Pattern (Right Triangle)
**Problem:** Print right triangle pattern with stars.
```csharp
public static void PrintTriangle(int n)
{
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= i; j++)
        {
            Console.Write("* ");
        }
        Console.WriteLine();
    }
}
```
**Time Complexity:** O(n²) - nested loops  
**Space Complexity:** O(1) - only loop variables

#### 41. Find LCM of Two Numbers
**Problem:** Find Least Common Multiple of two numbers.
```csharp
public static int FindLCM(int a, int b)
{
    return Math.Abs(a * b) / FindGCD(a, b);
}

private static int FindGCD(int a, int b)
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
**Time Complexity:** O(log min(a,b)) - depends on GCD  
**Space Complexity:** O(1) - constant space

#### 42. Check Perfect Number
**Problem:** Check if number equals sum of its proper divisors.
```csharp
public static bool IsPerfect(int n)
{
    if (n <= 1) return false;
    
    int sum = 1;
    for (int i = 2; i <= Math.Sqrt(n); i++)
    {
        if (n % i == 0)
        {
            sum += i;
            if (i != n / i) // Avoid adding same divisor twice for perfect squares
                sum += n / i;
        }
    }
    return sum == n;
}
```
**Time Complexity:** O(√n) - check divisors up to square root  
**Space Complexity:** O(1) - only sum variable

#### 43. Generate Prime Numbers up to N
**Problem:** Generate all prime numbers up to given limit.
```csharp
public static List<int> GeneratePrimes(int n)
{
    var primes = new List<int>();
    for (int i = 2; i <= n; i++)
    {
        bool isPrime = true;
        for (int j = 2; j <= Math.Sqrt(i); j++)
        {
            if (i % j == 0)
            {
                isPrime = false;
                break;
            }
        }
        if (isPrime) primes.Add(i);
    }
    return primes;
}
```
**Time Complexity:** O(n√n) - check each number up to √i  
**Space Complexity:** O(π(n)) - store prime numbers

#### 44. Calculate Sum of Natural Numbers
**Problem:** Calculate sum of first n natural numbers.
```csharp
public static int SumNaturals(int n)
{
    return n * (n + 1) / 2;
}

// Alternative iterative approach
public static int SumNaturalsIterative(int n)
{
    int sum = 0;
    for (int i = 1; i <= n; i++)
    {
        sum += i;
    }
    return sum;
}
```
**Time Complexity:** O(1) for formula, O(n) for iterative  
**Space Complexity:** O(1) - constant space

#### 45. Find Armstrong Number
**Problem:** Check if number equals sum of cubes of its digits.
```csharp
public static bool IsArmstrong(int number)
{
    int original = number;
    int sum = 0;
    int digits = CountDigits(number);
    
    while (number > 0)
    {
        int digit = number % 10;
        sum += (int)Math.Pow(digit, digits);
        number /= 10;
    }
    return sum == original;
}
```
**Time Complexity:** O(log n) - process each digit  
**Space Complexity:** O(1) - only variables

### **Search & Sort Algorithms (46-55)**

#### 46. Linear Search
**Problem:** Find element in array using linear search.
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
**Time Complexity:** O(n) - may need to check all elements  
**Space Complexity:** O(1) - only index variable

#### 47. Binary Search (Iterative)
**Problem:** Find element in sorted array using binary search.
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
**Time Complexity:** O(log n) - halve search space each iteration  
**Space Complexity:** O(1) - only variables

#### 48. Bubble Sort
**Problem:** Sort array using bubble sort algorithm.
```csharp
public static void BubbleSort(int[] arr)
{
    int n = arr.Length;
    for (int i = 0; i < n - 1; i++)
    {
        bool swapped = false;
        for (int j = 0; j < n - i - 1; j++)
        {
            if (arr[j] > arr[j + 1])
            {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = true;
            }
        }
        if (!swapped) break;
    }
}
```
**Time Complexity:** O(n²) worst case, O(n) best case  
**Space Complexity:** O(1) - in-place sorting

#### 49. Selection Sort
**Problem:** Sort array using selection sort algorithm.
```csharp
public static void SelectionSort(int[] arr)
{
    int n = arr.Length;
    for (int i = 0; i < n - 1; i++)
    {
        int minIndex = i;
        for (int j = i + 1; j < n; j++)
        {
            if (arr[j] < arr[minIndex])
                minIndex = j;
        }
        int temp = arr[i];
        arr[i] = arr[minIndex];
        arr[minIndex] = temp;
    }
}
```
**Time Complexity:** O(n²) - nested loops always  
**Space Complexity:** O(1) - in-place sorting

#### 50. Insertion Sort
**Problem:** Sort array using insertion sort algorithm.
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
**Time Complexity:** O(n²) worst case, O(n) best case  
**Space Complexity:** O(1) - in-place sorting

#### 51. Quick Sort (Simple Version)
**Problem:** Sort array using quicksort algorithm.
```csharp
public static void QuickSort(int[] arr, int low, int high)
{
    if (low < high)
    {
        int pivotIndex = Partition(arr, low, high);
        QuickSort(arr, low, pivotIndex - 1);
        QuickSort(arr, pivotIndex + 1, high);
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
            (arr[i], arr[j]) = (arr[j], arr[i]);
        }
    }
    (arr[i + 1], arr[high]) = (arr[high], arr[i + 1]);
    return i + 1;
}
```
**Time Complexity:** O(n log n) average, O(n²) worst case  
**Space Complexity:** O(log n) - recursion stack

#### 52. Merge Sort (Simple Version)
**Problem:** Sort array using merge sort algorithm.
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
    
    Array.Copy(temp, 0, arr, left, temp.Length);
}
```
**Time Complexity:** O(n log n) - always  
**Space Complexity:** O(n) - temporary arrays

#### 53. Find Kth Largest Element
**Problem:** Find kth largest element in unsorted array.
```csharp
public static int FindKthLargest(int[] arr, int k)
{
    Array.Sort(arr, (a, b) => b.CompareTo(a)); // Sort descending
    return arr[k - 1];
}

// Alternative: Using partial sorting
public static int FindKthLargestOptimal(int[] arr, int k)
{
    var maxHeap = new SortedSet<int>(Comparer<int>.Create((a, b) => b.CompareTo(a)));
    foreach (int num in arr.Take(k))
        maxHeap.Add(num);
    
    for (int i = k; i < arr.Length; i++)
    {
        if (arr[i] > maxHeap.Min)
        {
            maxHeap.Remove(maxHeap.Min);
            maxHeap.Add(arr[i]);
        }
    }
    return maxHeap.Min;
}
```
**Time Complexity:** O(n log n) for sorting, O(n log k) for heap  
**Space Complexity:** O(1) for sorting, O(k) for heap

#### 54. Binary Search in Rotated Array
**Problem:** Search in sorted array that has been rotated.
```csharp
public static int SearchRotated(int[] arr, int target)
{
    int left = 0, right = arr.Length - 1;
    
    while (left <= right)
    {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        
        if (arr[left] <= arr[mid]) // Left half is sorted
        {
            if (target >= arr[left] && target < arr[mid])
                right = mid - 1;
            else
                left = mid + 1;
        }
        else // Right half is sorted
        {
            if (target > arr[mid] && target <= arr[right])
                left = mid + 1;
            else
                right = mid - 1;
        }
    }
    return -1;
}
```
**Time Complexity:** O(log n) - binary search with rotation logic  
**Space Complexity:** O(1) - only variables

#### 55. Count Occurrences in Sorted Array
**Problem:** Count number of times target appears in sorted array.
```csharp
public static int CountOccurrences(int[] arr, int target)
{
    int first = FindFirst(arr, target);
    if (first == -1) return 0;
    
    int last = FindLast(arr, target);
    return last - first + 1;
}

private static int FindFirst(int[] arr, int target)
{
    int left = 0, right = arr.Length - 1, result = -1;
    while (left <= right)
    {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target)
        {
            result = mid;
            right = mid - 1; // Continue searching left
        }
        else if (arr[mid] < target)
            left = mid + 1;
        else
            right = mid - 1;
    }
    return result;
}

private static int FindLast(int[] arr, int target)
{
    int left = 0, right = arr.Length - 1, result = -1;
    while (left <= right)
    {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target)
        {
            result = mid;
            left = mid + 1; // Continue searching right
        }
        else if (arr[mid] < target)
            left = mid + 1;
        else
            right = mid - 1;
    }
    return result;
}
```
**Time Complexity:** O(log n) - two binary searches  
**Space Complexity:** O(1) - only variables

### **Recursion (56-65)**

#### 56. Factorial (Recursive)
**Problem:** Calculate factorial using recursion.
```csharp
public static long FactorialRecursive(int n)
{
    if (n <= 1)
        return 1;
    return n * FactorialRecursive(n - 1);
}
```
**Time Complexity:** O(n) - n recursive calls  
**Space Complexity:** O(n) - call stack depth

#### 57. Fibonacci (Recursive)
**Problem:** Calculate nth Fibonacci number using recursion.
```csharp
public static int FibonacciRecursive(int n)
{
    if (n <= 1)
        return n;
    return FibonacciRecursive(n - 1) + FibonacciRecursive(n - 2);
}

// Optimized with memoization
public static int FibonacciMemo(int n, Dictionary<int, int> memo = null)
{
    if (memo == null) memo = new Dictionary<int, int>();
    if (n <= 1) return n;
    if (memo.ContainsKey(n)) return memo[n];
    
    memo[n] = FibonacciMemo(n - 1, memo) + FibonacciMemo(n - 2, memo);
    return memo[n];
}
```
**Time Complexity:** O(2ⁿ) naive, O(n) with memoization  
**Space Complexity:** O(n) - call stack and memoization

#### 58. Power (Recursive)
**Problem:** Calculate power using recursion (optimized).
```csharp
public static double PowerRecursive(double baseNum, int exponent)
{
    if (exponent == 0) return 1;
    if (exponent < 0) return 1.0 / PowerRecursive(baseNum, -exponent);
    
    if (exponent % 2 == 0)
    {
        double half = PowerRecursive(baseNum, exponent / 2);
        return half * half;
    }
    else
    {
        return baseNum * PowerRecursive(baseNum, exponent - 1);
    }
}
```
**Time Complexity:** O(log n) - divide exponent by 2  
**Space Complexity:** O(log n) - recursion stack

#### 59. Binary Search (Recursive)
**Problem:** Implement binary search recursively.
```csharp
public static int BinarySearchRecursive(int[] arr, int target, int left, int right)
{
    if (left > right)
        return -1;
    
    int mid = left + (right - left) / 2;
    if (arr[mid] == target)
        return mid;
    
    if (arr[mid] > target)
        return BinarySearchRecursive(arr, target, left, mid - 1);
    else
        return BinarySearchRecursive(arr, target, mid + 1, right);
}
```
**Time Complexity:** O(log n) - halve search space  
**Space Complexity:** O(log n) - recursion stack

#### 60. Sum of Array (Recursive)
**Problem:** Calculate sum of array elements recursively.
```csharp
public static int SumArrayRecursive(int[] arr, int index = 0)
{
    if (index >= arr.Length)
        return 0;
    return arr[index] + SumArrayRecursive(arr, index + 1);
}
```
**Time Complexity:** O(n) - visit each element once  
**Space Complexity:** O(n) - recursion stack

#### 61. Reverse String (Recursive)
**Problem:** Reverse string using recursion.
```csharp
public static string ReverseStringRecursive(string s)
{
    if (s.Length <= 1)
        return s;
    return s[s.Length - 1] + ReverseStringRecursive(s.Substring(0, s.Length - 1));
}
```
**Time Complexity:** O(n²) - substring creation in each call  
**Space Complexity:** O(n²) - string storage and call stack

#### 62. Check Palindrome (Recursive)
**Problem:** Check if string is palindrome recursively.
```csharp
public static bool IsPalindromeRecursive(string s, int left = 0, int right = -1)
{
    if (right == -1) right = s.Length - 1;
    
    if (left >= right)
        return true;
    
    if (s[left] != s[right])
        return false;
    
    return IsPalindromeRecursive(s, left + 1, right - 1);
}
```
**Time Complexity:** O(n) - check half the characters  
**Space Complexity:** O(n) - recursion stack

#### 63. Find Maximum in Array (Recursive)
**Problem:** Find maximum element recursively.
```csharp
public static int FindMaxRecursive(int[] arr, int index = 0)
{
    if (index == arr.Length - 1)
        return arr[index];
    
    int maxOfRest = FindMaxRecursive(arr, index + 1);
    return Math.Max(arr[index], maxOfRest);
}
```
**Time Complexity:** O(n) - visit each element  
**Space Complexity:** O(n) - call stack

#### 64. Count Digits (Recursive)
**Problem:** Count digits in number recursively.
```csharp
public static int CountDigitsRecursive(int number)
{
    if (number == 0)
        return 1;
    
    number = Math.Abs(number);
    if (number < 10)
        return 1;
    
    return 1 + CountDigitsRecursive(number / 10);
}
```
**Time Complexity:** O(log n) - number of digits  
**Space Complexity:** O(log n) - recursion stack

#### 65. Tower of Hanoi
**Problem:** Solve Tower of Hanoi puzzle recursively.
```csharp
public static void TowerOfHanoi(int n, char source, char destination, char auxiliary)
{
    if (n == 1)
    {
        Console.WriteLine($"Move disk 1 from {source} to {destination}");
        return;
    }
    
    TowerOfHanoi(n - 1, source, auxiliary, destination);
    Console.WriteLine($"Move disk {n} from {source} to {destination}");
    TowerOfHanoi(n - 1, auxiliary, destination, source);
}
```
**Time Complexity:** O(2ⁿ) - exponential moves  
**Space Complexity:** O(n) - recursion stack depth

### **Advanced Problems (66-70)**

#### 66. Longest Common Subsequence
**Problem:** Find length of longest common subsequence between two strings.
```csharp
public static int LCS(string s1, string s2)
{
    int[,] dp = new int[s1.Length + 1, s2.Length + 1];
    
    for (int i = 1; i <= s1.Length; i++)
    {
        for (int j = 1; j <= s2.Length; j++)
        {
            if (s1[i - 1] == s2[j - 1])
                dp[i, j] = dp[i - 1, j - 1] + 1;
            else
                dp[i, j] = Math.Max(dp[i - 1, j], dp[i, j - 1]);
        }
    }
    return dp[s1.Length, s2.Length];
}
```
**Time Complexity:** O(m*n) - fill m×n table  
**Space Complexity:** O(m*n) - DP table storage

#### 67. Coin Change Problem
**Problem:** Find minimum coins needed to make given amount.
```csharp
public static int CoinChange(int[] coins, int amount)
{
    int[] dp = new int[amount + 1];
    Array.Fill(dp, amount + 1);
    dp[0] = 0;
    
    for (int i = 1; i <= amount; i++)
    {
        foreach (int coin in coins)
        {
            if (coin <= i)
                dp[i] = Math.Min(dp[i], dp[i - coin] + 1);
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
```
**Time Complexity:** O(amount × coins) - nested loops  
**Space Complexity:** O(amount) - DP array

#### 68. Valid Parentheses (Multiple Types)
**Problem:** Check if string has valid parentheses of multiple types.
```csharp
public static bool IsValidParentheses(string s)
{
    var stack = new Stack<char>();
    var pairs = new Dictionary<char, char> { {')', '('}, {'}', '{'}, {']', '['} };
    
    foreach (char c in s)
    {
        if (c == '(' || c == '{' || c == '[')
        {
            stack.Push(c);
        }
        else if (pairs.ContainsKey(c))
        {
            if (stack.Count == 0 || stack.Pop() != pairs[c])
                return false;
        }
    }
    return stack.Count == 0;
}
```
**Time Complexity:** O(n) - single pass through string  
**Space Complexity:** O(n) - stack storage

#### 69. Sliding Window Maximum
**Problem:** Find maximum in each sliding window of size k.
```csharp
public static int[] SlidingWindowMaximum(int[] arr, int k)
{
    if (arr.Length == 0 || k == 0) return new int[0];
    
    int[] result = new int[arr.Length - k + 1];
    var deque = new LinkedList<int>(); // Store indices
    
    for (int i = 0; i < arr.Length; i++)
    {
        // Remove indices outside current window
        while (deque.Count > 0 && deque.First.Value <= i - k)
            deque.RemoveFirst();
        
        // Remove indices with smaller values
        while (deque.Count > 0 && arr[deque.Last.Value] <= arr[i])
            deque.RemoveLast();
        
        deque.AddLast(i);
        
        // Add to result when window is complete
        if (i >= k - 1)
            result[i - k + 1] = arr[deque.First.Value];
    }
    return result;
}
```
**Time Complexity:** O(n) - each element added/removed once  
**Space Complexity:** O(k) - deque size limited to k

#### 70. Implement LRU Cache
**Problem:** Implement Least Recently Used cache with get/put operations.
```csharp
public class LRUCache
{
    private Dictionary<int, LinkedListNode<(int key, int value)>> cache;
    private LinkedList<(int key, int value)> order;
    private int capacity;

    public LRUCache(int capacity)
    {
        this.capacity = capacity;
        cache = new Dictionary<int, LinkedListNode<(int, int)>>();
        order = new LinkedList<(int, int)>();
    }

    public int Get(int key)
    {
        if (cache.ContainsKey(key))
        {
            var node = cache[key];
            order.Remove(node);
            order.AddFirst(node);
            return node.Value.value;
        }
        return -1;
    }

    public void Put(int key, int value)
    {
        if (cache.ContainsKey(key))
        {
            var node = cache[key];
            node.Value = (key, value);
            order.Remove(node);
            order.AddFirst(node);
        }
        else
        {
            if (cache.Count >= capacity)
            {
                var lastNode = order.Last;
                cache.Remove(lastNode.Value.key);
                order.RemoveLast();
            }
            
            var newNode = order.AddFirst((key, value));
            cache[key] = newNode;
        }
    }
}
```
**Time Complexity:** O(1) for both get and put operations  
**Space Complexity:** O(capacity) - dictionary and linked list storage

---

## Summary: Common Complexity Patterns

### **Time Complexity Patterns:**
- **O(1)**: Array access, arithmetic operations, hash operations
- **O(log n)**: Binary search, balanced tree operations, divide & conquer
- **O(n)**: Single loop, linear search, simple array operations
- **O(n log n)**: Efficient sorting (merge, heap, quick), divide & conquer with linear merge
- **O(n²)**: Nested loops, bubble/selection/insertion sort, naive string matching
- **O(2ⁿ)**: Recursive algorithms without memoization (naive Fibonacci)

### **Space Complexity Patterns:**
- **O(1)**: In-place algorithms, only using variables
- **O(log n)**: Recursive algorithms (call stack), binary search
- **O(n)**: Additional arrays, hash tables, memoization
- **O(n²)**: 2D arrays, dynamic programming tables

### **Key Analysis Tips:**
1. **Identify the input size** (n, m, etc.)
2. **Count nested loops** (multiply complexities)
3. **Consider best, average, and worst cases**
4. **Account for recursive call stack space**
5. **Distinguish between auxiliary and total space**
6. **Look for optimization opportunities** (early termination, memoization)

Practice these problems and focus on explaining your reasoning for each complexity analysis. Understanding the "why" behind each complexity is crucial for technical interviews!

--- 

For the following  Time Complexity Patterns:
- O(1) 
- O(log n)
- O(n)
- O(n log n)
- O(n²)
- O(2ⁿ)

and Space Complexity Patterns:
- O(1)
- O(log n)
- O(n)
- O(n²)

elaborate the complexity analysis techniques, with detailed explanations for each pattern

---

I'll provide a comprehensive guide to complexity analysis techniques, covering both time and space complexity patterns with detailed explanations and examples.This comprehensive guide covers all the complexity patterns you mentioned with detailed analysis techniques for each. The guide includes:

**For Time Complexity:**
- **O(1)**: Constant operations like array access
- **O(log n)**: Divide-and-conquer algorithms like binary search
- **O(n)**: Single-pass algorithms like linear search
- **O(n log n)**: Efficient sorting algorithms like merge sort
- **O(n²)**: Nested loops like bubble sort
- **O(2ⁿ)**: Exponential algorithms like naive Fibonacci

**For Space Complexity:**
- **O(1)**: Fixed variables, in-place algorithms
- **O(log n)**: Recursion stack depth in balanced algorithms
- **O(n)**: Linear data structures, copies of input
- **O(n²)**: 2D matrices, dynamic programming tables

Each section includes:
- Clear definitions and characteristics
- Code examples with detailed analysis
- Recognition patterns to identify complexity
- Mathematical reasoning behind the complexity
- Practical analysis techniques

The guide also covers advanced topics like amortized analysis, recurrence relations, and systematic steps for analyzing any algorithm. This should give you a solid foundation for complexity analysis across different algorithmic patterns.

# Complete Guide to Time and Space Complexity Analysis

## Introduction to Complexity Analysis

Complexity analysis is a fundamental technique in computer science for evaluating the efficiency of algorithms. It helps us understand how an algorithm's resource requirements (time and space) grow as the input size increases.

### Key Concepts
- **Big O Notation**: Describes the upper bound of an algorithm's growth rate
- **Input Size (n)**: The size of the data being processed
- **Worst-case Analysis**: Focus on the maximum resources needed
- **Asymptotic Behavior**: How the algorithm behaves as n approaches infinity

---

## Time Complexity Patterns

### O(1) - Constant Time

**Definition**: The algorithm's execution time remains constant regardless of input size.

**Characteristics**:
- No loops or recursion that depend on input size
- Direct access operations
- Simple arithmetic operations
- Array/hash table access by index/key

**Analysis Techniques**:
```python
# Example 1: Array access
def get_first_element(arr):
    return arr[0]  # Always takes same time regardless of array size

# Example 2: Simple calculations
def calculate_area(length, width):
    return length * width  # Constant number of operations
```

**Recognition Patterns**:
- Single statements or fixed number of statements
- Hash table lookups
- Stack push/pop operations
- Mathematical formulas

**Real-world Applications**:
- Database index lookups
- Cache retrieval
- Basic arithmetic operations

---

### O(log n) - Logarithmic Time

**Definition**: The algorithm's time grows logarithmically with input size, typically by repeatedly dividing the problem in half.

**Characteristics**:
- Each iteration reduces the problem size by a constant factor (usually half)
- Common in divide-and-conquer algorithms
- Extremely efficient even for large inputs

**Analysis Techniques**:
```python
# Example 1: Binary Search
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    
    while left <= right:          # Loop runs log n times
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1        # Eliminate half the elements
        else:
            right = mid - 1       # Eliminate half the elements
    
    return -1

# Analysis: At each step, we eliminate half the remaining elements
# Maximum iterations: log₂(n)
```

**Recognition Patterns**:
- Binary search variations
- Tree traversal (height-balanced trees)
- Algorithms that repeatedly divide input by constant factor
- Certain recursive algorithms with divide-and-conquer approach

**Mathematical Analysis**:
- If we can eliminate half the input at each step: log₂(n) iterations
- Tree height in balanced binary trees: log(n)
- Number of times you can divide n by 2 until you get 1: log₂(n)

---

### O(n) - Linear Time

**Definition**: The algorithm's time grows linearly with the input size.

**Characteristics**:
- Single pass through the data
- Each element is processed once
- Most common complexity for basic operations

**Analysis Techniques**:
```python
# Example 1: Linear Search
def linear_search(arr, target):
    for i in range(len(arr)):     # Loop runs n times
        if arr[i] == target:      # Constant time operation
            return i
    return -1

# Example 2: Sum of array
def sum_array(arr):
    total = 0
    for num in arr:               # Loop runs n times
        total += num              # Constant time operation
    return total
```

**Recognition Patterns**:
- Single loop iterating through input
- Processing each element once
- Simple traversals (arrays, linked lists)

**Analysis Steps**:
1. Count the number of times each operation executes
2. If there's a single loop over n elements: O(n)
3. Nested operations within the loop should be O(1) for overall O(n)

---

### O(n log n) - Linearithmic Time

**Definition**: Combination of linear and logarithmic growth, often seen in efficient sorting algorithms.

**Characteristics**:
- Usually involves dividing the problem and solving subproblems
- Optimal time complexity for comparison-based sorting
- Common in divide-and-conquer algorithms

**Analysis Techniques**:
```python
# Example 1: Merge Sort Analysis
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    # Divide: O(1)
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])      # T(n/2)
    right = merge_sort(arr[mid:])     # T(n/2)
    
    # Conquer: O(n)
    return merge(left, right)

# Recurrence Relation: T(n) = 2T(n/2) + O(n)
# Solution: T(n) = O(n log n)
```

**Recognition Patterns**:
- Efficient sorting algorithms (merge sort, heap sort, quick sort average case)
- Divide-and-conquer with linear merge step
- Tree operations with processing at each level

**Mathematical Analysis**:
- Divide problem into halves: log n levels
- Each level processes n elements: n operations per level
- Total: n × log n operations

---

### O(n²) - Quadratic Time

**Definition**: The algorithm's time grows quadratically with input size.

**Characteristics**:
- Often involves nested loops
- Each element is compared/processed with every other element
- Common in brute-force algorithms

**Analysis Techniques**:
```python
# Example 1: Bubble Sort
def bubble_sort(arr):
    n = len(arr)
    for i in range(n):           # Outer loop: n iterations
        for j in range(0, n-i-1): # Inner loop: decreasing iterations
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]

# Analysis: 
# Total comparisons ≈ n + (n-1) + (n-2) + ... + 1 = n(n-1)/2 = O(n²)

# Example 2: Finding all pairs
def find_all_pairs(arr):
    pairs = []
    for i in range(len(arr)):      # n iterations
        for j in range(i+1, len(arr)): # (n-1) + (n-2) + ... + 1 iterations
            pairs.append((arr[i], arr[j]))
    return pairs
```

**Recognition Patterns**:
- Nested loops over the same dataset
- Comparing every element with every other element
- Simple sorting algorithms (bubble, selection, insertion)

**Analysis Steps**:
1. Identify nested loop structure
2. Calculate iterations: outer loop × inner loop
3. For equal-sized loops: n × n = n²

---

### O(2ⁿ) - Exponential Time

**Definition**: The algorithm's time doubles with each additional input element.

**Characteristics**:
- Often involves exploring all possible combinations
- Recursive algorithms with multiple recursive calls
- Usually impractical for large inputs

**Analysis Techniques**:
```python
# Example 1: Fibonacci (naive approach)
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Analysis:
# T(n) = T(n-1) + T(n-2) + O(1)
# This creates a binary tree of recursive calls
# Height: n, Each level doubles the number of calls
# Total calls: approximately 2ⁿ

# Example 2: Generate all subsets
def generate_subsets(arr):
    if not arr:
        return [[]]
    
    first = arr[0]
    rest_subsets = generate_subsets(arr[1:])  # 2ⁿ⁻¹ subsets
    
    # For each existing subset, create two versions:
    # one without first element, one with first element
    return rest_subsets + [[first] + subset for subset in rest_subsets]
```

**Recognition Patterns**:
- Recursive algorithms with 2+ recursive calls per invocation
- Algorithms exploring all possible combinations/permutations
- Brute-force solutions to optimization problems

**Mathematical Analysis**:
- Each recursive call branches into multiple calls
- For binary recursion: 2ⁿ total calls in worst case
- Growth: 2¹, 2², 2³, ..., 2ⁿ

---

## Space Complexity Patterns

### O(1) - Constant Space

**Definition**: The algorithm uses a fixed amount of extra space regardless of input size.

**Characteristics**:
- Only uses a constant number of variables
- No data structures that grow with input
- In-place algorithms

**Analysis Techniques**:
```python
# Example 1: In-place array reversal
def reverse_array(arr):
    left, right = 0, len(arr) - 1  # 2 variables: O(1) space
    while left < right:
        arr[left], arr[right] = arr[right], arr[left]
        left += 1
        right -= 1

# Example 2: Finding maximum
def find_max(arr):
    max_val = arr[0]  # 1 variable: O(1) space
    for num in arr:
        if num > max_val:
            max_val = num
    return max_val
```

**Recognition Patterns**:
- Fixed number of variables
- In-place modifications
- Algorithms that only use pointers/indices

---

### O(log n) - Logarithmic Space

**Definition**: Space usage grows logarithmically with input size, typically due to recursion depth.

**Characteristics**:
- Usually from recursive call stack
- Common in divide-and-conquer algorithms
- Binary tree operations

**Analysis Techniques**:
```python
# Example: Binary Search (recursive)
def binary_search_recursive(arr, target, left=0, right=None):
    if right is None:
        right = len(arr) - 1
    
    if left > right:
        return -1
    
    mid = (left + right) // 2
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search_recursive(arr, target, mid + 1, right)
    else:
        return binary_search_recursive(arr, target, left, mid - 1)

# Space Analysis:
# Maximum recursion depth = log n
# Each recursive call uses O(1) space
# Total space: O(log n) for call stack
```

**Recognition Patterns**:
- Recursive algorithms with log n recursion depth
- Binary tree traversals
- Divide-and-conquer algorithms

---

### O(n) - Linear Space

**Definition**: Space usage grows linearly with input size.

**Characteristics**:
- Creating arrays/lists proportional to input
- Single-level data structures
- Most common space complexity

**Analysis Techniques**:
```python
# Example 1: Creating a copy
def duplicate_array(arr):
    new_arr = []  # This will grow to size n
    for element in arr:
        new_arr.append(element)
    return new_arr

# Example 2: Merge Sort
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])     # O(n/2) space for copy
    right = merge_sort(arr[mid:])    # O(n/2) space for copy
    
    return merge(left, right)        # O(n) space for merged result

# Space Analysis: O(n) for temporary arrays at each level
```

**Recognition Patterns**:
- Creating new data structures of size n
- Storing results that grow with input
- Hash tables with n entries

---

### O(n²) - Quadratic Space

**Definition**: Space usage grows quadratically with input size.

**Characteristics**:
- 2D arrays/matrices
- Storing all pairs of elements
- Dynamic programming tables

**Analysis Techniques**:
```python
# Example 1: 2D Matrix
def create_multiplication_table(n):
    table = []
    for i in range(n):           # n rows
        row = []
        for j in range(n):       # n columns per row
            row.append(i * j)
        table.append(row)
    return table                 # Total space: n × n = O(n²)

# Example 2: Dynamic Programming (Longest Common Subsequence)
def lcs_length(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]  # (m+1) × (n+1) matrix
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    
    return dp[m][n]
```

**Recognition Patterns**:
- 2D arrays or matrices
- Storing relationships between all pairs
- DP tables with two dimensions

---

## Advanced Analysis Techniques

### 1. Amortized Analysis
Used when operations have varying costs, but the average cost over a sequence is important.

### 2. Recurrence Relations
For recursive algorithms:
- Master Theorem
- Recursion Trees
- Substitution Method

### 3. Best, Average, and Worst Case
- **Best Case**: Minimum resources needed
- **Average Case**: Expected resources for typical input
- **Worst Case**: Maximum resources needed (most commonly analyzed)

### 4. Space-Time Tradeoffs
Often you can trade space for time or vice versa:
- Memoization: Use O(n) space to reduce time complexity
- In-place algorithms: Use O(1) space but might increase time complexity

---

## Practical Analysis Steps

### For Time Complexity:
1. **Identify the input size** (n, m, etc.)
2. **Count primitive operations** in terms of input size
3. **Focus on the highest-order term**
4. **Consider loops and recursion:**
   - Single loop over n elements: O(n)
   - Nested loops: multiply complexities
   - Recursive calls: solve recurrence relation

### For Space Complexity:
1. **Count extra space used** (excluding input)
2. **Consider:**
   - Local variables: usually O(1)
   - Data structures created: size relative to input
   - Recursion depth: call stack space
3. **Don't count:** input space (unless explicitly asked)

### Common Pitfalls:
- Confusing time and space complexity
- Forgetting about recursion stack space
- Not considering the dominant term
- Analyzing best case instead of worst case

This framework provides a systematic approach to analyzing any algorithm's complexity, helping you make informed decisions about algorithm selection and optimization.

--- 
