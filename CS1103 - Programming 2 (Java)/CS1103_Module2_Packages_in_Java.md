# CS 1103 – Programming 2 (Java)
## Module 2: Packages in Java

---

## Table of Contents

1. [What Is a Package?](#1-what-is-a-package)
2. [Why Packages Matter](#2-why-packages-matter)
3. [Built-in Java Packages](#3-built-in-java-packages)
4. [Creating Your Own Package](#4-creating-your-own-package)
5. [The import Statement](#5-the-import-statement)
6. [Access Modifiers and Packages](#6-access-modifiers-and-packages)
7. [Sub-packages](#7-sub-packages)
8. [Organizing a Multi-Package Project](#8-organizing-a-multi-package-project)
9. [JAR Files — Packaging for Distribution](#9-jar-files--packaging-for-distribution)
10. [Putting It All Together](#10-putting-it-all-together)
11. [Exercises](#11-exercises)

---

## 1. What Is a Package?

As your programs grow, you'll have dozens — even hundreds — of class files. Without organization, finding the right class becomes difficult and name collisions become likely. Imagine two different developers both writing a class called `User` — which one does Java use?

A **package** is Java's solution. It is a **namespace** that groups related classes and interfaces together, similar to how a file system uses folders to organize files.

```
com.myapp.models       ← classes representing data (User, Product, Order)
com.myapp.services     ← classes containing business logic
com.myapp.utils        ← helper and utility classes
com.myapp.gui          ← graphical interface classes
```

A package declaration at the top of a `.java` file tells Java which package that class belongs to:

```java
package com.myapp.models;

public class User {
    // ...
}
```

---

## 2. Why Packages Matter

Packages solve three real problems:

### 1. Organization
Related classes live together. A developer looking for data models knows to look in `com.myapp.models`, not hunt through hundreds of files.

### 2. Name Collision Avoidance
Two completely different classes can share the same name as long as they live in different packages:

```java
com.myapp.models.User      ← your app's User class
com.thirdpartylibrary.User ← a library's User class
```

They coexist without conflict because the **fully qualified name** (package + class name) is unique.

### 3. Access Control
Packages work with access modifiers (you saw `protected` in Module 6 of CS 1102). The package-private access level (no modifier) makes a class visible only within its own package — a powerful tool for hiding implementation details.

---

## 3. Built-in Java Packages

Java ships with hundreds of packages you've already been using. Here are the most important ones:

| Package | What it contains |
|---|---|
| `java.lang` | Core classes — `String`, `Math`, `System`, `Object`, `Integer` — **automatically imported** |
| `java.util` | Utilities — `ArrayList`, `HashMap`, `Scanner`, `Collections`, `List`, `Set`, `Map` |
| `java.io` | Input/output — `File`, `FileReader`, `BufferedReader`, `PrintWriter` |
| `java.nio` | Modern I/O — `Path`, `Files`, `Paths` (covered in Module 7) |
| `java.net` | Networking — `URL`, `Socket`, `ServerSocket` (covered in Module 7) |
| `java.time` | Date and time — `LocalDate`, `LocalDateTime`, `Period` |
| `java.math` | Big numbers — `BigInteger`, `BigDecimal` |
| `javax.swing` | GUI components — `JFrame`, `JButton`, `JLabel` |
| `java.sql` | Database connectivity — `Connection`, `Statement`, `ResultSet` (covered in Module 5) |

`java.lang` is special — Java imports it automatically, which is why you can use `String` and `System` without an import statement.

---

## 4. Creating Your Own Package

### Step 1 — Declare the Package

The `package` statement must be the **very first line** of code in your file (before imports, before the class):

```java
package com.myapp.models;

public class Product {

    private String name;
    private double price;

    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }

    public String getName()  { return name; }
    public double getPrice() { return price; }

    @Override
    public String toString() {
        return name + " ($" + price + ")";
    }
}
```

### Step 2 — Match Your Folder Structure

Java requires that the folder structure mirrors the package name. The class above must be saved at:

```
src/
└── com/
    └── myapp/
        └── models/
            └── Product.java
```

If the folder structure does not match the package declaration, the compiler will throw an error.

### Naming Conventions

Java package names follow strict conventions:

- All **lowercase** letters.
- Use your **reversed domain name** as the prefix — this guarantees global uniqueness.
- Separate levels with dots.

```
com.google.maps          ← Google's maps package
org.apache.commons.lang  ← Apache Commons library
edu.university.cs1103    ← an academic project
com.yourname.appname     ← your personal project
```

If you don't have a domain, use something like `com.yourname.projectname` — the convention still applies.

---

## 5. The `import` Statement

Once a class lives in a package, other classes need to import it before using it — unless they're in the same package.

### Importing a Specific Class

```java
package com.myapp.services;

import com.myapp.models.Product;   // import the single class you need

public class ProductService {

    public void displayProduct(Product p) {
        System.out.println(p);
    }
}
```

### Importing All Classes in a Package (Wildcard Import)

```java
import com.myapp.models.*;   // imports everything in the models package
```

This is convenient but considered less readable in professional code — it's unclear which classes are actually being used. Most IDEs and style guides prefer explicit single-class imports.

### No Import Needed When in the Same Package

If `ProductService` and `Product` are both in `com.myapp.models`, no import is required — they can see each other automatically.

### Fully Qualified Name (No Import)

You can also use a class without importing it by writing its full package path every time:

```java
com.myapp.models.Product p = new com.myapp.models.Product("Laptop", 999.99);
```

This is verbose and rarely used, but it's the solution when two classes from different packages share the same name:

```java
java.util.Date utilDate = new java.util.Date();
java.sql.Date  sqlDate  = new java.sql.Date(System.currentTimeMillis());
```

### Static Imports

You can import `static` members (methods or constants) so you don't need to prefix them with the class name:

```java
import static java.lang.Math.PI;
import static java.lang.Math.sqrt;
import static java.lang.Math.pow;

public class CircleCalc {
    public static void main(String[] args) {
        double radius = 5.0;
        double area = PI * pow(radius, 2);         // instead of Math.PI * Math.pow(...)
        double diagonal = sqrt(2) * radius;        // instead of Math.sqrt(...)
        System.out.printf("Area: %.2f%n", area);
        System.out.printf("Diagonal: %.2f%n", diagonal);
    }
}
```

Use static imports sparingly — overuse makes it unclear where a method comes from.

---

## 6. Access Modifiers and Packages

You've seen `public` and `private`. Now that you understand packages, all four access levels make complete sense:

| Modifier | Same Class | Same Package | Subclass (any package) | Any Class |
|---|---|---|---|---|
| `public` | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| *(none)* | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ |

### Package-Private (No Modifier)

A class, method, or variable with no access modifier is **package-private** — visible only within the same package. This is often called the "default" access level.

```java
package com.myapp.models;

class InternalHelper {   // no 'public' — only visible within com.myapp.models
    static String format(String s) {
        return s.trim().toLowerCase();
    }
}
```

`InternalHelper` is invisible to code outside `com.myapp.models`. This is powerful for hiding implementation details that other parts of the program don't need to know about.

### Class-Level Access

A **top-level class** (not nested inside another) can only be `public` or package-private. Making a class `public` means every other class can use it. Making it package-private limits it to its own package.

```java
package com.myapp.utils;

// Public — usable from anywhere
public class StringUtils {
    public static String capitalize(String s) {
        if (s == null || s.isEmpty()) return s;
        return Character.toUpperCase(s.charAt(0)) + s.substring(1).toLowerCase();
    }
}

// Package-private — only visible within com.myapp.utils
class InternalParser {
    // ...
}
```

---

## 7. Sub-packages

A sub-package is simply a package nested inside another by adding another dot-separated level to the name:

```
com.myapp
com.myapp.models
com.myapp.models.validation
```

**Important:** In Java, sub-packages are **not** automatically included when you import a parent package. `import com.myapp.*` imports classes directly in `com.myapp` — it does NOT import anything in `com.myapp.models`.

```java
import com.myapp.*;          // only classes directly in com.myapp
import com.myapp.models.*;   // classes in com.myapp.models
// These are two separate imports
```

Sub-packages are useful for further organising large packages:

```
com.myapp.models
com.myapp.models.validation   ← validators specific to models
com.myapp.services
com.myapp.services.external   ← services that call external APIs
```

---

## 8. Organizing a Multi-Package Project

Here is what a realistic, well-organized Java project structure looks like. This is a simple online store application:

```
store-app/
└── src/
    └── com/
        └── store/
            ├── models/
            │   ├── Product.java
            │   ├── Customer.java
            │   └── Order.java
            ├── services/
            │   ├── ProductService.java
            │   ├── OrderService.java
            │   └── CustomerService.java
            ├── utils/
            │   ├── StringUtils.java
            │   └── PriceFormatter.java
            └── Main.java
```

Let's see each file:

**com/store/models/Product.java**
```java
package com.store.models;

public class Product {
    private String name;
    private double price;
    private int stock;

    public Product(String name, double price, int stock) {
        this.name  = name;
        this.price = price;
        this.stock = stock;
    }

    public String getName()  { return name; }
    public double getPrice() { return price; }
    public int getStock()    { return stock; }

    public boolean isAvailable() { return stock > 0; }

    public void reduceStock(int quantity) {
        this.stock = Math.max(0, this.stock - quantity);
    }

    @Override
    public String toString() {
        return String.format("%-15s $%-8.2f Stock: %d", name, price, stock);
    }
}
```

**com/store/utils/PriceFormatter.java**
```java
package com.store.utils;

public class PriceFormatter {

    public static String format(double price) {
        return String.format("$%.2f", price);
    }

    public static String formatWithLabel(String label, double price) {
        return String.format("%-15s %s", label + ":", format(price));
    }
}
```

**com/store/services/ProductService.java**
```java
package com.store.services;

import com.store.models.Product;
import com.store.utils.PriceFormatter;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

public class ProductService {

    private List<Product> inventory = new ArrayList<>();

    public void addProduct(Product product) {
        inventory.add(product);
    }

    public List<Product> getAvailable() {
        return inventory.stream()
            .filter(Product::isAvailable)
            .collect(Collectors.toList());
    }

    public List<Product> getUnder(double maxPrice) {
        return inventory.stream()
            .filter(p -> p.getPrice() <= maxPrice)
            .collect(Collectors.toList());
    }

    public void printInventory() {
        System.out.println("=== Inventory ===");
        inventory.forEach(System.out::println);
        double total = inventory.stream()
            .mapToDouble(Product::getPrice).sum();
        System.out.println(PriceFormatter.formatWithLabel("Total value", total));
    }
}
```

**com/store/Main.java**
```java
package com.store;

import com.store.models.Product;
import com.store.services.ProductService;

public class Main {
    public static void main(String[] args) {
        ProductService service = new ProductService();
        service.addProduct(new Product("Laptop",     999.99, 5));
        service.addProduct(new Product("Phone",      699.99, 0));   // out of stock
        service.addProduct(new Product("Headphones", 149.99, 12));
        service.addProduct(new Product("Keyboard",   79.99,  8));

        service.printInventory();

        System.out.println("\n=== Available Products ===");
        service.getAvailable().forEach(System.out::println);

        System.out.println("\n=== Under $200 ===");
        service.getUnder(200).forEach(System.out::println);
    }
}
```

Output:
```
=== Inventory ===
Laptop          $999.99  Stock: 5
Phone           $699.99  Stock: 0
Headphones      $149.99  Stock: 12
Keyboard        $79.99   Stock: 8
Total value:    $1929.96

=== Available Products ===
Laptop          $999.99  Stock: 5
Headphones      $149.99  Stock: 12
Keyboard        $79.99   Stock: 8

=== Under $200 ===
Headphones      $149.99  Stock: 12
Keyboard        $79.99   Stock: 8
```

---

## 9. JAR Files — Packaging for Distribution

Once your project is complete, you can package all your compiled `.class` files into a single **JAR file** (Java ARchive). A JAR is essentially a ZIP file containing your bytecode, ready to be shared, deployed, or used as a library.

### Creating a JAR from the Command Line

First compile your code:
```bash
javac -d out src/com/store/**/*.java src/com/store/Main.java
```

Then create the JAR:
```bash
jar --create --file store-app.jar --main-class com.store.Main -C out .
```

### Running a JAR

```bash
java -jar store-app.jar
```

### Using a JAR as a Library

When your project depends on a third-party JAR (like a logging or JSON library), add it to the **classpath** at compile and run time:

```bash
javac -cp lib/gson.jar src/com/store/Main.java
java -cp lib/gson.jar:out com.store.Main
```

### In an IDE

In IntelliJ IDEA or VS Code, you add JAR dependencies through the project settings instead of the command line — the IDE manages the classpath for you. This is the standard workflow in real projects.

---

## 10. Putting It All Together

Here's a summary of the complete workflow for building a packaged Java project:

```
1. Plan your package structure
   └── com.myapp.models, com.myapp.services, com.myapp.utils

2. Create matching folder structure
   └── src/com/myapp/models/...

3. Add package declaration as first line in each file
   └── package com.myapp.models;

4. Import what you need in each file
   └── import com.myapp.models.Product;

5. Use access modifiers to control visibility
   └── public, protected, (none), private

6. Compile and run
   └── javac, java, or use your IDE

7. Package for distribution if needed
   └── jar --create ...
```

---

## 11. Exercises

---

### Exercise 1 – Your First Package

Create a package called `com.school.models` containing a class `Student` with:
- Private fields: `name` (String), `id` (int), `grade` (double).
- A constructor, getters, and a `toString()`.

Create a second package `com.school.app` containing a `Main` class that imports `Student`, creates three students, and prints them.

Make sure your folder structure matches the package names.

---

### Exercise 2 – Access Modifier Exploration

Create two classes in the same package `com.demo`:
- `ClassA` with one `public` field, one `protected` field, one package-private field, and one `private` field (all Strings with different values).
- `ClassB` that creates an instance of `ClassA` and tries to print all four fields.

Observe which ones compile and which ones cause errors. Then create `ClassC` in a **different** package `com.other` and repeat the experiment.

Document which access levels are visible from where.

---

### Exercise 3 – Utility Package

Create a package `com.utils` containing a class `MathUtils` with these static methods:
- `isPrime(int n)` — returns `true` if `n` is a prime number.
- `factorial(int n)` — returns `n!` (use a loop).
- `gcd(int a, int b)` — returns the greatest common divisor.
- `clamp(int value, int min, int max)` — returns `value` clamped between min and max.

In a `Main` class in `com.app`, import and test all four methods.

---

### Exercise 4 – Multi-Package Library System

Design a small library system across three packages:

**`com.library.models`**
- `Book` — title, author, isbn, available (boolean).
- `Member` — name, memberId, list of borrowed books.

**`com.library.services`**
- `LibraryService` — holds a list of books and members. Methods: `addBook`, `registerMember`, `borrowBook(memberId, isbn)`, `returnBook(memberId, isbn)`, `printAvailableBooks()`.

**`com.library.app`**
- `Main` — sets up books and members, borrows some, returns one, and prints the catalogue.

---

### Exercise 5 – Static Import Practice

Create a class `com.math.Geometry` with **static** methods:
- `circleArea(double radius)`
- `rectangleArea(double w, double h)`
- `triangleArea(double base, double height)`
- `hypotenuse(double a, double b)`

In `Main`, use **static imports** to call all four methods without the class prefix. Also use a static import of `Math.PI` directly.

---

### Exercise 6 – Package Refactor *(Challenge)*

Take the code you wrote for **Exercise 5 of CS 1102 Module 8** (the grade management system using `Map<String, List<Double>>`) and restructure it into a proper multi-package project:

```
com.gradebook.models     → Student class with name and list of scores
com.gradebook.services   → GradeService with all logic (add, average, top, at-risk)
com.gradebook.utils      → GradeUtils with static helper methods (letter grade, formatted output)
com.gradebook.app        → Main
```

Each package should only `import` what it actually needs. The `Main` class should only interact with `GradeService` — it should not directly touch `Student` or `GradeUtils`.

---

## Module Summary

Here's what you covered in Module 2:

- A **package** is a namespace that groups related classes, prevents name collisions, and controls access.
- Package names follow a reversed-domain convention and must be all lowercase.
- The `package` declaration must be the **first line** of every source file.
- The **folder structure must mirror the package name** exactly.
- `import` brings a class into scope; wildcard `.*` imports all classes in a package (but not sub-packages).
- **Static imports** let you use static methods and constants without the class prefix.
- Access levels from most to least open: `public` → `protected` → package-private → `private`.
- **Package-private** (no modifier) is a powerful tool for hiding implementation details within a package.
- A **JAR file** packages compiled bytecode for distribution or reuse as a library.
- Good projects follow a layered structure: models, services, utils, and app (main) packages.

---

**Next up:** Module 3 – Multithreading

---

*CS 1103 Programming 2 (Java) — Module 2 of 8*
