**Phase:** Level 1 - OOP
**Date Studied:**

---
## What Problem Does This Solve?

```text
Imagine building a payment system with multiple payment types.
Without inheritance:

class CreditCardPayment {
  private String cardNumber;
  private String holderName;
  private double amount;
  private String currency;
  private String status;
  private LocalDateTime createdAt;
  void validate() { /* validation logic */ }
  void process() { /* credit card processing */ }
  void refund() { /* refund logic */ }
  String getReceipt() { /* receipt */ }
}

class BkashPayment {
  private String phoneNumber;
  private double amount; // DUPLICATED
  private String currency; // DUPLICATED
  private String status; // DUPLICATED
  private LocalDateTime createdAt; // DUPLICATED
  void validate() { /* DIFFERENT validation */ }
  void process() { /* bKash processing */ }
  void refund() { /* refund logic — DUPLICATED */ }
  String getReceipt() { /* receipt — DUPLICATED */ }
}

class BankTransferPayment {
  private String bankAccount;
  private double amount; // DUPLICATED again
  private String currency; // DUPLICATED again
  // ... everything duplicated AGAIN
}

Problems:
→ amount, currency, status, createdAt duplicated in every class
→ getReceipt(), refund() logic repeated everywhere
→ Fix a bug in refund()? Update it in 5 places. Miss one → bug.
→ Can't treat all payments the same way in a list.
→ Adding a new payment type? Copy-paste everything again.

With inheritance:

class Payment { // parent — common stuff
  double amount;
  String currency;
  String status;
  LocalDateTime createdAt;
  void refund() { /* once */ }
  String getReceipt() { /* once */ }
}

class CreditCardPayment extends Payment { // child — unique stuff only
  String cardNumber;
  void process() { /* credit card specific */ }
}

class BkashPayment extends Payment { // child — unique stuff only
  String phoneNumber;
  void process() { /* bKash specific */ }
}

Benefits:
→ Common code written ONCE in parent → no duplication
→ Children only add what's UNIQUE to them
→ Fix bug in parent → ALL children get the fix
→ Can treat all payments as Payment objects in a list
→ Adding new payment type? Just extend Payment.
→ Real world mapped to code: a BkashPayment IS-A Payment
```

---

## 1. What Is Inheritance?

```text
Inheritance is a mechanism where a child class (subclass)
INHERITS fields and methods from a parent class (superclass).
The child class:
✅ Gets everything public/protected from the parent
✅ Can ADD new fields and methods
✅ Can OVERRIDE parent methods with different behavior
❌ Does NOT inherit private fields (but they still exist
in the object — just not directly accessible)
❌ Cannot extend multiple classes (Java is single inheritance)

KEY RELATIONSHIP: IS-A
Use inheritance only when the relationship is truly IS-A:
✅ CreditCardPayment IS-A Payment
✅ AdminUser IS-A User
✅ SavingsAccount IS-A BankAccount
✅ Circle IS-A Shape
❌ Car IS-A Engine (no! Car HAS-A Engine → use composition)
❌ OrderService IS-A Order (no! service processes orders)
The IS-A test:
"Can I say '[Child] is a [Parent]' and it makes sense?"
"A Dog is an Animal" → inheritance ✅
"A Dog is a Leash" → composition (Dog HAS-A Leash) ❌

TERMINOLOGY:
Parent class = Superclass = Base class
Child class = Subclass = Derived class
extends = the Java keyword for inheritance
```

---

## 2. The extends Keyword

```java
// File: InheritanceBasics.java
// PARENT CLASS — the common blueprint
class Animal {

    // Fields — will be inherited:
    protected String name; // protected: subclasses can access directly
    protected String species;
    protected int age;
    protected double weight;

    // Constructor:
    public Animal(String name, String species, int age, double weight) {
        this.name = name;
        this.species = species;
        this.age = age;
        this.weight = weight;
    }

    // Methods — will be inherited:
    public void eat() {
        System.out.println(name + " is eating.");
    }

    public void sleep() {
        System.out.println(name + " is sleeping.");
    }

    public void breathe() {
        System.out.println(name + " is breathing.");
    }

    // Method meant to be overridden (but has default behavior):
    public void makeSound() {
        System.out.println(name + " makes a sound.");
    }

    public String getInfo() {
        return String.format("%s (%s), age %d, %.1f kg",
                             name, species, age, weight);
    }

    @Override
    public String toString() {
        return "Animal{name='" + name + "', species='" + species + "'}";
    }
}

// CHILD CLASS — inherits from Animal
class Dog extends Animal {

    // Dog-specific fields (not in Animal):
    private String breed;
    private boolean isVaccinated;

    // Constructor — must call super() first:
    public Dog(String name, int age, double weight,
               String breed, boolean isVaccinated) {
        // Must call parent constructor first!
        // super() must be the FIRST statement
        super(name, "Canis lupus familiaris", age, weight);

        // Then initialize Dog-specific fields:
        this.breed = breed;
        this.isVaccinated = isVaccinated;
    }

    // Dog-specific methods (not in Animal):
    public void fetch() {
        System.out.println(name + " fetches the ball! 🎾");
    }

    public void wagTail() {
        System.out.println(name + " wags its tail! 🐕");
    }

    // OVERRIDE parent method — different behavior for Dog:
    @Override
    public void makeSound() {
        System.out.println(name + " barks: Woof! Woof! 🐶");
    }

    // EXTEND parent method — call parent then add more:
    @Override
    public String getInfo() {
        // Call parent's getInfo() then add Dog-specific info:
        return super.getInfo() + ", Breed: " + breed
               + ", Vaccinated: " + (isVaccinated ? "Yes" : "No");
    }

    public String getBreed() { return breed; }
    public boolean isVaccinated() { return isVaccinated; }
}

// ANOTHER CHILD CLASS:
class Cat extends Animal {

    private boolean isIndoor;
    private int livesRemaining; // cats have 9 lives 😄

    public Cat(String name, int age, double weight, boolean isIndoor) {
        super(name, "Felis catus", age, weight);
        this.isIndoor = isIndoor;
        this.livesRemaining = 9;
    }

    public void purr() {
        System.out.println(name + " purrs: Purrr... 😺");
    }

    public void scratch() {
        System.out.println(name + " scratches the furniture! 😤");
    }

    @Override
    public void makeSound() {
        System.out.println(name + " meows: Meow! 🐱");
    }

    // Override: Cat sleeps differently (yes, more):
    @Override
    public void sleep() {
        System.out.println(name + " is sleeping... again (16 hrs/day). 😴");
    }
}

public class InheritanceBasics {
    public static void main(String[] args) {

        // Create objects:
        Dog dog = new Dog("Rex", 3, 25.5, "German Shepherd", true);
        Cat cat = new Cat("Whiskers", 5, 4.2, true);

        System.out.println("=== Dog ===");
        System.out.println(dog.getInfo()); // includes breed (overridden)
        dog.eat(); // INHERITED from Animal
        dog.breathe(); // INHERITED from Animal
        dog.makeSound(); // OVERRIDDEN — "Woof!"
        dog.sleep(); // INHERITED from Animal
        dog.fetch(); // Dog-specific
        dog.wagTail(); // Dog-specific

        System.out.println("\n=== Cat ===");
        System.out.println(cat.getInfo()); // from Animal (not overridden)
        cat.eat(); // INHERITED
        cat.makeSound(); // OVERRIDDEN — "Meow!"
        cat.sleep(); // OVERRIDDEN — sleeps a lot
        cat.purr(); // Cat-specific

        System.out.println("\n=== Inherited Fields ===");
        // Dog has 'name' field even though it's defined in Animal:
        System.out.println("Dog's name: " + dog.name); // protected — accessible
        System.out.println("Cat's name: " + cat.name); // protected — accessible
        System.out.println("Dog's age: " + dog.age); // protected — accessible

        System.out.println("\n=== Type Checking ===");
        System.out.println("dog instanceof Dog: " + (dog instanceof Dog)); // true
        System.out.println("dog instanceof Animal: " + (dog instanceof Animal)); // true!
        System.out.println("cat instanceof Dog: " + (cat instanceof Dog)); // false
        System.out.println("cat instanceof Animal: " + (cat instanceof Animal)); // true!
        // Every Dog IS-A Animal. Not every Animal is a Dog.
    }
}
```

---

## 3. The super Keyword — Complete Guide

```java
public class SuperKeyword {

    static class Vehicle {

        protected String brand;
        protected String model;
        protected int year;
        protected double fuelLevel; // 0.0 to 1.0

        // Parent constructor:
        public Vehicle(String brand, String model, int year) {
            this.brand = brand;
            this.model = model;
            this.year = year;
            this.fuelLevel = 1.0; // full tank
            System.out.println("Vehicle constructor: " + brand + " " + model);
        }

        public void startEngine() {
            System.out.println(brand + " " + model + ": Engine started. Vroom!");
        }

        public void refuel(double amount) {
            fuelLevel = Math.min(1.0, fuelLevel + amount);
            System.out.printf("%s %s: Refueled. Fuel: %.0f%%%n",
                              brand, model, fuelLevel * 100);
        }

        public String getStatus() {
            return String.format("%s %s (%d) - Fuel: %.0f%%",
                                 brand, model, year, fuelLevel * 100);
        }

        @Override
        public String toString() {
            return "Vehicle{" + brand + " " + model + " " + year + "}";
        }
    }

    static class ElectricCar extends Vehicle {

        private double batteryLevel; // 0.0 to 1.0
        private int rangeKm;
        private boolean isCharging;

        // ─────────────────────────────────────────
        // super() — Call parent constructor
        // MUST be first statement in constructor
        // ─────────────────────────────────────────

        public ElectricCar(String brand, String model, int year,
                           int rangeKm) {
            super(brand, model, year); // MUST be first!
            // Now initialize ElectricCar-specific fields:
            this.batteryLevel = 1.0;
            this.rangeKm = rangeKm;
            this.isCharging = false;
            System.out.println("ElectricCar constructor: range " + rangeKm + "km");
        }

        // ─────────────────────────────────────────
        // super.method() — Call parent method
        // ─────────────────────────────────────────

        @Override
        public void startEngine() {
            // Call parent's version first:
            super.startEngine();
            // Then add electric-specific behavior:
            System.out.println(" ↳ Electric motor engaged. Silent mode.");
            System.out.printf(" ↳ Battery: %.0f%% | Range: ~%.0fkm%n",
                              batteryLevel * 100, rangeKm * batteryLevel);
        }

        // Override but don't use super — completely different behavior:
        @Override
        public void refuel(double amount) {
            // Electric cars don't refuel — they charge!
            // We override to prevent confusion, and redirect:
            System.out.println("Electric cars don't use fuel. Use charge() instead.");
        }

        // New method — no equivalent in parent:
        public void charge(double amount) {
            if (isCharging) {
                System.out.println("Already charging...");
                return;
            }
            isCharging = true;
            batteryLevel = Math.min(1.0, batteryLevel + amount);
            System.out.printf("Charging %s... Battery: %.0f%%%n",
                              brand, batteryLevel * 100);
            isCharging = false;
        }

        // Extend parent's getStatus:
        @Override
        public String getStatus() {
            return super.getStatus() + // call parent version
                   String.format(" | Battery: %.0f%% | Range: ~%.0fkm",
                                 batteryLevel * 100, rangeKm * batteryLevel);
        }

        public double getBatteryLevel() { return batteryLevel; }
        public int getRangeKm() { return rangeKm; }
    }

    static class HybridCar extends ElectricCar {

        private double fuelTankLevel; // additional fuel tank
        private String currentMode; // "ELECTRIC" or "HYBRID"

        // Constructor chains: HybridCar → ElectricCar → Vehicle
        public HybridCar(String brand, String model, int year,
                         int rangeKm) {
            super(brand, model, year, rangeKm); // calls ElectricCar constructor
            this.fuelTankLevel = 1.0;
            this.currentMode = "ELECTRIC";
            System.out.println("HybridCar constructor: dual mode");
        }

        @Override
        public void startEngine() {
            super.startEngine(); // calls ElectricCar's startEngine()
            System.out.println(" ↳ Hybrid system ready. Mode: " + currentMode);
        }

        // Hybrid CAN refuel (unlike pure electric):
        @Override
        public void refuel(double amount) {
            // Note: this calls Vehicle's refuel logic indirectly
            fuelTankLevel = Math.min(1.0, fuelTankLevel + amount);
            System.out.printf("Refueling hybrid... Fuel tank: %.0f%%%n",
                              fuelTankLevel * 100);
        }

        public void switchMode(String mode) {
            if (!mode.equals("ELECTRIC") && !mode.equals("HYBRID")) {
                throw new IllegalArgumentException("Mode must be ELECTRIC or HYBRID");
            }
            this.currentMode = mode;
            System.out.println("Switched to " + mode + " mode.");
        }

        @Override
        public String getStatus() {
            return super.getStatus() + // calls ElectricCar's getStatus()
                   String.format(" | Mode: %s | Fuel: %.0f%%",
                                 currentMode, fuelTankLevel * 100);
        }
    }

    public static void main(String[] args) {

        System.out.println("=== Creating Objects (constructor chain) ===");
        // Constructor execution order: Vehicle → ElectricCar → HybridCar
        HybridCar hybrid = new HybridCar("Toyota", "Prius", 2024, 50);

        System.out.println("\n=== Electric Car ===");
        ElectricCar tesla = new ElectricCar("Tesla", "Model 3", 2024, 570);

        tesla.startEngine(); // super.startEngine() → then electric behavior
        tesla.refuel(0.5); // overridden — tells user to charge instead
        tesla.charge(0.3); // electric-specific
        System.out.println(tesla.getStatus()); // extended from parent

        System.out.println("\n=== Hybrid Car ===");
        hybrid.startEngine();
        hybrid.charge(0.2); // inherited from ElectricCar
        hybrid.refuel(0.3); // overridden in HybridCar
        hybrid.switchMode("HYBRID"); // HybridCar-specific
        System.out.println(hybrid.getStatus()); // calls chain: Hybrid→Electric→Vehicle

        System.out.println("\n=== super chain ===");
        // hybrid.getStatus() calls:
        // HybridCar.getStatus()
        // → super.getStatus() = ElectricCar.getStatus()
        // → super.getStatus() = Vehicle.getStatus()
        // returns "Toyota Prius (2024) - Fuel: 100%"
        // returns "...| Battery: ... | Range: ..."
        // returns "...| Mode: ... | Fuel: ..."

        System.out.println("\n=== instanceof chain ===");
        System.out.println("hybrid instanceof HybridCar: " + (hybrid instanceof HybridCar)); // true
        System.out.println("hybrid instanceof ElectricCar: " + (hybrid instanceof ElectricCar)); // true!
        System.out.println("hybrid instanceof Vehicle: " + (hybrid instanceof Vehicle)); // true!
        System.out.println("hybrid instanceof Object: " + (hybrid instanceof Object)); // true!
    }
}
```

---

## 4. Method Overriding — The Rules

```java
public class MethodOverriding {

    static class Shape {
        protected String color;
        protected boolean filled;

        public Shape(String color, boolean filled) {
            this.color = color;
            this.filled = filled;
        }

        // These will be overridden:
        public double getArea() {
            return 0.0; // default — subclasses should override
        }

        public double getPerimeter() {
            return 0.0; // default — subclasses should override
        }

        public String describe() {
            return String.format("A %s %s shape", filled ? "filled" : "hollow", color);
        }

        // This will NOT be overridden — final method:
        public final String getColor() {
            return color; // cannot be overridden
        }

        @Override
        public String toString() {
            return String.format("Shape{color='%s', filled=%b, area=%.2f}",
                                 color, filled, getArea());
        }
    }

    // ─────────────────────────────────────────
    // OVERRIDING RULES (all must be followed):
    // ─────────────────────────────────────────

    static class Circle extends Shape {
        private double radius;

        public Circle(String color, boolean filled, double radius) {
            super(color, filled);
            this.radius = radius;
        }

        // RULE 1: Same method name
        // RULE 2: Same parameter list (no overloading here)
        // RULE 3: Return type must be same OR a subtype (covariant return)
        // RULE 4: Cannot reduce visibility (public stays public)
        // RULE 5: @Override annotation — catches errors at compile time

        @Override
        public double getArea() {
            return Math.PI * radius * radius;
        }

        @Override
        public double getPerimeter() {
            return 2 * Math.PI * radius;
        }

        @Override
        public String describe() {
            return super.describe() + String.format(" (Circle, radius=%.2f)", radius);
        }

        // ❌ CANNOT override getColor() — it's final in Shape:
        // @Override
        // public String getColor() { return "OVERRIDE"; } // COMPILE ERROR

        // ❌ CANNOT reduce access:
        // @Override
        // private double getArea() { ... } // COMPILE ERROR: can't make it private

        public double getRadius() { return radius; }
    }

    static class Rectangle extends Shape {
        private double width, height;

        public Rectangle(String color, boolean filled, double width, double height) {
            super(color, filled);
            this.width = width;
            this.height = height;
        }

        @Override
        public double getArea() {
            return width * height;
        }

        @Override
        public double getPerimeter() {
            return 2 * (width + height);
        }

        @Override
        public String describe() {
            return super.describe() +
                   String.format(" (Rectangle, %.2f×%.2f)", width, height);
        }
    }

    static class Square extends Rectangle {
        // Square IS-A Rectangle!

        public Square(String color, boolean filled, double side) {
            super(color, filled, side, side);
        }

        // Square can still override Rectangle's methods if needed,
        // or inherit them (which is fine since Rectangle's calculations work)

        @Override
        public String describe() {
            return "Square with side " +
                   Math.sqrt(getArea()); // use parent's getArea()
        }
    }

    // ─────────────────────────────────────────
    // @Override ANNOTATION — CRITICAL HABIT
    // ─────────────────────────────────────────

    static class WithoutAnnotation extends Shape {
        public WithoutAnnotation() { super("red", true); }

        // TYPO — this creates a NEW method, NOT an override!
        // Without @Override, compiler doesn't catch this:
        public double getarea() { // lowercase 'a' — NOT overriding!
            return 999; // never called polymorphically as getArea()
        }
    }

    static class WithAnnotation extends Shape {
        public WithAnnotation() { super("blue", true); }

        // @Override
        // public double getarea() { // COMPILE ERROR caught! "getarea" doesn't exist in Shape
        //     return 999;
        // }
        // Error: method does not override or implement a method from a supertype
        // This saved you from a silent bug!
    }

    public static void main(String[] args) {

        Circle circle = new Circle("red", true, 5.0);
        Rectangle rect = new Rectangle("blue", false, 4.0, 6.0);
        Square square = new Square("green", true, 3.0);

        System.out.println("=== Each Shape Calculates Its Own Area ===");
        System.out.printf("Circle area: %.2f%n", circle.getArea()); // 78.54
        System.out.printf("Rectangle area: %.2f%n", rect.getArea()); // 24.00
        System.out.printf("Square area: %.2f%n", square.getArea()); // 9.00

        System.out.printf("%nCircle perimeter: %.2f%n", circle.getPerimeter()); // 31.42
        System.out.printf("Rectangle perimeter: %.2f%n", rect.getPerimeter()); // 20.00

        System.out.println("\n=== describe() uses super.describe() ===");
        System.out.println(circle.describe());
        System.out.println(rect.describe());

        System.out.println("\n=== toString() uses overridden getArea() ===");
        System.out.println(circle); // calls overridden getArea() in toString!
        System.out.println(rect);

        System.out.println("\n=== final method cannot be overridden ===");
        System.out.println(circle.getColor()); // "red" — cannot be overridden
    }
}
```

---

## 5. Constructor Chaining in Inheritance

```java
public class ConstructorChaining {

    // Understanding the ORDER constructors execute in
    // is critical for debugging and design

    static class GrandParent {
        int a;

        public GrandParent() {
            this.a = 1;
            System.out.println("1. GrandParent() constructor runs. a=" + a);
        }

        public GrandParent(int a) {
            this.a = a;
            System.out.println("1. GrandParent(int) constructor runs. a=" + a);
        }
    }

    static class Parent extends GrandParent {
        int b;

        public Parent() {
            super(); // calls GrandParent() — implicit if not written
            this.b = 2;
            System.out.println("2. Parent() constructor runs. b=" + b);
        }

        public Parent(int a, int b) {
            super(a); // calls GrandParent(int)
            this.b = b;
            System.out.println("2. Parent(int, int) constructor runs. b=" + b);
        }
    }

    static class Child extends Parent {
        int c;

        public Child() {
            super(); // calls Parent() — which calls GrandParent()
            this.c = 3;
            System.out.println("3. Child() constructor runs. c=" + c);
        }

        public Child(int a, int b, int c) {
            super(a, b); // calls Parent(int, int) — which calls GrandParent(int)
            this.c = c;
            System.out.println("3. Child(int, int, int) constructor runs. c=" + c);
        }
    }

    // KEY RULES:
    // 1. Every constructor must call super() — either explicitly or implicitly
    // 2. If you don't write super(), Java adds super() automatically
    // (only works if parent has a no-arg constructor)
    // 3. If parent has NO no-arg constructor, child MUST explicitly call
    // super(args) with the right arguments
    // 4. Constructor runs top-down: grandparent → parent → child
    // 5. Instance initializers run after super() but before the rest of constructor

    static class NoArgRequired {
        String name;

        // No no-arg constructor:
        public NoArgRequired(String name) {
            this.name = name;
        }
    }

    static class ChildOfNoArg extends NoArgRequired {
        int value;

        public ChildOfNoArg(String name, int value) {
            super(name); // REQUIRED: parent has no no-arg constructor!
            // Without super(name): COMPILE ERROR
            this.value = value;
        }

        // ❌ This would NOT compile:
        // public ChildOfNoArg(int value) {
        //     this.value = value;
        //     // Java tries: super() — but NoArgRequired has no super()!
        //     // COMPILE ERROR: constructor NoArgRequired() not defined
        // }
    }

    public static void main(String[] args) {

        System.out.println("=== Child() — default chain ===");
        Child c1 = new Child();
        // Output:
        // 1. GrandParent() constructor runs. a=1
        // 2. Parent() constructor runs. b=2
        // 3. Child() constructor runs. c=3
        // TOP-DOWN execution: grandparent first, child last

        System.out.println("\n=== Child(int,int,int) — parameterized chain ===");
        Child c2 = new Child(10, 20, 30);
        // Output:
        // 1. GrandParent(int) constructor runs. a=10
        // 2. Parent(int, int) constructor runs. b=20
        // 3. Child(int, int, int) constructor runs. c=30

        System.out.println("\n=== After construction ===");
        System.out.println("c1: a=" + c1.a + " b=" + c1.b + " c=" + c1.c); // 1 2 3
        System.out.println("c2: a=" + c2.a + " b=" + c2.b + " c=" + c2.c); // 10 20 30

        System.out.println("\n=== NoArg required ===");
        ChildOfNoArg child = new ChildOfNoArg("test", 42);
        System.out.println(child.name + " " + child.value);
    }
}
```

---

## 6. Upcasting and Downcasting

```java
public class CastingDemo {

    static class Employee {
        protected String name;
        protected String department;
        protected double salary;

        public Employee(String name, String department, double salary) {
            this.name = name;
            this.department = department;
            this.salary = salary;
        }

        public void work() {
            System.out.println(name + " is working in " + department);
        }

        public String getSummary() {
            return String.format("%s (%s) - ৳%.0f", name, department, salary);
        }
    }

    static class Manager extends Employee {
        private int teamSize;

        public Manager(String name, String department,
                       double salary, int teamSize) {
            super(name, department, salary);
            this.teamSize = teamSize;
        }

        @Override
        public void work() {
            System.out.println(name + " is managing a team of " + teamSize);
        }

        public void conductReview() {
            System.out.println(name + " is conducting performance reviews.");
        }

        public int getTeamSize() { return teamSize; }
    }

    static class Developer extends Employee {
        private String programmingLanguage;

        public Developer(String name, String department,
                         double salary, String language) {
            super(name, department, salary);
            this.programmingLanguage = language;
        }

        @Override
        public void work() {
            System.out.println(name + " is writing " + programmingLanguage + " code.");
        }

        public void deployCode() {
            System.out.println(name + " is deploying to production! 🚀");
        }

        public String getLanguage() { return programmingLanguage; }
    }

    public static void main(String[] args) {

        // ─────────────────────────────────────────
        // UPCASTING — Implicit, automatic, always safe
        // Child reference → Parent type
        // ─────────────────────────────────────────

        Manager mgr = new Manager("Rahim", "Engineering", 120_000, 8);
        Developer dev = new Developer("Karim", "Engineering", 90_000, "Java");

        // Upcasting: Manager → Employee (child → parent)
        Employee emp1 = mgr; // implicit upcast — no cast syntax needed
        Employee emp2 = dev; // implicit upcast

        // emp1 and mgr point to the SAME object (the Manager)
        System.out.println("Same object? " + (emp1 == mgr)); // true

        // Through the Employee reference, we can only call Employee methods:
        emp1.work(); // calls Manager's OVERRIDDEN work()! (polymorphism!)
        emp1.getSummary(); // Employee's method
        // emp1.conductReview(); // COMPILE ERROR: Employee doesn't have this
        // emp1.getTeamSize(); // COMPILE ERROR: Employee doesn't have this

        // ─────────────────────────────────────────
        // DOWNCASTING — Explicit, risky, requires instanceof check
        // Parent reference → Child type (to access child-specific methods)
        // ─────────────────────────────────────────

        Employee empRef = new Manager("Hasan", "Finance", 100_000, 5);

        // Must cast explicitly to access Manager-specific methods:
        Manager manager = (Manager) empRef; // explicit downcast
        manager.conductReview(); // now accessible!
        System.out.println("Team size: " + manager.getTeamSize()); // 5

        // DANGEROUS: casting to WRONG type → ClassCastException at runtime!
        Employee anotherEmp = new Developer("Nabil", "IT", 85_000, "Python");
        try {
            Manager badCast = (Manager) anotherEmp; // Looks OK to compiler
            badCast.conductReview(); // ClassCastException at runtime!
        } catch (ClassCastException e) {
            System.out.println("ClassCastException: " + e.getMessage());
        }

        // ─────────────────────────────────────────
        // SAFE DOWNCASTING — Always use instanceof first
        // ─────────────────────────────────────────

        Employee[] team = {
            new Manager("Alice", "Engineering", 130_000, 10),
            new Developer("Bob", "Backend", 95_000, "Java"),
            new Developer("Charlie", "Frontend", 88_000, "TypeScript"),
            new Manager("Diana", "Product", 120_000, 6),
            new Employee("Eve", "HR", 75_000)
        };

        System.out.println("\n=== Processing Mixed Team ===");
        for (Employee e : team) {
            e.work(); // polymorphism — each works differently!

            // Safe downcast with instanceof:
            if (e instanceof Manager m) { // Pattern matching (Java 16+)
                m.conductReview();
                System.out.println(" → Team size: " + m.getTeamSize());
            } else if (e instanceof Developer d) {
                d.deployCode();
                System.out.println(" → Language: " + d.getLanguage());
            }
            // else: just a plain Employee — no special behavior
        }

        // ─────────────────────────────────────────
        // WHY UPCASTING IS POWERFUL
        // ─────────────────────────────────────────

        System.out.println("\n=== Polymorphic Processing ===");

        // Store different types in ONE list using parent type:
        java.util.List<Employee> employees = new java.util.ArrayList<>();
        employees.add(new Manager("Mgr1", "Eng", 120_000, 8));
        employees.add(new Developer("Dev1", "Eng", 90_000, "Java"));
        employees.add(new Developer("Dev2", "Eng", 85_000, "Python"));
        employees.add(new Manager("Mgr2", "Product", 110_000, 5));

        // Process ALL employees uniformly:
        double totalSalary = 0;
        for (Employee emp : employees) {
            emp.work(); // each type works differently!
            totalSalary += emp.salary;
        }
        System.out.printf("Total salary bill: ৳%.0f%n", totalSalary);
    }
}
```

---

## 7. The final Keyword with Inheritance

```java
public class FinalKeyword {

    // 'final' has three uses related to inheritance:

    // ─────────────────────────────────────────
    // 1. final CLASS — cannot be extended (inherited from)
    // ─────────────────────────────────────────

    public final class ImmutableConfig {
        private final String databaseUrl;
        private final int maxConnections;

        public ImmutableConfig(String url, int maxConn) {
            this.databaseUrl = url;
            this.maxConnections = maxConn;
        }

        public String getDatabaseUrl() { return databaseUrl; }
        public int getMaxConnections() { return maxConnections; }
    }

    // ❌ This would COMPILE ERROR:
    // class ExtendedConfig extends ImmutableConfig { } // cannot extend final class

    // Why make a class final?
    // → Security: String is final — no one can extend String and
    //   override its methods to create security holes
    // → Immutability guarantee: final class + final fields = truly immutable
    // → API stability: signals "this is not designed to be extended"

    // ─────────────────────────────────────────
    // 2. final METHOD — cannot be overridden
    // ─────────────────────────────────────────

    static class PaymentProcessor {
        private static final double PROCESSING_FEE_RATE = 0.02;

        // Template method — defines the algorithm skeleton:
        public final void processPayment(double amount) {
            // Cannot override this — the sequence is fixed:
            validateAmount(amount);
            double fee = calculateFee(amount); // can override
            double total = amount + fee;
            deductFromAccount(total); // can override
            generateReceipt(amount, fee, total); // can override
            notifyUser(); // can override
            System.out.println("Payment processed: ৳" + amount);
        }

        // These CAN be overridden:
        protected void validateAmount(double amount) {
            if (amount <= 0)
                throw new IllegalArgumentException("Amount must be positive");
        }

        protected double calculateFee(double amount) {
            return amount * PROCESSING_FEE_RATE;
        }

        protected void deductFromAccount(double total) {
            System.out.println("Deducting ৳" + total + " from account");
        }

        protected void generateReceipt(double amount, double fee, double total) {
            System.out.printf("Receipt: amount=৳%.2f fee=৳%.2f total=৳%.2f%n",
                              amount, fee, total);
        }

        protected void notifyUser() {
            System.out.println("SMS notification sent.");
        }
    }

    static class BkashProcessor extends PaymentProcessor {

        // ❌ Cannot override processPayment — it's final:
        // @Override
        // public void processPayment(double amount) { } // COMPILE ERROR

        // ✅ CAN override the non-final methods:
        @Override
        protected double calculateFee(double amount) {
            return amount * 0.015; // bKash charges 1.5% not 2%
        }

        @Override
        protected void notifyUser() {
            System.out.println("bKash notification sent to mobile.");
        }
    }

    // ─────────────────────────────────────────
    // 3. final FIELD — cannot be reassigned after initialization
    // ─────────────────────────────────────────

    static class OrderId {
        private final String id; // set once — never changes

        public OrderId(String prefix) {
            this.id = prefix + "-" + System.currentTimeMillis();
        }

        public String getId() { return id; }
        // No setId() — no need, it's final
    }

    public static void main(String[] args) {

        System.out.println("=== final method (template method) ===");
        PaymentProcessor standard = new PaymentProcessor();
        standard.processPayment(1000.0);

        System.out.println();

        BkashProcessor bkash = new BkashProcessor();
        bkash.processPayment(1000.0); // same processPayment() — different fee and notification

        System.out.println("\n=== final field ===");
        OrderId order = new OrderId("ORD");
        System.out.println("Order ID: " + order.getId());
        // order.id = "HACK"; // COMPILE ERROR: final field
    }
}
```

---

## 8. Inheritance vs Composition — When to Choose

```java
public class InheritanceVsComposition {

    // ─────────────────────────────────────────
    // THE IS-A vs HAS-A QUESTION
    //
    // Inheritance: "IS-A" relationship
    // Dog IS-A Animal → Dog extends Animal
    //
    // Composition: "HAS-A" relationship
    // Car HAS-A Engine → Car contains Engine as a field
    // ─────────────────────────────────────────

    // ─────────────────────────────────────────
    // COMPOSITION EXAMPLE — Use this more often than inheritance
    // ─────────────────────────────────────────

    // Engine is NOT a type of Car — Car HAS-A Engine
    static class Engine {
        private String type;
        private int horsepower;
        private boolean isRunning;

        public Engine(String type, int horsepower) {
            this.type = type;
            this.horsepower = horsepower;
            this.isRunning = false;
        }

        public void start() {
            isRunning = true;
            System.out.println(type + " engine started. " + horsepower + "hp");
        }

        public void stop() {
            isRunning = false;
            System.out.println(type + " engine stopped.");
        }

        public boolean isRunning() { return isRunning; }
        public int getHorsepower() { return horsepower; }
    }

    static class GPS {
        private String destination;

        public void setDestination(String destination) {
            this.destination = destination;
            System.out.println("GPS: Navigating to " + destination);
        }

        public String getDestination() { return destination; }
    }

    // Car uses composition: HAS-A Engine, HAS-A GPS
    static class Car {
        private String brand;
        private String model;
        private Engine engine; // composition — contains Engine
        private GPS gps; // composition — contains GPS

        public Car(String brand, String model, String engineType, int hp) {
            this.brand = brand;
            this.model = model;
            this.engine = new Engine(engineType, hp); // creates its own engine
            this.gps = new GPS();
        }

        // Delegate to engine:
        public void startCar() {
            engine.start();
            System.out.println(brand + " " + model + " ready to drive!");
        }

        public void stopCar() {
            engine.stop();
        }

        // Delegate to GPS:
        public void navigateTo(String destination) {
            gps.setDestination(destination);
        }

        public int getHorsepower() { return engine.getHorsepower(); }
    }

    // ─────────────────────────────────────────
    // WRONG USE OF INHERITANCE (AVOID):
    // ─────────────────────────────────────────

    // ❌ WRONG: Stack should NOT extend ArrayList
    // "A Stack IS-A ArrayList" — this makes push/pop available to internal list too
    // Users can call add(0, item) which bypasses stack behavior!
    // Java's original Stack class made this mistake — it's considered a design flaw

    // ❌ WRONG: Engine should NOT extend Car
    // An Engine is NOT a Car — composition is right

    // ─────────────────────────────────────────
    // FAVOR COMPOSITION OVER INHERITANCE
    // (effective Java Item 18 — famous advice)
    // ─────────────────────────────────────────

    // When to use INHERITANCE:
    // ✅ True IS-A relationship (Dog is Animal, AdminUser is User)
    // ✅ You control both parent and child (not extending library classes)
    // ✅ You need polymorphism (treat all subtypes as parent type)
    // ✅ Child genuinely is a specialization of parent

    // When to use COMPOSITION:
    // ✅ HAS-A relationship (Car has Engine)
    // ✅ You want to reuse behavior without subtyping
    // ✅ The parent class might change independently
    // ✅ You need flexibility to swap implementations
    // ✅ You want to avoid inheriting methods you don't want

    public static void main(String[] args) {
        Car myCar = new Car("Toyota", "Camry", "V6", 270);

        myCar.startCar();
        myCar.navigateTo("Chittagong");
        System.out.println("Horsepower: " + myCar.getHorsepower());
        myCar.stopCar();

        // Car doesn't expose Engine internals unnecessarily
        // Car delegates to Engine and GPS — clean separation
    }
}
```

---

## Build This — Complete Inheritance Practice

```java
// File: PaymentSystem.java
// A payment system using inheritance correctly
// Demonstrates all inheritance concepts
import java.time.LocalDateTime;
import java.util.*;

public class PaymentSystem {
    // ═══════════════════════════════════════
    // BASE CLASS — Common payment behavior
    // ═══════════════════════════════════════

    static abstract class Payment {
        // abstract means: subclasses MUST implement specific methods
        // (We'll fully cover abstract in Level 1.14)

        private static int counter = 1;

        private final String paymentId;
        private final double amount;
        private final String currency;
        private String status;
        private final LocalDateTime createdAt;
        private LocalDateTime processedAt;
        private String failureReason;

        protected static final double MIN_AMOUNT = 10.0;
        protected static final double MAX_AMOUNT = 1_000_000.0;

        // Constructor — validates common fields:
        public Payment(double amount, String currency) {
            if (amount < MIN_AMOUNT || amount > MAX_AMOUNT)
                throw new IllegalArgumentException(
                    "Amount must be between " + MIN_AMOUNT + " and " + MAX_AMOUNT);
            if (currency == null || currency.isBlank())
                throw new IllegalArgumentException("Currency required");

            this.paymentId = "PAY-" + String.format("%06d", counter++);
            this.amount = amount;
            this.currency = currency.toUpperCase();
            this.status = "PENDING";
            this.createdAt = LocalDateTime.now();
        }

        // Template method — defines the algorithm:
        public final boolean process() {
            if (!"PENDING".equals(status)) {
                System.out.println("❌ " + paymentId + ": Already " + status);
                return false;
            }

            System.out.println("\n[" + paymentId + "] Processing " +
                               getPaymentType() + " payment of " + formatAmount());

            // Step 1: Validate (subclass implements)
            if (!validatePaymentDetails()) {
                status = "FAILED";
                failureReason = "Validation failed";
                System.out.println(" ❌ Validation failed");
                return false;
            }
            System.out.println(" ✅ Validated");

            // Step 2: Execute (subclass implements)
            boolean success = executePayment();

            if (success) {
                status = "COMPLETED";
                processedAt = LocalDateTime.now();
                System.out.println(" ✅ Payment completed: " + formatAmount());
                onSuccess();
            } else {
                status = "FAILED";
                failureReason = getFailureReason();
                System.out.println(" ❌ Payment failed: " + failureReason);
            }

            return success;
        }

        public boolean refund() {
            if (!"COMPLETED".equals(status)) {
                System.out.println("❌ Can only refund COMPLETED payments");
                return false;
            }
            status = "REFUNDED";
            System.out.println("✅ Refunded: " + formatAmount());
            return true;
        }

        // Methods subclasses MUST override:
        protected abstract String getPaymentType();
        protected abstract boolean validatePaymentDetails();
        protected abstract boolean executePayment();

        // Methods subclasses CAN override:
        protected String getFailureReason() { return "Payment execution failed"; }
        protected void onSuccess() { } // hook — subclasses can add behavior

        // Common getters:
        public String getPaymentId() { return paymentId; }
        public double getAmount() { return amount; }
        public String getCurrency() { return currency; }
        public String getStatus() { return status; }
        public LocalDateTime getCreatedAt() { return createdAt; }

        // Computed:
        public String formatAmount() {
            return String.format("%s %.2f", currency, amount);
        }

        public boolean isCompleted() { return "COMPLETED".equals(status); }
        public boolean isFailed() { return "FAILED".equals(status); }

        @Override
        public String toString() {
            return String.format("[%s] %s %s %s → %s",
                                 paymentId, getPaymentType(),
                                 formatAmount(), status,
                                 failureReason != null ? "(" + failureReason + ")" : "");
        }
    }
    // ═══════════════════════════════════════
    // CREDIT CARD PAYMENT
    // ═══════════════════════════════════════

    static class CreditCardPayment extends Payment {

        private final String cardNumber; // masked after validation
        private final String holderName;
        private final String expiryMonth;
        private final String expiryYear;
        private final String cvv; // should be cleared after use
        private String maskedCardNumber;

        public CreditCardPayment(double amount, String currency,
                                 String cardNumber, String holderName,
                                 String expiryMonth, String expiryYear,
                                 String cvv) {
            super(amount, currency);
            this.cardNumber = cardNumber;
            this.holderName = holderName;
            this.expiryMonth = expiryMonth;
            this.expiryYear = expiryYear;
            this.cvv = cvv;
        }

        @Override
        protected String getPaymentType() { return "CREDIT_CARD"; }

        @Override
        protected boolean validatePaymentDetails() {
            if (cardNumber == null || !cardNumber.matches("\\d{16}")) {
                return false;
            }
            if (holderName == null || holderName.isBlank()) return false;
            if (cvv == null || !cvv.matches("\\d{3,4}")) return false;

            int month = Integer.parseInt(expiryMonth);
            int year = Integer.parseInt(expiryYear);
            if (month < 1 || month > 12) return false;
            if (year < 2024) return false; // expired

            // Mask card number after validation:
            maskedCardNumber = "****-****-****-" +
                               cardNumber.substring(cardNumber.length() - 4);
            return true;
        }

        @Override
        protected boolean executePayment() {
            // Simulate: 90% success rate
            return Math.random() > 0.1;
        }

        @Override
        protected void onSuccess() {
            System.out.println(" 📧 Receipt sent for card: " + maskedCardNumber);
        }

        public String getMaskedCardNumber() { return maskedCardNumber; }
        public String getHolderName() { return holderName; }
    }
    // ═══════════════════════════════════════
    // BKASH PAYMENT
    // ═══════════════════════════════════════

    static class BkashPayment extends Payment {

        private final String phoneNumber;
        private final String pin; // should be cleared
        private static final double BKASH_MAX = 25_000.0;
        private static final double FEE_RATE = 0.015;

        public BkashPayment(double amount, String phoneNumber, String pin) {
            super(amount, "BDT");
            if (amount > BKASH_MAX)
                throw new IllegalArgumentException(
                    "bKash max is ৳" + BKASH_MAX);
            this.phoneNumber = phoneNumber;
            this.pin = pin;
        }

        @Override
        protected String getPaymentType() { return "BKASH"; }

        @Override
        protected boolean validatePaymentDetails() {
            if (phoneNumber == null) return false;
            String cleaned = phoneNumber.replaceAll("[\\s\\-]", "");
            if (!cleaned.matches("(\\+88)?01[3-9]\\d{8}")) return false;
            if (pin == null || !pin.matches("\\d{5}")) return false; // bKash PIN is 5 digits
            return true;
        }

        @Override
        protected boolean executePayment() {
            double fee = getAmount() * FEE_RATE;
            double total = getAmount() + fee;
            System.out.printf(" 📱 bKash fee: ৳%.2f | Total deducted: ৳%.2f%n",
                              fee, total);
            return Math.random() > 0.05; // 95% success
        }

        @Override
        protected void onSuccess() {
            String masked = phoneNumber.substring(0, 3) +
                            "****" +
                            phoneNumber.substring(phoneNumber.length() - 4);
            System.out.println(" 📲 bKash notification sent to: " + masked);
        }

        @Override
        protected String getFailureReason() {
            return "bKash transaction declined";
        }

        public double getFeeAmount() { return getAmount() * FEE_RATE; }
    }
    // ═══════════════════════════════════════
    // BANK TRANSFER PAYMENT
    // ═══════════════════════════════════════

    static class BankTransferPayment extends Payment {

        private final String accountNumber;
        private final String bankName;
        private final String routingNumber;
        private static final int PROCESSING_DAYS = 3;

        public BankTransferPayment(double amount, String currency,
                                   String accountNumber, String bankName,
                                   String routingNumber) {
            super(amount, currency);
            this.accountNumber = accountNumber;
            this.bankName = bankName;
            this.routingNumber = routingNumber;
        }

        @Override
        protected String getPaymentType() { return "BANK_TRANSFER"; }

        @Override
        protected boolean validatePaymentDetails() {
            if (accountNumber == null || accountNumber.length() < 8) return false;
            if (bankName == null || bankName.isBlank()) return false;
            if (routingNumber == null || !routingNumber.matches("\\d{9}")) return false;
            return true;
        }

        @Override
        protected boolean executePayment() {
            System.out.printf(" 🏦 Bank transfer initiated to %s. " +
                              "ETA: %d business days.%n",
                              bankName, PROCESSING_DAYS);
            return Math.random() > 0.02; // 98% success
        }

        @Override
        protected void onSuccess() {
            System.out.println(" 📄 Wire transfer confirmation sent.");
        }

        public String getBankName() { return bankName; }
        public int getProcessingDays() { return PROCESSING_DAYS; }
    }
    // ═══════════════════════════════════════
    // PAYMENT PROCESSOR (uses polymorphism)
    // ═══════════════════════════════════════

    static class PaymentGateway {
        private final List<Payment> transactions = new ArrayList<>();

        public boolean process(Payment payment) {
            boolean result = payment.process(); // polymorphic!
            transactions.add(payment);
            return result;
        }

        public void printReport() {
            System.out.println("\n╔═══════════════════════════════════════════╗");
            System.out.println("║         PAYMENT GATEWAY REPORT            ║");
            System.out.println("╠═══════════════════════════════════════════╣");

            long completed = transactions.stream().filter(Payment::isCompleted).count();
            long failed = transactions.stream().filter(Payment::isFailed).count();
            double revenue = transactions.stream()
                .filter(Payment::isCompleted)
                .mapToDouble(Payment::getAmount)
                .sum();

            System.out.printf("║ Total Transactions: %-20d║%n", transactions.size());
            System.out.printf("║ Completed : %-20d║%n", completed);
            System.out.printf("║ Failed : %-20d║%n", failed);
            System.out.printf("║ Total Revenue : BDT %-16.2f║%n", revenue);
            System.out.println("╠═══════════════════════════════════════════╣");

            for (Payment p : transactions) {
                System.out.printf("║ %-41s║%n",
                                  p.toString().length() > 41
                                  ? p.toString().substring(0, 38) + "..."
                                  : p.toString());
            }
            System.out.println("╚═══════════════════════════════════════════╝");
        }
    }
    // ═══════════════════════════════════════
    // MAIN
    // ═══════════════════════════════════════

    public static void main(String[] args) {

        PaymentGateway gateway = new PaymentGateway();

        // Credit card payments:
        gateway.process(new CreditCardPayment(
            1500.0, "BDT",
            "4532015112830366", "Rahim Ahmed",
            "12", "2025", "123"));

        gateway.process(new CreditCardPayment(
            500.0, "BDT",
            "1234567890123456", "Karim Hassan",
            "06", "2026", "456"));

        // bKash payments:
        gateway.process(new BkashPayment(
            2000.0, "01712345678", "12345"));

        gateway.process(new BkashPayment(
            800.0, "+8801812345678", "98765"));

        // Bank transfer:
        gateway.process(new BankTransferPayment(
            50000.0, "BDT",
            "1234567890", "Dutch-Bangla Bank",
            "123456789"));

        // Invalid payment:
        try {
            gateway.process(new BkashPayment(
                30000.0, "01712345678", "12345")); // exceeds bKash limit
        } catch (IllegalArgumentException e) {
            System.out.println("\n❌ Rejected: " + e.getMessage());
        }

        // Refund a completed payment:
        System.out.println("\n=== Refund Demo ===");
        Payment p = new CreditCardPayment(
            300.0, "BDT",
            "4532015112830366", "Hasan Ali",
            "08", "2025", "789");
        gateway.process(p);
        if (p.isCompleted()) {
            p.refund();
        }

        // Report — processes ALL payments polymorphically:
        gateway.printReport();
    }
}
```

---

## Exercises

```text
EXERCISE 1: Shape Hierarchy
  Create ShapeHierarchy.java
  Design this inheritance tree:

  Shape (parent)
  → fields: color, filled
  → abstract methods: getArea(), getPerimeter()
  → concrete: describe(), toString()

  Circle extends Shape
  → radius
  → getArea(), getPerimeter()
  → getDiameter()

  Rectangle extends Shape
  → width, height
  → getArea(), getPerimeter()
  → isSquare()

  Triangle extends Shape
  → a, b, c (three sides)
  → getArea() using Heron's formula
  → getPerimeter()
  → getType() → Equilateral/Isosceles/Scalene

  RightTriangle extends Triangle
  → hypotenuse, leg1, leg2
  → Constructor validates Pythagorean theorem
  → Override getArea() using ½ * leg1 * leg2

  Test: create array of all shapes, compute total area.

EXERCISE 2: Employee Hierarchy
  Create EmployeeHierarchy.java

  Employee (parent)
  → id (auto), name, department, baseSalary, hireDate
  → abstract calculateMonthlyPay()
  → concrete: getYearsOfService(), getSummary(), promote(double increment)

  FullTimeEmployee extends Employee
  → healthInsurance (monthly cost), paidLeaveDays
  → calculateMonthlyPay() = baseSalary - healthInsurance
  → takeLeave(int days)

  PartTimeEmployee extends Employee
  → hoursPerWeek, hourlyRate
  → calculateMonthlyPay() = hoursPerWeek * hourlyRate * 4.33
  → No health insurance

  Contractor extends Employee
  → contractEndDate, dailyRate
  → calculateMonthlyPay() = dailyRate * 22 (working days)
  → isContractExpired()

  Manager extends FullTimeEmployee
  → teamSize, bonus (percentage of baseSalary)
  → calculateMonthlyPay() = super.calculateMonthlyPay() + bonus
  → addTeamMember(), removeTeamMember()

  Test: list of all employees, total monthly payroll.

EXERCISE 3: Vehicle Hierarchy
  Create VehicleFleet.java

  Vehicle → Car, Truck, Motorcycle, ElectricVehicle
  ElectricVehicle → ElectricCar (extends both Vehicle and ElectricVehicle concepts)

  Each has: fuelType, engineSize OR batteryCapacity
  Common: startEngine(), stopEngine(), calculateFuelCost(double distance)
  Override appropriately.
  Fleet: list of all vehicles, total fuel cost for 500km.

EXERCISE 4: Constructor Chain Deep Dive
  Create ConstructorDeepDive.java
  Build a 4-level hierarchy:
  A → B extends A → C extends B → D extends C
  Each level has different constructors.
  Add print statements in every constructor.
  Create objects using different constructors.
  Draw/print the execution order for each creation.
  Then add: each level has a field.
  Show that D has access to A's protected fields.

EXERCISE 5: Fix the Design
  Create DesignFix.java
  Given this BAD inheritance:

  class Stack extends ArrayList {
    public void push(Integer item) { add(item); }
    public Integer pop() { return remove(size()-1); }
    public Integer peek() { return get(size()-1); }
  }
  // Problem: Stack inherits add(0, item), addAll(), etc.
  // Users can bypass push() and add to middle of stack!

  Refactor using COMPOSITION:
  Stack USES ArrayList internally.
  Only expose: push(), pop(), peek(), isEmpty(), size()
  Test that users CANNOT bypass the stack contract.
  Push to GitHub: "feat: payment system with inheritance"
```

---

## Common Mistakes

```text
MISTAKE 1: Using inheritance for IS-A relationships that aren't really IS-A
  class UserService extends User { } // Service IS-A User? No!
  class OrderController extends Order { } // No!
  Fix: use composition. Service HAS-A repository.

MISTAKE 2: Forgetting @Override annotation
  public double getarea() { } // TYPO! Creates new method, not override.
  Fix: always use @Override → compiler catches typos.

MISTAKE 3: Not calling super() in constructor
  class Dog extends Animal {
    public Dog(String name) {
      // Forgot super(name)!
      // Java adds super() automatically IF Animal has no-arg constructor
      // If Animal has no no-arg constructor → COMPILE ERROR
    }
  }
  Fix: explicitly call super(args) as first statement.

MISTAKE 4: Calling super() not as first statement
  public Dog(String name) {
    this.name = name; // COMPILE ERROR: super() must be first
    super(name);
  }
  Fix: super() must be the VERY FIRST statement.

MISTAKE 5: Accessing private parent fields directly in child
  class Child extends Parent {
    void method() {
      this.privateField = "value"; // COMPILE ERROR: private!
    }
  }
  Fix: use protected fields OR public/protected getters/setters.

MISTAKE 6: Overriding with reduced visibility
  // Parent:
  public void process() { }
  // Child:
  protected void process() { } // COMPILE ERROR: cannot reduce visibility
  Fix: override with same or broader visibility.

MISTAKE 7: Overriding static methods (they're not truly overridden!)
  // Static methods are hidden, not overridden:
  Parent p = new Child();
  p.staticMethod(); // calls PARENT's static method, not Child's!
  // This is a common interview trap.
  Fix: don't rely on polymorphism for static methods.

MISTAKE 8: Deep inheritance hierarchies
  A → B → C → D → E → F → G (7 levels deep)
  Hard to understand, hard to change, hard to debug.
  Fix: max 3-4 levels. After that, consider composition.

MISTAKE 9: Assuming instanceof check is always needed
  // If you need instanceof everywhere, inheritance is wrong:
  for (Employee e : employees) {
    if (e instanceof Manager) { ((Manager)e).doManagerThing(); }
    else if (e instanceof Developer) { ((Developer)e).doDevThing(); }
  }
  Fix: use polymorphism — add doYourThing() to Employee,
  override in each subclass. No instanceof needed.

MISTAKE 10: Extending classes you don't own/control
  class MyString extends String { } // COMPILE ERROR: String is final
  class MyList extends ArrayList { } // dangerous — ArrayList can change
  Fix: wrap/compose instead of extending library classes.
```

---

## Interview Questions

**Q: What is inheritance in Java? What are its benefits and drawbacks?**
A: Inheritance is a mechanism where a child class (subclass) acquires fields and methods from a parent class (superclass) using the extends keyword. Benefits: code reuse (write common logic once in parent), polymorphism (treat all subtypes as parent type), modeling real-world hierarchies. Drawbacks: tight coupling (child depends on parent internals — fragile base class problem), deep hierarchies become hard to understand, can lead to wrong designs if IS-A isn't truly there (prefer composition in those cases). Effective Java recommends favoring composition over inheritance unless the IS-A relationship is genuinely clear and you control both classes.

**Q: What is the difference between method overloading and overriding?**
A: Overloading: same method name, different parameter list, in the SAME class. Compile-time decision. Doesn't require inheritance. Example: calculateArea(double radius) and calculateArea(double w, double h). Overriding: same method name, same parameter list, in a CHILD class. Runtime decision (polymorphism). Requires inheritance. Child provides its own implementation of a parent method. Rules for overriding: same signature, return type same or subtype (covariant), cannot reduce visibility, @Override annotation catches mistakes at compile time.

**Q: What is the use of the super keyword?**
A: super refers to the parent class. Three uses: 1) super() — call parent constructor, must be first statement in child constructor. Required when parent has no no-arg constructor. 2) super.method() — call parent's version of an overridden method, useful when you want to extend (not replace) parent behavior. 3) super.field — access parent's field (rarely needed if using encapsulation properly). super cannot be used in static methods.

**Q: What happens when you upcast and downcast in Java?**
A: Upcasting: assigning child reference to parent type variable. Automatic and safe. Dog dog = new Dog(); Animal a = dog; Through 'a' you can only call Animal methods, but overridden methods still call Dog's version (polymorphism). Downcasting: casting parent reference back to child type. Must be explicit: Dog d = (Dog) a; Required to access child-specific methods. DANGEROUS if the actual object isn't that type — throws ClassCastException at runtime. Always check with instanceof before downcasting. Modern Java: use pattern matching: if (a instanceof Dog d) { d.fetch(); }

**Q: Can a constructor be inherited? What about static methods?**
A: Constructors cannot be inherited. A Dog doesn't inherit Animal's constructor — it must define its own. But the child constructor MUST call the parent constructor via super() (explicitly or implicitly if parent has no-arg constructor). Static methods can appear in both parent and child with the same signature, but they are NOT overridden — they are HIDDEN. When you call a static method through a reference, Java uses the declared type of the reference (compile-time), not the actual object type (runtime). This is a common interview trick question.

**Q: What is the difference between final class, final method, and final field?**
A: final class: cannot be extended. No subclass can be created. Example: String, Integer. Use for security, immutability guarantee. final method: cannot be overridden in subclasses. The method implementation is locked. Use in template method pattern to preserve algorithm structure. Overriding individual hook methods is fine, but the main orchestrating method is final. final field: cannot be reassigned after initialization (in constructor or at declaration). Makes a field a constant per-object (if instance field) or a constant period (if also static). Use for IDs, timestamps, configuration values that must not change.

---

## Key Takeaways

```text
1. Inheritance = child class gets fields and methods from parent.
   Use ONLY for true IS-A relationships.
   Prefer composition (HAS-A) when in doubt.

2. extends keyword creates the inheritance link.
   Java supports SINGLE inheritance (one parent only).
   But a class can implement MULTIPLE interfaces (Level 1.14).

3. super() calls parent constructor — MUST be first statement.
   If no super() written: Java adds super() implicitly.
   Requires parent to have a no-arg constructor.
   Constructor order: TOP-DOWN (grandparent → parent → child).

4. super.method() calls parent's version of an overridden method.
   Use to EXTEND (not replace) parent behavior.

5. @Override annotation is non-negotiable professional habit.
   Catches typos at compile time. Without it: silent bugs.

6. Method overriding rules:
   Same name + same params + same (or subtype) return type.
   Cannot reduce visibility (public → private not allowed).
   Can broaden visibility (protected → public is fine).

7. UPCASTING (child → parent): automatic, always safe.
   Enables polymorphism — different objects, same interface.
   Can only call parent's methods through parent reference.
   But OVERRIDDEN methods still call the child's version!

8. DOWNCASTING (parent → child): explicit cast, RISKY.
   Always check instanceof before downcasting.
   Use Java 16+ pattern matching: if (obj instanceof Dog d)

9. final modifier:
   final class → no subclasses (String, Integer)
   final method → no overriding in children
   final field → no reassignment after construction

10. Deep hierarchies (>3-4 levels) = design smell.
    If you need instanceof everywhere = polymorphism not working.
    Favor composition over inheritance (Effective Java Item 18).
```

---