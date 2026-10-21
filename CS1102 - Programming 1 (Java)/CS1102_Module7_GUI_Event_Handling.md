# CS 1102 – Programming 1 (Java)
## Module 7: Graphical User Interfaces (GUI) with Event Handling

---

## Table of Contents

1. [What Is a GUI?](#1-what-is-a-gui)
2. [Introducing Java Swing](#2-introducing-java-swing)
3. [Your First Window](#3-your-first-window)
4. [Adding Components](#4-adding-components)
5. [Layout Managers](#5-layout-managers)
6. [Event Handling](#6-event-handling)
7. [Common Components in Depth](#7-common-components-in-depth)
8. [Building a Complete GUI Application](#8-building-a-complete-gui-application)
9. [Dialogs and Pop-ups](#9-dialogs-and-pop-ups)
10. [GUI and OOP Together](#10-gui-and-oop-together)
11. [Exercises](#11-exercises)

---

## 1. What Is a GUI?

Up until now, every program you've written has worked through the **terminal** — text in, text out. That's called a **Command-Line Interface (CLI)**.

A **Graphical User Interface (GUI)** replaces typed commands with visual elements that users can interact with using a mouse and keyboard:

- Windows and panels
- Buttons, text fields, checkboxes, and dropdowns
- Menus and toolbars
- Dialogs and pop-ups

Almost every application you use daily — a browser, a word processor, a calculator — has a GUI.

---

## 2. Introducing Java Swing

Java has two main GUI libraries built into the JDK:

| Library | Notes |
|---|---|
| **AWT** (Abstract Window Toolkit) | The original, older library. Uses the operating system's native components. |
| **Swing** | Built on top of AWT. Draws its own components — looks consistent across all platforms. More flexible and widely used. |

A newer library called **JavaFX** also exists and is more modern, but Swing remains the most commonly taught because it's simpler to start with and built right into the JDK.

In this module we use **Swing**.

### Key Swing Imports

```java
import javax.swing.*;   // JFrame, JButton, JLabel, JTextField, etc.
import java.awt.*;      // FlowLayout, BorderLayout, Color, Font, etc.
import java.awt.event.*;// ActionListener, ActionEvent, etc.
```

---

## 3. Your First Window

A Swing application starts with a **`JFrame`** — the main window container.

```java
import javax.swing.*;

public class FirstWindow {
    public static void main(String[] args) {

        JFrame frame = new JFrame("My First Window");

        frame.setSize(400, 300);                         // width x height in pixels
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE); // close app on X
        frame.setLocationRelativeTo(null);               // center on screen
        frame.setVisible(true);                          // make it appear
    }
}
```

Run this and you'll see an empty 400×300 window titled "My First Window."

### Essential JFrame Methods

| Method | What it does |
|---|---|
| `setTitle(String)` | Sets the window title bar text |
| `setSize(int, int)` | Sets width and height in pixels |
| `setVisible(boolean)` | Shows (`true`) or hides (`false`) the window |
| `setDefaultCloseOperation(...)` | What happens when the user clicks X |
| `setLocationRelativeTo(null)` | Centers the window on screen |
| `setResizable(boolean)` | Whether the user can resize the window |

---

## 4. Adding Components

A window is just a frame — you fill it with **components** (also called widgets): labels, buttons, text fields, and more.

Add components to the frame's **content pane** using `add()`.

### JLabel — Displaying Text

```java
JLabel label = new JLabel("Hello, Swing!");
frame.add(label);
```

### JButton — A Clickable Button

```java
JButton button = new JButton("Click Me");
frame.add(button);
```

### JTextField — Single-Line Text Input

```java
JTextField textField = new JTextField(20);   // 20 = approximate column width
frame.add(textField);
```

### JTextArea — Multi-Line Text Input

```java
JTextArea textArea = new JTextArea(5, 20);   // 5 rows, 20 columns
frame.add(new JScrollPane(textArea));         // wrap in scroll pane
```

### JCheckBox and JRadioButton

```java
JCheckBox checkBox = new JCheckBox("I agree");
frame.add(checkBox);

JRadioButton radio = new JRadioButton("Option A");
frame.add(radio);
```

### A Simple Window with Multiple Components

```java
import javax.swing.*;

public class ComponentDemo {
    public static void main(String[] args) {

        JFrame frame = new JFrame("Component Demo");
        frame.setSize(300, 200);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLocationRelativeTo(null);
        frame.setLayout(new FlowLayout());   // arrange components in a row

        frame.add(new JLabel("Name:"));
        frame.add(new JTextField(15));
        frame.add(new JButton("Submit"));

        frame.setVisible(true);
    }
}
```

---

## 5. Layout Managers

A **Layout Manager** controls how components are arranged inside a container. Without one, components stack on top of each other.

### FlowLayout

Arranges components left to right, wrapping to the next line when needed. The default for `JPanel`.

```java
frame.setLayout(new FlowLayout());
```

```
[ Label ][ TextField ][ Button ]
```

### BorderLayout

Divides the container into five regions: North, South, East, West, Center. The default for `JFrame`.

```java
frame.setLayout(new BorderLayout());

frame.add(new JButton("North"),  BorderLayout.NORTH);
frame.add(new JButton("South"),  BorderLayout.SOUTH);
frame.add(new JButton("East"),   BorderLayout.EAST);
frame.add(new JButton("West"),   BorderLayout.WEST);
frame.add(new JButton("Center"), BorderLayout.CENTER);
```

```
+----------+
|  North   |
+--+----+--+
|W |    |E |
|  |Ctr |  |
+--+----+--+
|  South   |
+----------+
```

### GridLayout

Arranges components in a grid of equal-sized cells.

```java
frame.setLayout(new GridLayout(3, 2));   // 3 rows, 2 columns

frame.add(new JButton("1"));
frame.add(new JButton("2"));
frame.add(new JButton("3"));
frame.add(new JButton("4"));
frame.add(new JButton("5"));
frame.add(new JButton("6"));
```

```
[ 1 ][ 2 ]
[ 3 ][ 4 ]
[ 5 ][ 6 ]
```

### JPanel — A Container Inside a Container

`JPanel` is an invisible container you can use to group components and apply different layouts to different sections:

```java
JPanel topPanel = new JPanel(new FlowLayout());
topPanel.add(new JLabel("Search:"));
topPanel.add(new JTextField(15));
topPanel.add(new JButton("Go"));

JPanel bottomPanel = new JPanel(new GridLayout(1, 3));
bottomPanel.add(new JButton("File"));
bottomPanel.add(new JButton("Edit"));
bottomPanel.add(new JButton("Help"));

frame.setLayout(new BorderLayout());
frame.add(topPanel,    BorderLayout.NORTH);
frame.add(bottomPanel, BorderLayout.SOUTH);
```

Using nested `JPanel`s is the standard way to build complex layouts.

---

## 6. Event Handling

A static window isn't very useful. **Event handling** is what makes GUI apps interactive — your program responds when a user clicks, types, or moves the mouse.

Java uses the **event listener** pattern:
1. An **event** occurs (e.g. a button click).
2. A **listener** object is notified.
3. The listener runs a method in response.

### ActionListener — The Most Common Listener

`ActionListener` responds to button clicks, menu selections, and Enter key presses in text fields.

It has one method you must implement: `actionPerformed(ActionEvent e)`.

### Method 1 — Anonymous Inner Class (Traditional)

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class ButtonDemo {
    public static void main(String[] args) {

        JFrame frame = new JFrame("Button Demo");
        frame.setSize(300, 150);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLocationRelativeTo(null);
        frame.setLayout(new FlowLayout());

        JLabel label  = new JLabel("Press the button!");
        JButton button = new JButton("Click Me");

        button.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                label.setText("You clicked the button!");
            }
        });

        frame.add(label);
        frame.add(button);
        frame.setVisible(true);
    }
}
```

### Method 2 — Lambda Expression (Modern, Cleaner)

Since `ActionListener` has only one method, you can replace the whole anonymous class with a **lambda expression**:

```java
button.addActionListener(e -> label.setText("You clicked the button!"));
```

This is shorter and just as correct. You'll see both styles in Java code, but lambdas are preferred in modern Java.

### Reading from a TextField

```java
JTextField nameField = new JTextField(15);
JLabel greeting     = new JLabel("Enter your name above.");
JButton button      = new JButton("Greet");

button.addActionListener(e -> {
    String name = nameField.getText();   // read what the user typed
    greeting.setText("Hello, " + name + "!");
});
```

---

## 7. Common Components in Depth

### JComboBox — Dropdown Menu

```java
String[] options = {"Red", "Green", "Blue"};
JComboBox<String> colorPicker = new JComboBox<>(options);

colorPicker.addActionListener(e -> {
    String selected = (String) colorPicker.getSelectedItem();
    System.out.println("Selected: " + selected);
});
```

### JCheckBox — Toggle Option

```java
JCheckBox darkMode = new JCheckBox("Enable Dark Mode");

darkMode.addActionListener(e -> {
    if (darkMode.isSelected()) {
        System.out.println("Dark mode ON");
    } else {
        System.out.println("Dark mode OFF");
    }
});
```

### JRadioButton — Select One from a Group

Radio buttons must be grouped so only one can be selected at a time:

```java
JRadioButton smallBtn  = new JRadioButton("Small");
JRadioButton mediumBtn = new JRadioButton("Medium");
JRadioButton largeBtn  = new JRadioButton("Large");

ButtonGroup sizeGroup = new ButtonGroup();   // groups them logically
sizeGroup.add(smallBtn);
sizeGroup.add(mediumBtn);
sizeGroup.add(largeBtn);
```

### JSlider — Numerical Range Input

```java
JSlider slider = new JSlider(0, 100, 50);   // min, max, initial value
JLabel valueLabel = new JLabel("50");

slider.addChangeListener(e -> {
    valueLabel.setText(String.valueOf(slider.getValue()));
});
```

### Styling Components

```java
label.setFont(new Font("Arial", Font.BOLD, 18));
label.setForeground(Color.BLUE);
button.setBackground(Color.ORANGE);
panel.setBackground(new Color(240, 240, 240));   // light gray using RGB
```

---

## 8. Building a Complete GUI Application

Let's build a simple tip calculator — a realistic app that ties together layouts, components, and events.

```java
import javax.swing.*;
import java.awt.*;

public class TipCalculator {

    public static void main(String[] args) {
        JFrame frame = new JFrame("Tip Calculator");
        frame.setSize(320, 220);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLocationRelativeTo(null);
        frame.setLayout(new GridLayout(5, 2, 10, 10));

        // Components
        JLabel billLabel    = new JLabel("  Bill Amount ($):");
        JTextField billField = new JTextField();

        JLabel tipLabel    = new JLabel("  Tip Percentage (%):");
        JTextField tipField = new JTextField();

        JLabel totalLabel   = new JLabel("  Total:");
        JLabel totalDisplay = new JLabel("--");
        totalDisplay.setFont(new Font("Arial", Font.BOLD, 16));

        JLabel tipAmtLabel   = new JLabel("  Tip Amount:");
        JLabel tipAmtDisplay = new JLabel("--");

        JButton calcButton = new JButton("Calculate");
        JButton clearButton = new JButton("Clear");

        // Event handling
        calcButton.addActionListener(e -> {
            try {
                double bill = Double.parseDouble(billField.getText());
                double tipPct = Double.parseDouble(tipField.getText());
                double tipAmt = bill * tipPct / 100;
                double total  = bill + tipAmt;

                tipAmtDisplay.setText(String.format("$%.2f", tipAmt));
                totalDisplay.setText(String.format("$%.2f", total));

            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(frame,
                    "Please enter valid numbers.",
                    "Input Error",
                    JOptionPane.ERROR_MESSAGE);
            }
        });

        clearButton.addActionListener(e -> {
            billField.setText("");
            tipField.setText("");
            totalDisplay.setText("--");
            tipAmtDisplay.setText("--");
        });

        // Add to frame
        frame.add(billLabel);    frame.add(billField);
        frame.add(tipLabel);     frame.add(tipField);
        frame.add(tipAmtLabel);  frame.add(tipAmtDisplay);
        frame.add(totalLabel);   frame.add(totalDisplay);
        frame.add(calcButton);   frame.add(clearButton);

        frame.setVisible(true);
    }
}
```

This app:
- Takes two inputs from the user (bill and tip %).
- Calculates and displays tip amount and total.
- Shows an error dialog for invalid input.
- Clears everything on the Clear button.

---

## 9. Dialogs and Pop-ups

`JOptionPane` provides ready-made pop-up dialogs without needing to build them yourself.

### Message Dialog — Show information

```java
JOptionPane.showMessageDialog(frame, "Operation completed!");
```

### Confirm Dialog — Ask yes/no

```java
int choice = JOptionPane.showConfirmDialog(frame, "Are you sure you want to quit?");

if (choice == JOptionPane.YES_OPTION) {
    System.exit(0);
}
```

### Input Dialog — Get text from user

```java
String name = JOptionPane.showInputDialog(frame, "Enter your name:");
System.out.println("Hello, " + name);
```

### Message Types

```java
JOptionPane.showMessageDialog(frame, "Saved!", "Success",
    JOptionPane.INFORMATION_MESSAGE);   // info icon

JOptionPane.showMessageDialog(frame, "File not found!", "Error",
    JOptionPane.ERROR_MESSAGE);         // error icon

JOptionPane.showMessageDialog(frame, "Low battery!", "Warning",
    JOptionPane.WARNING_MESSAGE);       // warning icon
```

---

## 10. GUI and OOP Together

A clean, professional way to build Swing apps is to put your GUI in its own class that **extends `JFrame`**. This uses the OOP skills from Modules 5 and 6 and keeps your code organized.

```java
import javax.swing.*;
import java.awt.*;

public class CounterApp extends JFrame {

    private int count = 0;
    private JLabel countLabel;

    // Constructor — builds the GUI
    public CounterApp() {
        setTitle("Click Counter");
        setSize(250, 150);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setLayout(new FlowLayout(FlowLayout.CENTER, 20, 20));

        countLabel = new JLabel("Count: 0");
        countLabel.setFont(new Font("Arial", Font.BOLD, 18));

        JButton incrementBtn = new JButton("+ Add");
        JButton resetBtn     = new JButton("Reset");

        incrementBtn.addActionListener(e -> {
            count++;
            countLabel.setText("Count: " + count);
        });

        resetBtn.addActionListener(e -> {
            count = 0;
            countLabel.setText("Count: 0");
        });

        add(countLabel);
        add(incrementBtn);
        add(resetBtn);
    }

    public static void main(String[] args) {
        new CounterApp().setVisible(true);
    }
}
```

Benefits of this structure:
- The class **is** the window — clean and self-contained.
- Instance variables hold the state (like `count`) and components (like `countLabel`).
- The constructor sets everything up — just like any other object.
- `main` becomes a one-liner.

This is the standard pattern you'll see in real Java GUI projects.

---

## 11. Exercises

---

### Exercise 1 – Hello GUI

Create a window that contains:
- A `JLabel` saying "What is your name?"
- A `JTextField` for input.
- A `JButton` labeled "Say Hello".
- Another `JLabel` (initially empty) that updates to `"Hello, [name]!"` when the button is clicked.

Use `FlowLayout` or `GridLayout` to arrange the components.

---

### Exercise 2 – Simple Calculator

Build a GUI calculator that:
1. Has two `JTextField` inputs for numbers.
2. Has four buttons: `+`, `-`, `*`, `/`.
3. Displays the result in a `JLabel`.
4. Shows an error dialog if either input is not a valid number.
5. Shows an error if the user tries to divide by zero.

---

### Exercise 3 – Color Picker

Build a window with:
- Three `JSlider` components (each 0–255) for Red, Green, and Blue values.
- A large `JPanel` that displays the color produced by those RGB values.
- Labels showing the current value of each slider.

When a slider is moved, the panel color and labels should update immediately.

**Hint:** Use `panel.setBackground(new Color(r, g, b))` and `panel.repaint()`.

---

### Exercise 4 – To-Do List

Build a simple to-do list app:
1. A `JTextField` to type a new task.
2. An "Add" button that adds the task to a `JList` or `JTextArea`.
3. A "Clear All" button that empties the list.
4. Show a warning dialog if the user tries to add an empty task.

---

### Exercise 5 – Temperature Converter GUI

Build a GUI version of the temperature converter from Module 3:
1. A `JTextField` for the temperature value.
2. Two `JRadioButton`s — "Celsius to Fahrenheit" and "Fahrenheit to Celsius" — grouped in a `ButtonGroup`.
3. A "Convert" button.
4. A `JLabel` that shows the result.

---

### Exercise 6 – Login Screen *(Challenge)*

Build a login screen using the class-extends-JFrame pattern:
1. A username `JTextField`.
2. A password `JPasswordField` (hides input with dots).
3. A "Login" button.

Hard-code a valid username (`"admin"`) and password (`"java123"`).

- If the credentials are correct: show a success message dialog and disable both fields and the button.
- If wrong: show an error dialog and clear the password field.
- After 3 failed attempts: show a dialog saying the account is locked and disable the form.

---

## Module Summary

Here's what you covered in Module 7:

- **GUI** programs use visual components instead of text commands.
- **Swing** is Java's built-in GUI library — `JFrame` is the main window.
- Components include `JLabel`, `JButton`, `JTextField`, `JTextArea`, `JCheckBox`, `JRadioButton`, `JComboBox`, and `JSlider`.
- **Layout managers** control arrangement: `FlowLayout` (row), `BorderLayout` (five regions), `GridLayout` (grid).
- **`JPanel`** groups components and can have its own layout.
- **Event handling** uses listeners — `addActionListener` responds to button clicks.
- **Lambda expressions** are a concise way to write single-method listeners.
- **`JOptionPane`** provides ready-made message, confirm, and input dialogs.
- The best practice is to **extend `JFrame`** in your own class for clean, OOP-based GUI apps.

---

**Next up:** Module 8 – Collection API and Java 8 Features

---

*CS 1102 Programming 1 (Java) — Module 7 of 8*
