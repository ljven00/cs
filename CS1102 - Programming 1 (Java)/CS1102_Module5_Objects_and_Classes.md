# CS 1102 – Programming 1 (Java)
## Module 5: Objects and Classes

---

## Table of Contents

1. [What Is Object-Oriented Programming?](#1-what-is-object-oriented-programming)
2. [Classes as Blueprints](#2-classes-as-blueprints)
3. [Instance Variables and Methods](#3-instance-variables-and-methods)
4. [Constructors](#4-constructors)
5. [Creating and Using Objects](#5-creating-and-using-objects)
6. [The `this` Keyword](#6-the-this-keyword)
7. [Getters and Setters](#7-getters-and-setters)
8. [Multiple Constructors](#8-multiple-constructors)
9. [Object Arrays and ArrayLists](#9-object-arrays-and-arraylists)
10. [The `toString` Method](#10-the-tostring-method)
11. [Putting It All Together](#11-putting-it-all-together)
12. [Exercises](#12-exercises)

---

## 1. What Is Object-Oriented Programming?

So far, your programs have been a collection of variables and methods — all lumped together in one class. That works for small programs, but as programs grow, it gets hard to manage.

**Object-Oriented Programming (OOP)** is a way of organizing code around real-world "things" — called **objects**. Each object bundles together:

- **Data** — what the object *knows* (its properties).
- **Behavior** — what the object *can do* (its actions).

Think about a real-world **car**:

| What it knows (data) | What it can do (behavior) |
|----------------------|---------------------------|
| Make, model, year    | Start, stop, accelerate   |
| Current speed        | Turn left, turn right      |
| Fuel level           | Refuel                    |

In Java, you represent this by creating a **class** called `Car` that holds that data and those behaviors. You can then create as many individual car **objects** as you need — each with their own make, model, and speed.

OOP makes code:
- **Organized** — related data and behavior stay together.
- **Reusable** — one class definition, unlimited objects.
- **Easier to maintain** — change one class without affecting unrelated code.

---

## 2. Classes as Blueprints

A **class** is a blueprint. It describes what an object will look like and what it can do — but it isn't the object itself.

An **object** is a specific instance created from that blueprint.

The analogy:
- A **class** is like an architectural blueprint for a house.
- An **object** is an actual house built from that blueprint.
- You can build many houses from one blueprint, each at a different address with different paint colors.

Here's a minimal class definition:

```java
public class Dog {
    // This is just the blueprint — no object yet
}
```

A class file should be named exactly after the class — so this goes in `Dog.java`.

---

## 3. Instance Variables and Methods

**Instance variables** (also called fields or attributes) hold the data for each object. Unlike `static` variables which are shared across the whole class, each object gets its **own copy** of instance variables.

**Instance methods** define what an object can do.

```java
public class Dog {

    // Instance variables — each Dog object has its own
    String name;
    String breed;
    int age;

    // Instance method — behavior every Dog can perform
    void bark() {
        System.out.println(name + " says: Woof!");
    }

    void describe() {
        System.out.println(name + " is a " + age + "-year-old " + breed + ".");
    }
}
```

Notice there's no `static` on the instance variables or instance methods. That's the key difference from Module 3 — these belong to individual objects, not the class itself.

---

## 4. Constructors

A **constructor** is a special method that runs automatically when you create a new object. Its job is to **set up the initial state** of the object — give its instance variables their starting values.

Rules for constructors:
- The name must **exactly match the class name**.
- It has **no return type** (not even `void`).

```java
public class Dog {

    String name;
    String breed;
    int age;

    // Constructor
    Dog(String name, String breed, int age) {
        this.name = name;
        this.breed = breed;
        this.age = age;
    }

    void bark() {
        System.out.println(name + " says: Woof!");
    }

    void describe() {
        System.out.println(name + " is a " + age + "-year-old " + breed + ".");
    }
}
```

Don't worry about `this` yet — we'll cover it in Section 6.

### Default Constructor

If you don't write any constructor, Java quietly provides one behind the scenes — a **default constructor** with no parameters that sets all variables to their default values (`0`, `false`, `null`).

Once you write your own constructor, Java's default one goes away.

---

## 5. Creating and Using Objects

To create an object, use the `new` keyword followed by the constructor:

```java
Dog myDog = new Dog("Rex", "Labrador", 3);
```

Breaking this down:

```
Dog      myDog  =  new  Dog("Rex", "Labrador", 3);
^type    ^name     ^keyword  ^constructor call
```

### Accessing Instance Variables

Use **dot notation** — the object name, a dot, then the variable or method name:

```java
System.out.println(myDog.name);    // Rex
System.out.println(myDog.age);     // 3
```

### Calling Instance Methods

```java
myDog.bark();       // Rex says: Woof!
myDog.describe();   // Rex is a 3-year-old Labrador.
```

### Multiple Objects from One Class

```java
Dog dog1 = new Dog("Rex", "Labrador", 3);
Dog dog2 = new Dog("Bella", "Poodle", 5);
Dog dog3 = new Dog("Max", "Beagle", 2);

dog1.bark();      // Rex says: Woof!
dog2.bark();      // Bella says: Woof!
dog3.describe();  // Max is a 2-year-old Beagle.
```

Each object is independent. Changing `dog1.name` has no effect on `dog2.name`.

---

## 6. The `this` Keyword

Look back at the constructor:

```java
Dog(String name, String breed, int age) {
    this.name = name;
    this.breed = breed;
    this.age = age;
}
```

Here, the **parameter** `name` and the **instance variable** `name` have the same name. Java would be confused about which `name` you mean.

`this` refers to **the current object** — the one being constructed or operating. So:

- `this.name` → the instance variable of this object.
- `name` (without `this`) → the parameter passed into the constructor.

Without `this`, the assignment `name = name` would just assign the parameter to itself — the instance variable would never get set.

You can also use `this` in regular instance methods:

```java
void birthday() {
    this.age++;   // increments this object's age
    System.out.println(this.name + " is now " + this.age);
}
```

When there's no name conflict, `this` is optional — but many developers always write it for clarity.

---

## 7. Getters and Setters

In the examples so far, instance variables are **publicly accessible** — any code can read or change them directly:

```java
myDog.age = -5;   // This compiles fine, but makes no real-world sense
```

A best practice in OOP is to make instance variables **private** — hidden from outside code — and provide controlled access through **getter** and **setter** methods. This is called **encapsulation**.

```java
public class Dog {

    private String name;    // private — only accessible within this class
    private String breed;
    private int age;

    Dog(String name, String breed, int age) {
        this.name = name;
        this.breed = breed;
        setAge(age);   // use setter for validation
    }

    // Getter — returns the value
    public String getName() {
        return name;
    }

    public String getBreed() {
        return breed;
    }

    public int getAge() {
        return age;
    }

    // Setter — sets the value, with optional validation
    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        if (age >= 0) {
            this.age = age;
        } else {
            System.out.println("Age cannot be negative. Setting to 0.");
            this.age = 0;
        }
    }
}
```

Now outside code must go through the methods:

```java
Dog myDog = new Dog("Rex", "Labrador", 3);

System.out.println(myDog.getName());   // Rex
myDog.setAge(-5);                      // Age cannot be negative. Setting to 0.
System.out.println(myDog.getAge());    // 0
```

### Naming Convention

| Method type | Pattern       | Example        |
|-------------|---------------|----------------|
| Getter      | `getVariable` | `getName()`    |
| Setter      | `setVariable` | `setName()`    |
| Boolean getter | `isVariable` | `isAvailable()` |

---

## 8. Multiple Constructors

Just like regular methods, constructors can be **overloaded** — you can have more than one constructor with different parameter lists. This gives callers flexibility in how they create objects.

```java
public class Book {

    private String title;
    private String author;
    private int year;
    private double price;

    // Constructor with all details
    Book(String title, String author, int year, double price) {
        this.title = title;
        this.author = author;
        this.year = year;
        this.price = price;
    }

    // Constructor without price (price defaults to 0.0)
    Book(String title, String author, int year) {
        this.title = title;
        this.author = author;
        this.year = year;
        this.price = 0.0;
    }

    // Constructor with only title
    Book(String title) {
        this.title = title;
        this.author = "Unknown";
        this.year = 0;
        this.price = 0.0;
    }

    void describe() {
        System.out.println("\"" + title + "\" by " + author + " (" + year + ") - $" + price);
    }
}
```

```java
Book b1 = new Book("The Hobbit", "J.R.R. Tolkien", 1937, 12.99);
Book b2 = new Book("Clean Code", "Robert Martin", 2008);
Book b3 = new Book("Unknown Title");

b1.describe();   // "The Hobbit" by J.R.R. Tolkien (1937) - $12.99
b2.describe();   // "Clean Code" by Robert Martin (2008) - $0.0
b3.describe();   // "Unknown Title" by Unknown (0) - $0.0
```

---

## 9. Object Arrays and ArrayLists

You can store objects in arrays and ArrayLists just like primitive values.

### Array of Objects

```java
Dog[] dogs = new Dog[3];
dogs[0] = new Dog("Rex", "Labrador", 3);
dogs[1] = new Dog("Bella", "Poodle", 5);
dogs[2] = new Dog("Max", "Beagle", 2);

for (Dog dog : dogs) {
    dog.describe();
}
```

Output:
```
Rex is a 3-year-old Labrador.
Bella is a 5-year-old Poodle.
Max is a 2-year-old Beagle.
```

### ArrayList of Objects

```java
import java.util.ArrayList;

ArrayList<Dog> kennel = new ArrayList<>();
kennel.add(new Dog("Rex", "Labrador", 3));
kennel.add(new Dog("Bella", "Poodle", 5));
kennel.add(new Dog("Max", "Beagle", 2));

for (Dog dog : kennel) {
    dog.bark();
}
```

Output:
```
Rex says: Woof!
Bella says: Woof!
Max says: Woof!
```

---

## 10. The `toString` Method

When you print an object directly, Java doesn't know how to display it nicely:

```java
Dog myDog = new Dog("Rex", "Labrador", 3);
System.out.println(myDog);   // Something like: Dog@6d06d69c
```

That odd output is Java's default — the class name and a memory address. To fix this, **override** the `toString` method inside your class. Java calls `toString` automatically whenever an object is converted to a String.

```java
public class Dog {

    private String name;
    private String breed;
    private int age;

    Dog(String name, String breed, int age) {
        this.name = name;
        this.breed = breed;
        this.age = age;
    }

    @Override
    public String toString() {
        return name + " (" + breed + ", age " + age + ")";
    }
}
```

```java
Dog myDog = new Dog("Rex", "Labrador", 3);
System.out.println(myDog);   // Rex (Labrador, age 3)
```

`@Override` is an **annotation** that tells Java you're intentionally replacing an inherited method. It's optional but highly recommended — Java will warn you if you misspell the method name.

---

## 11. Putting It All Together

Here's a complete, realistic example — a `BankAccount` class with private fields, a constructor, getters/setters, and meaningful methods.

**File: BankAccount.java**
```java
public class BankAccount {

    private String owner;
    private double balance;

    BankAccount(String owner, double initialBalance) {
        this.owner = owner;
        this.balance = initialBalance >= 0 ? initialBalance : 0;
    }

    public String getOwner() {
        return owner;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            System.out.println("Deposited $" + amount + ". New balance: $" + balance);
        } else {
            System.out.println("Deposit amount must be positive.");
        }
    }

    public void withdraw(double amount) {
        if (amount > balance) {
            System.out.println("Insufficient funds.");
        } else if (amount <= 0) {
            System.out.println("Withdrawal amount must be positive.");
        } else {
            balance -= amount;
            System.out.println("Withdrew $" + amount + ". New balance: $" + balance);
        }
    }

    @Override
    public String toString() {
        return "Account[" + owner + "] Balance: $" + balance;
    }
}
```

**File: Main.java**
```java
public class Main {
    public static void main(String[] args) {
        BankAccount account1 = new BankAccount("Alice", 500.00);
        BankAccount account2 = new BankAccount("Bob", 200.00);

        account1.deposit(150.00);
        account1.withdraw(75.00);
        account2.withdraw(300.00);   // Insufficient funds

        System.out.println(account1);
        System.out.println(account2);
    }
}
```

Output:
```
Deposited $150.0. New balance: $650.0
Withdrew $75.0. New balance: $575.0
Insufficient funds.
Account[Alice] Balance: $575.0
Account[Bob] Balance: $200.0
```

---

## 12. Exercises

---

### Exercise 1 – Student Class

Create a class called `Student` with the following:

**Private instance variables:**
- `name` (String)
- `studentId` (int)
- `gpa` (double)

**Constructor:** takes all three values.

**Getters and setters** for all three fields. The `gpa` setter should reject values outside the range 0.0 to 4.0.

**A method** `isHonorRoll()` that returns `true` if the GPA is 3.5 or higher.

**A `toString` method** that returns something like:
```
Student[ID: 1021] Maria - GPA: 3.8
```

In `main`, create 3 students, print each one, and print whether they're on the honor roll.

---

### Exercise 2 – Rectangle Class

Create a class called `Rectangle` with:

**Private instance variables:** `width` (double) and `height` (double).

**Constructor:** takes width and height.

**Methods:**
- `getArea()` — returns width × height.
- `getPerimeter()` — returns 2 × (width + height).
- `isSquare()` — returns `true` if width equals height.
- `toString()` — returns something like: `Rectangle[4.0 x 6.0]`

In `main`, create two rectangles and print their area, perimeter, and whether they are squares.

---

### Exercise 3 – ArrayList of Students

Using your `Student` class from Exercise 1:

1. Create an `ArrayList<Student>` and add at least 5 students.
2. Print all students.
3. Loop through and print only the students on the honor roll.
4. Find and print the student with the highest GPA.

---

### Exercise 4 – Multiple Constructors

Create a class called `Movie` with:

**Private instance variables:** `title` (String), `director` (String), `year` (int), `rating` (double).

**Three constructors:**
- All four fields.
- Title, director, year (rating defaults to `0.0`).
- Title only (director defaults to `"Unknown"`, year to `0`, rating to `0.0`).

**A `toString` method** and a method `isHighlyRated()` that returns `true` if rating is above `8.0`.

Create one object with each constructor and print them all.

---

### Exercise 5 – Simple Library *(Challenge)*

Build two classes:

**`Book` class** with private fields: `title`, `author`, `isCheckedOut` (boolean). Include a constructor (title and author, checked out defaults to `false`), a `checkOut()` method, a `returnBook()` method, and a `toString`.

**`Library` class** with an `ArrayList<Book>` and these methods:
- `addBook(Book book)`
- `checkOutBook(String title)` — finds the book by title and checks it out (print a message if not found or already checked out).
- `returnBook(String title)` — returns the book.
- `printAvailableBooks()` — prints all books not currently checked out.

In `main`, create a library, add 4 books, check out 2, return 1, and print available books.

---

## Module Summary

Here's what you covered in Module 5:

- **OOP** organizes code around objects that bundle data and behavior together.
- A **class** is a blueprint; an **object** is a specific instance of that class.
- **Instance variables** hold each object's data; **instance methods** define what it can do.
- A **constructor** sets up an object's initial state when it's created with `new`.
- The `this` keyword refers to the current object, and resolves naming conflicts.
- **Encapsulation** means making fields `private` and accessing them through **getters and setters**.
- Constructors can be **overloaded** to give callers multiple ways to create objects.
- Objects can be stored in **arrays** and **ArrayLists**.
- Overriding **`toString`** lets you print objects in a readable format.

---

**Next up:** Module 6 – OOP Paradigms (Inheritance, Polymorphism, Abstraction & Interfaces)

---

*CS 1102 Programming 1 (Java) — Module 5 of 8*
