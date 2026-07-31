**Phase:** Level 1 — Java Fundamentals
**Date Studied:**

---
## What Problem Does This Solve?
Without loops, repetition is manual and impossible to scale:
```java
System.out.println("Processing order 1");
System.out.println("Processing order 2");
System.out.println("Processing order 3");
// ... what if there are 100,000 orders?
// You can't write 100,000 lines.
// You don't even KNOW how many orders there will be at compile time.
```

```
Real applications need repetition:
  → Send email to every user in the database (could be 5 million)
  → Process every item in a shopping cart (could be 1 or 100)
  → Retry a failed network request up to 3 times
  → Read every line of a log file
  → Check every product for low stock
  → Calculate interest for every account every month
  → Search through results until you find the match
```

Loops solve this:
```java
  for (every order in orders) {
      processOrder(order); // one line, handles any number
  }
```
**Without loops:** programs are fixed-length scripts.
**With loops:** programs handle any amount of data dynamically.

Loops are in EVERY real program ever written.
You will write loops hundreds of times per week as a backend engineer.

---
## The Big Picture - All Loop Types in Java
```text
JAVA LOOP TYPES:

┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  for loop         → when you know HOW MANY times            │
│                     (count-controlled)                      │
│                                                             │
│  while loop       → when you DON'T know how many times      │
│                     (condition-controlled, check BEFORE)    │
│                                                             │
│  do-while loop    → runs AT LEAST ONCE                      │
│                     (condition-controlled, check AFTER)     │
│                                                             │
│  for-each loop    → iterate over collections/arrays         │
│  (enhanced for)     (the most common in real backend code)  │
│                                                             │
│  + break          → exit loop early                         │
│  + continue       → skip current iteration                  │
│  + return         → exit method (also exits any loop)       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---
## 1. The for Loop

### Basic Structure
```java
public class ForLoop {
    public static void main(String[] args) {
        
        // SYNTAX:
        // for (initialization; condition; update) {
        //     // body: runs while condition is true
        // }
        
        // THREE PARTS:
        // initialization → runs ONCE before loop starts
        // condition      → checked BEFORE each iteration
        //                  if true → run body
        //                  if false → exit loop
        // update         → runs AFTER each iteration
        
        // ─────────────────────────────────────────
        // BASIC COUNT UP
        // ─────────────────────────────────────────
        
        for (int i = 0; i < 5; i++) {
            System.out.println("Iteration: " + i);
        }
        // Output:
        // Iteration: 0
        // Iteration: 1
        // Iteration: 2
        // Iteration: 3
        // Iteration: 4
        
        // STEP BY STEP EXECUTION:
        // 1. int i = 0       → initialize i to 0
        // 2. i < 5 → true    → run body (print "Iteration: 0")
        // 3. i++             → i becomes 1
        // 4. i < 5 → true    → run body (print "Iteration: 1")
        // 5. i++             → i becomes 2
        // ... continues ...
        // 6. i < 5 → false (i is 5) → EXIT loop
        
        // NOTE: i is NOT accessible AFTER the loop
        // It only exists within the for loop's scope
        // System.out.println(i); // COMPILE ERROR
        
        // ─────────────────────────────────────────
        // COMMON VARIATIONS
        // ─────────────────────────────────────────
        
        // Count from 1 to 10 (inclusive)
        for (int i = 1; i <= 10; i++) {
            System.out.print(i + " "); // 1 2 3 4 5 6 7 8 9 10
        }
        System.out.println();
        
        // Count DOWN
        for (int i = 5; i >= 1; i--) {
            System.out.print(i + " "); // 5 4 3 2 1
        }
        System.out.println();
        
        // Count by 2s (even numbers)
        for (int i = 0; i <= 20; i += 2) {
            System.out.print(i + " "); // 0 2 4 6 8 10 12 14 16 18 20
        }
        System.out.println();
        
        // Count by 5s
        for (int i = 0; i <= 50; i += 5) {
            System.out.print(i + " "); // 0 5 10 15 20 25 30 35 40 45 50
        }
        System.out.println();
        
        // Powers of 2
        for (int i = 1; i <= 1024; i *= 2) {
            System.out.print(i + " "); // 1 2 4 8 16 32 64 128 256 512 1024
        }
        System.out.println();
        
        // ─────────────────────────────────────────
        // for WITH ARRAYS (very common)
        // ─────────────────────────────────────────
        
        int[] scores = {85, 92, 78, 95, 88};
        int sum = 0;
        
        for (int i = 0; i < scores.length; i++) {
            sum += scores[i];
            System.out.printf("scores[%d] = %d%n", i, scores[i]);
        }
        
        double average = (double) sum / scores.length;
        System.out.printf("Sum: %d, Average: %.2f%n", sum, average);
        
        // ─────────────────────────────────────────
        // OPTIONAL PARTS — for loop flexibility
        // ─────────────────────────────────────────
        
        // All three parts are OPTIONAL (but semicolons are NOT)
        
        // No initialization (variable declared before loop):
        int j = 0;
        for (; j < 3; j++) {
            System.out.print(j + " "); // 0 1 2
        }
        System.out.println("j after loop: " + j); // 3 (accessible here)
        
        // No update (update inside body):
        for (int k = 0; k < 3; ) {
            System.out.print(k + " ");
            k++; // update inside body
        }
        System.out.println();
        
        // Infinite loop (all parts empty — use break to exit):
        int count = 0;
        for (;;) { // same as while(true)
            if (count >= 3) break;
            System.out.print(count + " ");
            count++;
        }
        System.out.println();
        
        // Multiple initializations and updates:
        for (int a = 0, b = 10; a <= b; a++, b--) {
            System.out.printf("a=%d, b=%d%n", a, b);
        }
        // a=0,b=10 → a=1,b=9 → a=2,b=8 → ... → a=5,b=5 → exit (a>b)
    }
}
```

### Nested for Loops
```java
public class NestedForLoops {
    public static void main(String[] args) {
        
        // A loop inside a loop = nested loop
        // Inner loop runs COMPLETELY for each outer iteration
        // Total iterations = outer × inner
        
        // ─────────────────────────────────────────
        // BASIC NESTED LOOP
        // ─────────────────────────────────────────
        
        // Outer runs 3 times, inner runs 3 times each
        // Total: 3 × 3 = 9 iterations
        
        for (int i = 1; i <= 3; i++) {
            for (int j = 1; j <= 3; j++) {
                System.out.printf("(%d,%d) ", i, j);
            }
            System.out.println(); // new line after each row
        }
        // Output:
        // (1,1) (1,2) (1,3)
        // (2,1) (2,2) (2,3)
        // (3,1) (3,2) (3,3)
        
        // ─────────────────────────────────────────
        // MULTIPLICATION TABLE
        // ─────────────────────────────────────────
        
        System.out.println("\nMultiplication Table:");
        System.out.print("   ");
        for (int col = 1; col <= 10; col++) {
            System.out.printf("%4d", col);
        }
        System.out.println();
        System.out.println("   " + "─".repeat(40));
        
        for (int row = 1; row <= 10; row++) {
            System.out.printf("%2d │", row);
            for (int col = 1; col <= 10; col++) {
                System.out.printf("%4d", row * col);
            }
            System.out.println();
        }
        
        // ─────────────────────────────────────────
        // STAR PATTERN (common interview question)
        // ─────────────────────────────────────────
        
        int size = 5;
        
        // Right triangle
        System.out.println("\nRight triangle:");
        for (int row = 1; row <= size; row++) {
            for (int col = 1; col <= row; col++) {
                System.out.print("★ ");
            }
            System.out.println();
        }
        // ★
        // ★ ★
        // ★ ★ ★
        // ★ ★ ★ ★
        // ★ ★ ★ ★ ★
        
        // ─────────────────────────────────────────
        // 2D ARRAY PROCESSING
        // ─────────────────────────────────────────
        
        // Like a spreadsheet/table:
        int[][] matrix = {
            {1,  2,  3,  4},
            {5,  6,  7,  8},
            {9,  10, 11, 12}
        };
        
        System.out.println("\n2D Array (Matrix):");
        for (int row = 0; row < matrix.length; row++) {
            for (int col = 0; col < matrix[row].length; col++) {
                System.out.printf("%4d", matrix[row][col]);
            }
            System.out.println();
        }
        
        // ─────────────────────────────────────────
        // ⚠️ NESTED LOOP PERFORMANCE WARNING
        // ─────────────────────────────────────────
        
        // O(n²) complexity — grows fast!
        // n=100   → 10,000 iterations    (fine)
        // n=1000  → 1,000,000 iterations (slow)
        // n=10000 → 100,000,000 iterations (very slow)
        
        // In backend: never loop over database results in nested loops
        // Use SQL JOINs instead (database optimizes this)
        // Nested loops on large data = performance disaster
        
        // Maximum practical nesting in backend code: 2 levels
        // 3+ levels = time to rethink your algorithm
    }
}
```

---
## 2. The while Loop
```java
public class WhileLoop {
    public static void main(String[] args) {
        
        // SYNTAX:
        // while (condition) {
        //     // body: runs as long as condition is true
        // }
        
        // Condition checked BEFORE each iteration.
        // If condition is false from the start → body NEVER runs.
        
        // USE WHEN: you don't know how many iterations upfront.
        // The loop runs until SOMETHING CHANGES to make condition false.
        
        // ─────────────────────────────────────────
        // BASIC while LOOP
        // ─────────────────────────────────────────
        
        int count = 0;
        while (count < 5) {
            System.out.println("Count: " + count);
            count++; // MUST change something or it's infinite!
        }
        // Output:
        // Count: 0
        // Count: 1
        // Count: 2
        // Count: 3
        // Count: 4
        
        // ─────────────────────────────────────────
        // RETRY PATTERN — while shines here
        // ─────────────────────────────────────────
        
        // Simulate: retry connecting to database (up to 3 times)
        int maxRetries = 3;
        int attempts = 0;
        boolean connected = false;
        
        while (!connected && attempts < maxRetries) {
            attempts++;
            System.out.println("Connection attempt " + attempts + "...");
            
            // Simulate success on 3rd attempt
            if (attempts == 3) {
                connected = true;
                System.out.println("✅ Connected to database!");
            } else {
                System.out.println("❌ Failed. Retrying in 1 second...");
                // Thread.sleep(1000); // would pause in real code
            }
        }
        
        if (!connected) {
            System.out.println("❌ Could not connect after " + maxRetries + " attempts");
        }
        
        // ─────────────────────────────────────────
        // READING UNTIL CONDITION MET
        // ─────────────────────────────────────────
        
        // Simulate processing a stream of numbers until we find 0
        int[] dataStream = {3, 7, 12, 5, 0, 8, 4}; // 0 is the "stop" signal
        int index = 0;
        int streamSum = 0;
        
        while (index < dataStream.length && dataStream[index] != 0) {
            streamSum += dataStream[index];
            System.out.println("Processing: " + dataStream[index]);
            index++;
        }
        System.out.println("Stopped at 0. Sum so far: " + streamSum); // 27
        
        // ─────────────────────────────────────────
        // COLLATZ CONJECTURE — unknown iterations
        // ─────────────────────────────────────────
        
        // Classic example: we don't know how many steps until n = 1
        int n = 27;
        int steps = 0;
        System.out.print("\nCollatz sequence from " + n + ": " + n);
        
        while (n != 1) {
            if (n % 2 == 0) {
                n = n / 2;
            } else {
                n = 3 * n + 1;
            }
            System.out.print(" → " + n);
            steps++;
        }
        System.out.println("\nSteps: " + steps); // 111 steps for 27!
        
        // ─────────────────────────────────────────
        // FIBONACCI SEQUENCE
        // ─────────────────────────────────────────
        
        System.out.println("\nFibonacci below 1000:");
        int prev = 0, curr = 1;
        
        while (curr < 1000) {
            System.out.print(curr + " ");
            int next = prev + curr;
            prev = curr;
            curr = next;
        }
        System.out.println();
        
        // ─────────────────────────────────────────
        // DIGIT SUM (while = natural fit)
        // ─────────────────────────────────────────
        
        int number = 123456789;
        int digitSum = 0;
        int temp = Math.abs(number); // handle negative
        
        while (temp > 0) {
            digitSum += temp % 10; // get last digit
            temp /= 10;            // remove last digit
        }
        System.out.println("Digit sum of " + number + " = " + digitSum);
        // 1+2+3+4+5+6+7+8+9 = 45
        
        // ─────────────────────────────────────────
        // INFINITE LOOP — while(true)
        // ─────────────────────────────────────────
        
        // Very common pattern in:
        // Server loops (keep server running)
        // Menu-driven programs
        // Consumer threads (Kafka consumer loop)
        // Game loops
        
        // while (true) {
        //     Request request = server.acceptConnection();
        //     handleRequest(request); // process each connection
        // }
        // ← this is essentially how servers work!
        
        // For demo, use a limited version:
        int menuChoice = 0;
        int menuIteration = 0; // prevent actual infinite loop in demo
        
        while (true) {
            menuIteration++;
            // Simulate user choosing to exit on 3rd try
            menuChoice = (menuIteration == 3) ? 0 : menuIteration;
            
            System.out.println("\nMenu choice: " + menuChoice);
            
            if (menuChoice == 0) {
                System.out.println("Exiting...");
                break; // EXIT the infinite loop
            } else if (menuChoice == 1) {
                System.out.println("Viewing orders...");
            } else if (menuChoice == 2) {
                System.out.println("Viewing profile...");
            }
        }
    }
}
```

---
## 3. The do-while Loop
```java
public class DoWhileLoop {
    public static void main(String[] args) {
        
        // SYNTAX:
        // do {
        //     // body: runs at least ONCE
        // } while (condition);
        //         ← NOTE the semicolon here!
        
        // KEY DIFFERENCE from while:
        // while: checks condition BEFORE body (body might never run)
        // do-while: checks condition AFTER body (body ALWAYS runs at least once)
        
        // USE WHEN: you always want to run the body at least once,
        // then check if you should continue.
        
        // ─────────────────────────────────────────
        // BASIC do-while
        // ─────────────────────────────────────────
        
        int count = 0;
        do {
            System.out.println("Count: " + count);
            count++;
        } while (count < 5);
        // Output: same as while loop example
        // Count: 0 through Count: 4
        
        // ─────────────────────────────────────────
        // THE CRITICAL DIFFERENCE
        // ─────────────────────────────────────────
        
        // while: body might not run at all
        int x = 10;
        while (x < 5) {
            System.out.println("while: This never prints"); // skipped
        }
        
        // do-while: body ALWAYS runs at least once
        int y = 10;
        do {
            System.out.println("do-while: This ALWAYS prints once"); // prints!
        } while (y < 5); // condition false → stops after first run
        
        // ─────────────────────────────────────────
        // INPUT VALIDATION PATTERN
        // ─────────────────────────────────────────
        
        // Classic use: menu/input that must run at least once.
        // You HAVE to show the menu before you can check the input.
        
        // Simulate user entering wrong PIN twice, then correct:
        String[] userInputs = {"0000", "1111", "1234"}; // simulated input
        int inputIndex = 0;
        
        final String CORRECT_PIN = "1234";
        int pinAttempts = 0;
        String enteredPin;
        
        do {
            enteredPin = userInputs[inputIndex++]; // simulate user typing
            pinAttempts++;
            System.out.println("Attempt " + pinAttempts + ": entered " + enteredPin);
            
            if (!enteredPin.equals(CORRECT_PIN)) {
                System.out.println("Wrong PIN. Try again.");
            }
        } while (!enteredPin.equals(CORRECT_PIN) && pinAttempts < 3);
        
        if (enteredPin.equals(CORRECT_PIN)) {
            System.out.println("✅ PIN correct! Access granted.");
        } else {
            System.out.println("❌ Too many failed attempts. Card blocked.");
        }
        
        // ─────────────────────────────────────────
        // GAME-STYLE LOOP (play at least once)
        // ─────────────────────────────────────────
        
        int score = 0;
        int round = 0;
        boolean playAgain;
        
        do {
            round++;
            score += (int)(Math.random() * 100); // simulate scoring
            System.out.printf("Round %d: score = %d (total: %d)%n",
                              round, score % 100, score);
            
            // Simulate: stop after 3 rounds in this demo
            playAgain = round < 3;
            
        } while (playAgain);
        
        System.out.println("Game over! Final score: " + score);
        
        // ─────────────────────────────────────────
        // WHEN TO USE do-while (rare in backend)
        // ─────────────────────────────────────────
        
        // Honest assessment:
        // do-while is RARE in backend web development.
        // Most backend logic uses for or while.
        // BUT: understanding it is important for interviews
        // and reading legacy code.
        
        // Common do-while use cases:
        // → ATM/menu interaction (must show menu first)
        // → PIN/password retry logic
        // → Game loops
        // → Parsing: read at least one token, then continue
        // → Polling: check at least once, then decide to continue
    }
}
```

---
## 4. The for-each Loop (Enhanced for)
```java
public class ForEachLoop {
    public static void main(String[] args) {
        
        // SYNTAX:
        // for (ElementType variable : collection) {
        //     // use variable (current element)
        // }
        
        // READS AS: "for each element in collection"
        // Automatically gets each element, one at a time.
        // No index management. No bounds checking. Cleaner.
        
        // WORKS WITH:
        // ✅ Arrays
        // ✅ ArrayList, LinkedList, other List implementations
        // ✅ HashSet, TreeSet, other Set implementations
        // ✅ HashMap (via keySet(), values(), entrySet())
        // ✅ Any class implementing Iterable<T>
        
        // ─────────────────────────────────────────
        // WITH ARRAYS
        // ─────────────────────────────────────────
        
        int[] numbers = {10, 20, 30, 40, 50};
        
        // Old way (for loop):
        System.out.println("for loop:");
        for (int i = 0; i < numbers.length; i++) {
            System.out.print(numbers[i] + " ");
        }
        System.out.println();
        
        // Better way (for-each):
        System.out.println("for-each:");
        for (int num : numbers) {
            System.out.print(num + " ");
        }
        System.out.println();
        // Output: 10 20 30 40 50 (same result, cleaner code)
        
        // Computing sum with for-each:
        int sum = 0;
        for (int num : numbers) {
            sum += num;
        }
        System.out.println("Sum: " + sum); // 150
        
        // ─────────────────────────────────────────
        // WITH String ARRAYS
        // ─────────────────────────────────────────
        
        String[] products = {"Laptop", "Phone", "Tablet", "Watch"};
        
        System.out.println("\nProduct Catalog:");
        for (String product : products) {
            System.out.println("  • " + product);
        }
        
        // Find specific item:
        String target = "Tablet";
        for (String product : products) {
            if (product.equals(target)) {
                System.out.println("Found: " + target);
                break; // found it, stop searching
            }
        }
        
        // ─────────────────────────────────────────
        // WITH ArrayList (Collections)
        // ─────────────────────────────────────────
        
        java.util.List<String> cities = new java.util.ArrayList<>();
        cities.add("Dhaka");
        cities.add("Chittagong");
        cities.add("Sylhet");
        cities.add("Rajshahi");
        cities.add("Khulna");
        
        System.out.println("\nCities in Bangladesh:");
        for (String city : cities) {
            System.out.println("  → " + city);
        }
        
        // Filter while iterating:
        System.out.println("\nCities with more than 5 characters:");
        for (String city : cities) {
            if (city.length() > 5) {
                System.out.println("  " + city + " (" + city.length() + " chars)");
            }
        }
        
        // ─────────────────────────────────────────
        // WITH HashMap
        // ─────────────────────────────────────────
        
        java.util.Map<String, Double> productPrices = new java.util.HashMap<>();
        productPrices.put("Laptop", 75000.0);
        productPrices.put("Phone", 45000.0);
        productPrices.put("Tablet", 35000.0);
        productPrices.put("Watch", 15000.0);
        
        // Iterate over keys:
        System.out.println("\nProduct names:");
        for (String productName : productPrices.keySet()) {
            System.out.println("  " + productName);
        }
        
        // Iterate over values:
        System.out.println("\nProduct prices:");
        double total = 0;
        for (double price : productPrices.values()) {
            total += price;
            System.out.printf("  ৳%.2f%n", price);
        }
        System.out.printf("Total: ৳%.2f%n", total);
        
        // Iterate over entries (key-value pairs): ← MOST COMMON
        System.out.println("\nFull catalog:");
        for (java.util.Map.Entry<String, Double> entry : productPrices.entrySet()) {
            System.out.printf("  %-10s : ৳%.2f%n",
                              entry.getKey(), entry.getValue());
        }
        
        // ─────────────────────────────────────────
        // LIMITATION: CANNOT MODIFY THE COLLECTION WHILE ITERATING
        // ─────────────────────────────────────────
        
        java.util.List<Integer> nums = new java.util.ArrayList<>();
        nums.add(1); nums.add(2); nums.add(3); nums.add(4); nums.add(5);
        
        // ❌ This throws ConcurrentModificationException:
        // for (Integer n : nums) {
        //     if (n % 2 == 0) {
        //         nums.remove(n); // CANNOT remove while for-each iterating!
        //     }
        // }
        
        // ✅ Fix 1: Use Iterator with remove():
        java.util.Iterator<Integer> iterator = nums.iterator();
        while (iterator.hasNext()) {
            int n = iterator.next();
            if (n % 2 == 0) {
                iterator.remove(); // safe removal via Iterator
            }
        }
        System.out.println("\nAfter removing evens: " + nums); // [1, 3, 5]
        
        // ✅ Fix 2: Use removeIf() (Java 8+ — more elegant):
        nums = new java.util.ArrayList<>(
            java.util.Arrays.asList(1, 2, 3, 4, 5));
        nums.removeIf(n -> n % 2 == 0);
        System.out.println("removeIf evens: " + nums); // [1, 3, 5]
        
        // ✅ Fix 3: Collect to new list then replace
        
        // ─────────────────────────────────────────
        // LIMITATION: NO INDEX ACCESS
        // ─────────────────────────────────────────
        
        // for-each doesn't give you the index.
        // If you NEED the index, use regular for loop:
        
        String[] items = {"apple", "banana", "cherry"};
        
        // Need index: use regular for
        for (int i = 0; i < items.length; i++) {
            System.out.printf("items[%d] = %s%n", i, items[i]);
        }
        
        // Don't need index: use for-each
        for (String item : items) {
            System.out.println(item.toUpperCase());
        }
        
        // ─────────────────────────────────────────
        // REAL-WORLD BACKEND USAGE
        // ─────────────────────────────────────────
        
        // Processing a list of orders:
        java.util.List<String> orderIds = java.util.List.of(
            "ORD-001", "ORD-002", "ORD-003"
        );
        
        System.out.println("\nProcessing orders:");
        for (String orderId : orderIds) {
            System.out.println("Processing " + orderId + "...");
            // In real code: service.processOrder(orderId);
        }
        
        // Validating a list of emails:
        java.util.List<String> emails = java.util.List.of(
            "rahim@example.com", "invalid-email", "karim@test.com", ""
        );
        
        System.out.println("\nEmail validation:");
        for (String email : emails) {
            boolean isValid = email.contains("@") && !email.isEmpty();
            System.out.printf("  %-25s → %s%n",
                              email,
                              isValid ? "✅ Valid" : "❌ Invalid");
        }
    }
}
```

---
## 5. break and continue
```java
public class BreakAndContinue {
    public static void main(String[] args) {
        
        // break    → EXIT the loop immediately (skip remaining iterations)
        // continue → SKIP the current iteration (jump to next iteration)
        
        // ─────────────────────────────────────────
        // break — EXIT EARLY
        // ─────────────────────────────────────────
        
        System.out.println("=== break examples ===");
        
        // Find first negative number in array:
        int[] data = {5, 12, 8, -3, 14, -7, 20};
        int firstNegative = -1;
        
        for (int num : data) {
            if (num < 0) {
                firstNegative = num;
                break; // found it — stop searching, no need to check rest
            }
        }
        System.out.println("First negative: " + firstNegative); // -3
        
        // Search by name:
        String[] names = {"Rahim", "Karim", "Hasan", "Nabil", "Rafiq"};
        String searchFor = "Hasan";
        boolean found = false;
        
        for (int i = 0; i < names.length; i++) {
            if (names[i].equals(searchFor)) {
                System.out.println("Found '" + searchFor + "' at index " + i);
                found = true;
                break; // no need to check remaining names
            }
        }
        if (!found) {
            System.out.println("'" + searchFor + "' not found");
        }
        
        // break in while — process until sentinel value:
        int[] stream = {3, 8, 12, -1, 5, 9}; // -1 is the "stop" signal
        int idx = 0;
        int streamTotal = 0;
        
        while (idx < stream.length) {
            if (stream[idx] == -1) {
                break; // -1 is sentinel — stop processing
            }
            streamTotal += stream[idx];
            idx++;
        }
        System.out.println("Stream total (before -1): " + streamTotal); // 23
        
        // ─────────────────────────────────────────
        // continue — SKIP CURRENT ITERATION
        // ─────────────────────────────────────────
        
        System.out.println("\n=== continue examples ===");
        
        // Print only ODD numbers:
        System.out.print("Odd numbers 1-10: ");
        for (int i = 1; i <= 10; i++) {
            if (i % 2 == 0) {
                continue; // even number → skip to next iteration
            }
            System.out.print(i + " "); // only odd numbers reach here
        }
        System.out.println(); // 1 3 5 7 9
        
        // Skip null/invalid values in processing:
        String[] emails = {"rahim@test.com", null, "", "karim@test.com",
                           "invalid", "nabil@test.com"};
        
        System.out.println("Valid emails:");
        for (String email : emails) {
            // Skip null
            if (email == null) {
                System.out.println("  [skipping null]");
                continue;
            }
            // Skip empty
            if (email.isBlank()) {
                System.out.println("  [skipping empty]");
                continue;
            }
            // Skip invalid (no @ symbol)
            if (!email.contains("@")) {
                System.out.println("  [skipping invalid: " + email + "]");
                continue;
            }
            // Only valid emails reach here:
            System.out.println("  ✅ " + email);
        }
        
        // ─────────────────────────────────────────
        // break vs continue — visual comparison
        // ─────────────────────────────────────────
        
        System.out.println("\n--- break: stops at 5 ---");
        for (int i = 1; i <= 10; i++) {
            if (i == 5) break;
            System.out.print(i + " "); // 1 2 3 4
        }
        System.out.println();
        
        System.out.println("--- continue: skips 5 ---");
        for (int i = 1; i <= 10; i++) {
            if (i == 5) continue;
            System.out.print(i + " "); // 1 2 3 4 6 7 8 9 10
        }
        System.out.println();
        
        // ─────────────────────────────────────────
        // LABELED break/continue (rare but important to know)
        // ─────────────────────────────────────────
        
        // Without labels, break/continue only affect the INNERMOST loop
        // Labels let you break/continue an OUTER loop
        
        System.out.println("\nLabeled break (exit outer loop):");
        
        outerLoop: // label for outer loop
        for (int i = 1; i <= 3; i++) {
            for (int j = 1; j <= 3; j++) {
                if (i == 2 && j == 2) {
                    System.out.println("Breaking outer loop at i=" + i + ", j=" + j);
                    break outerLoop; // exits the OUTER loop entirely
                }
                System.out.println("  i=" + i + ", j=" + j);
            }
        }
        // Output:
        // i=1, j=1
        // i=1, j=2
        // i=1, j=3
        // i=2, j=1
        // Breaking outer loop at i=2, j=2
        // (stops completely — no i=2,j=3, no i=3)
        
        System.out.println("\nLabeled continue (skip to next outer iteration):");
        
        outer: // label
        for (int i = 1; i <= 3; i++) {
            for (int j = 1; j <= 3; j++) {
                if (j == 2) {
                    continue outer; // skip rest of inner, go to next i
                }
                System.out.println("  i=" + i + ", j=" + j);
            }
        }
        // Output:
        // i=1, j=1
        // i=2, j=1
        // i=3, j=1
        // (j=2 and j=3 are never printed — continue outer skips them)
        
        // LABELED loops: rare in professional code.
        // Usually means you should refactor into a method with return.
        // Good to know for interviews and reading others' code.
    }
}
```

---
## 6. Common Loop Patterns in Backend Engineering
```java
public class BackendLoopPatterns {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // PATTERN 1: Accumulator
        // Sum/collect values from a collection
        // ─────────────────────────────────────────
        
        double[] orderAmounts = {150.0, 340.50, 89.99, 520.0, 225.75};
        
        double total = 0;
        double maxOrder = Double.MIN_VALUE;
        double minOrder = Double.MAX_VALUE;
        
        for (double amount : orderAmounts) {
            total    += amount;                            // running sum
            maxOrder  = Math.max(maxOrder, amount);       // track max
            minOrder  = Math.min(minOrder, amount);       // track min
        }
        
        double average = total / orderAmounts.length;
        
        System.out.printf("Total   : ৳%.2f%n", total);
        System.out.printf("Average : ৳%.2f%n", average);
        System.out.printf("Highest : ৳%.2f%n", maxOrder);
        System.out.printf("Lowest  : ৳%.2f%n", minOrder);
        
        // ─────────────────────────────────────────
        // PATTERN 2: Filter and collect
        // Select elements matching a condition
        // ─────────────────────────────────────────
        
        java.util.List<String> allUsers = java.util.List.of(
            "Rahim", "Karim", "admin_Hasan", "Nabil", "admin_Rafiq", "Tariq"
        );
        
        java.util.List<String> adminUsers = new java.util.ArrayList<>();
        java.util.List<String> regularUsers = new java.util.ArrayList<>();
        
        for (String user : allUsers) {
            if (user.startsWith("admin_")) {
                adminUsers.add(user);
            } else {
                regularUsers.add(user);
            }
        }
        
        System.out.println("\nAdmins: " + adminUsers);
        System.out.println("Regular: " + regularUsers);
        
        // ─────────────────────────────────────────
        // PATTERN 3: Transform (map)
        // Convert each element to a different form
        // ─────────────────────────────────────────
        
        String[] rawPrices = {"1000", "2500", "750", "3200"};
        double[] prices = new double[rawPrices.length];
        
        for (int i = 0; i < rawPrices.length; i++) {
            prices[i] = Double.parseDouble(rawPrices[i]); // String → double
        }
        
        System.out.println("\nParsed prices:");
        for (double price : prices) {
            System.out.printf("  ৳%.2f%n", price);
        }
        
        // ─────────────────────────────────────────
        // PATTERN 4: Build a result string
        // ─────────────────────────────────────────
        
        String[] tags = {"java", "spring-boot", "backend", "api"};
        
        // Build comma-separated string:
        StringBuilder tagBuilder = new StringBuilder();
        for (int i = 0; i < tags.length; i++) {
            tagBuilder.append(tags[i]);
            if (i < tags.length - 1) {
                tagBuilder.append(", "); // add comma except after last
            }
        }
        System.out.println("\nTags: " + tagBuilder); // java, spring-boot, backend, api
        
        // Cleaner way: String.join()
        String tagString = String.join(", ", tags);
        System.out.println("Tags: " + tagString); // same result
        
        // ─────────────────────────────────────────
        // PATTERN 5: Count occurrences
        // ─────────────────────────────────────────
        
        String[] statuses = {"PAID", "PENDING", "PAID", "CANCELLED",
                             "PAID", "PENDING", "PAID", "SHIPPED"};
        
        java.util.Map<String, Integer> statusCounts = new java.util.HashMap<>();
        
        for (String status : statuses) {
            // If key exists: increment. If not: start at 1.
            statusCounts.put(status, statusCounts.getOrDefault(status, 0) + 1);
        }
        
        System.out.println("\nOrder status counts:");
        for (java.util.Map.Entry<String, Integer> entry : statusCounts.entrySet()) {
            System.out.printf("  %-12s : %d%n", entry.getKey(), entry.getValue());
        }
        
        // ─────────────────────────────────────────
        // PATTERN 6: Pagination simulation
        // ─────────────────────────────────────────
        
        // Simulate 50 products, show page 2 (size 10)
        int totalProducts = 50;
        int pageSize = 10;
        int pageNumber = 2; // 0-indexed
        
        int startIndex = pageNumber * pageSize;
        int endIndex = Math.min(startIndex + pageSize, totalProducts);
        
        System.out.printf("\nPage %d (items %d-%d of %d):%n",
                          pageNumber + 1, startIndex + 1, endIndex, totalProducts);
        
        for (int i = startIndex; i < endIndex; i++) {
            System.out.printf("  Product #%02d: Item %d%n", i + 1, i + 1);
        }
        
        // ─────────────────────────────────────────
        // PATTERN 7: Retry with exponential backoff
        // Common in distributed systems / API calls
        // ─────────────────────────────────────────
        
        int maxAttempts = 4;
        long baseDelayMs = 100; // 100ms base delay
        
        System.out.println("\nRetry with exponential backoff:");
        
        for (int attempt = 1; attempt <= maxAttempts; attempt++) {
            System.out.printf("Attempt %d...", attempt);
            
            // Simulate: succeed on 4th attempt
            boolean success = (attempt == maxAttempts);
            
            if (success) {
                System.out.println(" ✅ Success!");
                break;
            } else {
                long delay = baseDelayMs * (long) Math.pow(2, attempt - 1);
                System.out.printf(" ❌ Failed. Waiting %dms...%n", delay);
                // Thread.sleep(delay); // would actually wait in real code
            }
        }
        
        // ─────────────────────────────────────────
        // PATTERN 8: Batch processing
        // Process large dataset in chunks
        // ─────────────────────────────────────────
        
        int totalRecords = 1000;
        int batchSize = 100;
        
        System.out.println("\nBatch processing " + totalRecords + " records:");
        
        for (int batchStart = 0; batchStart < totalRecords; batchStart += batchSize) {
            int batchEnd = Math.min(batchStart + batchSize, totalRecords);
            int currentBatch = (batchStart / batchSize) + 1;
            System.out.printf("  Processing batch %d: records %d-%d%n",
                              currentBatch, batchStart + 1, batchEnd);
            // In real code: processBatch(records.subList(batchStart, batchEnd));
        }
    }
}
```

---
## 7. Loop Performance - What Every Backend Engineer Knows
```java
public class LoopPerformance {
    public static void main(String[] args) {
        
        // ─────────────────────────────────────────
        // O(n) — LINEAR: grows with data size
        // ─────────────────────────────────────────
        
        // ONE loop over n items = O(n)
        // 100 items → 100 operations
        // 1,000,000 items → 1,000,000 operations
        // ACCEPTABLE for most cases
        
        int[] items = new int[1000];
        int linearSum = 0;
        for (int item : items) {
            linearSum += item; // O(n) — fine
        }
        
        // ─────────────────────────────────────────
        // O(n²) — QUADRATIC: grows with data size SQUARED
        // ─────────────────────────────────────────
        
        // TWO nested loops over n items = O(n²)
        // 100 items → 10,000 operations
        // 1,000 items → 1,000,000 operations
        // 10,000 items → 100,000,000 operations ← SLOW!
        // DANGEROUS for large datasets
        
        // ─────────────────────────────────────────
        // COMMON BACKEND PERFORMANCE MISTAKES
        // ─────────────────────────────────────────
        
        // MISTAKE: Loading all data then looping
        // ❌ Bad (N+1 query problem):
        // List<User> users = userRepository.findAll(); // load ALL users
        // for (User user : users) {
        //     List<Order> orders = orderRepository.findByUser(user); // N queries!
        //     // 1000 users = 1001 database queries!
        // }
        
        // ✅ Good: Use JOIN in database query → ONE query
        // List<UserWithOrders> = userRepository.findAllWithOrders();
        
        // MISTAKE: String concatenation in loop
        System.out.println("String concatenation performance:");
        
        long startTime = System.currentTimeMillis();
        String badResult = "";
        for (int i = 0; i < 10000; i++) {
            badResult += i; // creates NEW string each time → O(n²)
        }
        long badTime = System.currentTimeMillis() - startTime;
        
        startTime = System.currentTimeMillis();
        StringBuilder goodResult = new StringBuilder();
        for (int i = 0; i < 10000; i++) {
            goodResult.append(i); // modifies same object → O(n)
        }
        long goodTime = System.currentTimeMillis() - startTime;
        
        System.out.printf("  String +=     : %d ms%n", badTime);
        System.out.printf("  StringBuilder : %d ms%n", goodTime);
        System.out.println("  StringBuilder is typically much faster!");
        
        // ─────────────────────────────────────────
        // MOVE INVARIANTS OUTSIDE THE LOOP
        // ─────────────────────────────────────────
        
        // ❌ Recalculates list.size() every iteration (minor but bad habit):
        java.util.List<String> bigList = new java.util.ArrayList<>();
        for (int i = 0; i < 1000; i++) bigList.add("item" + i);
        
        // ❌ Less efficient:
        for (int i = 0; i < bigList.size(); i++) { // size() called each time
            // ... (compiler often optimizes this, but don't rely on it)
        }
        
        // ✅ Better:
        int size = bigList.size(); // calculate ONCE before loop
        for (int i = 0; i < size; i++) {
            // ...
        }
        
        // ❌ Expensive operation inside loop:
        // for (String item : items) {
        //     DatabaseResult result = database.query("SELECT..."); // BAD!
        //     // n database calls for n items!
        // }
        
        // ✅ Move it before the loop:
        // DatabaseResult result = database.query("SELECT..."); // ONCE
        // for (String item : items) {
        //     process(item, result); // reuse the result
        // }
        
        // ─────────────────────────────────────────
        // INFINITE LOOP DETECTION
        // ─────────────────────────────────────────
        
        // Common causes of infinite loops:
        // 1. Forgot to update loop variable:
        //    int i = 0;
        //    while (i < 10) { System.out.println(i); } // forgot i++!
        
        // 2. Off-by-one in condition:
        //    for (int i = 0; i <= arr.length; i++) { // should be <, not <=
        //        arr[i]; // ArrayIndexOutOfBoundsException at arr.length!
        //    }
        
        // 3. Condition never becomes false:
        //    while (balance > 0) { balance += interest; } // balance grows, never ≤ 0
        
        // 4. break condition inside loop that never triggers:
        //    while(true) { if (condition) break; } // condition never true
        
        // DEBUGGING infinite loops:
        // → Add print statement inside loop to see what's happening
        // → Use IntelliJ debugger — pause and inspect variables
        // → Add a safety counter: if (iterations++ > 1000000) break;
        System.out.println("Loop performance analysis complete.");
    }
}
```

---
## Build This - Complete Loop Practice Program
```java
// File: LoopMasterpiece.java
// A comprehensive program demonstrating all loop types
// in a realistic backend engineering context

import java.util.*;

public class LoopMasterpiece {
    
    public static void main(String[] args) {
        System.out.println("╔══════════════════════════════════════════╗");
        System.out.println("║      BACKEND ANALYTICS DASHBOARD          ║");
        System.out.println("╚══════════════════════════════════════════╝\n");
        
        // ─────────────────────────────────────────
        // DATA SETUP
        // ─────────────────────────────────────────
        double[] dailySales = {
            12500, 18900, 7800, 22100, 15600,
            9300,  4200,  31000, 28750, 19400,
            11200, 25600, 8900, 17300, 21800
        };
        
        String[] products = {
            "Laptop", "Phone", "Tablet",
            "Watch", "Earbuds", "Charger",
            "Case", "Keyboard", "Mouse"
        };
        
        int[] productSold = {45, 120, 67, 89, 203, 156, 310, 78, 94};
        double[] productPrice = {
            75000, 45000, 35000,
            15000, 8000, 2500,
            500, 3500, 2000
        };
        
        String[] orderStatuses = {
            "DELIVERED", "PROCESSING", "SHIPPED",
            "DELIVERED", "CANCELLED", "DELIVERED",
            "PROCESSING", "SHIPPED", "DELIVERED",
            "DELIVERED", "CANCELLED", "PROCESSING"
        };
        
        // ─────────────────────────────────────────
        // SECTION 1: SALES STATISTICS (for loop)
        // ─────────────────────────────────────────
        System.out.println("📊 DAILY SALES ANALYSIS (Last 15 Days)");
        System.out.println("─".repeat(44));
        
        double totalSales = 0;
        double maxSales = Double.MIN_VALUE;
        double minSales = Double.MAX_VALUE;
        int maxDay = 0, minDay = 0;
        
        for (int i = 0; i < dailySales.length; i++) {
            totalSales += dailySales[i];
            if (dailySales[i] > maxSales) {
                maxSales = dailySales[i];
                maxDay = i + 1;
            }
            if (dailySales[i] < minSales) {
                minSales = dailySales[i];
                minDay = i + 1;
            }
        }
        
        double avgSales = totalSales / dailySales.length;
        
        System.out.printf("Total Revenue : ৳%,.2f%n", totalSales);
        System.out.printf("Average Daily : ৳%,.2f%n", avgSales);
        System.out.printf("Best Day      : Day %d (৳%,.2f)%n", maxDay, maxSales);
        System.out.printf("Worst Day     : Day %d (৳%,.2f)%n", minDay, minSales);
        
        // Count days above average:
        int daysAboveAvg = 0;
        for (double sale : dailySales) {
            if (sale > avgSales) daysAboveAvg++;
        }
        System.out.printf("Days > Average: %d / %d%n%n", daysAboveAvg, dailySales.length);
        
        // ─────────────────────────────────────────
        // SECTION 2: SALES BAR CHART (nested loops)
        // ─────────────────────────────────────────
        System.out.println("📈 SALES BAR CHART (each ■ = ৳2000)");
        System.out.println("─".repeat(44));
        
        for (int i = 0; i < dailySales.length; i++) {
            System.out.printf("Day %2d │", i + 1);
            
            int bars = (int)(dailySales[i] / 2000);
            for (int b = 0; b < bars; b++) {
                System.out.print("■");
            }
            System.out.printf(" ৳%,.0f%n", dailySales[i]);
        }
        System.out.println();
        
        // ─────────────────────────────────────────
        // SECTION 3: PRODUCT REVENUE (for-each)
        // ─────────────────────────────────────────
        System.out.println("📦 PRODUCT REVENUE REPORT");
        System.out.println("─".repeat(44));
        System.out.printf("%-12s %6s %10s %12s%n",
                          "Product", "Sold", "Price", "Revenue");
        System.out.println("─".repeat(44));
        
        double totalRevenue = 0;
        double maxRevenue = 0;
        String topProduct = "";
        
        for (int i = 0; i < products.length; i++) {
            double revenue = productSold[i] * productPrice[i];
            totalRevenue += revenue;
            
            if (revenue > maxRevenue) {
                maxRevenue = revenue;
                topProduct = products[i];
            }
            
            System.out.printf("%-12s %6d %10.0f %12.0f%n",
                              products[i],
                              productSold[i],
                              productPrice[i],
                              revenue);
        }
        
        System.out.println("─".repeat(44));
        System.out.printf("%-12s %6s %10s %12.0f%n",
                          "TOTAL", "", "", totalRevenue);
        System.out.printf("Top Product   : %s (৳%,.0f)%n%n",
                          topProduct, maxRevenue);
        
        // ─────────────────────────────────────────
        // SECTION 4: ORDER STATUS (while + Map)
        // ─────────────────────────────────────────
        System.out.println("🚚 ORDER STATUS BREAKDOWN");
        System.out.println("─".repeat(44));
        
        Map<String, Integer> statusCount = new HashMap<>();
        int idx = 0;
        
        // Count with while loop
        while (idx < orderStatuses.length) {
            String status = orderStatuses[idx];
            statusCount.put(status, statusCount.getOrDefault(status, 0) + 1);
            idx++;
        }
        
        // Display with for-each
        for (Map.Entry<String, Integer> entry : statusCount.entrySet()) {
            int cnt = entry.getValue();
            double pct = (double) cnt / orderStatuses.length * 100;
            
            System.out.printf("%-12s : %2d orders (%5.1f%%) ",
                              entry.getKey(), cnt, pct);
            
            // Mini bar for each status
            for (int b = 0; b < cnt; b++) System.out.print("●");
            System.out.println();
        }
        System.out.println();
        
        // ─────────────────────────────────────────
        // SECTION 5: LOW STOCK ALERT (break/continue)
        // ─────────────────────────────────────────
        System.out.println("⚠️  INVENTORY ALERTS");
        System.out.println("─".repeat(44));
        
        int[] stockLevels = {150, 23, 8, 0, 45, 3, 200, 15, 67};
        int criticalCount = 0;
        
        for (int i = 0; i < products.length; i++) {
            int stock = stockLevels[i];
            
            // Skip products with healthy stock
            if (stock > 30) {
                continue;
            }
            
            // Flag critical/low
            String alert;
            if (stock == 0) {
                alert = "🔴 OUT OF STOCK";
                criticalCount++;
            } else if (stock <= 5) {
                alert = "🔴 CRITICAL";
                criticalCount++;
            } else if (stock <= 15) {
                alert = "🟡 LOW";
            } else {
                alert = "🟠 REORDER SOON";
            }
            
            System.out.printf("%-12s : %3d units → %s%n",
                              products[i], stock, alert);
        }
        
        if (criticalCount > 0) {
            System.out.printf("\n🚨 %d product(s) need IMMEDIATE restocking!%n", criticalCount);
        }
        System.out.println();
        
        // ─────────────────────────────────────────
        // SECTION 6: BATCH PROCESSING SIMULATION
        // ─────────────────────────────────────────
        System.out.println("⚙️  BATCH INVOICE GENERATION");
        System.out.println("─".repeat(44));
        
        int totalInvoices = 47;
        int batchSize = 10;
        int batchNum = 0;
        
        for (int start = 0; start < totalInvoices; start += batchSize) {
            int end = Math.min(start + batchSize, totalInvoices);
            batchNum++;
            System.out.printf("Batch %d: Generating invoices %d-%d... ✅%n",
                              batchNum, start + 1, end);
        }
        System.out.printf("Complete: %d invoices generated in %d batches.%n%n",
                          totalInvoices, batchNum);
        
        // ─────────────────────────────────────────
        // FINAL SUMMARY
        // ─────────────────────────────────────────
        System.out.println("╔══════════════════════════════════════════╗");
        System.out.println("║             EXECUTIVE SUMMARY             ║");
        System.out.println("╠══════════════════════════════════════════╣");
        System.out.printf( "║  Total Sales Revenue : ৳%,12.2f     ║%n", totalSales);
        System.out.printf( "║  Total Product Revenue: ৳%,12.2f     ║%n", totalRevenue);
        System.out.printf( "║  Top Product         : %-18s ║%n", topProduct);
        System.out.printf( "║  Critical Stock Items: %-18d ║%n", criticalCount);
        System.out.printf( "║  Orders Tracked      : %-18d ║%n", orderStatuses.length);
        System.out.println("╚══════════════════════════════════════════╝");
    }
}
```

---
## Exercises
```text
EXERCISE 1: Number Patterns
  Create NumberPatterns.java
  Print these patterns using nested loops:
  
  Pattern A (right triangle of numbers):
  1
  1 2
  1 2 3
  1 2 3 4
  1 2 3 4 5
  
  Pattern B (square):
  * * * * *
  * * * * *
  * * * * *
  * * * * *
  * * * * *
  
  Pattern C (pyramid):
      *
     ***
    *****
   *******
  *********
  
  Pattern D (diamond):
  Print the pyramid then a mirrored version below it.

EXERCISE 2: Number Analysis
  Create NumberAnalysis.java
  Given array: {23, 7, -4, 15, 42, -8, 0, 33, -12, 19, 8, -1}
  Using only loops (no Streams API yet):
  - Count positives, negatives, zeros
  - Find sum, average, max, min
  - Find second largest number
  - Reverse the array (swap in place)
  - Check if array is sorted
  - Count how many numbers > average
  Print a full analysis report.

EXERCISE 3: String Processing
  Create StringProcessor.java
  Given a sentence: "the quick brown fox jumps over the lazy dog"
  Using loops:
  - Count total characters (excluding spaces)
  - Count words
  - Count each vowel (a,e,i,o,u) occurrence
  - Find the longest word
  - Capitalize first letter of each word
  - Reverse each word (not the sentence)
  - Check if it's a pangram (contains all 26 letters)
  Print all results.

EXERCISE 4: FizzBuzz (Classic Interview Warm-up)
  Create FizzBuzz.java
  Print numbers 1-100 where:
  - Multiple of 3: print "Fizz"
  - Multiple of 5: print "Buzz"
  - Multiple of both: print "FizzBuzz"
  - Otherwise: print the number
  Then answer:
  - How many Fizz? How many Buzz? How many FizzBuzz?
  Use continue where appropriate.

EXERCISE 5: Loan Amortization Schedule
  Create LoanSchedule.java
  Given: principal=500000 BDT, annual rate=9%, term=12 months
  For each month, calculate and print:
  - Month number
  - Opening balance
  - Monthly payment (use formula or approximate)
  - Interest portion
  - Principal portion
  - Closing balance
  Use a while loop (loop while balance > 0).
  Print a formatted table.
  Push to GitHub: "feat: add loan amortization calculator"
```

---
## Common Mistakes
```text
MISTAKE 1: Off-by-one error (most common loop bug)
  for (int i = 0; i <= arr.length; i++) // should be <, not <=
      arr[i]; // ArrayIndexOutOfBoundsException at last iteration!
  
  Remember: array indices are 0 to length-1
  Use i < arr.length (not <=)

MISTAKE 2: Modifying collection during for-each
  for (String item : list) {
      list.remove(item); // ConcurrentModificationException!
  }
  Fix: use Iterator.remove() or list.removeIf()

MISTAKE 3: Infinite loop — forgot to update variable
  int i = 0;
  while (i < 10) {
      System.out.println(i);
      // forgot i++! → runs forever
  }

MISTAKE 4: Integer division in loop condition/calculation
  for (int i = 0; i < totalItems / pageSize; i++) // might lose last page
  Fix: use Math.ceil() or (totalItems + pageSize - 1) / pageSize

MISTAKE 5: Creating objects inside loop unnecessarily
  for (int i = 0; i < 1000; i++) {
      SimpleDateFormat sdf = new SimpleDateFormat("dd/MM/yyyy"); // BAD
      // Creates 1000 objects!
  }
  Fix: create sdf ONCE before the loop

MISTAKE 6: String += in loops
  String result = "";
  for (int i = 0; i < 10000; i++) {
      result += i; // Creates new String each iteration → O(n²)
  }
  Fix: use StringBuilder.append()

MISTAKE 7: break exits only the INNERMOST loop
  for (int i = 0; i < 3; i++) {
      for (int j = 0; j < 3; j++) {
          if (j == 1) break; // only exits inner loop!
      }
      // outer continues...
  }
  Fix: use labeled break or boolean flag or refactor to method

MISTAKE 8: do-while semicolon forgotten
  do {
      // body
  } while (condition)  // COMPILE ERROR: missing semicolon!
  Fix: } while (condition);

MISTAKE 9: N+1 query problem with loops and database
  for (User user : users) {
      List<Order> orders = db.getOrdersByUser(user.getId()); // N queries!
  }
  Fix: use database JOIN to get all data in ONE query

MISTAKE 10: Using == in loop to compare Strings
  for (String status : statuses) {
      if (status == "ACTIVE") { ... } // unreliable!
  }
  Fix: if ("ACTIVE".equals(status))
```

---
## Interview Questions

**Q: What is the difference between while and do-while?**
A: Both repeat a block while a condition is true. The difference is WHEN the condition is checked. while checks BEFORE the body runs — if the condition is initially false, the body never executes. do-while checks AFTER the body runs — the body ALWAYS executes at least once, then the condition determines whether to repeat. Use do-while when you need to run the body at least once before checking (like showing a menu, then checking if the user wants to exit).

**Q: When would you use a for loop vs a while loop?**
A: Use a for loop when you know the number of iterations upfront — iterating over an array with a known size, counting up/down a fixed number of times, or iterating over a collection. Use a while loop when the number of iterations depends on runtime conditions — retrying until success, processing until a sentinel value, reading until end of file, or running a server loop indefinitely. for-each is preferred over both when simply iterating over all elements of a collection.

**Q: What is the for-each loop and what are its limitations?**
A: The enhanced for loop (for-each) provides clean syntax for iterating all elements of an array or any Iterable collection. It's preferred when you don't need the index. Limitations: 1) You cannot modify the collection while iterating (throws ConcurrentModificationException). 2) You cannot access the current index directly. 3) You cannot iterate in reverse. 4) You cannot skip elements mid-iteration without continue. When you need index access, modification, or reverse iteration, use a regular for loop with Iterator.

**Q: What is the difference between break and continue?**
A: break exits the current loop entirely — execution jumps to the first statement after the loop. continue skips the rest of the current iteration and jumps to the next iteration (for the for loop: the update runs first, then condition check). break is used when you've found what you need and further iterations are unnecessary. continue is used to skip elements that don't meet a condition without exiting the loop entirely.

**Q: What is the N+1 query problem?**
A: A common performance anti-pattern where a loop makes N additional database queries — one for each item in an initial result set of N items. Example: fetching 1000 users then calling database for each user's orders inside the loop = 1001 queries. The fix is to use a JOIN in the database query to fetch all required data in one query, or use batch loading. In Spring Boot with JPA, the N+1 problem is solved with @EntityGraph, JOIN FETCH, or @BatchSize.

**Q: Explain labeled break with an example.**
A: Normally break only exits the innermost loop. A labeled break exits a specific outer loop. You define a label (identifier followed by colon) before the outer loop, then use break labelName to exit it. Example: searching a 2D grid — when you find the target, you want to exit both loops, not just the inner one. Alternative approaches are extracting the nested loops into a method (use return instead of break) or using a boolean flag. Labeled breaks are rare in modern code and often signal a need for refactoring.

---
## Key Takeaways
```text
1. FOUR loop types — choose based on use case:
   for      → known count, index needed, array traversal
   while    → unknown count, condition-driven, retry logic
   do-while → must run at least once (menus, PIN retry)
   for-each → iterating ALL elements of any collection (preferred)

2. for loop anatomy:
   for (init; condition; update) { body }
   init runs ONCE. condition checked BEFORE each iteration.
   update runs AFTER each iteration. All parts optional.

3. for-each is the DEFAULT choice for collections.
   Cleaner, less error-prone, no index management.
   Use regular for only when you need the index.

4. break = EXIT the loop immediately.
   continue = SKIP to next iteration.
   Both only affect the INNERMOST loop by default.
   Labeled break/continue affects a specific outer loop.

5. while(true) with break = common server/consumer pattern.
   Kafka consumers, server accept loops use this.

6. NEVER modify a collection during for-each.
   Use Iterator.remove() or list.removeIf() instead.

7. Performance rules:
   Avoid nested loops on large datasets → O(n²)
   Use StringBuilder in loops, not String +=
   Move invariants (expensive calculations) BEFORE the loop
   Never query database inside a loop → N+1 problem

8. Infinite loops are caused by:
   Forgetting to update the loop variable
   Condition that never becomes false
   break that never triggers

9. Off-by-one errors:
   Arrays: i < arr.length (not i <= arr.length)
   Index starts at 0, last index is length-1.

10. Common backend patterns using loops:
    Accumulator (sum/max/min)
    Filter and collect
    Transform (convert each element)
    Count occurrences (with HashMap)
    Batch processing (process in chunks)
    Retry with backoff
```

---