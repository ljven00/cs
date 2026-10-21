# CS 1102 – Programming 1 (Java)
## Module 6: OOP Paradigms

---

## Table of Contents

1. [The Four Pillars of OOP](#1-the-four-pillars-of-oop)
2. [Inheritance](#2-inheritance)
3. [The `super` Keyword](#3-the-super-keyword)
4. [Method Overriding](#4-method-overriding)
5. [Polymorphism](#5-polymorphism)
6. [Abstract Classes](#6-abstract-classes)
7. [Interfaces](#7-interfaces)
8. [Abstract Class vs Interface](#8-abstract-class-vs-interface)
9. [Encapsulation (Revisited)](#9-encapsulation-revisited)
10. [Putting It All Together](#10-putting-it-all-together)
11. [Exercises](#11-exercises)

---

## 1. The Four Pillars of OOP

Object-Oriented Programming is built on four core ideas. You've already touched on one of them:

| Pillar | What it means |
|---|---|
| **Encapsulation** | Bundling data and methods together; hiding internal details with `private`. |
| **Inheritance** | A class can inherit variables and methods from another class. |
| **Polymorphism** | One interface, many forms — the same method name behaves differently depending on the object. |
| **Abstraction** | Hiding complex implementation details and exposing only what's necessary. |

This module covers all four in depth.

---

## 2. Inheritance

Imagine you're building a program about animals. A `Dog` and a `Cat` both have a name, an age, and can eat and sleep. Without inheritance, you'd copy those fields and methods into both classes — a lot of repetition.

**Inheritance** lets one class (the **child** or **subclass**) automatically receive all the fields and methods of another class (the **parent** or **superclass**). The child can then add its own unique behavior on top.

```
Animal          ← parent class (superclass)
├── Dog         ← child class (subclass)
└── Cat         ← child class (subclass)
```

Use the `extends` keyword to inherit from a parent:

### Parent Class

```java
public class Animal {

    private String name;
    private int age;

    Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public int getAge()     { return age; }

    public void eat() {
        System.out.println(name + " is eating.");
    }

    public void sleep() {
        System.out.println(name + " is sleeping.");
    }

    @Override
    public String toString() {
        return name + " (age " + age + ")";
    }
}
```

### Child Classes

```java
public class Dog extends Animal {

    private String breed;

    Dog(String name, int age, String breed) {
        super(name, age);       // call the parent constructor
        this.breed = breed;
    }

    public String getBreed() { return breed; }

    public void fetch() {
        System.out.println(getName() + " fetches the ball!");
    }
}
```

```java
public class Cat extends Animal {

    private boolean isIndoor;

    Cat(String name, int age, boolean isIndoor) {
        super(name, age);
        this.isIndoor = isIndoor;
    }

    public void purr() {
        System.out.println(getName() + " purrs contentedly.");
    }
}
```

### Using the Subclasses

```java
Dog dog = new Dog("Rex", 3, "Labrador");
Cat cat = new Cat("Whiskers", 5, true);

dog.eat();      // Inherited from Animal → Rex is eating.
dog.sleep();    // Inherited from Animal → Rex is sleeping.
dog.fetch();    // Unique to Dog        → Rex fetches the ball!

cat.eat();      // Inherited from Animal → Whiskers is eating.
cat.purr();     // Unique to Cat         → Whiskers purrs contentedly.

System.out.println(dog);    // Rex (age 3)
System.out.println(cat);    // Whiskers (age 5)
```

A `Dog` is an `Animal`. A `Cat` is an `Animal`. This relationship is called **"is-a"** — the foundation of inheritance.

> **One rule:** Java supports **single inheritance** only — a class can extend only one parent class. (Interfaces, covered later, work around this.)

---

## 3. The `super` Keyword

`super` refers to the **parent class**. You've already seen it used to call the parent's constructor:

```java
Dog(String name, int age, String breed) {
    super(name, age);   // calls Animal's constructor
    this.breed = breed;
}
```

This must be the **first line** of the child constructor if used. It's how the parent gets to set up its own fields before the child adds its own.

You can also use `super` to call a parent's method directly:

```java
public class Dog extends Animal {

    @Override
    public String toString() {
        return super.toString() + " | Breed: " + breed;
        //     ^^^^^^^^^^^^^^^^
        //     calls Animal's toString()
    }
}
```

```java
Dog dog = new Dog("Rex", 3, "Labrador");
System.out.println(dog);   // Rex (age 3) | Breed: Labrador
```

---

## 4. Method Overriding

A child class can **replace** a parent's method with its own version. This is called **overriding**. Java will use the child's version when the method is called on a child object.

Use `@Override` to signal your intent (and let Java catch typos):

```java
public class Animal {
    public void makeSound() {
        System.out.println("...");
    }
}

public class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
}

public class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow!");
    }
}
```

```java
Dog dog = new Dog("Rex", 3, "Labrador");
Cat cat = new Cat("Whiskers", 5, true);

dog.makeSound();   // Woof!
cat.makeSound();   // Meow!
```

### Overriding vs Overloading

These two terms sound similar but are very different:

| | Overriding | Overloading |
|---|---|---|
| Where | Child class replaces parent's method | Same class, different parameter lists |
| Same name? | Yes | Yes |
| Same parameters? | Yes | No — parameters differ |
| Decided at | Runtime | Compile time |

---

## 5. Polymorphism

**Polymorphism** means "many forms." In Java, it means you can refer to a child object using a parent-type variable — and Java will call the right version of overridden methods automatically.

```java
Animal a1 = new Dog("Rex", 3, "Labrador");
Animal a2 = new Cat("Whiskers", 5, true);
Animal a3 = new Animal("Unknown", 1);

a1.makeSound();   // Woof!  ← Dog's version
a2.makeSound();   // Meow!  ← Cat's version
a3.makeSound();   // ...    ← Animal's version
```

All three variables are of type `Animal`, but Java looks at what **actual object** they point to and calls that object's version of `makeSound()`. This is called **dynamic dispatch** and happens at runtime.

### Why is this useful?

You can write code that works with the parent type and it handles any subclass automatically:

```java
static void makeAllSounds(Animal[] animals) {
    for (Animal animal : animals) {
        animal.makeSound();   // calls each animal's actual version
    }
}
```

```java
Animal[] animals = {
    new Dog("Rex", 3, "Labrador"),
    new Cat("Whiskers", 5, true),
    new Dog("Max", 2, "Beagle")
};

makeAllSounds(animals);
```

Output:
```
Woof!
Meow!
Woof!
```

You didn't need to check the type or write separate methods for dogs and cats. Polymorphism handles it.

---

## 6. Abstract Classes

Sometimes a parent class is too general to have a meaningful implementation of a method. What does a generic `Animal` sound like? It depends on the specific animal.

An **abstract class** lets you define a method signature without writing the body — forcing every subclass to provide its own implementation. You can't create an object directly from an abstract class.

Use the `abstract` keyword:

```java
public abstract class Shape {

    private String color;

    Shape(String color) {
        this.color = color;
    }

    public String getColor() { return color; }

    // Abstract method — no body, must be implemented by subclasses
    public abstract double getArea();
    public abstract double getPerimeter();

    // Regular method — inherited as-is
    public void describe() {
        System.out.println("A " + color + " shape with area " + getArea());
    }
}
```

Now any class extending `Shape` **must** implement `getArea()` and `getPerimeter()`:

```java
public class Circle extends Shape {

    private double radius;

    Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    public double getArea() {
        return Math.PI * radius * radius;
    }

    @Override
    public double getPerimeter() {
        return 2 * Math.PI * radius;
    }
}
```

```java
public class Rectangle extends Shape {

    private double width, height;

    Rectangle(String color, double width, double height) {
        super(color);
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
}
```

```java
Shape c = new Circle("red", 5);
Shape r = new Rectangle("blue", 4, 6);

c.describe();   // A red shape with area 78.53981633974483
r.describe();   // A blue shape with area 24.0

// Shape s = new Shape("green");   ← ERROR: cannot instantiate abstract class
```

Abstract classes are useful when:
- You want to share common code between subclasses.
- You want to enforce that subclasses implement certain methods.
- The parent concept is too general to make objects from.

---

## 7. Interfaces

An **interface** is a step further than an abstract class. It defines a **contract** — a list of methods that any implementing class must provide. Interfaces contain:

- Abstract methods (by default — no body needed)
- Constants (variables that are `public static final` by default)
- (Since Java 8) `default` methods with a body

A class **implements** an interface using the `implements` keyword.

```java
public interface Printable {
    void print();   // every implementing class must provide this
}

public interface Saveable {
    void save(String filename);
}
```

```java
public class Report implements Printable, Saveable {

    private String content;

    Report(String content) {
        this.content = content;
    }

    @Override
    public void print() {
        System.out.println("Printing report:\n" + content);
    }

    @Override
    public void save(String filename) {
        System.out.println("Saving report to " + filename);
    }
}
```

```java
Report report = new Report("Q3 Sales Summary");
report.print();
report.save("q3_report.txt");
```

Output:
```
Printing report:
Q3 Sales Summary
Saving report to q3_report.txt
```

### A class can implement multiple interfaces

```java
public class Document implements Printable, Saveable {
    // must implement both print() and save()
}
```

This is how Java works around its single-inheritance limitation. A class can only extend **one** parent, but it can implement **many** interfaces.

### Interfaces as Types

Like abstract classes, interfaces can be used as variable types:

```java
Printable p = new Report("Annual Summary");
p.print();   // works — Report implements Printable
```

---

## 8. Abstract Class vs Interface

Both enforce method implementation on subclasses, but they serve different purposes:

| Feature | Abstract Class | Interface |
|---|---|---|
| Can have instance variables? | Yes | No (only constants) |
| Can have constructors? | Yes | No |
| Can have method bodies? | Yes | Only `default` methods |
| A class can extend how many? | One | Many |
| Use when... | Sharing common code among related classes | Defining a capability any unrelated class can adopt |

A helpful way to think about it:

- **Abstract class** → "is-a" relationship. A `Dog` *is an* `Animal`.
- **Interface** → "can-do" relationship. A `Report` *can be* printed. A `Robot` *can also be* printed. They're unrelated, but both can fulfill the `Printable` contract.

---

## 9. Encapsulation (Revisited)

You learned about encapsulation in Module 5 — making fields `private` and exposing them through getters and setters. Now that you understand inheritance, here's an important addition:

### Access Modifiers

| Modifier | Accessible from |
|---|---|
| `public` | Anywhere |
| `private` | Only within the same class |
| `protected` | Same class, same package, and subclasses |
| *(no modifier)* | Same package only |

`protected` is especially useful in inheritance — it lets subclasses access parent fields directly without making them fully public:

```java
public class Animal {
    protected String name;   // accessible in Dog, Cat, etc.
    private int age;         // NOT accessible in subclasses directly
}

public class Dog extends Animal {
    void showName() {
        System.out.println(name);   // works — protected
    }
    void showAge() {
        // System.out.println(age);  ← ERROR — private
        System.out.println(getAge());   // must use getter
    }
}
```

---

## 10. Putting It All Together

Here's a complete example using inheritance, abstract classes, polymorphism, and encapsulation together — a simple employee payment system.

**File: Employee.java**
```java
public abstract class Employee {

    private String name;
    private int id;

    Employee(String name, int id) {
        this.name = name;
        this.id = id;
    }

    public String getName() { return name; }
    public int getId()      { return id; }

    // Every employee type calculates pay differently
    public abstract double calculatePay();

    public void printPaySlip() {
        System.out.println("--- Pay Slip ---");
        System.out.println("Name: " + name);
        System.out.println("ID:   " + id);
        System.out.printf("Pay:  $%.2f%n", calculatePay());
    }
}
```

**File: FullTimeEmployee.java**
```java
public class FullTimeEmployee extends Employee {

    private double monthlySalary;

    FullTimeEmployee(String name, int id, double monthlySalary) {
        super(name, id);
        this.monthlySalary = monthlySalary;
    }

    @Override
    public double calculatePay() {
        return monthlySalary;
    }
}
```

**File: PartTimeEmployee.java**
```java
public class PartTimeEmployee extends Employee {

    private double hourlyRate;
    private int hoursWorked;

    PartTimeEmployee(String name, int id, double hourlyRate, int hoursWorked) {
        super(name, id);
        this.hourlyRate = hourlyRate;
        this.hoursWorked = hoursWorked;
    }

    @Override
    public double calculatePay() {
        return hourlyRate * hoursWorked;
    }
}
```

**File: Main.java**
```java
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {

        ArrayList<Employee> staff = new ArrayList<>();
        staff.add(new FullTimeEmployee("Alice", 101, 4500.00));
        staff.add(new PartTimeEmployee("Bob", 102, 18.50, 80));
        staff.add(new FullTimeEmployee("Carlos", 103, 5200.00));
        staff.add(new PartTimeEmployee("Diana", 104, 22.00, 60));

        for (Employee emp : staff) {
            emp.printPaySlip();   // polymorphism — each calls its own calculatePay()
            System.out.println();
        }
    }
}
```

Output:
```
--- Pay Slip ---
Name: Alice
ID:   101
Pay:  $4500.00

--- Pay Slip ---
Name: Bob
ID:   102
Pay:  $1480.00

--- Pay Slip ---
Name: Carlos
ID:   103
Pay:  $5200.00

--- Pay Slip ---
Name: Diana
ID:   104
Pay:  $1320.00
```

The `for` loop works on all employee types without any `if/else` checking — that's polymorphism doing its job.

---

## 11. Exercises

---

### Exercise 1 – Vehicle Hierarchy

Create a parent class `Vehicle` with:
- Private fields: `make` (String), `model` (String), `speed` (int, starts at 0).
- Constructor taking `make` and `model`.
- Methods: `accelerate(int amount)` adds to speed, `brake(int amount)` reduces speed (minimum 0), `toString()` showing make, model, and speed.

Create two subclasses:
- `Car` — adds a `numDoors` (int) field and a `honk()` method.
- `Motorcycle` — adds a `hasSidecar` (boolean) field and a `wheelie()` method.

In `main`, create one of each, call some inherited and unique methods, and print both objects.

---

### Exercise 2 – Abstract Shape

Create an abstract class `Shape` with:
- A `color` field (protected).
- An abstract method `getArea()`.
- An abstract method `getPerimeter()`.
- A concrete method `printInfo()` that prints the color, area, and perimeter.

Implement three subclasses: `Circle`, `Rectangle`, and `Triangle`.

In `main`, create one of each, put them in an `ArrayList<Shape>`, and call `printInfo()` on each.

---

### Exercise 3 – Interface Practice

Create two interfaces:
- `Chargeable` with a method `charge(double amount)`.
- `Refundable` with a method `refund(double amount)`.

Create a class `CreditCard` that implements both. It should have a `balance` field and print a message when charged or refunded.

In `main`, create a `CreditCard` object, charge it twice, refund once, and print the final balance.

---

### Exercise 4 – Polymorphism in Action

Create a parent class `Instrument` with a method `play()`.

Create three subclasses: `Guitar`, `Piano`, and `Drums`, each overriding `play()` with a unique message.

Write a static method `performConcert(Instrument[] instruments)` that calls `play()` on each instrument.

In `main`, create an array with one of each instrument and pass it to `performConcert()`.

---

### Exercise 5 – Full OOP System *(Challenge)*

Design a small school system with the following:

**Abstract class `Person`:**
- Fields: `name`, `id`.
- Abstract method `getRole()`.
- Concrete `toString()` using `getRole()`.

**Class `Student` extends `Person`:**
- Additional field: `gpa` (double).
- `getRole()` returns `"Student"`.
- Method `isPassingWith(double minGpa)`.

**Class `Teacher` extends `Person`:**
- Additional field: `subject` (String).
- `getRole()` returns `"Teacher"`.
- Method `teach()` prints a message.

**Interface `Schedulable`:**
- Method `getSchedule()` returns a String.

Have **both** `Student` and `Teacher` implement `Schedulable` with their own schedules.

In `main`, create 2 students and 2 teachers, store them in an `ArrayList<Person>`, and print all of them using `toString()`. Then separately print each person's schedule.

---

## Module Summary

Here's what you covered in Module 6:

- **Inheritance** (`extends`) lets a child class reuse fields and methods from a parent.
- `super` calls the parent's constructor or methods.
- **Method overriding** lets a child replace a parent's method with its own version — use `@Override`.
- **Polymorphism** lets you use a parent-type variable to hold child objects; Java calls the right method at runtime.
- **Abstract classes** define incomplete blueprints — they can have abstract methods that subclasses must implement; you cannot create objects from them directly.
- **Interfaces** define a contract of methods a class must provide; a class can implement multiple interfaces.
- **Abstract class** suits "is-a" relationships with shared code; **interface** suits "can-do" capabilities across unrelated classes.
- **Access modifiers** (`public`, `private`, `protected`) control visibility across the class hierarchy.

---

**Next up:** Module 7 – Graphical User Interfaces (GUI) with Event Handling

---

*CS 1102 Programming 1 (Java) — Module 6 of 8*
