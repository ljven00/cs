# CS 1103 – Programming 2 (Java)
## Module 4: I/O and Applets

---

## Table of Contents

1. [What Is I/O?](#1-what-is-io)
2. [The I/O Stream Model](#2-the-io-stream-model)
3. [Byte Streams](#3-byte-streams)
4. [Character Streams](#4-character-streams)
5. [Buffered Streams](#5-buffered-streams)
6. [Reading and Writing Text Files](#6-reading-and-writing-text-files)
7. [Reading and Writing Binary Files](#7-reading-and-writing-binary-files)
8. [The File Class](#8-the-file-class)
9. [The Modern NIO.2 API](#9-the-modern-nio2-api)
10. [Object Serialization](#10-object-serialization)
11. [Standard Input and Output](#11-standard-input-and-output)
12. [Java Applets — History and Context](#12-java-applets--history-and-context)
13. [Putting It All Together](#13-putting-it-all-together)
14. [Exercises](#14-exercises)

---

## 1. What Is I/O?

**I/O** stands for **Input/Output** — the flow of data into and out of a program. So far your programs have read input from the keyboard (`Scanner`) and written output to the console (`System.out`). That's I/O too, but there's much more:

| Source / Destination | Example |
|---|---|
| Keyboard | User typing at the console |
| Console / Terminal | `System.out.println(...)` |
| Files | Reading a config file, writing a log |
| Network sockets | Sending/receiving data over the internet |
| Byte arrays in memory | Processing data without touching the disk |
| Other programs / pipes | Reading output from another process |

Java handles all of these through a unified abstraction: **streams**.

---

## 2. The I/O Stream Model

A **stream** is a sequence of data flowing in one direction. There are two kinds:

- **Input stream** — data flows *into* your program (reading).
- **Output stream** — data flows *out of* your program (writing).

And two categories based on what unit of data they work with:

| Category | Unit | When to use |
|---|---|---|
| **Byte streams** | 8-bit bytes | Images, audio, video, binary files |
| **Character streams** | 16-bit Unicode characters | Text files, CSV, JSON, logs |

The entire Java I/O system is built from these four combinations. All stream classes live in `java.io`.

```
java.io
├── InputStream       ← abstract base for all byte input
│   ├── FileInputStream
│   └── ByteArrayInputStream
├── OutputStream      ← abstract base for all byte output
│   ├── FileOutputStream
│   └── ByteArrayOutputStream
├── Reader            ← abstract base for all character input
│   ├── FileReader
│   └── StringReader
└── Writer            ← abstract base for all character output
    ├── FileWriter
    └── StringWriter
```

You rarely use the base classes directly — you work with specific subclasses, and often **wrap** them in buffered or decorator streams for better performance.

---

## 3. Byte Streams

`FileInputStream` and `FileOutputStream` read and write raw bytes. They are the foundation for all binary file I/O.

### Writing Bytes

```java
import java.io.FileOutputStream;
import java.io.IOException;

public class ByteWriteDemo {
    public static void main(String[] args) {
        byte[] data = {72, 101, 108, 108, 111};   // ASCII for "Hello"

        try (FileOutputStream fos = new FileOutputStream("output.dat")) {
            fos.write(data);
            System.out.println("Bytes written successfully.");
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

### Reading Bytes

```java
import java.io.FileInputStream;
import java.io.IOException;

public class ByteReadDemo {
    public static void main(String[] args) {
        try (FileInputStream fis = new FileInputStream("output.dat")) {
            int byteValue;
            while ((byteValue = fis.read()) != -1) {   // -1 means end of file
                System.out.print((char) byteValue);
            }
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

Output:
```
Hello
```

### The try-with-resources Statement

Notice the `try (...)` syntax — this is called **try-with-resources** (introduced in Java 7). It automatically closes the stream when the block ends, even if an exception is thrown. You should always use this pattern with streams to avoid resource leaks.

```java
try (FileInputStream fis = new FileInputStream("file.txt")) {
    // use fis
}   // fis.close() is called automatically here
```

Any class that implements `AutoCloseable` (including all stream classes) works with try-with-resources.

---

## 4. Character Streams

For text data, always use character streams — they handle Unicode correctly and are simpler to work with than byte streams for text.

### FileWriter — Writing Text

```java
import java.io.FileWriter;
import java.io.IOException;

public class TextWriteDemo {
    public static void main(String[] args) {
        try (FileWriter writer = new FileWriter("notes.txt")) {
            writer.write("Hello, Java I/O!\n");
            writer.write("This is a text file.\n");
            writer.write("Line three.\n");
            System.out.println("Text written successfully.");
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

By default `FileWriter` **overwrites** the file. To **append** instead:

```java
new FileWriter("notes.txt", true)   // true = append mode
```

### FileReader — Reading Text

```java
import java.io.FileReader;
import java.io.IOException;

public class TextReadDemo {
    public static void main(String[] args) {
        try (FileReader reader = new FileReader("notes.txt")) {
            int ch;
            while ((ch = reader.read()) != -1) {
                System.out.print((char) ch);
            }
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

Reading one character at a time works but is slow. In practice, you always wrap readers and writers in **buffered** streams.

---

## 5. Buffered Streams

Unbuffered streams make one system call per byte or character — extremely slow for large files. **Buffered streams** read or write a large chunk at a time (the buffer), dramatically improving performance.

### BufferedReader and BufferedWriter

```java
import java.io.*;

public class BufferedDemo {
    public static void main(String[] args) throws IOException {

        // Writing with buffer
        try (BufferedWriter writer = new BufferedWriter(new FileWriter("story.txt"))) {
            writer.write("Once upon a time...");
            writer.newLine();   // platform-independent line separator
            writer.write("There was a Java programmer.");
            writer.newLine();
            writer.write("They learned I/O and lived happily ever after.");
        }

        // Reading with buffer — line by line
        try (BufferedReader reader = new BufferedReader(new FileReader("story.txt"))) {
            String line;
            int lineNumber = 1;
            while ((line = reader.readLine()) != null) {   // null means end of file
                System.out.printf("%2d: %s%n", lineNumber++, line);
            }
        }
    }
}
```

Output:
```
 1: Once upon a time...
 2: There was a Java programmer.
 3: They learned I/O and lived happily ever after.
```

`readLine()` is the method you'll use most often — it reads one complete line at a time, stripping the newline character automatically.

### The Wrapping Pattern

Java's I/O classes are designed to be layered (a **decorator pattern**):

```java
new BufferedReader(new FileReader("file.txt"))
//  ^buffered        ^reads characters from file
```

You can add more layers:

```java
new PrintWriter(new BufferedWriter(new FileWriter("output.txt")))
//  ^printf support  ^buffered        ^writes to file
```

---

## 6. Reading and Writing Text Files

This is the most common I/O task. Here are clean, complete patterns for everyday use.

### Writing a List of Lines

```java
import java.io.*;
import java.util.List;

public class FileUtils {

    public static void writeLines(String filename, List<String> lines) throws IOException {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(filename))) {
            for (String line : lines) {
                writer.write(line);
                writer.newLine();
            }
        }
    }

    public static List<String> readLines(String filename) throws IOException {
        List<String> lines = new java.util.ArrayList<>();
        try (BufferedReader reader = new BufferedReader(new FileReader(filename))) {
            String line;
            while ((line = reader.readLine()) != null) {
                lines.add(line);
            }
        }
        return lines;
    }
}
```

```java
// Write
List<String> data = List.of("Alice,30", "Bob,25", "Carlos,35");
FileUtils.writeLines("people.csv", data);

// Read back
List<String> loaded = FileUtils.readLines("people.csv");
loaded.forEach(System.out::println);
```

### PrintWriter — Formatted Text Output

`PrintWriter` wraps a `Writer` and adds `print`, `println`, and `printf` — just like `System.out`:

```java
try (PrintWriter pw = new PrintWriter(new BufferedWriter(new FileWriter("report.txt")))) {
    pw.println("=== Sales Report ===");
    pw.printf("%-15s %10s%n", "Product", "Revenue");
    pw.printf("%-15s %10.2f%n", "Laptop",  49999.95);
    pw.printf("%-15s %10.2f%n", "Phone",   28750.00);
    pw.printf("%-15s %10.2f%n", "Tablet",  15200.50);
}
```

---

## 7. Reading and Writing Binary Files

For binary data (images, audio, custom formats), use `DataInputStream` and `DataOutputStream`. These wrap byte streams and let you read/write Java primitives directly.

### Writing Binary Data

```java
import java.io.*;

public class BinaryWriteDemo {
    public static void main(String[] args) throws IOException {
        try (DataOutputStream dos = new DataOutputStream(
                new BufferedOutputStream(new FileOutputStream("data.bin")))) {

            dos.writeInt(42);
            dos.writeDouble(3.14159);
            dos.writeBoolean(true);
            dos.writeUTF("Hello Binary");   // writes a length-prefixed UTF string
        }
        System.out.println("Binary data written.");
    }
}
```

### Reading Binary Data

You must read values back **in the same order** they were written:

```java
try (DataInputStream dis = new DataInputStream(
        new BufferedInputStream(new FileInputStream("data.bin")))) {

    int    number  = dis.readInt();
    double pi      = dis.readDouble();
    boolean flag   = dis.readBoolean();
    String message = dis.readUTF();

    System.out.println(number);    // 42
    System.out.println(pi);        // 3.14159
    System.out.println(flag);      // true
    System.out.println(message);   // Hello Binary
}
```

---

## 8. The File Class

`java.io.File` represents a file or directory path. It does not read or write content — it manages the path and metadata.

```java
import java.io.File;

File file = new File("notes.txt");

System.out.println(file.exists());          // true or false
System.out.println(file.getName());         // notes.txt
System.out.println(file.getAbsolutePath()); // full path
System.out.println(file.length());          // size in bytes
System.out.println(file.canRead());         // true/false
System.out.println(file.isFile());          // true
System.out.println(file.isDirectory());     // false
```

### Creating and Deleting Files

```java
File newFile = new File("temp.txt");
boolean created = newFile.createNewFile();   // creates the file if it doesn't exist
System.out.println("Created: " + created);

newFile.delete();   // delete the file
```

### Working with Directories

```java
File dir = new File("myFolder");
dir.mkdir();    // creates a single directory
dir.mkdirs();   // creates the directory and any necessary parent directories

// List contents
File[] contents = dir.listFiles();
if (contents != null) {
    for (File f : contents) {
        System.out.println((f.isDirectory() ? "[DIR] " : "[FILE] ") + f.getName());
    }
}
```

---

## 9. The Modern NIO.2 API

Java 7 introduced `java.nio.file` — a modern, more powerful alternative to `java.io.File`. The key classes are `Path`, `Paths`, and `Files`.

### Path and Paths

`Path` represents a file or directory location. It's more flexible and robust than `File`:

```java
import java.nio.file.*;

Path path = Paths.get("notes.txt");
Path absPath = path.toAbsolutePath();

System.out.println(path.getFileName());   // notes.txt
System.out.println(path.getParent());     // parent directory
System.out.println(absPath);              // full absolute path
```

### Files — Reading and Writing in One Line

The `Files` utility class makes common operations extremely concise:

```java
import java.nio.file.*;
import java.util.List;

// Write all lines at once
List<String> lines = List.of("Line 1", "Line 2", "Line 3");
Files.write(Paths.get("output.txt"), lines);

// Read all lines at once
List<String> read = Files.readAllLines(Paths.get("output.txt"));
read.forEach(System.out::println);

// Read entire file as one string
String content = Files.readString(Paths.get("output.txt"));   // Java 11+
System.out.println(content);

// Write a single string
Files.writeString(Paths.get("message.txt"), "Hello NIO.2!");  // Java 11+
```

### Checking and Managing Files

```java
Path p = Paths.get("myfile.txt");

Files.exists(p)               // does it exist?
Files.isReadable(p)           // can we read it?
Files.isDirectory(p)          // is it a directory?
Files.size(p)                 // size in bytes
Files.copy(source, target)    // copy a file
Files.move(source, target)    // move or rename a file
Files.delete(p)               // delete
```

### Walking a Directory Tree

```java
import java.nio.file.*;
import java.io.IOException;

Files.walk(Paths.get("myFolder"))
    .filter(Files::isRegularFile)
    .forEach(System.out::println);
```

This prints every file in `myFolder` and all its subdirectories — without writing a recursive method yourself.

### NIO.2 vs java.io.File

| Task | Old (java.io.File) | New (NIO.2) |
|---|---|---|
| Read all lines | Loop with BufferedReader | `Files.readAllLines(path)` |
| Write all lines | Loop with BufferedWriter | `Files.write(path, lines)` |
| Check existence | `file.exists()` | `Files.exists(path)` |
| Copy file | Manual byte loop | `Files.copy(src, dest)` |
| Walk directory | Recursive method | `Files.walk(path)` |

For new code, prefer NIO.2 — it's cleaner and more powerful.

---

## 10. Object Serialization

**Serialization** converts a Java object into a sequence of bytes that can be saved to a file or sent over a network. **Deserialization** is the reverse — reconstructing the object from those bytes.

### Making a Class Serializable

A class must implement the `Serializable` marker interface (it has no methods — it just tells Java the class is safe to serialize):

```java
import java.io.Serializable;

public class Student implements Serializable {

    private static final long serialVersionUID = 1L;   // version identifier

    private String name;
    private int id;
    private double gpa;

    public Student(String name, int id, double gpa) {
        this.name = name;
        this.id   = id;
        this.gpa  = gpa;
    }

    @Override
    public String toString() {
        return String.format("Student[%d] %s - GPA: %.2f", id, name, gpa);
    }
}
```

The `serialVersionUID` is a version number. If you later change the class, update this number to avoid compatibility problems when reading old files.

### Serializing (Writing an Object)

```java
import java.io.*;

Student s = new Student("Alice", 1001, 3.85);

try (ObjectOutputStream oos = new ObjectOutputStream(
        new BufferedOutputStream(new FileOutputStream("student.ser")))) {
    oos.writeObject(s);
    System.out.println("Object saved: " + s);
}
```

### Deserializing (Reading an Object)

```java
try (ObjectInputStream ois = new ObjectInputStream(
        new BufferedInputStream(new FileInputStream("student.ser")))) {
    Student loaded = (Student) ois.readObject();
    System.out.println("Object loaded: " + loaded);
}
```

Output:
```
Object saved: Student[1001] Alice - GPA: 3.85
Object loaded: Student[1001] Alice - GPA: 3.85
```

### Serializing a List of Objects

```java
List<Student> students = new ArrayList<>();
students.add(new Student("Alice",  1001, 3.85));
students.add(new Student("Bob",    1002, 3.42));
students.add(new Student("Carlos", 1003, 3.91));

// Save
try (ObjectOutputStream oos = new ObjectOutputStream(
        new FileOutputStream("students.ser"))) {
    oos.writeObject(students);
}

// Load
try (ObjectInputStream ois = new ObjectInputStream(
        new FileInputStream("students.ser"))) {
    List<Student> loaded = (List<Student>) ois.readObject();
    loaded.forEach(System.out::println);
}
```

### The `transient` Keyword

Mark a field `transient` to exclude it from serialization (useful for passwords, cached data, or non-serializable objects):

```java
private transient String password;   // will NOT be saved to disk
```

---

## 11. Standard Input and Output

You've been using these since Module 1, but now you understand streams — so let's see what they really are.

| Stream | Type | Description |
|---|---|---|
| `System.in` | `InputStream` | Standard input (keyboard by default) |
| `System.out` | `PrintStream` | Standard output (console) |
| `System.err` | `PrintStream` | Standard error output (console, usually in red) |

`System.in` is a raw `InputStream`. That's why you wrap it:

```java
Scanner scanner = new Scanner(System.in);
// or
BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
```

`System.err` is for error messages — it stays separate from normal output so errors can be logged or redirected independently:

```java
System.out.println("Normal output");
System.err.println("Error message");   // usually appears in red in IDEs
```

### Redirecting Standard Streams

You can redirect `System.out` to a file programmatically:

```java
PrintStream fileOut = new PrintStream(new FileOutputStream("log.txt"));
System.setOut(fileOut);

System.out.println("This goes to log.txt, not the console.");
```

---

## 12. Java Applets — History and Context

The Java curriculum includes **Applets** as a historical topic. Understanding their context is important for any Java developer.

### What Were Applets?

Java **Applets** were small Java programs designed to run inside a web browser. When Java launched in 1995, this was revolutionary — dynamic, interactive content running right in the browser window.

An Applet extended the `JApplet` class (a special container), and browsers used a **Java Plugin** to execute the bytecode:

```java
// Historical example — do NOT use this today
import java.applet.Applet;
import java.awt.Graphics;

public class HelloApplet extends Applet {
    @Override
    public void paint(Graphics g) {
        g.drawString("Hello, World!", 50, 50);
    }
}
```

### Why Applets Are Dead

By the early 2010s, applets were effectively obsolete:

- **Security vulnerabilities** — the Java browser plugin was repeatedly exploited.
- **Browser support dropped** — Chrome, Firefox, and Edge all removed support.
- **Better alternatives emerged** — HTML5, CSS3, and JavaScript made interactive web content possible without plugins.
- **Official deprecation** — Java 9 deprecated applets; **Java 17 removed them entirely**.

### What to Use Instead

| Old Applet Use Case | Modern Alternative |
|---|---|
| Interactive web content | JavaScript / TypeScript with HTML5 Canvas |
| Rich desktop applications | Java Swing or JavaFX |
| Web applications with Java backend | Spring Boot with REST APIs |
| Embedded Java UI | JavaFX WebView |

### The Applet Lifecycle (For Reference)

Even though applets are gone, the lifecycle pattern appears in other frameworks (Android, JavaFX), so it's worth knowing:

| Method | When called |
|---|---|
| `init()` | Once when the applet loads — like a constructor |
| `start()` | When the browser tab becomes active |
| `paint(Graphics g)` | When the applet needs to draw itself |
| `stop()` | When the tab is hidden or the user leaves |
| `destroy()` | When the applet is permanently unloaded |

This init → start → stop → destroy pattern is the foundation of component lifecycles in many modern frameworks.

---

## 13. Putting It All Together

Here's a complete student record system that saves and loads data using both text files and serialization:

```java
import java.io.*;
import java.nio.file.*;
import java.util.*;

public class StudentRecordSystem {

    static final String TEXT_FILE   = "students.csv";
    static final String BINARY_FILE = "students.ser";

    // --- Model ---
    static class Student implements Serializable {
        private static final long serialVersionUID = 1L;
        String name;
        int    id;
        double gpa;

        Student(String name, int id, double gpa) {
            this.name = name; this.id = id; this.gpa = gpa;
        }

        static Student fromCSV(String line) {
            String[] parts = line.split(",");
            return new Student(parts[0].trim(),
                               Integer.parseInt(parts[1].trim()),
                               Double.parseDouble(parts[2].trim()));
        }

        String toCSV() {
            return name + "," + id + "," + gpa;
        }

        @Override
        public String toString() {
            return String.format("%-12s ID: %04d  GPA: %.2f", name, id, gpa);
        }
    }

    // --- Text I/O ---
    static void saveToCSV(List<Student> students) throws IOException {
        List<String> lines = new ArrayList<>();
        for (Student s : students) lines.add(s.toCSV());
        Files.write(Paths.get(TEXT_FILE), lines);
        System.out.println("Saved " + students.size() + " records to " + TEXT_FILE);
    }

    static List<Student> loadFromCSV() throws IOException {
        List<Student> students = new ArrayList<>();
        for (String line : Files.readAllLines(Paths.get(TEXT_FILE))) {
            if (!line.isBlank()) students.add(Student.fromCSV(line));
        }
        System.out.println("Loaded " + students.size() + " records from " + TEXT_FILE);
        return students;
    }

    // --- Binary I/O ---
    static void saveToBinary(List<Student> students) throws IOException {
        try (ObjectOutputStream oos = new ObjectOutputStream(
                new FileOutputStream(BINARY_FILE))) {
            oos.writeObject(students);
        }
        System.out.println("Serialized " + students.size() + " records to " + BINARY_FILE);
    }

    @SuppressWarnings("unchecked")
    static List<Student> loadFromBinary() throws IOException, ClassNotFoundException {
        try (ObjectInputStream ois = new ObjectInputStream(
                new FileInputStream(BINARY_FILE))) {
            List<Student> students = (List<Student>) ois.readObject();
            System.out.println("Deserialized " + students.size() + " records.");
            return students;
        }
    }

    // --- Main ---
    public static void main(String[] args) throws IOException, ClassNotFoundException {

        List<Student> students = new ArrayList<>();
        students.add(new Student("Alice",   1001, 3.85));
        students.add(new Student("Bob",     1002, 3.42));
        students.add(new Student("Carlos",  1003, 3.91));
        students.add(new Student("Diana",   1004, 2.75));
        students.add(new Student("Ethan",   1005, 3.60));

        System.out.println("=== Original Records ===");
        students.forEach(System.out::println);

        // Save and reload via CSV
        System.out.println("\n--- CSV Round-Trip ---");
        saveToCSV(students);
        List<Student> fromCSV = loadFromCSV();
        fromCSV.forEach(System.out::println);

        // Save and reload via serialization
        System.out.println("\n--- Serialization Round-Trip ---");
        saveToBinary(students);
        List<Student> fromBinary = loadFromBinary();
        fromBinary.forEach(System.out::println);

        // Stats using NIO.2
        System.out.println("\n--- File Info ---");
        System.out.printf("CSV file size:    %d bytes%n", Files.size(Paths.get(TEXT_FILE)));
        System.out.printf("Binary file size: %d bytes%n", Files.size(Paths.get(BINARY_FILE)));
    }
}
```

---

## 14. Exercises

---

### Exercise 1 – File Writer and Reader

Write a program that:
1. Asks the user to enter up to 5 sentences (one per line, blank line to stop).
2. Saves them to a file called `journal.txt`.
3. Reads the file back and prints each line numbered.
4. Prints the total number of lines and total characters saved.

---

### Exercise 2 – CSV Contact Book

Build a contact book that saves to and loads from a CSV file (`contacts.csv`):

Each line in the file should have the format: `Name,Phone,Email`

Your program should:
1. Load existing contacts from the file on startup (if it exists).
2. Display a menu: Add Contact / List All / Search by Name / Save and Exit.
3. Search should be case-insensitive and print all matching contacts.
4. Save writes all contacts back to the CSV on exit.

---

### Exercise 3 – Word Count Tool

Write a program that:
1. Accepts a filename from the user.
2. Reads the file using `BufferedReader`.
3. Counts and prints:
   - Total lines
   - Total words
   - Total characters (including spaces)
   - The 5 most frequently used words (use a `HashMap` and sort by value)

Handle `FileNotFoundException` with a friendly message.

---

### Exercise 4 – NIO.2 File Manager

Write a program using only the NIO.2 API (`Files`, `Paths`) that:
1. Creates a directory called `workspace`.
2. Creates three text files inside it: `todo.txt`, `notes.txt`, `ideas.txt`.
3. Writes some content to each.
4. Lists all files in the directory with their sizes.
5. Copies `todo.txt` to `todo_backup.txt`.
6. Deletes the original `todo.txt`.
7. Verifies the deletion with `Files.exists()`.

---

### Exercise 5 – Object Serialization

Create a `Product` class (with `name`, `category`, `price`, `quantity`) that implements `Serializable`.

Write a program that:
1. Creates a list of 5 products.
2. Saves the entire list to `products.ser`.
3. Loads it back and prints all products.
4. Adds one more product to the loaded list.
5. Saves the updated list and confirms the count is now 6.

---

### Exercise 6 – Append Log *(Challenge)*

Build a simple logging system:

Create a `Logger` class with:
- A `log(String level, String message)` method that appends a timestamped entry to `app.log`.
- Format: `[YYYY-MM-DD HH:mm:ss] [LEVEL] message`
- Use `java.time.LocalDateTime` for the timestamp.
- Log levels: `INFO`, `WARN`, `ERROR`.

Also add a `printLog()` method that reads and prints the entire log file.

In `main`, call `log()` several times with different levels and messages, then print the full log.

---

### Exercise 7 – Binary Student Records *(Challenge)*

Write a program that stores student records in a custom binary format using `DataOutputStream`:

For each student write: `int id`, `double gpa`, `UTF name` (in that order).

Then read them back with `DataInputStream`.

Requirements:
1. Write 5 students to `records.bin`.
2. Read them back and display each record.
3. Compute and display the average GPA from the binary file alone (do not use the original list).
4. Handle `EOFException` to detect the end of the file cleanly.

---

## Module Summary

Here's what you covered in Module 4:

- **I/O streams** are the universal abstraction for data flowing in and out of a program.
- **Byte streams** (`FileInputStream`, `FileOutputStream`) handle raw binary data.
- **Character streams** (`FileReader`, `FileWriter`) handle text with Unicode support.
- **Buffered streams** wrap other streams for much better performance — always use them.
- `BufferedReader.readLine()` and `PrintWriter.println()` are the workhorses of text file I/O.
- **try-with-resources** automatically closes streams — always use it.
- The **NIO.2 API** (`Files`, `Paths`) is the modern alternative — cleaner and more powerful than `java.io.File`.
- **Serialization** converts objects to bytes for storage or transmission; classes must implement `Serializable`.
- `transient` excludes sensitive fields from serialization.
- **Applets** are fully deprecated and removed from modern Java — replaced by Swing, JavaFX, and web technologies.
- The applet lifecycle pattern (init → start → stop → destroy) lives on in modern frameworks.

---

**Next up:** Module 5 – Introduction to Java Database Connectivity (JDBC)

---

*CS 1103 Programming 2 (Java) — Module 4 of 8*
