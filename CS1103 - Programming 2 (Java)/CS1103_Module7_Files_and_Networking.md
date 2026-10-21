# CS 1103 – Programming 2 (Java)
## Module 7: Files and Networking

---

## Table of Contents

1. [Overview — Files and Networking Together](#1-overview--files-and-networking-together)
2. [Advanced File Handling with NIO.2](#2-advanced-file-handling-with-nio2)
3. [Watching a Directory for Changes](#3-watching-a-directory-for-changes)
4. [Random Access Files](#4-random-access-files)
5. [Introduction to Networking](#5-introduction-to-networking)
6. [URLs and HTTP Connections](#6-urls-and-http-connections)
7. [Sockets — TCP Communication](#7-sockets--tcp-communication)
8. [Building a TCP Server](#8-building-a-tcp-server)
9. [Building a TCP Client](#9-building-a-tcp-client)
10. [Multithreaded Server](#10-multithreaded-server)
11. [UDP Communication](#11-udp-communication)
12. [Combining Files and Networking](#12-combining-files-and-networking)
13. [Putting It All Together](#13-putting-it-all-together)
14. [Exercises](#14-exercises)

---

## 1. Overview — Files and Networking Together

In Module 4 you learned the foundations of Java I/O — streams, text files, binary files, and serialization. This module goes deeper and wider:

**Files (Advanced):**
- Navigating and searching directory trees
- Watching a folder for live file changes
- Reading and writing to specific positions in a file (random access)

**Networking:**
- Fetching data from URLs (HTTP)
- Two-way communication between programs over a network using **sockets**
- Building a server that handles multiple clients simultaneously

Files and networking are closely linked in real applications — a server reads configuration files, logs to disk, and serves files to clients over the network. By the end of this module you'll understand how both sides work.

---

## 2. Advanced File Handling with NIO.2

You saw the basics of NIO.2 in Module 4. Here we go further with directory traversal, file searching, and metadata.

### Walking a Directory Tree

`Files.walk()` produces a `Stream<Path>` of every file and directory under a given root — recursively:

```java
import java.nio.file.*;
import java.io.IOException;

public class DirectoryWalker {
    public static void main(String[] args) throws IOException {
        Path root = Paths.get(".");   // current directory

        Files.walk(root)
            .filter(Files::isRegularFile)
            .filter(p -> p.toString().endsWith(".java"))
            .forEach(p -> System.out.println(p.toAbsolutePath()));
    }
}
```

This finds every `.java` file under the current directory and all subdirectories — in one stream pipeline.

### Walking with Depth Limit

```java
Files.walk(root, 2)   // only go 2 levels deep
    .filter(Files::isDirectory)
    .forEach(System.out::println);
```

### Finding Files with `Files.find()`

More expressive than `walk()` — takes a `BiPredicate<Path, BasicFileAttributes>`:

```java
Files.find(root, 10,
    (path, attrs) -> attrs.isRegularFile() && attrs.size() > 1024)
    .forEach(p -> System.out.println(p + " (" + p.toFile().length() + " bytes)"));
```

This finds all regular files larger than 1 KB up to 10 levels deep.

### Reading File Metadata

```java
import java.nio.file.attribute.BasicFileAttributes;

Path path = Paths.get("notes.txt");
BasicFileAttributes attrs = Files.readAttributes(path, BasicFileAttributes.class);

System.out.println("Size:         " + attrs.size() + " bytes");
System.out.println("Created:      " + attrs.creationTime());
System.out.println("Last modified:" + attrs.lastModifiedTime());
System.out.println("Is directory: " + attrs.isDirectory());
System.out.println("Is regular:   " + attrs.isRegularFile());
```

### Copying and Moving Files

```java
Path source = Paths.get("original.txt");
Path target = Paths.get("backup/copy.txt");

// Copy — fails if target exists
Files.copy(source, target);

// Copy — replace target if it exists
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);

// Move (rename)
Files.move(source, Paths.get("renamed.txt"), StandardCopyOption.REPLACE_EXISTING);
```

### Deleting Files and Directories

```java
Files.delete(path);                 // throws NoSuchFileException if absent
Files.deleteIfExists(path);         // silent if absent

// Delete a non-empty directory tree
Files.walk(Paths.get("tempFolder"))
    .sorted(Comparator.reverseOrder())   // delete contents before the folder
    .forEach(p -> {
        try { Files.delete(p); } catch (IOException e) {}
    });
```

---

## 3. Watching a Directory for Changes

`WatchService` lets your program **monitor a directory** and be notified when files are created, modified, or deleted — without polling.

```java
import java.nio.file.*;

public class DirectoryWatcher {
    public static void main(String[] args) throws Exception {

        Path dir = Paths.get("watched");
        Files.createDirectories(dir);

        WatchService watcher = FileSystems.getDefault().newWatchService();

        dir.register(watcher,
            StandardWatchEventKinds.ENTRY_CREATE,
            StandardWatchEventKinds.ENTRY_MODIFY,
            StandardWatchEventKinds.ENTRY_DELETE);

        System.out.println("Watching: " + dir.toAbsolutePath());
        System.out.println("(Create, edit, or delete files in the folder)");

        while (true) {
            WatchKey key = watcher.take();   // blocks until an event occurs

            for (WatchEvent<?> event : key.pollEvents()) {
                WatchEvent.Kind<?> kind = event.kind();
                Path filename = (Path) event.context();

                System.out.printf("[%s] %s%n", kind.name(), filename);
            }

            boolean valid = key.reset();
            if (!valid) break;   // directory no longer accessible
        }
    }
}
```

Run this program, then create or modify files in the `watched` folder — you'll see the events print in real time.

---

## 4. Random Access Files

`BufferedReader` reads a file sequentially from start to finish. **`RandomAccessFile`** lets you jump to any byte position in the file — reading or writing anywhere:

```java
import java.io.RandomAccessFile;
import java.io.IOException;

public class RandomAccessDemo {
    public static void main(String[] args) throws IOException {

        // Create a file and write some data
        try (RandomAccessFile raf = new RandomAccessFile("data.bin", "rw")) {

            // Write 5 integers (each int = 4 bytes)
            for (int i = 1; i <= 5; i++) {
                raf.writeInt(i * 100);   // 100, 200, 300, 400, 500
            }

            System.out.println("File length: " + raf.length() + " bytes");  // 20

            // Jump to the 3rd integer (index 2, offset = 2 * 4 = 8 bytes)
            raf.seek(8);
            System.out.println("Value at position 8: " + raf.readInt());   // 300

            // Overwrite the 2nd integer
            raf.seek(4);
            raf.writeInt(999);

            // Read all values back
            raf.seek(0);   // go back to the beginning
            for (int i = 0; i < 5; i++) {
                System.out.println("Position " + (i * 4) + ": " + raf.readInt());
            }
        }
    }
}
```

Output:
```
File length: 20 bytes
Value at position 8: 300
Position 0: 100
Position 4: 999
Position 8: 300
Position 12: 400
Position 16: 500
```

### File Open Modes

| Mode | Meaning |
|---|---|
| `"r"` | Read only |
| `"rw"` | Read and write |
| `"rws"` | Read/write + sync content and metadata immediately |
| `"rwd"` | Read/write + sync content immediately |

### Use Case — Fixed-Size Records

`RandomAccessFile` is ideal for databases of fixed-size records — you can calculate the exact byte offset of any record and jump to it directly:

```java
static final int RECORD_SIZE = 4 + 20 + 8;   // int id + 20-char name + double gpa = 32 bytes

static void writeRecord(RandomAccessFile raf, int id, String name, double gpa)
        throws IOException {
    raf.writeInt(id);
    // Pad or truncate name to exactly 20 chars
    byte[] nameBytes = new byte[20];
    byte[] src = name.getBytes();
    System.arraycopy(src, 0, nameBytes, 0, Math.min(src.length, 20));
    raf.write(nameBytes);
    raf.writeDouble(gpa);
}

static void readRecord(RandomAccessFile raf, int recordIndex) throws IOException {
    raf.seek((long) recordIndex * RECORD_SIZE);
    int    id   = raf.readInt();
    byte[] nameBytes = new byte[20];
    raf.read(nameBytes);
    String name = new String(nameBytes).trim();
    double gpa  = raf.readDouble();
    System.out.printf("ID: %d  Name: %-20s  GPA: %.2f%n", id, name, gpa);
}
```

---

## 5. Introduction to Networking

When two programs communicate over a network, they follow a set of rules called a **protocol**. The most important ones in Java networking:

| Protocol | Layer | Description |
|---|---|---|
| **IP** (Internet Protocol) | Network | Routes packets to the right machine using IP addresses |
| **TCP** (Transmission Control Protocol) | Transport | Reliable, ordered, connection-based stream of bytes |
| **UDP** (User Datagram Protocol) | Transport | Fast, connectionless, no delivery guarantee |
| **HTTP/HTTPS** | Application | Request-response protocol for the web |

### IP Addresses and Ports

- An **IP address** identifies a machine on a network (e.g. `192.168.1.5` or `127.0.0.1` for localhost).
- A **port** identifies a specific service on that machine (1–65535).
  - Port 80 = HTTP, Port 443 = HTTPS, Port 22 = SSH.
  - Ports above 1024 are free for your own applications.

A connection is identified by: **IP address + Port**. Think of the IP as a building address and the port as the apartment number.

### The Client-Server Model

```
Client                          Server
  │                               │
  │── connect to IP:Port ────────>│
  │                               │── accept connection
  │<─────────── response ─────────│
  │── request ───────────────────>│
  │<─────────── response ─────────│
  │── close ─────────────────────>│
```

The **server** waits (listens) on a port. The **client** initiates the connection. Java's `java.net` package provides everything you need for both sides.

---

## 6. URLs and HTTP Connections

The simplest way to fetch data from the internet in Java is through the `URL` class.

### Reading a Web Page or API

```java
import java.net.*;
import java.io.*;

public class UrlDemo {
    public static void main(String[] args) throws Exception {

        URL url = new URL("https://jsonplaceholder.typicode.com/todos/1");
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();

        conn.setRequestMethod("GET");
        conn.setRequestProperty("Accept", "application/json");

        int responseCode = conn.getResponseCode();
        System.out.println("Response Code: " + responseCode);   // 200

        try (BufferedReader reader = new BufferedReader(
                new InputStreamReader(conn.getInputStream()))) {

            String line;
            StringBuilder response = new StringBuilder();
            while ((line = reader.readLine()) != null) {
                response.append(line);
            }
            System.out.println("Response: " + response);
        }

        conn.disconnect();
    }
}
```

Output:
```
Response Code: 200
Response: {  "userId": 1,  "id": 1,  "title": "delectus aut autem",  "completed": false}
```

### Sending a POST Request

```java
URL url = new URL("https://jsonplaceholder.typicode.com/posts");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();

conn.setRequestMethod("POST");
conn.setRequestProperty("Content-Type", "application/json");
conn.setDoOutput(true);   // allow writing to the connection

String json = "{\"title\":\"foo\",\"body\":\"bar\",\"userId\":1}";

try (OutputStream os = conn.getOutputStream();
     OutputStreamWriter writer = new OutputStreamWriter(os)) {
    writer.write(json);
}

System.out.println("Response: " + conn.getResponseCode());   // 201 Created
conn.disconnect();
```

### HTTP Response Codes

| Code | Meaning |
|---|---|
| `200` | OK — request succeeded |
| `201` | Created — resource was created |
| `400` | Bad Request — invalid input |
| `401` | Unauthorized — authentication required |
| `404` | Not Found — resource doesn't exist |
| `500` | Internal Server Error — server-side problem |

---

## 7. Sockets — TCP Communication

A **socket** is one end of a two-way communication channel between two programs. Think of it like a phone call — once connected, both sides can talk and listen.

Java's TCP socket classes:

| Class | Role |
|---|---|
| `ServerSocket` | Listens on a port, accepts incoming connections (server side) |
| `Socket` | Represents one connection endpoint (both client and server use this) |

Once a connection is established, both ends get a `Socket`. Reading and writing through a socket works just like reading and writing any I/O stream.

```
Server                          Client
ServerSocket(8080)              
  └── accept() ─────────────── new Socket("localhost", 8080)
       └── Socket              └── Socket
           ├── InputStream         ├── InputStream
           └── OutputStream        └── OutputStream
```

---

## 8. Building a TCP Server

```java
import java.net.*;
import java.io.*;

public class SimpleServer {

    static final int PORT = 8080;

    public static void main(String[] args) {
        System.out.println("Server starting on port " + PORT + "...");

        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            System.out.println("Waiting for a client...");

            // accept() blocks until a client connects
            try (Socket clientSocket = serverSocket.accept();
                 BufferedReader in  = new BufferedReader(
                     new InputStreamReader(clientSocket.getInputStream()));
                 PrintWriter    out = new PrintWriter(
                     clientSocket.getOutputStream(), true)) {

                System.out.println("Client connected: "
                    + clientSocket.getInetAddress().getHostAddress());

                // Read a message from the client
                String message = in.readLine();
                System.out.println("Received: " + message);

                // Send a response
                out.println("Echo: " + message);
                System.out.println("Response sent. Connection closing.");
            }

        } catch (IOException e) {
            System.out.println("Server error: " + e.getMessage());
        }
    }
}
```

Key points:
- `new ServerSocket(PORT)` — opens the server on port 8080.
- `serverSocket.accept()` — **blocks** until a client connects, then returns a `Socket`.
- `PrintWriter(..., true)` — the `true` enables **auto-flush** so each `println` is sent immediately.
- The server echoes the message back with "Echo: " prepended.

---

## 9. Building a TCP Client

```java
import java.net.*;
import java.io.*;

public class SimpleClient {

    static final String HOST = "localhost";
    static final int    PORT = 8080;

    public static void main(String[] args) {
        System.out.println("Connecting to " + HOST + ":" + PORT + "...");

        try (Socket socket    = new Socket(HOST, PORT);
             PrintWriter out  = new PrintWriter(socket.getOutputStream(), true);
             BufferedReader in = new BufferedReader(
                 new InputStreamReader(socket.getInputStream()))) {

            System.out.println("Connected.");

            // Send a message to the server
            out.println("Hello, Server!");

            // Read the server's response
            String response = in.readLine();
            System.out.println("Server says: " + response);

        } catch (IOException e) {
            System.out.println("Client error: " + e.getMessage());
        }
    }
}
```

### Running the Example

1. Start `SimpleServer` first — it will print "Waiting for a client..."
2. Start `SimpleClient` — it sends "Hello, Server!" and prints the echo response.

Server output:
```
Server starting on port 8080...
Waiting for a client...
Client connected: 127.0.0.1
Received: Hello, Server!
Response sent. Connection closing.
```

Client output:
```
Connecting to localhost:8080...
Connected.
Server says: Echo: Hello, Server!
```

### Conversation Loop

A more realistic server/client exchange with multiple messages:

```java
// Client side — conversation loop
Scanner keyboard = new Scanner(System.in);

try (Socket socket   = new Socket("localhost", 8080);
     PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
     BufferedReader in = new BufferedReader(
         new InputStreamReader(socket.getInputStream()))) {

    String userInput;
    while (!(userInput = keyboard.nextLine()).equalsIgnoreCase("quit")) {
        out.println(userInput);
        System.out.println("Server: " + in.readLine());
    }
    out.println("quit");
}
```

---

## 10. Multithreaded Server

The simple server in Section 8 handles only one client then stops. A real server must handle many clients simultaneously — each in its own thread.

```java
import java.net.*;
import java.io.*;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class MultiThreadedServer {

    static final int PORT = 8080;
    static final AtomicInteger clientCount = new AtomicInteger(0);

    public static void main(String[] args) throws IOException {
        ExecutorService pool = Executors.newFixedThreadPool(10);

        System.out.println("Multi-threaded server listening on port " + PORT);

        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            while (true) {
                Socket clientSocket = serverSocket.accept();   // wait for next client
                int id = clientCount.incrementAndGet();
                System.out.println("Client #" + id + " connected.");
                pool.submit(new ClientHandler(clientSocket, id));   // handle in thread pool
            }
        }
    }
}
```

```java
class ClientHandler implements Runnable {

    private final Socket socket;
    private final int clientId;

    ClientHandler(Socket socket, int clientId) {
        this.socket   = socket;
        this.clientId = clientId;
    }

    @Override
    public void run() {
        try (BufferedReader in  = new BufferedReader(
                 new InputStreamReader(socket.getInputStream()));
             PrintWriter    out = new PrintWriter(
                 socket.getOutputStream(), true)) {

            out.println("Welcome! You are client #" + clientId);

            String line;
            while ((line = in.readLine()) != null) {
                if (line.equalsIgnoreCase("quit")) break;
                System.out.println("Client #" + clientId + ": " + line);
                out.println("Echo [" + clientId + "]: " + line.toUpperCase());
            }

        } catch (IOException e) {
            System.out.println("Client #" + clientId + " error: " + e.getMessage());
        } finally {
            try { socket.close(); } catch (IOException e) {}
            System.out.println("Client #" + clientId + " disconnected.");
        }
    }
}
```

### How It Works

```
ServerSocket.accept() ─┐
    ↓ (client 1)       │ main loop keeps
ClientHandler → pool   │ waiting for more
    ↓                  │ clients
ServerSocket.accept() ←┘
    ↓ (client 2)
ClientHandler → pool
```

The main thread does nothing but accept connections and hand them off to the thread pool. Each `ClientHandler` runs in a pool thread and handles all messages for one client independently.

---

## 11. UDP Communication

**UDP (User Datagram Protocol)** is simpler and faster than TCP — but unreliable. Packets (called **datagrams**) may arrive out of order, be duplicated, or be dropped. Use it for:

- Real-time applications where speed matters more than reliability (video streaming, gaming, VoIP).
- Simple request-response where you retry if no answer comes.

Java uses `DatagramSocket` and `DatagramPacket` for UDP:

### UDP Server (Receiver)

```java
import java.net.*;

public class UdpServer {
    public static void main(String[] args) throws Exception {
        DatagramSocket socket = new DatagramSocket(9090);
        byte[] buffer = new byte[1024];

        System.out.println("UDP server listening on port 9090...");

        while (true) {
            DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
            socket.receive(packet);   // blocks until data arrives

            String message = new String(packet.getData(), 0, packet.getLength());
            System.out.println("Received: " + message
                + " from " + packet.getAddress() + ":" + packet.getPort());

            // Echo back to sender
            byte[] response = ("Echo: " + message).getBytes();
            DatagramPacket reply = new DatagramPacket(
                response, response.length,
                packet.getAddress(), packet.getPort());
            socket.send(reply);
        }
    }
}
```

### UDP Client (Sender)

```java
import java.net.*;

public class UdpClient {
    public static void main(String[] args) throws Exception {
        DatagramSocket socket = new DatagramSocket();
        InetAddress server = InetAddress.getByName("localhost");

        String message = "Hello UDP!";
        byte[] data = message.getBytes();

        DatagramPacket packet = new DatagramPacket(data, data.length, server, 9090);
        socket.send(packet);
        System.out.println("Sent: " + message);

        // Wait for echo
        byte[] buffer = new byte[1024];
        DatagramPacket response = new DatagramPacket(buffer, buffer.length);
        socket.receive(response);
        System.out.println("Response: " + new String(response.getData(), 0, response.getLength()));

        socket.close();
    }
}
```

Output (Client):
```
Sent: Hello UDP!
Response: Echo: Hello UDP!
```

### TCP vs UDP Summary

| Feature | TCP | UDP |
|---|---|---|
| Connection | Required — handshake before data | None — just send |
| Reliability | Guaranteed delivery, correct order | No guarantee |
| Speed | Slower (overhead for reliability) | Faster (minimal overhead) |
| Use cases | File transfer, web, email, chat | Streaming, gaming, DNS, VoIP |
| Java classes | `Socket`, `ServerSocket` | `DatagramSocket`, `DatagramPacket` |

---

## 12. Combining Files and Networking

A common real-world pattern: a server reads from or writes to files in response to client requests.

### File Server — Client Requests a File

```java
// Server side — sends a file's content to a client
class FileRequestHandler implements Runnable {
    private final Socket socket;

    FileRequestHandler(Socket socket) { this.socket = socket; }

    @Override
    public void run() {
        try (BufferedReader  in  = new BufferedReader(
                 new InputStreamReader(socket.getInputStream()));
             PrintWriter     out = new PrintWriter(
                 socket.getOutputStream(), true)) {

            String filename = in.readLine();   // client sends filename
            Path path = Paths.get("files/" + filename);

            if (Files.exists(path) && Files.isReadable(path)) {
                out.println("OK");
                Files.lines(path).forEach(out::println);
                out.println("END");
            } else {
                out.println("ERROR: File not found");
            }

        } catch (IOException e) {
            System.out.println("Handler error: " + e.getMessage());
        } finally {
            try { socket.close(); } catch (IOException e) {}
        }
    }
}
```

```java
// Client side — requests a file from the server
try (Socket socket    = new Socket("localhost", 8080);
     PrintWriter out  = new PrintWriter(socket.getOutputStream(), true);
     BufferedReader in = new BufferedReader(
         new InputStreamReader(socket.getInputStream()))) {

    out.println("readme.txt");   // request the file

    String status = in.readLine();
    if ("OK".equals(status)) {
        System.out.println("=== File Contents ===");
        String line;
        while (!(line = in.readLine()).equals("END")) {
            System.out.println(line);
        }
    } else {
        System.out.println(status);
    }
}
```

### Logging Server Activity to a File

```java
// Append each request to a log file using Files.writeString / Files.write
static synchronized void log(String message) {
    try {
        String entry = LocalDateTime.now() + " " + message + "\n";
        Files.writeString(
            Paths.get("server.log"),
            entry,
            StandardOpenOption.CREATE,
            StandardOpenOption.APPEND
        );
    } catch (IOException e) {
        System.err.println("Log error: " + e.getMessage());
    }
}
```

---

## 13. Putting It All Together

Here's a complete multi-client chat server and client. The server broadcasts each message to all connected clients:

**ChatServer.java**
```java
import java.net.*;
import java.io.*;
import java.util.*;
import java.util.concurrent.*;

public class ChatServer {

    static final int PORT = 8080;
    static final List<PrintWriter> clients = new CopyOnWriteArrayList<>();

    static void broadcast(String message) {
        clients.forEach(client -> client.println(message));
    }

    public static void main(String[] args) throws IOException {
        ExecutorService pool = Executors.newCachedThreadPool();
        System.out.println("Chat server running on port " + PORT);

        try (ServerSocket server = new ServerSocket(PORT)) {
            while (true) {
                Socket socket = server.accept();
                pool.submit(() -> {
                    try (BufferedReader in  = new BufferedReader(
                             new InputStreamReader(socket.getInputStream()));
                         PrintWriter    out = new PrintWriter(
                             socket.getOutputStream(), true)) {

                        clients.add(out);

                        out.println("Enter your name:");
                        String name = in.readLine();
                        broadcast("*** " + name + " joined the chat ***");

                        String line;
                        while ((line = in.readLine()) != null) {
                            if (line.equalsIgnoreCase("/quit")) break;
                            broadcast(name + ": " + line);
                        }

                        clients.remove(out);
                        broadcast("*** " + name + " left the chat ***");

                    } catch (IOException e) {
                        System.out.println("Client error: " + e.getMessage());
                    } finally {
                        try { socket.close(); } catch (IOException e) {}
                    }
                });
            }
        }
    }
}
```

**ChatClient.java**
```java
import java.net.*;
import java.io.*;
import java.util.Scanner;

public class ChatClient {

    static final String HOST = "localhost";
    static final int    PORT = 8080;

    public static void main(String[] args) throws IOException {
        Socket socket        = new Socket(HOST, PORT);
        PrintWriter out      = new PrintWriter(socket.getOutputStream(), true);
        BufferedReader in    = new BufferedReader(
            new InputStreamReader(socket.getInputStream()));
        Scanner keyboard     = new Scanner(System.in);

        // Thread to read incoming messages
        Thread reader = new Thread(() -> {
            try {
                String line;
                while ((line = in.readLine()) != null) {
                    System.out.println(line);
                }
            } catch (IOException e) {
                System.out.println("Disconnected.");
            }
        });
        reader.setDaemon(true);
        reader.start();

        // Main thread sends messages
        String input;
        while (!(input = keyboard.nextLine()).equalsIgnoreCase("/quit")) {
            out.println(input);
        }
        out.println("/quit");
        socket.close();
    }
}
```

To test: start `ChatServer`, then start two or more `ChatClient` instances in separate terminals. Messages typed in one client appear in all others.

---

## 14. Exercises

---

### Exercise 1 – Directory Report

Write a program that:
1. Asks the user for a directory path.
2. Uses `Files.walk()` to list all files (not directories) inside it recursively.
3. Prints each file's name, size in bytes, and last-modified date.
4. Prints a summary: total files found, total size in bytes, and the largest file.

---

### Exercise 2 – File Search Tool

Build a command-line file search tool that:
1. Accepts a root directory and a search term from the user.
2. Searches all `.txt` files recursively.
3. Prints the filename and line number for every line containing the search term (case-insensitive).
4. Prints the total number of matches found.

---

### Exercise 3 – Random Access Record Store

Create a program that stores fixed-size `Employee` records in a binary file using `RandomAccessFile`:

Each record: `int id` (4 bytes) + `double salary` (8 bytes) + 30-byte name = 42 bytes total.

Implement:
- `writeEmployee(raf, id, name, salary)` — writes at the correct offset.
- `readEmployee(raf, recordIndex)` — reads by record number.
- `updateSalary(raf, recordIndex, newSalary)` — updates just the salary field.

In `main`, write 5 employees, read them all back, update one salary, then read all again to confirm the change.

---

### Exercise 4 – HTTP Weather Fetcher

Write a program that fetches current weather data from the Open-Meteo API (free, no API key needed):

```
https://api.open-meteo.com/v1/forecast?latitude=25.67&longitude=-100.31&current_weather=true
```

Parse the JSON response manually (look for `"temperature"` and `"windspeed"`) and print a friendly weather summary. Handle `IOException` and non-200 response codes gracefully.

---

### Exercise 5 – TCP Calculator Server

Build a calculator server:

**Server:** Listens on port 9000. Accepts expressions like `"10 + 5"`, `"3.14 * 2"`, computes the result, and sends it back. Handles `+`, `-`, `*`, `/`. Use a thread pool to support multiple clients.

**Client:** Reads expressions from the user in a loop, sends them to the server, and prints the result. Type `"quit"` to exit.

Handle division by zero on the server side — send back an error message instead of crashing.

---

### Exercise 6 – File Transfer *(Challenge)*

Build a TCP file transfer system:

**Server:** Listens on port 7000. When a client connects, it:
1. Receives the desired filename from the client.
2. If the file exists in a `"server_files"` directory, sends `"OK"` then the file as raw bytes.
3. If not found, sends `"NOT_FOUND"`.

**Client:** Asks the user for a filename. Connects to the server, requests the file, and if found, saves it locally to a `"client_downloads"` folder. Print download progress (bytes received).

Use `InputStream`/`OutputStream` for the binary transfer (not `BufferedReader`) so it works for any file type (text or binary).

---

### Exercise 7 – Multi-Client Chat with History *(Challenge)*

Extend the chat server from Section 13:

1. Save every message to a `chat_history.txt` file (append mode, with timestamps).
2. When a new client connects, send them the last 10 messages from the history file before entering the live chat.
3. Add a `/list` command the client can type to see all connected user names.
4. Add a `/whisper [name] [message]` command for private messages between users.

---

## Module Summary

Here's what you covered in Module 7:

- `Files.walk()` and `Files.find()` traverse directory trees as streams.
- `BasicFileAttributes` provides file metadata — size, dates, type.
- `WatchService` monitors a directory for live file changes without polling.
- `RandomAccessFile` reads and writes to any byte offset — ideal for fixed-size record databases.
- **Networking** in Java uses `java.net` — IP addresses identify machines, ports identify services.
- `URL` and `HttpURLConnection` fetch data from the web with GET and POST requests.
- **TCP sockets** — `ServerSocket` listens, `Socket` connects — provide reliable two-way communication.
- A **multithreaded server** uses a thread pool to handle many clients simultaneously.
- **UDP** (`DatagramSocket`, `DatagramPacket`) is faster but unreliable — used for real-time applications.
- Files and networking combine naturally — servers read config files, log to disk, and serve files over sockets.

---

**Next up:** Module 8 – Advanced GUI Programming

---

*CS 1103 Programming 2 (Java) — Module 7 of 8*
