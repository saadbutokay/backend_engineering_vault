**Phase:** Level 1 — Java Fundamentals
**Date Studied:**

---
## What Problem Does This Solve?
You know how to store ONE value in a variable:
```java
  int score = 95;
  String name = "Rahim";
```

```text
But real applications deal with COLLECTIONS of values:
  → 1000 student scores in a database query result
  → All product names in a catalog
  → Every character in a user's password to validate
  → All tags on a blog post
  → All items in a shopping cart
```

Without arrays or strings:
```java
  int score1 = 85, score2 = 92, score3 = 78; // ... score1000 = ???
  // Impossible to scale. Impossible to loop over. Useless.
```

With arrays:
```java
  int[] scores = new int[1000];
  // Now you can store, access, loop, sort, search — any size.
```

```  
Strings are even more fundamental:
  → Every username, email, password is a String
  → Every API request body contains Strings
  → Every database column of text is a String
  → Every HTTP header is a String
  → Every log message is a String
  
Arrays and Strings are the most frequently used
data structures in ALL of Java programming.
Every backend system you build will use both — constantly.
```

---
## PART 1 - ARRAYS
An array is a FIXED-SIZE, ORDERED collection of elements
of the SAME TYPE, stored in CONTIGUOUS memory locations.
```text
FIXED-SIZE:    Once created, its size CANNOT change.
               Want 11 elements? Create a new array.
               This is the main limitation of arrays.

ORDERED:       Elements have positions (indices).
               Index starts at 0. Last index = length - 1.
               Order is preserved.

SAME TYPE:     int[] only holds ints.
               String[] only holds Strings.
               Cannot mix types in one array.

CONTIGUOUS:    Elements are stored next to each other in memory.
               This makes random access O(1) — instant by index.
               This makes arrays very fast for access.

VISUAL:
  int[] scores = {85, 92, 78, 95, 88};

  Index:   [0]  [1]  [2]  [3]  [4]
           ┌────┬────┬────┬────┬────┐
  Values:  │ 85 │ 92 │ 78 │ 95 │ 88 │
           └────┴────┴────┴────┴────┘
  Memory:  addr addr addr addr addr
           +0   +4   +8   +12  +16  (int = 4 bytes each)

  scores[0] = 85   ← first element
  scores[4] = 88   ← last element (length-1)
  scores[5] = ???  ← ArrayIndexOutOfBoundsException!
```

### 1.1 Declaring, Creating, Initializing Arrays
```java
public class ArrayCreation {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // METHOD 1: Declare then create (specify size)
        // ─────────────────────────────────────────
        
        // Step 1: Declare (tells Java the TYPE and NAME)
        int[] scores;
        // 'scores' is a reference variable — currently null
        // No array exists yet
        
        // Step 2: Create (allocate memory)
        scores = new int[5];
        // Creates array of 5 ints on the HEAP
        // All elements initialized to DEFAULT values:
        //   int/long/short/byte → 0
        //   double/float → 0.0
        //   boolean → false
        //   char → '\u0000'
        //   Object/String → null
        
        // Declare and create in one line (most common):
        int[] ages = new int[10];
        double[] prices = new double[100];
        boolean[] flags = new boolean[5];
        String[] names = new String[3]; // elements are null initially
        
        System.out.println("ages[0] = " + ages[0]);     // 0 (default)
        System.out.println("prices[0] = " + prices[0]); // 0.0 (default)
        System.out.println("flags[0] = " + flags[0]);   // false (default)
        System.out.println("names[0] = " + names[0]);   // null (default)
        
        // ─────────────────────────────────────────
        // METHOD 2: Array literal (declare + create + initialize)
        // ─────────────────────────────────────────
        
        // Values are specified directly in {}
        int[] topScores = {95, 88, 92, 79, 85};
        // Size is inferred from the number of values (5 here)
        
        String[] days = {"Monday", "Tuesday", "Wednesday",
                         "Thursday", "Friday", "Saturday", "Sunday"};
        
        double[] rates = {0.05, 0.10, 0.15, 0.20};
        
        char[] vowels = {'a', 'e', 'i', 'o', 'u'};
        
        // ─────────────────────────────────────────
        // METHOD 3: new keyword with values (anonymous array)
        // ─────────────────────────────────────────
        
        // Useful when passing arrays directly to methods:
        printArray(new int[]{1, 2, 3, 4, 5}); // no variable needed
        
        // Or when reassigning to an existing variable:
        topScores = new int[]{100, 95, 90, 85, 80}; // replace the array
        
        // ─────────────────────────────────────────
        // ARRAY LENGTH
        // ─────────────────────────────────────────
        
        // .length property (NOT a method — no parentheses!)
        System.out.println("days.length = " + days.length);       // 7
        System.out.println("topScores.length = " + topScores.length); // 5
        
        // length is FINAL — cannot change
        // days.length = 10; // COMPILE ERROR: cannot assign to final field
        
        // ─────────────────────────────────────────
        // ACCESSING AND MODIFYING ELEMENTS
        // ─────────────────────────────────────────
        
        int[] numbers = new int[5];
        
        // Assign values by index:
        numbers[0] = 10;
        numbers[1] = 20;
        numbers[2] = 30;
        numbers[3] = 40;
        numbers[4] = 50;
        
        // Read values by index:
        System.out.println(numbers[2]); // 30
        System.out.println(numbers[4]); // 50
        
        // Modify:
        numbers[2] = 999;
        System.out.println(numbers[2]); // 999
        
        // Last element — use length-1:
        System.out.println(numbers[numbers.length - 1]); // 50
        
        // ArrayIndexOutOfBoundsException:
        try {
            System.out.println(numbers[5]); // index 5 doesn't exist (0-4)
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Index out of bounds: " + e.getMessage());
        }
        
        // ─────────────────────────────────────────
        // DYNAMIC SIZE (size from variable)
        // ─────────────────────────────────────────
        
        int userCount = 50; // could come from database
        String[] usernames = new String[userCount];
        System.out.println("Created array for " + usernames.length + " users");
    }
    
    static void printArray(int[] arr) {
        System.out.print("[");
        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i]);
            if (i < arr.length - 1) System.out.print(", ");
        }
        System.out.println("]");
    }
}
```

### 1.2 Iterating Arrays
```java
import java.util.Arrays;

public class ArrayIteration {
    public static void main(String[] args) {
        
        int[] numbers = {10, 25, 8, 42, 17, 33, 6, 51, 29, 14};
        
        // ─────────────────────────────────────────
        // METHOD 1: for loop (use when you need index)
        // ─────────────────────────────────────────
        
        System.out.println("=== for loop (with index) ===");
        for (int i = 0; i < numbers.length; i++) {
            System.out.printf("[%d] = %d%n", i, numbers[i]);
        }
        
        // ─────────────────────────────────────────
        // METHOD 2: for-each (use when you DON'T need index)
        // ─────────────────────────────────────────
        
        System.out.println("\n=== for-each (no index needed) ===");
        int sum = 0;
        for (int num : numbers) {
            sum += num;
            System.out.print(num + " ");
        }
        System.out.println("\nSum: " + sum);
        
        // ─────────────────────────────────────────
        // METHOD 3: while loop (for conditional iteration)
        // ─────────────────────────────────────────
        
        // Find first number > 30:
        int idx = 0;
        while (idx < numbers.length && numbers[idx] <= 30) {
            idx++;
        }
        if (idx < numbers.length) {
            System.out.println("\nFirst > 30: numbers[" + idx + "] = " + numbers[idx]);
        }
        
        // ─────────────────────────────────────────
        // COMMON ARRAY OPERATIONS
        // ─────────────────────────────────────────
        
        // Find max and min:
        int max = numbers[0];
        int min = numbers[0];
        for (int num : numbers) {
            if (num > max) max = num;
            if (num < min) min = num;
        }
        System.out.println("\nMax: " + max + ", Min: " + min); // 51, 6
        
        // Find average:
        double average = (double) sum / numbers.length;
        System.out.printf("Average: %.2f%n", average);
        
        // Count elements matching condition:
        int countAboveAvg = 0;
        for (int num : numbers) {
            if (num > average) countAboveAvg++;
        }
        System.out.println("Above average: " + countAboveAvg);
        
        // Linear search (find index of value):
        int target = 42;
        int foundIdx = -1;
        for (int i = 0; i < numbers.length; i++) {
            if (numbers[i] == target) {
                foundIdx = i;
                break;
            }
        }
        System.out.println("Found " + target + " at index: " + foundIdx); // 3
        
        // Check if array contains a value:
        boolean contains = false;
        for (int num : numbers) {
            if (num == 99) { contains = true; break; }
        }
        System.out.println("Contains 99: " + contains); // false
        
        // ─────────────────────────────────────────
        // REVERSE ITERATION
        // ─────────────────────────────────────────
        
        System.out.print("\nReverse: ");
        for (int i = numbers.length - 1; i >= 0; i--) {
            System.out.print(numbers[i] + " ");
        }
        System.out.println();
        
        // ─────────────────────────────────────────
        // ARRAYS UTILITY CLASS
        // ─────────────────────────────────────────
        
        // Arrays.toString() — print array nicely:
        System.out.println("\nArrays.toString: " + Arrays.toString(numbers));
        // [10, 25, 8, 42, 17, 33, 6, 51, 29, 14]
        
        // Arrays.sort() — sort in place:
        int[] sorted = Arrays.copyOf(numbers, numbers.length); // copy first
        Arrays.sort(sorted);
        System.out.println("Sorted: " + Arrays.toString(sorted));
        // [6, 8, 10, 14, 17, 25, 29, 33, 42, 51]
        
        // Arrays.binarySearch() — O(log n) search (array MUST be sorted):
        int searchResult = Arrays.binarySearch(sorted, 29);
        System.out.println("Binary search for 29: index " + searchResult); // 6
        
        // Arrays.copyOf() — copy with new size:
        int[] shorter = Arrays.copyOf(numbers, 5); // first 5 elements
        int[] longer = Arrays.copyOf(numbers, 15); // extra slots = 0
        System.out.println("Shorter: " + Arrays.toString(shorter));
        System.out.println("Longer: " + Arrays.toString(longer));
        
        // Arrays.copyOfRange() — copy a portion:
        int[] middle = Arrays.copyOfRange(numbers, 3, 7); // index 3 to 6
        System.out.println("Middle: " + Arrays.toString(middle));
        // [42, 17, 33, 6]
        
        // Arrays.fill() — fill with a value:
        int[] filled = new int[5];
        Arrays.fill(filled, 7);
        System.out.println("Filled: " + Arrays.toString(filled)); // [7,7,7,7,7]
        
        // Arrays.equals() — compare contents:
        int[] a = {1, 2, 3};
        int[] b = {1, 2, 3};
        int[] c = {1, 2, 4};
        System.out.println("a equals b: " + Arrays.equals(a, b)); // true
        System.out.println("a equals c: " + Arrays.equals(a, c)); // false
        // NOTE: a == b would compare references (almost always false!)
    }
}
```

### 1.3 Common Array Algorithms
```java
import java.util.Arrays;

public class ArrayAlgorithms {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // REVERSE AN ARRAY IN PLACE
        // ─────────────────────────────────────────
        
        int[] arr = {1, 2, 3, 4, 5};
        System.out.println("Before: " + Arrays.toString(arr));
        
        // Two-pointer approach:
        int left = 0, right = arr.length - 1;
        while (left < right) {
            // Swap arr[left] and arr[right]
            int temp = arr[left];
            arr[left] = arr[right];
            arr[right] = temp;
            left++;
            right--;
        }
        System.out.println("Reversed: " + Arrays.toString(arr));
        // [5, 4, 3, 2, 1]
        
        // ─────────────────────────────────────────
        // REMOVE DUPLICATES (without Collections)
        // ─────────────────────────────────────────
        
        int[] withDups = {1, 3, 3, 5, 5, 5, 7, 9, 9};
        // Array must be SORTED first
        // Count unique elements:
        int uniqueCount = 1; // first element is always unique
        for (int i = 1; i < withDups.length; i++) {
            if (withDups[i] != withDups[i - 1]) {
                uniqueCount++;
            }
        }
        
        int[] unique = new int[uniqueCount];
        unique[0] = withDups[0];
        int pos = 1;
        for (int i = 1; i < withDups.length; i++) {
            if (withDups[i] != withDups[i - 1]) {
                unique[pos++] = withDups[i];
            }
        }
        System.out.println("Unique: " + Arrays.toString(unique));
        // [1, 3, 5, 7, 9]
        
        // ─────────────────────────────────────────
        // ROTATE ARRAY (by k positions)
        // ─────────────────────────────────────────
        
        int[] original = {1, 2, 3, 4, 5};
        int k = 2; // rotate right by 2
        
        // Method: reverse three times
        int n = original.length;
        k = k % n; // handle k > n
        
        reverse(original, 0, n - 1);   // reverse all
        reverse(original, 0, k - 1);   // reverse first k
        reverse(original, k, n - 1);   // reverse remaining
        
        System.out.println("Rotated right by " + k + ": " + Arrays.toString(original));
        // [4, 5, 1, 2, 3]
        
        // ─────────────────────────────────────────
        // MERGE TWO SORTED ARRAYS
        // ─────────────────────────────────────────
        
        int[] sorted1 = {1, 3, 5, 7, 9};
        int[] sorted2 = {2, 4, 6, 8, 10};
        int[] merged = new int[sorted1.length + sorted2.length];
        
        int i = 0, j = 0, m = 0;
        while (i < sorted1.length && j < sorted2.length) {
            if (sorted1[i] <= sorted2[j]) {
                merged[m++] = sorted1[i++];
            } else {
                merged[m++] = sorted2[j++];
            }
        }
        // Copy remaining elements:
        while (i < sorted1.length) merged[m++] = sorted1[i++];
        while (j < sorted2.length) merged[m++] = sorted2[j++];
        
        System.out.println("Merged: " + Arrays.toString(merged));
        // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
        
        // ─────────────────────────────────────────
        // FIND SECOND LARGEST
        // ─────────────────────────────────────────
        
        int[] nums = {5, 1, 9, 3, 7, 2, 8};
        int first = Integer.MIN_VALUE;
        int second = Integer.MIN_VALUE;
        
        for (int num : nums) {
            if (num > first) {
                second = first;
                first = num;
            } else if (num > second && num != first) {
                second = num;
            }
        }
        System.out.println("Largest: " + first);  // 9
        System.out.println("Second: " + second);  // 8
        
        // ─────────────────────────────────────────
        // FREQUENCY COUNT
        // ─────────────────────────────────────────
        
        int[] grades = {85, 92, 85, 78, 92, 95, 85, 78, 92, 100};
        // Use a map-like approach (manual, without HashMap for now):
        // Since grades 0-100, use index as key:
        int[] frequency = new int[101]; // indices 0-100
        for (int grade : grades) {
            frequency[grade]++;
        }
        
        System.out.println("\nGrade frequencies:");
        for (int g = 0; g <= 100; g++) {
            if (frequency[g] > 0) {
                System.out.printf("Grade %3d: %d time(s)%n", g, frequency[g]);
            }
        }
    }
    
    // Helper: reverse portion of array
    static void reverse(int[] arr, int start, int end) {
        while (start < end) {
            int temp = arr[start];
            arr[start] = arr[end];
            arr[end] = temp;
            start++;
            end--;
        }
    }
}
```

### 1.4 2D Arrays (Multi-dimensional)
```java
import java.util.Arrays;

public class TwoDArrays {
    public static void main(String[] args) {
        
        // 2D array = array of arrays
        // Like a table/grid/matrix
        // Declared as: type[][] name
        
        // ─────────────────────────────────────────
        // CREATING 2D ARRAYS
        // ─────────────────────────────────────────
        
        // Method 1: specify dimensions
        int[][] matrix = new int[3][4]; // 3 rows, 4 columns
        // All elements initialized to 0
        
        // Method 2: array literal
        int[][] grid = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };
        
        // Method 3: jagged array (rows of different lengths)
        int[][] jagged = new int[3][];
        jagged[0] = new int[]{1};
        jagged[1] = new int[]{2, 3};
        jagged[2] = new int[]{4, 5, 6};
        
        // ─────────────────────────────────────────
        // ACCESSING 2D ARRAY
        // ─────────────────────────────────────────
        
        // grid[row][column]
        System.out.println("grid[0][0] = " + grid[0][0]); // 1
        System.out.println("grid[1][2] = " + grid[1][2]); // 6
        System.out.println("grid[2][1] = " + grid[2][1]); // 8
        
        // Dimensions:
        System.out.println("Rows: " + grid.length);       // 3
        System.out.println("Cols: " + grid[0].length);    // 3
        
        // ─────────────────────────────────────────
        // ITERATING 2D ARRAY
        // ─────────────────────────────────────────
        
        System.out.println("\nGrid:");
        for (int row = 0; row < grid.length; row++) {
            for (int col = 0; col < grid[row].length; col++) {
                System.out.printf("%4d", grid[row][col]);
            }
            System.out.println();
        }
        //    1   2   3
        //    4   5   6
        //    7   8   9
        
        // for-each with 2D:
        System.out.println("\nfor-each 2D:");
        for (int[] row : grid) {        // each row is int[]
            for (int val : row) {       // each value in row
                System.out.printf("%4d", val);
            }
            System.out.println();
        }
        
        // Arrays.deepToString() for 2D:
        System.out.println(Arrays.deepToString(grid));
        // [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
        
        // ─────────────────────────────────────────
        // REAL-WORLD: Grade Sheet
        // ─────────────────────────────────────────
        
        // rows = students, cols = assignment scores
        String[] students = {"Rahim", "Karim", "Hasan", "Nabil"};
        int[][] studentScores = {
            {85, 92, 78, 95},  // Rahim's scores
            {72, 68, 80, 75},  // Karim's scores
            {90, 95, 88, 92},  // Hasan's scores
            {65, 70, 60, 55}   // Nabil's scores
        };
        String[] assignments = {"HW1", "HW2", "Mid", "Final"};
        
        System.out.println("\n=== GRADE SHEET ===");
        System.out.printf("%-10s", "Student");
        for (String asgn : assignments) {
            System.out.printf("%7s", asgn);
        }
        System.out.printf("%8s%n", "Average");
        System.out.println("─".repeat(50));
        
        for (int s = 0; s < students.length; s++) {
            System.out.printf("%-10s", students[s]);
            int total = 0;
            for (int a = 0; a < assignments.length; a++) {
                System.out.printf("%7d", studentScores[s][a]);
                total += studentScores[s][a];
            }
            double avg = (double) total / assignments.length;
            System.out.printf("%8.1f%n", avg);
        }
    }
}
```

---
## PART 2 - STRINGS
String in Java is NOT a primitive. String is a CLASS (java.lang.String). Strings are OBJECTS.

BUT: String is special - Java gives it primitive-like syntax:
```java
  String name = "Rahim"; // looks like a primitive assignment
  // but it's actually an object creation!
```

KEY PROPERTY: Strings are IMMUTABLE. Once created, the content CANNOT be changed.
```java
String s = "Hello";
s.toUpperCase(); // creates a NEW String "HELLO"
                   // s still refers to "Hello"!
```
To "change" a String, you always get a NEW String. The original is untouched.

```  
WHY IMMUTABLE?
  → Thread-safety (multiple threads can read same String safely)
  → Security (passwords, file paths can't be changed mid-operation)
  → Enables String Pool optimization (safe to reuse the same object)
  → Enables caching hashCode (used in HashMap keys)

STRING POOL:
  Java keeps a pool of String literals to save memory.
  
  String s1 = "Hello"; // stored in pool
  String s2 = "Hello"; // reuses SAME object from pool!
  String s3 = new String("Hello"); // FORCED new object on heap
  
  s1 == s2  → true  (same pool object)
  s1 == s3  → false (s3 is a different object)
  s1.equals(s3) → true (same content)
  
  LESSON: Always use .equals() to compare String content.
          Never use == for Strings.
```
### 2.1 Creating and Comparing Strings
```java
public class StringCreation {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // CREATING STRINGS
        // ─────────────────────────────────────────
        
        // Most common: string literal (uses pool)
        String name = "Rahim Ahmed";
        
        // Explicit new (avoids pool — rarely needed)
        String name2 = new String("Rahim Ahmed");
        
        // From char array:
        char[] chars = {'J', 'a', 'v', 'a'};
        String fromChars = new String(chars); // "Java"
        
        // Empty string:
        String empty = "";
        String emptyNew = new String(); // same as ""
        
        // null is NOT a string (no object):
        String nullStr = null;
        
        // Concatenation creates new string:
        String first = "Rahim";
        String last = "Ahmed";
        String full = first + " " + last; // "Rahim Ahmed" (new object)
        
        // From other types:
        String fromInt = String.valueOf(42);       // "42"
        String fromDouble = String.valueOf(3.14);  // "3.14"
        String fromBool = String.valueOf(true);    // "true"
        String fromChar = String.valueOf('A');     // "A"
        
        System.out.println("fromChars: " + fromChars);
        System.out.println("full: " + full);
        System.out.println("fromInt: " + fromInt);
        
        // ─────────────────────────────────────────
        // COMPARING STRINGS — THE GOLDEN RULES
        // ─────────────────────────────────────────
        
        String s1 = "hello";
        String s2 = "hello";
        String s3 = new String("hello");
        String s4 = "HELLO";
        
        // == compares REFERENCES (memory addresses):
        System.out.println("\n== comparison:");
        System.out.println(s1 == s2);  // true  (both from pool → same object)
        System.out.println(s1 == s3);  // false (s3 is new object)
        
        // .equals() compares CONTENT — ALWAYS use this:
        System.out.println("\n.equals() comparison:");
        System.out.println(s1.equals(s2));  // true ✓
        System.out.println(s1.equals(s3));  // true ✓
        System.out.println(s1.equals(s4));  // false (case-sensitive)
        
        // .equalsIgnoreCase() — case insensitive:
        System.out.println(s1.equalsIgnoreCase(s4)); // true
        
        // Safe null comparison (literal first):
        String userInput = null;
        System.out.println("hello".equals(userInput));           // false (no NPE)
        // System.out.println(userInput.equals("hello"));        // NPE!
        
        // .compareTo() — lexicographic ordering (for sorting):
        // returns negative if s1 < s2
        // returns 0 if s1 == s2
        // returns positive if s1 > s2
        System.out.println("\ncompareTo:");
        System.out.println("apple".compareTo("banana")); // negative (a < b)
        System.out.println("banana".compareTo("apple")); // positive
        System.out.println("apple".compareTo("apple"));  // 0
        
        // compareToIgnoreCase():
        System.out.println("APPLE".compareToIgnoreCase("apple")); // 0
        
        // .contentEquals() — compare with StringBuilder:
        StringBuilder sb = new StringBuilder("hello");
        System.out.println(s1.contentEquals(sb)); // true
        
        // String.isEmpty() vs String.isBlank():
        String empty2 = "";
        String whitespace = "   ";
        String notEmpty = "hello";
        
        System.out.println("\nempty vs blank:");
        System.out.println(empty2.isEmpty());      // true (length == 0)
        System.out.println(whitespace.isEmpty());  // false (has spaces)
        System.out.println(empty2.isBlank());      // true (empty or only whitespace)
        System.out.println(whitespace.isBlank());  // true ← KEY DIFFERENCE
        System.out.println(notEmpty.isBlank());    // false
    }
}
```

### 2.2 Essential String Methods
```java
public class StringMethods {
    public static void main(String[] args) {
        
        String str = "  Hello, World!  ";
        
        // ─────────────────────────────────────────
        // LENGTH AND ACCESS
        // ─────────────────────────────────────────
        
        System.out.println(str.length());          // 17
        System.out.println(str.charAt(7));         // W (0-indexed, after 2 spaces + "Hello, ")
        System.out.println(str.charAt(0));         // ' ' (space)
        
        // ─────────────────────────────────────────
        // SEARCHING
        // ─────────────────────────────────────────
        
        String clean = "Hello, World!";
        
        System.out.println(clean.indexOf('o'));         // 4  (first 'o')
        System.out.println(clean.indexOf('o', 5));     // 8  (first 'o' after index 5)
        System.out.println(clean.lastIndexOf('o'));     // 8  (last 'o')
        System.out.println(clean.indexOf("World"));    // 7  (substring)
        System.out.println(clean.indexOf("xyz"));      // -1 (not found)
        
        System.out.println(clean.contains("World"));   // true
        System.out.println(clean.contains("xyz"));     // false
        System.out.println(clean.startsWith("Hello")); // true
        System.out.println(clean.endsWith("!"));       // true
        System.out.println(clean.startsWith("World", 7)); // true (at index 7)
        
        // ─────────────────────────────────────────
        // EXTRACTING / SUBSTRING
        // ─────────────────────────────────────────
        
        // substring(start) — from start to end
        System.out.println(clean.substring(7));     // "World!"
        
        // substring(start, end) — start (inclusive) to end (EXCLUSIVE)
        System.out.println(clean.substring(7, 12)); // "World" (indices 7,8,9,10,11)
        System.out.println(clean.substring(0, 5));  // "Hello"
        
        // charAt for single char:
        System.out.println(clean.charAt(0));  // 'H'
        
        // ─────────────────────────────────────────
        // CASE CONVERSION
        // ─────────────────────────────────────────
        
        String mixed = "Hello World";
        System.out.println(mixed.toUpperCase()); // "HELLO WORLD"
        System.out.println(mixed.toLowerCase()); // "hello world"
        
        // Original is UNCHANGED (strings are immutable):
        System.out.println(mixed); // "Hello World" — unchanged
        
        // ─────────────────────────────────────────
        // TRIMMING WHITESPACE
        // ─────────────────────────────────────────
        
        String padded = "  Hello  ";
        System.out.println(padded.trim());        // "Hello" (removes leading/trailing)
        System.out.println(padded.strip());       // "Hello" (Unicode-aware, Java 11+)
        System.out.println(padded.stripLeading()); // "Hello  " (only leading)
        System.out.println(padded.stripTrailing()); // "  Hello" (only trailing)
        
        // ─────────────────────────────────────────
        // REPLACING
        // ─────────────────────────────────────────
        
        String original = "I love cats. Cats are great. CATS!";
        
        // Replace single char:
        System.out.println(original.replace('a', '@')); // "I love c@ts. C@ts @re gre@t. CATS!"
        
        // Replace substring (case-sensitive):
        System.out.println(original.replace("cats", "dogs"));
        // "I love dogs. Cats are great. CATS!" (only lowercase "cats" replaced)
        
        // Replace all matching regex:
        System.out.println(original.replaceAll("[Cc][Aa][Tt][Ss]", "dogs"));
        // "I love dogs. dogs are great. dogs!" (all variations)
        
        // Replace first match only:
        System.out.println(original.replaceFirst("[Cc]ats", "birds"));
        // "I love birds. Cats are great. CATS!"
        
        // ─────────────────────────────────────────
        // SPLITTING
        // ─────────────────────────────────────────
        
        String csv = "Rahim,Karim,Hasan,Nabil,Rafiq";
        String[] parts = csv.split(",");
        System.out.println("Parts: " + parts.length); // 5
        for (String part : parts) {
            System.out.println("  " + part);
        }
        
        // Split with limit:
        String[] limited = csv.split(",", 3); // max 3 parts
        System.out.println(java.util.Arrays.toString(limited));
        // [Rahim, Karim, Hasan,Nabil,Rafiq]
        
        // Split by whitespace:
        String sentence = "the quick brown fox";
        String[] words = sentence.split("\\s+"); // one or more whitespace
        System.out.println("Words: " + words.length); // 4
        
        // Split by multiple delimiters:
        String mixed2 = "one,two;three|four";
        String[] all = mixed2.split("[,;|]");
        System.out.println(java.util.Arrays.toString(all));
        // [one, two, three, four]
        
        // ─────────────────────────────────────────
        // JOINING
        // ─────────────────────────────────────────
        
        // String.join() — join with delimiter:
        String joined = String.join(", ", "Java", "Spring", "PostgreSQL");
        System.out.println(joined); // Java, Spring, PostgreSQL
        
        // Join array:
        String[] techs = {"Docker", "Kubernetes", "AWS"};
        System.out.println(String.join(" | ", techs)); // Docker | Kubernetes | AWS
        
        // ─────────────────────────────────────────
        // CONVERSION
        // ─────────────────────────────────────────
        
        // String to char array:
        char[] chars = "Hello".toCharArray();
        System.out.println(chars[0]); // H
        
        // Char array to String:
        String backToStr = new String(chars);
        System.out.println(backToStr); // Hello
        
        // String to bytes:
        byte[] bytes = "Hello".getBytes();
        System.out.println(bytes.length); // 5
        
        // ─────────────────────────────────────────
        // FORMATTING
        // ─────────────────────────────────────────
        
        // String.format() — same as printf but returns String
        String formatted = String.format("Name: %-15s Age: %3d CGPA: %.2f",
                                          "Rahim Ahmed", 21, 3.15);
        System.out.println(formatted);
        // Name: Rahim Ahmed      Age:  21 CGPA: 3.15
        
        // formatted() shorthand (Java 15+):
        String f2 = "Hello, %s! You have %d messages.".formatted("Rahim", 5);
        System.out.println(f2);
        // Hello, Rahim! You have 5 messages.
        
        // ─────────────────────────────────────────
        // REPEAT (Java 11+)
        // ─────────────────────────────────────────
        
        System.out.println("─".repeat(30)); // ──────────────────────────────
        System.out.println("Ha".repeat(5)); // HaHaHaHaHa
        
        // ─────────────────────────────────────────
        // CHECKING CONTENT
        // ─────────────────────────────────────────
        
        String password = "SecurePass123!";
        System.out.println(password.matches(".*[A-Z].*"));   // true (has uppercase)
        System.out.println(password.matches(".*\\d.*"));     // true (has digit)
        System.out.println(password.matches("[a-z]+"));      // false (not all lowercase)
        
        // ─────────────────────────────────────────
        // USEFUL COMBINATIONS
        // ─────────────────────────────────────────
        
        // Normalize user input (very common in backend):
        String userInput = "  RAHIM@EXAMPLE.COM  ";
        String normalized = userInput.trim().toLowerCase();
        System.out.println(normalized); // "rahim@example.com"
        
        // Extract domain from email:
        String email = "rahim@example.com";
        int atIdx = email.indexOf('@');
        String domain = email.substring(atIdx + 1);
        System.out.println("Domain: " + domain); // "example.com"
        
        // Check if string is numeric:
        String numStr = "12345";
        boolean isNumeric = numStr.matches("\\d+");
        System.out.println("Is numeric: " + isNumeric); // true
        
        // Mask sensitive data (credit card, password):
        String cardNumber = "4532015112830366";
        String masked = "*".repeat(cardNumber.length() - 4)
                      + cardNumber.substring(cardNumber.length() - 4);
        System.out.println("Masked: " + masked); // ************0366
    }
}
```

### 2.4 StringBuilder - Mutable Strings
```java
public class StringBuilderGuide {
    public static void main(String[] args) {
        
        // WHY StringBuilder?
        // String is immutable: every operation creates a NEW object.
        // In a loop: 1000 iterations → 1000 intermediate String objects
        // This is SLOW and wastes memory.
        
        // StringBuilder is MUTABLE: modifies the SAME object.
        // Perfect for building strings piece by piece.
        
        // ─────────────────────────────────────────
        // CREATING StringBuilder
        // ─────────────────────────────────────────
        
        StringBuilder sb = new StringBuilder();          // empty
        StringBuilder sb2 = new StringBuilder("Hello"); // with initial value
        StringBuilder sb3 = new StringBuilder(100);     // with initial capacity
        
        // ─────────────────────────────────────────
        // CORE METHODS
        // ─────────────────────────────────────────
        
        StringBuilder builder = new StringBuilder();
        
        // append() — add to end:
        builder.append("Hello");
        builder.append(", ");
        builder.append("World");
        builder.append("!");
        System.out.println(builder); // Hello, World!
        
        // Method chaining (append returns 'this'):
        StringBuilder chained = new StringBuilder()
            .append("User: ")
            .append("Rahim")
            .append(", Age: ")
            .append(21)
            .append(", Active: ")
            .append(true);
        System.out.println(chained); // User: Rahim, Age: 21, Active: true
        
        // insert() — add at specific position:
        StringBuilder s = new StringBuilder("Hello World");
        s.insert(5, ",");   // insert "," at index 5
        System.out.println(s); // Hello, World
        
        // delete() — remove characters:
        s.delete(5, 6); // remove index 5 to 5 (exclusive 6)
        System.out.println(s); // Hello World (comma removed)
        
        // deleteCharAt() — remove one character:
        s.deleteCharAt(5); // remove space
        System.out.println(s); // HelloWorld
        
        // replace() — replace portion:
        s.replace(5, 10, " Java"); // replace "World" with " Java"
        System.out.println(s); // Hello Java
        
        // reverse():
        StringBuilder rev = new StringBuilder("Hello");
        rev.reverse();
        System.out.println(rev); // olleH
        
        // charAt() and setCharAt():
        StringBuilder mod = new StringBuilder("Hello");
        System.out.println(mod.charAt(0));   // H
        mod.setCharAt(0, 'J');
        System.out.println(mod);             // Jello
        
        // length() and capacity():
        StringBuilder cap = new StringBuilder();
        System.out.println("Length: " + cap.length());     // 0
        System.out.println("Capacity: " + cap.capacity()); // 16 (default)
        
        // Convert to String:
        String result = builder.toString();
        System.out.println("Final: " + result);
        
        // indexOf() — search:
        StringBuilder search = new StringBuilder("Hello World Hello");
        System.out.println(search.indexOf("Hello"));     // 0
        System.out.println(search.lastIndexOf("Hello")); // 12
        
        // ─────────────────────────────────────────
        // PERFORMANCE COMPARISON
        // ─────────────────────────────────────────
        
        int iterations = 50_000;
        
        // String += (SLOW):
        long start = System.currentTimeMillis();
        String slow = "";
        for (int i = 0; i < iterations; i++) {
            slow += i;
        }
        long slowTime = System.currentTimeMillis() - start;
        
        // StringBuilder (FAST):
        start = System.currentTimeMillis();
        StringBuilder fast = new StringBuilder();
        for (int i = 0; i < iterations; i++) {
            fast.append(i);
        }
        String fastResult = fast.toString();
        long fastTime = System.currentTimeMillis() - start;
        
        System.out.printf("%nPerformance (%,d iterations):%n", iterations);
        System.out.printf("String +=     : %d ms%n", slowTime);
        System.out.printf("StringBuilder : %d ms%n", fastTime);
        System.out.println("Same length: " + (slow.length() == fastResult.length()));
        
        // ─────────────────────────────────────────
        // REAL-WORLD USE CASES
        // ─────────────────────────────────────────
        
        // Building SQL query:
        String table = "users";
        boolean hasWhere = true;
        boolean hasOrder = true;
        
        StringBuilder sql = new StringBuilder("SELECT * FROM ")
            .append(table);
        
        if (hasWhere) {
            sql.append(" WHERE active = true");
        }
        if (hasOrder) {
            sql.append(" ORDER BY created_at DESC");
        }
        sql.append(" LIMIT 10");
        
        System.out.println("\nSQL: " + sql);
        
        // Building CSV line:
        String[] fields = {"Rahim", "rahim@test.com", "21", "Dhaka"};
        StringBuilder csv = new StringBuilder();
        for (int i = 0; i < fields.length; i++) {
            csv.append(fields[i]);
            if (i < fields.length - 1) csv.append(",");
        }
        System.out.println("CSV: " + csv);
        
        // Building HTML:
        StringBuilder html = new StringBuilder();
        html.append("<ul>\n");
        String[] items = {"Home", "Products", "About", "Contact"};
        for (String item : items) {
            html.append("  <li>").append(item).append("</li>\n");
        }
        html.append("</ul>");
        System.out.println("\nHTML:\n" + html);
        
        // ─────────────────────────────────────────
        // StringBuffer (thread-safe StringBuilder)
        // ─────────────────────────────────────────
        
        // StringBuffer = synchronized StringBuilder
        // Use when multiple THREADS share the same builder
        // Slower than StringBuilder due to synchronization
        // In practice: rarely needed — use StringBuilder
        // unless you specifically need thread safety
        
        StringBuffer threadSafe = new StringBuffer("thread safe");
        // Same API as StringBuilder
    }
}
```

### 2.5 String in Real Backend Engineering
```java
public class StringInBackend {
    public static void main(String[] args) {
        
        // These patterns appear CONSTANTLY in Spring Boot code
        
        // ─────────────────────────────────────────
        // 1. EMAIL VALIDATION
        // ─────────────────────────────────────────
        
        String[] emails = {
            "rahim@example.com",
            "invalid",
            "@nodomain.com",
            "noatsign.com",
            "rahim@",
            "valid.email+tag@domain.co.uk"
        };
        
        System.out.println("=== Email Validation ===");
        for (String email : emails) {
            System.out.printf("%-35s → %s%n",
                              email,
                              isValidEmail(email) ? "✅ Valid" : "❌ Invalid");
        }
        
        // ─────────────────────────────────────────
        // 2. URL / PATH PARSING
        // ─────────────────────────────────────────
        
        System.out.println("\n=== URL Parsing ===");
        String url = "https://api.example.com/v1/users/123/orders?page=2&size=10";
        
        // Extract protocol:
        String protocol = url.substring(0, url.indexOf("://"));
        System.out.println("Protocol: " + protocol);
        
        // Extract domain:
        int domainStart = url.indexOf("://") + 3;
        int domainEnd = url.indexOf("/", domainStart);
        String domain = url.substring(domainStart, domainEnd);
        System.out.println("Domain: " + domain);
        
        // Extract path (before ?):
        int pathEnd = url.indexOf("?");
        String path = url.substring(domainEnd, pathEnd);
        System.out.println("Path: " + path);
        
        // Extract query string:
        String queryString = url.substring(pathEnd + 1);
        System.out.println("Query: " + queryString);
        
        // Parse query params:
        System.out.println("Query params:");
        for (String param : queryString.split("&")) {
            String[] kv = param.split("=");
            System.out.printf("  %s = %s%n", kv[0], kv[1]);
        }
        
        // ─────────────────────────────────────────
        // 3. USER INPUT SANITIZATION
        // ─────────────────────────────────────────
        
        System.out.println("\n=== Input Sanitization ===");
        String[] inputs = {
            "  RAHIM  ",
            "  rahim@EXAMPLE.COM  ",
            "  +8801712345678  ",
            "  Bangladesh  "
        };
        
        for (String input : inputs) {
            String sanitized = sanitize(input);
            System.out.printf("'%s' → '%s'%n", input, sanitized);
        }
        
        // ─────────────────────────────────────────
        // 4. TEMPLATE FILLING
        // ─────────────────────────────────────────
        
        System.out.println("\n=== Email Template ===");
        String template = """
                Dear {name},
                
                Your order #{orderId} has been {status}.
                Total amount: {amount}
                
                Thank you for shopping with us!
                """;
        
        String filled = template
            .replace("{name}", "Rahim Ahmed")
            .replace("{orderId}", "ORD-2024-001")
            .replace("{status}", "confirmed")
            .replace("{amount}", "৳1,250.00");
        
        System.out.println(filled);
        
        // ─────────────────────────────────────────
        // 5. PASSWORD VALIDATION
        // ─────────────────────────────────────────
        
        System.out.println("=== Password Validation ===");
        String[] passwords = {
            "short",
            "nouppercase1!",
            "NoDigit!",
            "NoSpecial1",
            "Valid@Pass1",
            "SuperSecure@123"
        };
        
        for (String pwd : passwords) {
            System.out.printf("%-20s → %s%n", pwd, validatePassword(pwd));
        }
        
        // ─────────────────────────────────────────
        // 6. JSON-LIKE FORMATTING (manual, before Jackson)
        // ─────────────────────────────────────────
        
        System.out.println("\n=== Manual JSON Building ===");
        String name = "Rahim Ahmed";
        String email = "rahim@example.com";
        int age = 21;
        boolean active = true;
        
        String json = String.format("""
                {
                  "name": "%s",
                  "email": "%s",
                  "age": %d,
                  "active": %b
                }""",
                name, email, age, active);
        
        System.out.println(json);
        // In real code, use Jackson ObjectMapper — never manual JSON building
    }
    
    // Helper methods:
    
    static boolean isValidEmail(String email) {
        if (email == null || email.isBlank()) return false;
        int atIdx = email.indexOf('@');
        if (atIdx <= 0) return false;              // no @ or @ is first char
        if (atIdx == email.length() - 1) return false; // @ is last char
        String domain = email.substring(atIdx + 1);
        return domain.contains(".") && !domain.startsWith(".");
    }
    
    static String sanitize(String input) {
        if (input == null) return "";
        return input.trim().toLowerCase();
    }
    
    static String validatePassword(String pwd) {
        if (pwd == null || pwd.length() < 8)
            return "❌ Too short (min 8 chars)";
        if (!pwd.matches(".*[A-Z].*"))
            return "❌ Missing uppercase letter";
        if (!pwd.matches(".*[0-9].*"))
            return "❌ Missing digit";
        if (!pwd.matches(".*[!@#$%^&*].*"))
            return "❌ Missing special char (!@#$%^&*)";
        return "✅ Strong password";
    }
}
```

---
## Build This - Complete Array & String Practice
```java
// File: DataProcessor.java
// Realistic data processing system using arrays and strings

import java.util.Arrays;

public class DataProcessor {
    
    // ─────────────────────────────────────────
    // DATA
    // ─────────────────────────────────────────
    
    static String[] rawCustomerData = {
        "  RAHIM AHMED  , rahim@example.com , 25 , Dhaka   ",
        "  KARIM HASSAN , karim@test.org   , 32 , Chittagong",
        "  HASAN ALI    , hasan@test.com   , 28 , Sylhet   ",
        "  NABIL KHAN   , invalid-email    , -5 , Khulna   ",
        "  RAFIQ MIA    , rafiq@demo.net   , 45 , Rajshahi ",
        "  TARIQ ALAM   ,                 , 22 , Dhaka    ", // missing email
    };
    
    static int[] monthlyRevenue = {
        45000, 52000, 38000, 61000, 55000, 48000,
        70000, 82000, 67000, 75000, 90000, 105000
    };
    
    static String[] orderLog = {
        "ORD-001:DELIVERED:1500",
        "ORD-002:PROCESSING:3200",
        "ORD-003:CANCELLED:800",
        "ORD-004:DELIVERED:4500",
        "ORD-005:SHIPPED:2100",
        "ORD-006:DELIVERED:950",
        "ORD-007:PROCESSING:7800",
        "ORD-008:CANCELLED:1200",
        "ORD-009:DELIVERED:3300",
        "ORD-010:SHIPPED:5600"
    };
    
    static String[] months = {
        "Jan","Feb","Mar","Apr","May","Jun",
        "Jul","Aug","Sep","Oct","Nov","Dec"
    };
    
    // ─────────────────────────────────────────
    // MAIN
    // ─────────────────────────────────────────
    
    public static void main(String[] args) {
        processCustomers();
        System.out.println();
        analyzeRevenue();
        System.out.println();
        processOrders();
    }
    
    // ─────────────────────────────────────────
    // SECTION 1: CUSTOMER PROCESSING
    // ─────────────────────────────────────────
    
    static void processCustomers() {
        printHeader("CUSTOMER DATA PROCESSING");
        
        int valid = 0, invalid = 0;
        
        for (String raw : rawCustomerData) {
            String[] fields = raw.split(",");
            
            // Extract and sanitize each field
            String name  = fields[0].trim();
            String email = fields[1].trim();
            String ageStr = fields[2].trim();
            String city  = fields[3].trim();
            
            // Normalize
            name  = toTitleCase(name);
            email = email.toLowerCase();
            
            // Validate
            boolean emailOk = isValidEmail(email);
            boolean ageOk   = isValidAge(ageStr);
            
            if (emailOk && ageOk) {
                valid++;
                int age = Integer.parseInt(ageStr);
                System.out.printf("✅ %-20s │ %-25s │ %3d │ %s%n",
                                  name, email, age, city);
            } else {
                invalid++;
                StringBuilder reason = new StringBuilder("❌ ");
                reason.append(name).append(" → ");
                if (!emailOk) reason.append("[invalid email] ");
                if (!ageOk)   reason.append("[invalid age] ");
                System.out.println(reason.toString().trim());
            }
        }
        
        System.out.println("─".repeat(70));
        System.out.printf("Valid: %d  │  Invalid: %d  │  Total: %d%n",
                          valid, invalid, rawCustomerData.length);
    }
    
    // ─────────────────────────────────────────
    // SECTION 2: REVENUE ANALYSIS
    // ─────────────────────────────────────────
    
    static void analyzeRevenue() {
        printHeader("MONTHLY REVENUE ANALYSIS");
        
        // Statistics
        int total = 0;
        int maxRev = monthlyRevenue[0], maxMonth = 0;
        int minRev = monthlyRevenue[0], minMonth = 0;
        
        for (int i = 0; i < monthlyRevenue.length; i++) {
            total += monthlyRevenue[i];
            if (monthlyRevenue[i] > maxRev) { maxRev = monthlyRevenue[i]; maxMonth = i; }
            if (monthlyRevenue[i] < minRev) { minRev = monthlyRevenue[i]; minMonth = i; }
        }
        
        double avg = (double) total / monthlyRevenue.length;
        
        // Sort a copy for median
        int[] sorted = Arrays.copyOf(monthlyRevenue, monthlyRevenue.length);
        Arrays.sort(sorted);
        double median = sorted.length % 2 == 0
            ? (sorted[sorted.length/2 - 1] + sorted[sorted.length/2]) / 2.0
            : sorted[sorted.length / 2];
        
        // Bar chart
        System.out.printf("%-5s %-10s %s%n", "Month", "Revenue", "Chart");
        System.out.println("─".repeat(55));
        
        for (int i = 0; i < months.length; i++) {
            int bars = monthlyRevenue[i] / 5000;
            String bar = "█".repeat(bars);
            String trend = i > 0
                ? (monthlyRevenue[i] > monthlyRevenue[i-1] ? "↑" : "↓")
                : " ";
            System.out.printf("%-5s %,9d %s %s%n",
                              months[i], monthlyRevenue[i], trend, bar);
        }
        
        System.out.println("─".repeat(55));
        System.out.printf("Total  : %,d%n", total);
        System.out.printf("Average: %,.0f%n", avg);
        System.out.printf("Median : %,.0f%n", median);
        System.out.printf("Best   : %s (%,d)%n", months[maxMonth], maxRev);
        System.out.printf("Worst  : %s (%,d)%n", months[minMonth], minRev);
        
        // Growth rate (last month vs first month)
        double growth = ((double)(monthlyRevenue[11] - monthlyRevenue[0])
                        / monthlyRevenue[0]) * 100;
        System.out.printf("YoY Growth: %.1f%%%n", growth);
    }
    
    // ─────────────────────────────────────────
    // SECTION 3: ORDER LOG PROCESSING
    // ─────────────────────────────────────────
    
    static void processOrders() {
        printHeader("ORDER LOG ANALYSIS");
        
        int delivered = 0, processing = 0, shipped = 0, cancelled = 0;
        long totalDeliveredRevenue = 0;
        long totalCancelledRevenue = 0;
        String highestOrder = "";
        long highestAmount = 0;
        
        System.out.printf("%-10s %-12s %10s%n", "Order", "Status", "Amount");
        System.out.println("─".repeat(35));
        
        for (String log : orderLog) {
            String[] parts = log.split(":");
            String orderId = parts[0];
            String status  = parts[1];
            long amount    = Long.parseLong(parts[2]);
            
            // Count by status
            switch (status) {
                case "DELIVERED"  -> { delivered++;  totalDeliveredRevenue += amount; }
                case "PROCESSING" -> processing++;
                case "SHIPPED"    -> shipped++;
                case "CANCELLED"  -> { cancelled++;  totalCancelledRevenue += amount; }
            }
            
            // Track highest
            if (amount > highestAmount) {
                highestAmount = amount;
                highestOrder = orderId;
            }
            
            // Status icon
            String icon = switch (status) {
                case "DELIVERED"  -> "✅";
                case "PROCESSING" -> "⚙️ ";
                case "SHIPPED"    -> "🚚";
                case "CANCELLED"  -> "❌";
                default           -> "❓";
            };
            
            System.out.printf("%-10s %s %-10s %,9d%n",
                              orderId, icon, status, amount);
        }
        
        System.out.println("─".repeat(35));
        System.out.printf("%-10s %-12s %,9d%n", "DELIVERED", delivered + " orders",
                          totalDeliveredRevenue);
        System.out.printf("%-10s %-12s %,9d%n", "CANCELLED", cancelled + " orders",
                          totalCancelledRevenue);
        System.out.printf("Processing: %d  Shipped: %d%n", processing, shipped);
        System.out.printf("Highest: %s (৳%,d)%n", highestOrder, highestAmount);
        
        double successRate = (double) delivered / orderLog.length * 100;
        System.out.printf("Success Rate: %.1f%%%n", successRate);
    }
    
    // ─────────────────────────────────────────
    // HELPER METHODS
    // ─────────────────────────────────────────
    
    static void printHeader(String title) {
        System.out.println("╔" + "═".repeat(50) + "╗");
        System.out.printf("║  %-48s║%n", title);
        System.out.println("╚" + "═".repeat(50) + "╝");
    }
    
    static String toTitleCase(String s) {
        if (s == null || s.isBlank()) return "";
        String[] words = s.toLowerCase().split("\\s+");
        StringBuilder result = new StringBuilder();
        for (int i = 0; i < words.length; i++) {
            if (words[i].isEmpty()) continue;
            result.append(Character.toUpperCase(words[i].charAt(0)));
            result.append(words[i].substring(1));
            if (i < words.length - 1) result.append(" ");
        }
        return result.toString();
    }
    
    static boolean isValidEmail(String email) {
        if (email == null || email.isBlank()) return false;
        int at = email.indexOf('@');
        if (at <= 0 || at == email.length() - 1) return false;
        String domain = email.substring(at + 1);
        return domain.contains(".") && !domain.endsWith(".");
    }
    
    static boolean isValidAge(String ageStr) {
        try {
            int age = Integer.parseInt(ageStr.trim());
            return age >= 0 && age <= 120;
        } catch (NumberFormatException e) {
            return false;
        }
    }
}
```

---
## Exercises
```text
EXERCISE 1: Array Fundamentals
  Create ArrayFundamentals.java
  Given: int[] data = {64, 34, 25, 12, 22, 11, 90, 45, 67, 3, 88, 17}
  Without using Arrays utility class:
  - Find max, min, sum, average
  - Count even and odd numbers
  - Find all numbers divisible by 3
  - Reverse the array in place
  - Rotate left by 3 positions
  - Check if it contains 45 (linear search)
  - Sort using bubble sort (implement manually)
  - After sorting: binary search for 67
  Print each result.

EXERCISE 2: 2D Array Grid
  Create GridGame.java
  Create a 5x5 grid filled with random 0s and 1s.
  (0 = empty, 1 = filled)
  - Print the grid visually (. for 0, █ for 1)
  - Count total filled cells
  - Count filled cells in each row and column
  - Find the row with most filled cells
  - Find the column with most filled cells
  - Count 2x2 blocks that are all filled

EXERCISE 3: String Manipulation
  Create StringManipulator.java
  Given a paragraph of text:
  "Java is a powerful language. Spring Boot makes
   Java backend development easier. Every backend
   engineer should know Java and Spring Boot."
  - Count words total
  - Count unique words (case-insensitive)
  - Find the most frequent word
  - Replace all "Java" with "Java 21"
  - Extract all words containing "Spring"
  - Reverse each sentence (not the whole string)
  - Check if paragraph is a pangram
  - Count vowels and consonants

EXERCISE 4: String + Array Combined
  Create CSVProcessor.java
  Given CSV data (hardcoded as String[]):
  "id,name,email,city,score"
  "1,Rahim Ahmed,rahim@test.com,Dhaka,85"
  "2,Karim Hassan,karim@test.org,Chittagong,92"
  "3,Hasan Ali,hasan@test.com,Sylhet,78"
  "4,Nabil Khan,invalid-email,Khulna,95"
  "5,Rafiq Mia,rafiq@demo.net,Rajshahi,88"

  Parse the CSV:
  - Store data in parallel arrays (id[], name[], email[], etc.)
  - Skip the header row
  - Validate emails, mark invalid
  - Find highest and lowest scorer
  - Calculate average score
  - Sort by score (descending) — sort all parallel arrays together
  - Print formatted table
  - Print summary statistics

EXERCISE 5: StringBuilder Challenge
  Create TextBuilder.java
  Build these using ONLY StringBuilder:
  a) A formatted HTML table from arrays of data
  b) A markdown report with headers, bullets, code blocks
  c) A CSV export string from an array of objects
  d) A URL query string builder:
     buildQueryString(Map<String, String>) →
     "key1=value1&key2=value2&key3=value3"
  Measure and compare performance vs String concatenation.
  Push to GitHub: "feat: arrays and string processing"
```

---
## Common Mistakes
```text
MISTAKE 1: ArrayIndexOutOfBoundsException
  int[] arr = {1, 2, 3};
  System.out.println(arr[3]); // ERROR! Valid indices: 0, 1, 2
  Fix: always use i < arr.length (not <=) in loops

MISTAKE 2: Comparing arrays with ==
  int[] a = {1, 2, 3};
  int[] b = {1, 2, 3};
  a == b → false (compares references, not content)
  Fix: Arrays.equals(a, b) → true ✓

MISTAKE 3: Forgetting String is immutable
  String s = "hello";
  s.toUpperCase(); // does NOTHING to s!
  System.out.println(s); // still "hello"
  Fix: String upper = s.toUpperCase(); // capture the result

MISTAKE 4: Using == to compare Strings
  String a = new String("hello");
  String b = new String("hello");
  a == b → false
  Fix: a.equals(b) → true ✓
  Or: "hello".equals(userInput) → null-safe ✓

MISTAKE 5: NullPointerException on null String methods
  String s = null;
  s.length(); // NPE!
  Fix: null check first, or use "literal".equals(s)

MISTAKE 6: String += in loops
  String result = "";
  for (int i = 0; i < 10000; i++) result += i; // SLOW O(n²)
  Fix: StringBuilder.append() → O(n)

MISTAKE 7: Not capturing String method results
  String email = "  USER@EXAMPLE.COM  ";
  email.trim().toLowerCase(); // discarded!
  System.out.println(email); // unchanged!
  Fix: email = email.trim().toLowerCase(); // reassign

MISTAKE 8: Forgetting substring end is EXCLUSIVE
  "Hello".substring(1, 3) → "el" (indices 1 and 2, NOT 3)
  "Hello".substring(1, 4) → "ell" (indices 1, 2, 3)

MISTAKE 9: Split result surprise
  "a,b,,c".split(",") → ["a", "b", "", "c"] (4 elements)
  "".split(",") → [""] (ONE empty string, not zero)
  ",".split(",") → [] (zero elements — trailing empty removed!)

MISTAKE 10: Modifying array while iterating with for-each
  // Cannot remove from array (arrays are fixed size)
  // Cannot modify elements' references via for-each variable
  for (String s : arr) {
      s = s.toUpperCase(); // modifies local variable, NOT array!
  }
  Fix: use indexed for loop: arr[i] = arr[i].toUpperCase();
```

---
## Interview Questions

**Q: What are the limitations of arrays in Java?**
A: Arrays have three main limitations: 1) Fixed size — once created, the size cannot change. To add more elements, you must create a new larger array and copy elements. 2) Homogeneous — all elements must be the same type. 3) No built-in high-level operations — sorting, searching, and filtering require manual implementation or utility methods. These limitations led to the Collections Framework (ArrayList, HashMap etc.) which provide dynamic sizing and rich operations. Arrays are still used for primitives (int[], double[]) and performance-critical code where Collections overhead matters.

**Q: Why are Strings immutable in Java?**
A: String immutability provides several benefits: 1) Thread safety: multiple threads can read the same String concurrently without synchronization. 2) Security: passwords, file paths, network connections use Strings — immutability prevents accidental or malicious modification after validation. 3) String Pool: Java can safely reuse String objects in the pool because immutable objects can be shared. 4) Caching hashCode: String caches its hash (used in HashMap keys) because the content never changes.

**Q: What is the String Pool?**
A: The String Pool (also called String intern pool) is a special area in the heap where Java stores unique String literals. When you write String s = "hello", Java first checks if "hello" exists in the pool — if yes, reuses the same object; if no, creates it and adds to the pool. This saves memory when many variables hold the same string value. String s1 = "hello"; String s2 = "hello"; makes s1 == s2 true because they reference the same pool object. new String("hello") bypasses the pool and creates a new object. That's why you should always compare String content with .equals() not == — == compares references, not content.

**Q: When would you use StringBuilder over String?**
A: Use StringBuilder when building a String through multiple operations, especially in loops. String concatenation with + creates a new String object on each operation — in a loop of N iterations, this creates N intermediate objects and runs in O(n²) time. StringBuilder modifies the same underlying char array, running in O(n) time. Use String for simple, single-operation concatenation (the compiler optimizes these to StringBuilder anyway). Use StringBuilder explicitly for loops, conditional building, or when appending many pieces. StringBuffer is the thread-safe version of StringBuilder — use it only when multiple threads share the same builder.

**Q: What is the difference between String.equals() and String.equalsIgnoreCase()?**
A: equals() compares String content case-sensitively: "Hello".equals("hello") → false. equalsIgnoreCase() compares content ignoring case: "Hello".equalsIgnoreCase("HELLO") → true. In backend engineering, equalsIgnoreCase() is commonly used for comparing HTTP method strings, role names, status codes, and any user-provided input where case should not matter. Always use equals() or equalsIgnoreCase() for String comparison, never == (which compares references, not content).

**Q: How do you handle a null String safely?**
A: Three approaches: 1) Null check before use: if (str != null && str.length() > 0). 2) Put the literal first: "expected".equals(str) — this never throws NPE even if str is null, because you're calling equals() on the literal, not the variable. 3) Use String utility methods: Objects.isNull(str), str == null || str.isBlank() with proper guard clauses. In modern Java, use Optional`<String>` for return types that might be absent. In Spring Boot, use @NotNull and @NotBlank annotations for input validation.

---

## Key Takeaways
```text
ARRAYS:
1. Fixed size. Ordered. Same type. 0-indexed.
   Size = length property (not method, no parentheses).
   Last valid index = length - 1. Always.

2. THREE creation styles:
   new int[5]         → all zeros (default values)
   {1, 2, 3}          → literal with values
   new int[]{1,2,3}   → anonymous (for passing directly)

3. Use for loop when you need the INDEX.
   Use for-each when you need ALL ELEMENTS.
   Use while when you need CONDITIONAL iteration.

4. Arrays.toString() to print. Arrays.equals() to compare.
   Never use == to compare arrays (compares references).
   Arrays.sort(), Arrays.copyOf(), Arrays.fill() are essential.

5. 2D array = int[][]. Access with [row][col].
   Rows = arr.length. Cols = arr[0].length.

STRINGS:
6. String is immutable. Every operation returns a NEW String.
   ALWAYS capture the result: s = s.toUpperCase(); not just s.toUpperCase();

7. ALWAYS use .equals() to compare String content. NEVER ==.
   Safer pattern: "literal".equals(variable) — null-safe.

8. String Pool: literals are reused. new String() bypasses pool.
   This is why s1 == s2 can be true for literals but false for new.

9. Essential methods to know cold:
   length(), charAt(), indexOf(), contains()
   substring(), split(), replace(), trim()/strip()
   toUpperCase(), toLowerCase(), startsWith(), endsWith()
   isEmpty(), isBlank(), equals(), equalsIgnoreCase()
   String.valueOf(), String.join(), String.format()

10. StringBuilder for string building in loops.
    String += in loops = O(n²) = SLOW.
    StringBuilder.append() = O(n) = FAST.
    Always use StringBuilder when building strings piece by piece.
```

---