# CS 1103 – Programming 2 (Java)
## Module 8: Advanced GUI Programming

---

## Table of Contents

1. [Beyond the Basics — What Makes a GUI "Advanced"?](#1-beyond-the-basics--what-makes-a-gui-advanced)
2. [Structuring GUI Apps with MVC](#2-structuring-gui-apps-with-mvc)
3. [Advanced Layout Managers](#3-advanced-layout-managers)
4. [Custom Painting and Graphics](#4-custom-painting-and-graphics)
5. [Advanced Event Handling](#5-advanced-event-handling)
6. [Menus and Toolbars](#6-menus-and-toolbars)
7. [Advanced Components](#7-advanced-components)
8. [Dialogs — Custom and Built-in](#8-dialogs--custom-and-built-in)
9. [Swing Workers — Background Tasks in the GUI](#9-swing-workers--background-tasks-in-the-gui)
10. [Connecting GUI to Data — Tables and Lists](#10-connecting-gui-to-data--tables-and-lists)
11. [Drag and Drop](#11-drag-and-drop)
12. [Putting It All Together](#12-putting-it-all-together)
13. [Exercises](#13-exercises)
14. [Course Wrap-Up](#14-course-wrap-up)

---

## 1. Beyond the Basics — What Makes a GUI "Advanced"?

In Module 7 of CS 1102 you learned the fundamentals of Swing: windows, buttons, text fields, layouts, and basic event listeners. That knowledge lets you build functional interfaces.

Advanced GUI programming is about building interfaces that are:

- **Well-structured** — logic separated from presentation using design patterns.
- **Visually rich** — custom graphics, colors, and animations.
- **Responsive** — long operations run in the background, keeping the UI smooth.
- **Data-driven** — tables and lists that reflect live data automatically.
- **Professional** — menus, toolbars, keyboard shortcuts, and custom dialogs.

This module covers all of these. By the end you'll have the skills to build desktop applications that look and behave like real software.

---

## 2. Structuring GUI Apps with MVC

As GUI apps grow, mixing UI code with business logic creates a tangled mess. The **Model-View-Controller (MVC)** pattern solves this by separating responsibilities into three distinct layers:

| Layer | Responsibility | In Swing |
|---|---|---|
| **Model** | Holds the data and business logic | Plain Java classes, `DefaultTableModel` |
| **View** | Displays the data, receives user input | `JFrame`, `JPanel`, all components |
| **Controller** | Responds to user actions, updates Model and View | Event listeners / `ActionListener` |

```
User interaction
      ↓
   View  ──── notifies ────> Controller
      ↑                           │
      └──── updates ─────── Model ┘
```

### A Simple MVC Example

**Model — pure data, no UI:**
```java
public class CounterModel {
    private int count = 0;

    public void increment()  { count++; }
    public void decrement()  { count = Math.max(0, count - 1); }
    public void reset()      { count = 0; }
    public int  getCount()   { return count; }
}
```

**View — pure UI, no logic:**
```java
import javax.swing.*;
import java.awt.*;

public class CounterView extends JFrame {

    JLabel  countLabel  = new JLabel("0", SwingConstants.CENTER);
    JButton incButton   = new JButton("+1");
    JButton decButton   = new JButton("-1");
    JButton resetButton = new JButton("Reset");

    public CounterView() {
        setTitle("MVC Counter");
        setSize(300, 180);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        countLabel.setFont(new Font("Arial", Font.BOLD, 48));

        JPanel buttonPanel = new JPanel(new FlowLayout());
        buttonPanel.add(incButton);
        buttonPanel.add(decButton);
        buttonPanel.add(resetButton);

        setLayout(new BorderLayout());
        add(countLabel,  BorderLayout.CENTER);
        add(buttonPanel, BorderLayout.SOUTH);
    }

    public void updateCount(int count) {
        countLabel.setText(String.valueOf(count));
    }
}
```

**Controller — wires model to view:**
```java
public class CounterController {

    private final CounterModel model;
    private final CounterView  view;

    public CounterController(CounterModel model, CounterView view) {
        this.model = model;
        this.view  = view;
        attachListeners();
    }

    private void attachListeners() {
        view.incButton.addActionListener(e -> {
            model.increment();
            view.updateCount(model.getCount());
        });
        view.decButton.addActionListener(e -> {
            model.decrement();
            view.updateCount(model.getCount());
        });
        view.resetButton.addActionListener(e -> {
            model.reset();
            view.updateCount(model.getCount());
        });
    }
}
```

**Main:**
```java
public class Main {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            CounterModel model = new CounterModel();
            CounterView  view  = new CounterView();
            new CounterController(model, view);
            view.setVisible(true);
        });
    }
}
```

### `SwingUtilities.invokeLater()`

All Swing operations **must run on the Event Dispatch Thread (EDT)** — Swing's single UI thread. Wrapping your startup in `SwingUtilities.invokeLater()` ensures this. Updating UI components from a background thread causes subtle bugs.

---

## 3. Advanced Layout Managers

In Module 7 of CS 1102 you learned `FlowLayout`, `BorderLayout`, and `GridLayout`. Two more give you much finer control.

### GridBagLayout — The Most Flexible Layout

`GridBagLayout` arranges components in a grid where cells can span multiple rows and columns, and each component can have its own alignment and padding.

Every component gets a `GridBagConstraints` object:

```java
import java.awt.*;
import javax.swing.*;

public class GridBagDemo extends JFrame {

    public GridBagDemo() {
        setTitle("GridBagLayout Demo");
        setSize(350, 200);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        setLayout(new GridBagLayout());
        GridBagConstraints c = new GridBagConstraints();
        c.insets = new Insets(5, 5, 5, 5);   // padding around each component
        c.fill   = GridBagConstraints.HORIZONTAL;

        // Row 0
        c.gridx = 0; c.gridy = 0; c.gridwidth = 1;
        add(new JLabel("Name:"), c);

        c.gridx = 1; c.gridy = 0; c.gridwidth = 2;
        add(new JTextField(20), c);

        // Row 1
        c.gridx = 0; c.gridy = 1; c.gridwidth = 1;
        add(new JLabel("Email:"), c);

        c.gridx = 1; c.gridy = 1; c.gridwidth = 2;
        add(new JTextField(20), c);

        // Row 2 — button spanning all 3 columns
        c.gridx = 0; c.gridy = 2; c.gridwidth = 3;
        add(new JButton("Submit"), c);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new GridBagDemo().setVisible(true));
    }
}
```

### Key `GridBagConstraints` Fields

| Field | Purpose |
|---|---|
| `gridx`, `gridy` | Column and row position |
| `gridwidth`, `gridheight` | How many columns/rows this component spans |
| `fill` | How to stretch the component: `HORIZONTAL`, `VERTICAL`, `BOTH`, `NONE` |
| `anchor` | Where to place component within its cell: `NORTHWEST`, `CENTER`, etc. |
| `weightx`, `weighty` | How much extra space this column/row absorbs (0.0–1.0) |
| `insets` | External padding (top, left, bottom, right) |

### BoxLayout — Stack Vertically or Horizontally

```java
JPanel panel = new JPanel();
panel.setLayout(new BoxLayout(panel, BoxLayout.Y_AXIS));   // stack vertically

panel.add(new JLabel("Username"));
panel.add(Box.createVerticalStrut(5));   // 5px gap
panel.add(new JTextField(20));
panel.add(Box.createVerticalStrut(10));
panel.add(new JButton("Login"));
```

`BoxLayout.X_AXIS` stacks horizontally. `Box.createVerticalStrut(n)` and `Box.createHorizontalStrut(n)` add fixed-size spacing between components.

---

## 4. Custom Painting and Graphics

Every Swing component can draw custom graphics by overriding `paintComponent(Graphics g)`.

**Always:**
- Extend `JPanel` for custom drawing.
- Call `super.paintComponent(g)` as the first line to clear the background.
- Cast `g` to `Graphics2D` for advanced drawing features.

```java
import javax.swing.*;
import java.awt.*;
import java.awt.geom.*;

public class DrawingPanel extends JPanel {

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);                  // always call this first
        Graphics2D g2 = (Graphics2D) g;

        // Enable antialiasing — smoother edges
        g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                            RenderingHints.VALUE_ANTIALIAS_ON);

        // Fill a rectangle
        g2.setColor(Color.BLUE);
        g2.fillRect(20, 20, 100, 60);

        // Draw a circle outline
        g2.setColor(Color.RED);
        g2.setStroke(new BasicStroke(3));
        g2.drawOval(150, 20, 80, 80);

        // Draw a filled ellipse
        g2.setColor(new Color(0, 180, 0, 150));   // semi-transparent green
        g2.fillOval(130, 50, 120, 60);

        // Draw a line
        g2.setColor(Color.BLACK);
        g2.setStroke(new BasicStroke(2, BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND));
        g2.drawLine(20, 120, 280, 120);

        // Draw text with a custom font
        g2.setFont(new Font("Serif", Font.BOLD | Font.ITALIC, 22));
        g2.setColor(Color.DARK_GRAY);
        g2.drawString("Java 2D Graphics", 50, 160);

        // Draw a gradient-filled rectangle
        GradientPaint gradient = new GradientPaint(
            20, 180, Color.CYAN,
            220, 220, Color.MAGENTA);
        g2.setPaint(gradient);
        g2.fillRoundRect(20, 180, 260, 40, 15, 15);
    }
}
```

```java
public class GraphicsDemo extends JFrame {
    public GraphicsDemo() {
        setTitle("Graphics Demo");
        setSize(320, 260);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        add(new DrawingPanel());
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new GraphicsDemo().setVisible(true));
    }
}
```

### Simple Animation with a Timer

`javax.swing.Timer` fires an `ActionEvent` repeatedly at a set interval — perfect for animation:

```java
import javax.swing.*;
import java.awt.*;

public class AnimatedBall extends JPanel {

    private int x = 0, y = 150;
    private int dx = 3, dy = 2;
    private final int RADIUS = 20;

    public AnimatedBall() {
        setBackground(Color.WHITE);
        setPreferredSize(new Dimension(400, 300));

        Timer timer = new Timer(16, e -> {   // ~60 FPS
            x += dx;
            y += dy;

            // Bounce off walls
            if (x < 0 || x + RADIUS * 2 > getWidth())  dx = -dx;
            if (y < 0 || y + RADIUS * 2 > getHeight()) dy = -dy;

            repaint();   // triggers paintComponent
        });
        timer.start();
    }

    @Override
    protected void paintComponent(Graphics g) {
        super.paintComponent(g);
        g.setColor(Color.ORANGE);
        g.fillOval(x, y, RADIUS * 2, RADIUS * 2);
    }
}
```

```java
JFrame frame = new JFrame("Bouncing Ball");
frame.add(new AnimatedBall());
frame.pack();
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
frame.setLocationRelativeTo(null);
frame.setVisible(true);
```

---

## 5. Advanced Event Handling

Beyond `ActionListener`, Swing has many specialized listener types.

### Mouse Events

```java
import java.awt.event.*;

panel.addMouseListener(new MouseAdapter() {
    @Override
    public void mouseClicked(MouseEvent e) {
        System.out.println("Clicked at (" + e.getX() + ", " + e.getY() + ")");
        if (e.getClickCount() == 2) System.out.println("Double-click!");
    }

    @Override
    public void mouseEntered(MouseEvent e) { panel.setBackground(Color.LIGHT_GRAY); }

    @Override
    public void mouseExited(MouseEvent e)  { panel.setBackground(Color.WHITE); }
});

panel.addMouseMotionListener(new MouseMotionAdapter() {
    @Override
    public void mouseDragged(MouseEvent e) {
        System.out.println("Dragging at (" + e.getX() + ", " + e.getY() + ")");
    }
});
```

`MouseAdapter` is an abstract class that provides empty implementations of all `MouseListener` methods — override only the ones you need.

### Keyboard Events

```java
textField.addKeyListener(new KeyAdapter() {
    @Override
    public void keyPressed(KeyEvent e) {
        if (e.getKeyCode() == KeyEvent.VK_ENTER) {
            System.out.println("Enter pressed: " + textField.getText());
        }
        if (e.isControlDown() && e.getKeyCode() == KeyEvent.VK_S) {
            System.out.println("Ctrl+S — save!");
        }
    }
});
```

### Key Bindings — The Better Way for Keyboard Shortcuts

`KeyListener` only works when the component has focus. **Key bindings** work application-wide:

```java
// Bind Escape key to close the window, regardless of which component has focus
InputMap  inputMap  = panel.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW);
ActionMap actionMap = panel.getActionMap();

inputMap.put(KeyStroke.getKeyStroke(KeyEvent.VK_ESCAPE, 0), "close");
actionMap.put("close", new AbstractAction() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.exit(0);
    }
});
```

### Focus Events

```java
textField.addFocusListener(new FocusAdapter() {
    @Override
    public void focusGained(FocusEvent e) {
        textField.setBackground(new Color(255, 255, 200));   // yellow when focused
    }

    @Override
    public void focusLost(FocusEvent e) {
        textField.setBackground(Color.WHITE);
    }
});
```

---

## 6. Menus and Toolbars

Professional applications use menus and toolbars to expose actions without cluttering the main window.

### JMenuBar, JMenu, JMenuItem

```java
import javax.swing.*;
import java.awt.event.*;

public class MenuDemo extends JFrame {

    public MenuDemo() {
        setTitle("Menu Demo");
        setSize(400, 300);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Create the menu bar
        JMenuBar menuBar = new JMenuBar();

        // File menu
        JMenu fileMenu = new JMenu("File");
        fileMenu.setMnemonic(KeyEvent.VK_F);   // Alt+F opens the menu

        JMenuItem newItem  = new JMenuItem("New",  KeyEvent.VK_N);
        JMenuItem openItem = new JMenuItem("Open", KeyEvent.VK_O);
        JMenuItem saveItem = new JMenuItem("Save", KeyEvent.VK_S);
        JMenuItem exitItem = new JMenuItem("Exit", KeyEvent.VK_X);

        // Keyboard shortcuts
        newItem.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_N, ActionEvent.CTRL_MASK));
        saveItem.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_S, ActionEvent.CTRL_MASK));

        exitItem.addActionListener(e -> System.exit(0));

        fileMenu.add(newItem);
        fileMenu.add(openItem);
        fileMenu.add(saveItem);
        fileMenu.addSeparator();
        fileMenu.add(exitItem);

        // Edit menu
        JMenu editMenu = new JMenu("Edit");
        editMenu.add(new JMenuItem("Cut"));
        editMenu.add(new JMenuItem("Copy"));
        editMenu.add(new JMenuItem("Paste"));

        // View menu with checkboxes and radio buttons
        JMenu viewMenu = new JMenu("View");

        JCheckBoxMenuItem darkMode = new JCheckBoxMenuItem("Dark Mode");
        darkMode.addActionListener(e ->
            System.out.println("Dark mode: " + darkMode.isSelected()));

        ButtonGroup sizeGroup = new ButtonGroup();
        JRadioButtonMenuItem smallText  = new JRadioButtonMenuItem("Small Text");
        JRadioButtonMenuItem mediumText = new JRadioButtonMenuItem("Medium Text", true);
        JRadioButtonMenuItem largeText  = new JRadioButtonMenuItem("Large Text");
        sizeGroup.add(smallText);
        sizeGroup.add(mediumText);
        sizeGroup.add(largeText);

        viewMenu.add(darkMode);
        viewMenu.addSeparator();
        viewMenu.add(smallText);
        viewMenu.add(mediumText);
        viewMenu.add(largeText);

        menuBar.add(fileMenu);
        menuBar.add(editMenu);
        menuBar.add(viewMenu);
        setJMenuBar(menuBar);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new MenuDemo().setVisible(true));
    }
}
```

### JToolBar

```java
JToolBar toolbar = new JToolBar();
toolbar.setFloatable(false);   // prevent undocking

JButton newBtn  = new JButton("New");
JButton openBtn = new JButton("Open");
JButton saveBtn = new JButton("Save");

toolbar.add(newBtn);
toolbar.add(openBtn);
toolbar.addSeparator();
toolbar.add(saveBtn);

frame.add(toolbar, BorderLayout.NORTH);
```

### Popup (Right-Click) Menu

```java
JPopupMenu popup = new JPopupMenu();
popup.add(new JMenuItem("Cut"));
popup.add(new JMenuItem("Copy"));
popup.add(new JMenuItem("Paste"));

panel.addMouseListener(new MouseAdapter() {
    @Override
    public void mousePressed(MouseEvent e)  { showPopup(e); }

    @Override
    public void mouseReleased(MouseEvent e) { showPopup(e); }

    private void showPopup(MouseEvent e) {
        if (e.isPopupTrigger()) {
            popup.show(e.getComponent(), e.getX(), e.getY());
        }
    }
});
```

---

## 7. Advanced Components

### JTabbedPane — Multiple Panels in Tabs

```java
JTabbedPane tabs = new JTabbedPane();

tabs.addTab("Profile",  new ProfilePanel());
tabs.addTab("Settings", new SettingsPanel());
tabs.addTab("About",    new JLabel("App v1.0", SwingConstants.CENTER));

tabs.setTabPlacement(JTabbedPane.TOP);   // TOP, BOTTOM, LEFT, RIGHT

// Listen for tab changes
tabs.addChangeListener(e ->
    System.out.println("Switched to tab: " + tabs.getTitleAt(tabs.getSelectedIndex())));

frame.add(tabs);
```

### JSplitPane — Resizable Split View

```java
JTextArea editor = new JTextArea();
JTextArea preview = new JTextArea();
preview.setEditable(false);

JSplitPane split = new JSplitPane(
    JSplitPane.HORIZONTAL_SPLIT,
    new JScrollPane(editor),
    new JScrollPane(preview));

split.setDividerLocation(300);   // initial divider position in pixels
split.setResizeWeight(0.5);      // both sides grow equally when window resizes
frame.add(split);
```

### JScrollPane — Add Scrolling to Any Component

```java
JTextArea area = new JTextArea(10, 30);
JScrollPane scroll = new JScrollPane(area);
scroll.setVerticalScrollBarPolicy(JScrollPane.VERTICAL_SCROLLBAR_ALWAYS);
frame.add(scroll);
```

### JProgressBar

```java
JProgressBar progressBar = new JProgressBar(0, 100);
progressBar.setValue(0);
progressBar.setStringPainted(true);   // show percentage text

// Update value
progressBar.setValue(65);             // 65%
progressBar.setIndeterminate(true);   // spinning animation when progress unknown
```

### JSpinner — Numeric/Date Picker

```java
SpinnerNumberModel model = new SpinnerNumberModel(5, 1, 100, 1);
// initial value, min, max, step
JSpinner spinner = new JSpinner(model);

spinner.addChangeListener(e ->
    System.out.println("Value: " + spinner.getValue()));
```

---

## 8. Dialogs — Custom and Built-in

### JFileChooser — Open and Save Files

```java
JFileChooser chooser = new JFileChooser();

// File filter — only show .txt files
chooser.setFileFilter(new javax.swing.filechooser.FileNameExtensionFilter(
    "Text Files (*.txt)", "txt"));

int result = chooser.showOpenDialog(frame);   // or showSaveDialog()

if (result == JFileChooser.APPROVE_OPTION) {
    File selected = chooser.getSelectedFile();
    System.out.println("Selected: " + selected.getAbsolutePath());
}
```

### JColorChooser — Pick a Color

```java
Color chosen = JColorChooser.showDialog(frame, "Choose a Color", Color.BLUE);
if (chosen != null) {
    panel.setBackground(chosen);
}
```

### Custom JDialog

For complex input forms, build a `JDialog` — a modal window that blocks the parent until dismissed:

```java
public class LoginDialog extends JDialog {

    private JTextField  usernameField = new JTextField(15);
    private JPasswordField passwordField = new JPasswordField(15);
    private boolean approved = false;

    public LoginDialog(JFrame parent) {
        super(parent, "Login", true);   // true = modal
        setSize(280, 160);
        setLocationRelativeTo(parent);
        setLayout(new GridBagLayout());

        GridBagConstraints c = new GridBagConstraints();
        c.insets = new Insets(5, 5, 5, 5);
        c.fill   = GridBagConstraints.HORIZONTAL;

        c.gridx = 0; c.gridy = 0; add(new JLabel("Username:"), c);
        c.gridx = 1;              add(usernameField, c);
        c.gridx = 0; c.gridy = 1; add(new JLabel("Password:"), c);
        c.gridx = 1;              add(passwordField, c);

        JButton loginBtn  = new JButton("Login");
        JButton cancelBtn = new JButton("Cancel");

        loginBtn.addActionListener(e -> { approved = true;  dispose(); });
        cancelBtn.addActionListener(e -> { approved = false; dispose(); });

        JPanel buttons = new JPanel();
        buttons.add(loginBtn);
        buttons.add(cancelBtn);

        c.gridx = 0; c.gridy = 2; c.gridwidth = 2;
        add(buttons, c);
    }

    public boolean isApproved()   { return approved; }
    public String  getUsername()  { return usernameField.getText(); }
    public String  getPassword()  { return new String(passwordField.getPassword()); }
}
```

```java
LoginDialog dialog = new LoginDialog(frame);
dialog.setVisible(true);   // blocks until dialog is closed

if (dialog.isApproved()) {
    System.out.println("Logging in as: " + dialog.getUsername());
}
```

---

## 9. Swing Workers — Background Tasks in the GUI

Long-running operations (file loading, network requests, database queries) must **never run on the EDT**. If they do, the UI freezes — buttons stop responding, the window won't redraw.

`SwingWorker<T, V>` runs a task in a background thread and safely updates the UI when done:

- `T` — the final result type (returned by `doInBackground()`).
- `V` — intermediate progress update type (published by `publish()`).

```java
import javax.swing.*;
import java.util.List;

public class DataLoaderWorker extends SwingWorker<String, Integer> {

    private final JProgressBar progressBar;
    private final JLabel       statusLabel;

    DataLoaderWorker(JProgressBar progressBar, JLabel statusLabel) {
        this.progressBar = progressBar;
        this.statusLabel = statusLabel;
    }

    @Override
    protected String doInBackground() throws Exception {
        // Runs on background thread — NOT the EDT
        for (int i = 0; i <= 100; i += 10) {
            Thread.sleep(300);   // simulate work
            publish(i);          // send progress to process()
        }
        return "Data loaded successfully!";
    }

    @Override
    protected void process(List<Integer> chunks) {
        // Runs on EDT — safe to update UI
        int latestProgress = chunks.get(chunks.size() - 1);
        progressBar.setValue(latestProgress);
        statusLabel.setText("Loading... " + latestProgress + "%");
    }

    @Override
    protected void done() {
        // Runs on EDT after doInBackground() completes
        try {
            String result = get();   // retrieves the return value of doInBackground()
            statusLabel.setText(result);
            progressBar.setValue(100);
        } catch (Exception e) {
            statusLabel.setText("Error: " + e.getMessage());
        }
    }
}
```

```java
JProgressBar bar    = new JProgressBar(0, 100);
JLabel       status = new JLabel("Press Load to start.");
JButton      loadBtn = new JButton("Load Data");

loadBtn.addActionListener(e -> {
    loadBtn.setEnabled(false);
    new DataLoaderWorker(bar, status).execute();   // start background work
    loadBtn.setEnabled(true);                      // re-enable immediately
});
```

### SwingWorker Summary

| Method | Thread | Purpose |
|---|---|---|
| `doInBackground()` | Background | Long-running task — never touch UI here |
| `publish(V... chunks)` | Background | Send intermediate updates |
| `process(List<V>)` | EDT | Receive and display intermediate updates |
| `done()` | EDT | Called when `doInBackground()` finishes |
| `get()` | EDT (in `done()`) | Retrieve the result from `doInBackground()` |
| `execute()` | EDT | Start the worker |

---

## 10. Connecting GUI to Data — Tables and Lists

### JTable with DefaultTableModel

```java
import javax.swing.*;
import javax.swing.table.*;

String[] columns = {"ID", "Name", "Age", "GPA"};
Object[][] data  = {
    {1, "Alice",  20, 3.85},
    {2, "Bob",    22, 3.42},
    {3, "Carlos", 21, 3.91}
};

DefaultTableModel model = new DefaultTableModel(data, columns) {
    @Override
    public boolean isCellEditable(int row, int col) {
        return col != 0;   // make ID column read-only
    }
};

JTable table = new JTable(model);
table.setRowHeight(24);
table.getTableHeader().setReorderingAllowed(false);

// Make columns sortable
table.setAutoCreateRowSorter(true);

JScrollPane scroll = new JScrollPane(table);
frame.add(scroll);
```

### Adding and Removing Rows

```java
// Add a row
model.addRow(new Object[]{4, "Diana", 23, 3.67});

// Remove selected row
int selectedRow = table.getSelectedRow();
if (selectedRow >= 0) {
    int modelRow = table.convertRowIndexToModel(selectedRow);  // account for sorting
    model.removeRow(modelRow);
}

// Update a cell
model.setValueAt(3.95, 0, 3);   // row 0, column 3 (GPA)
```

### Listening for Selection

```java
table.getSelectionModel().addListSelectionListener(e -> {
    if (!e.getValueIsAdjusting()) {
        int row = table.getSelectedRow();
        if (row >= 0) {
            int modelRow = table.convertRowIndexToModel(row);
            System.out.println("Selected: " + model.getValueAt(modelRow, 1));
        }
    }
});
```

### JList — Simpler List Display

```java
DefaultListModel<String> listModel = new DefaultListModel<>();
listModel.addElement("Alice");
listModel.addElement("Bob");
listModel.addElement("Carlos");

JList<String> list = new JList<>(listModel);
list.setSelectionMode(ListSelectionModel.SINGLE_SELECTION);

list.addListSelectionListener(e -> {
    if (!e.getValueIsAdjusting()) {
        System.out.println("Selected: " + list.getSelectedValue());
    }
});

// Add / remove items dynamically
listModel.addElement("Diana");
listModel.remove(0);
```

---

## 11. Drag and Drop

Swing has built-in support for drag and drop between components and even between applications.

### Enabling Drag and Drop on a Text Component

```java
JTextField source = new JTextField("Drag me!");
JTextField target = new JTextField("Drop here");

source.setDragEnabled(true);   // enable dragging
target.setTransferHandler(new TransferHandler("text"));   // accept text drops
```

### Custom Drag Source and Drop Target

```java
JLabel dragLabel = new JLabel("Drag this label");
dragLabel.setTransferHandler(new TransferHandler("text"));

dragLabel.addMouseListener(new MouseAdapter() {
    @Override
    public void mousePressed(MouseEvent e) {
        TransferHandler handler = ((JComponent) e.getSource()).getTransferHandler();
        handler.exportAsDrag((JComponent) e.getSource(), e, TransferHandler.COPY);
    }
});

JTextArea dropArea = new JTextArea(5, 20);
dropArea.setTransferHandler(new TransferHandler("text"));
```

---

## 12. Putting It All Together

Here's a complete Student Manager application using MVC, `JTable`, menus, a custom dialog, file saving, and `SwingWorker` for a simulated load operation:

```java
import javax.swing.*;
import javax.swing.table.*;
import java.awt.*;
import java.awt.event.*;
import java.io.*;
import java.nio.file.*;
import java.util.*;

public class StudentManagerApp extends JFrame {

    // --- Model ---
    private final DefaultTableModel tableModel;

    // --- View components ---
    private final JTable      table;
    private final JLabel      statusBar;
    private final JProgressBar progressBar;

    public StudentManagerApp() {
        setTitle("Student Manager");
        setSize(700, 500);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        // Table
        String[] cols = {"ID", "Name", "Age", "GPA"};
        tableModel = new DefaultTableModel(cols, 0) {
            @Override public boolean isCellEditable(int r, int c) { return c != 0; }
        };
        table = new JTable(tableModel);
        table.setAutoCreateRowSorter(true);
        table.setRowHeight(24);
        table.getTableHeader().setReorderingAllowed(false);

        // Status bar
        statusBar   = new JLabel("Ready.");
        progressBar = new JProgressBar(0, 100);
        progressBar.setVisible(false);

        JPanel south = new JPanel(new BorderLayout());
        south.add(statusBar,   BorderLayout.CENTER);
        south.add(progressBar, BorderLayout.EAST);

        // Main layout
        setLayout(new BorderLayout());
        add(createToolBar(),         BorderLayout.NORTH);
        add(new JScrollPane(table),  BorderLayout.CENTER);
        add(south,                   BorderLayout.SOUTH);
        setJMenuBar(createMenuBar());

        loadSampleData();
    }

    // --- Menu ---
    private JMenuBar createMenuBar() {
        JMenuBar bar = new JMenuBar();
        JMenu file = new JMenu("File");
        JMenu edit = new JMenu("Edit");

        JMenuItem save = new JMenuItem("Save to CSV");
        JMenuItem exit = new JMenuItem("Exit");
        save.setAccelerator(KeyStroke.getKeyStroke(KeyEvent.VK_S, ActionEvent.CTRL_MASK));
        save.addActionListener(e -> saveToCSV());
        exit.addActionListener(e -> System.exit(0));
        file.add(save); file.addSeparator(); file.add(exit);

        JMenuItem addItem    = new JMenuItem("Add Student");
        JMenuItem deleteItem = new JMenuItem("Delete Selected");
        addItem.addActionListener(e    -> showAddDialog());
        deleteItem.addActionListener(e -> deleteSelected());
        edit.add(addItem); edit.add(deleteItem);

        bar.add(file); bar.add(edit);
        return bar;
    }

    // --- Toolbar ---
    private JToolBar createToolBar() {
        JToolBar bar = new JToolBar();
        bar.setFloatable(false);

        JButton addBtn    = new JButton("Add");
        JButton deleteBtn = new JButton("Delete");
        JButton loadBtn   = new JButton("Simulate Load");

        addBtn.addActionListener(e    -> showAddDialog());
        deleteBtn.addActionListener(e -> deleteSelected());
        loadBtn.addActionListener(e   -> simulateLoad());

        bar.add(addBtn); bar.add(deleteBtn);
        bar.addSeparator();
        bar.add(loadBtn);
        return bar;
    }

    // --- Dialogs ---
    private void showAddDialog() {
        JTextField nameField = new JTextField(15);
        JTextField ageField  = new JTextField(5);
        JTextField gpaField  = new JTextField(5);

        JPanel form = new JPanel(new GridLayout(3, 2, 5, 5));
        form.add(new JLabel("Name:")); form.add(nameField);
        form.add(new JLabel("Age:"));  form.add(ageField);
        form.add(new JLabel("GPA:"));  form.add(gpaField);

        int result = JOptionPane.showConfirmDialog(this, form,
            "Add Student", JOptionPane.OK_CANCEL_OPTION);

        if (result == JOptionPane.OK_OPTION) {
            try {
                String name = nameField.getText().trim();
                int    age  = Integer.parseInt(ageField.getText().trim());
                double gpa  = Double.parseDouble(gpaField.getText().trim());
                int    id   = tableModel.getRowCount() + 1;

                if (name.isEmpty()) throw new IllegalArgumentException("Name required.");
                if (gpa < 0 || gpa > 4.0) throw new IllegalArgumentException("GPA must be 0.0–4.0.");

                tableModel.addRow(new Object[]{id, name, age, gpa});
                statusBar.setText("Added: " + name);
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(this,
                    "Invalid number format.", "Error", JOptionPane.ERROR_MESSAGE);
            } catch (IllegalArgumentException ex) {
                JOptionPane.showMessageDialog(this,
                    ex.getMessage(), "Validation Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    }

    private void deleteSelected() {
        int row = table.getSelectedRow();
        if (row < 0) {
            JOptionPane.showMessageDialog(this, "Please select a row to delete.");
            return;
        }
        int confirm = JOptionPane.showConfirmDialog(this,
            "Delete selected student?", "Confirm", JOptionPane.YES_NO_OPTION);
        if (confirm == JOptionPane.YES_OPTION) {
            String name = (String) tableModel.getValueAt(
                table.convertRowIndexToModel(row), 1);
            tableModel.removeRow(table.convertRowIndexToModel(row));
            statusBar.setText("Deleted: " + name);
        }
    }

    // --- File I/O ---
    private void saveToCSV() {
        try {
            List<String> lines = new ArrayList<>();
            lines.add("ID,Name,Age,GPA");
            for (int i = 0; i < tableModel.getRowCount(); i++) {
                lines.add(tableModel.getValueAt(i, 0) + ","
                    + tableModel.getValueAt(i, 1) + ","
                    + tableModel.getValueAt(i, 2) + ","
                    + tableModel.getValueAt(i, 3));
            }
            Files.write(Paths.get("students.csv"), lines);
            statusBar.setText("Saved to students.csv (" + tableModel.getRowCount() + " records)");
        } catch (IOException e) {
            JOptionPane.showMessageDialog(this,
                "Save failed: " + e.getMessage(), "Error", JOptionPane.ERROR_MESSAGE);
        }
    }

    // --- SwingWorker ---
    private void simulateLoad() {
        progressBar.setVisible(true);
        progressBar.setValue(0);
        statusBar.setText("Loading...");

        new SwingWorker<Void, Integer>() {
            @Override
            protected Void doInBackground() throws Exception {
                for (int i = 0; i <= 100; i += 5) {
                    Thread.sleep(80);
                    publish(i);
                }
                return null;
            }

            @Override
            protected void process(java.util.List<Integer> chunks) {
                progressBar.setValue(chunks.get(chunks.size() - 1));
            }

            @Override
            protected void done() {
                progressBar.setVisible(false);
                statusBar.setText("Load complete.");
            }
        }.execute();
    }

    private void loadSampleData() {
        Object[][] sample = {
            {1, "Alice",  20, 3.85},
            {2, "Bob",    22, 3.42},
            {3, "Carlos", 21, 3.91},
            {4, "Diana",  23, 3.67}
        };
        for (Object[] row : sample) tableModel.addRow(row);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new StudentManagerApp().setVisible(true));
    }
}
```

This application combines: MVC structure, `JTable` with sorting, menus and toolbar, a custom add dialog with validation, file saving with NIO.2, a confirmation dialog, `SwingWorker` for background progress, and a status bar — a complete, professional-grade desktop application.

---

## 13. Exercises

---

### Exercise 1 – MVC Temperature Converter

Build a temperature converter using the MVC pattern:

**Model:** Holds the current temperature and unit. Methods: `setCelsius(double)`, `setFahrenheit(double)`, `getCelsius()`, `getFahrenheit()`.

**View:** Two `JTextField`s (one for Celsius, one for Fahrenheit), two labels, a `JComboBox` for scale selection.

**Controller:** When either field changes, update the model and refresh the other field.

Keep all conversion logic in the Model — the View should not do any math.

---

### Exercise 2 – Drawing Application

Build a simple drawing app:

1. A `DrawingPanel` that extends `JPanel`.
2. Track mouse drag events to draw lines (store a list of points or line segments).
3. A toolbar with color buttons (Red, Blue, Green, Black) and a stroke-width `JSpinner`.
4. A "Clear" button that wipes the canvas.
5. A "Save" button that prompts with `JFileChooser` and saves the drawing as a list of coordinates to a text file.

---

### Exercise 3 – Tabbed Settings Panel

Build a settings window using `JTabbedPane` with three tabs:

**Display tab:** Font size `JSpinner`, dark mode `JCheckBox`, theme `JComboBox` (Light/Dark/System).

**Profile tab:** Name, email, and bio `JTextArea` fields.

**Notifications tab:** `JCheckBox` items for Email, SMS, and Push notifications with a frequency `JRadioButton` group (Instant / Daily / Weekly).

A "Save" button at the bottom prints all current settings to the console.

---

### Exercise 4 – Sortable Data Table

Build a contact manager with a `JTable`:

1. Columns: Name, Phone, Email, City.
2. A toolbar with Add, Edit (opens a dialog pre-filled with selected row), and Delete buttons.
3. A search `JTextField` that filters the table in real time as the user types (filter by name).
4. Click any column header to sort by that column.
5. Save all contacts to `contacts.csv` on exit; load them on startup.

---

### Exercise 5 – Progress and Background Work

Build an app with:

1. A `JTextArea` (read-only) to display output.
2. A "Start Processing" button that launches a `SwingWorker` simulating 50 steps of work (each step sleeps 100ms and appends a line to the text area).
3. A `JProgressBar` that updates with each step.
4. A "Cancel" button that calls `worker.cancel(true)` and prints "Cancelled."
5. The UI must stay fully responsive during processing.

---

### Exercise 6 – Mini Text Editor *(Challenge)*

Build a functional text editor application:

**Features:**
- A `JTextArea` in a `JScrollPane` as the main editing area.
- **File menu:** New, Open (`JFileChooser`), Save, Save As, Exit. Track unsaved changes — prompt before closing.
- **Edit menu:** Cut, Copy, Paste, Select All, Find & Replace (custom `JDialog`).
- **Format menu:** Word wrap toggle, font chooser (name, size, bold, italic).
- **View menu:** Show/hide status bar (shows line count and character count).
- A toolbar with New, Open, Save buttons.
- **Keyboard shortcuts** for all menu actions (Ctrl+N, Ctrl+O, Ctrl+S, Ctrl+Z, etc.).
- Title bar shows the current filename and an asterisk (`*`) when there are unsaved changes.

---

## Module Summary

Here's what you covered in Module 8:

- **MVC** separates Model (data), View (UI), and Controller (logic) — cleaner, testable code.
- **`SwingUtilities.invokeLater()`** ensures all UI operations run on the Event Dispatch Thread.
- **`GridBagLayout`** is the most flexible layout — components can span rows/columns with fine-grained control.
- **`BoxLayout`** stacks components vertically or horizontally with `Box.createVerticalStrut()` for spacing.
- **Custom painting** uses `paintComponent(Graphics g)` — cast to `Graphics2D` for antialiasing, gradients, and shapes.
- **`javax.swing.Timer`** drives animation by repeatedly calling `repaint()`.
- **`MouseAdapter`**, **`KeyAdapter`**, and **key bindings** handle rich user interaction.
- **Menus** (`JMenuBar`, `JMenu`, `JMenuItem`) and **toolbars** (`JToolBar`) organize actions professionally.
- **Advanced components:** `JTabbedPane`, `JSplitPane`, `JProgressBar`, `JSpinner`, `JFileChooser`, `JColorChooser`.
- **Custom `JDialog`** blocks the parent window and collects complex user input.
- **`SwingWorker<T, V>`** runs background tasks without freezing the UI — use `doInBackground()`, `publish()`, `process()`, and `done()`.
- **`JTable`** with `DefaultTableModel` displays and manages tabular data; `setAutoCreateRowSorter(true)` adds column sorting.
- **`JList`** with `DefaultListModel` provides a simpler dynamic list component.

---

## 14. Course Wrap-Up

Congratulations — you have completed **CS 1103 Programming 2 (Java)**! 🎉

Here's a summary of everything you've mastered across both courses:

### CS 1102 — Programming 1

| Module | Topic |
|---|---|
| 1 | Introduction to Java — JVM, types, Scanner |
| 2 | Control Structures — if/else, loops, switch |
| 3 | Static Methods & Member Variables — methods, scope |
| 4 | Arrays & ArrayLists — fixed and dynamic collections |
| 5 | Objects & Classes — constructors, encapsulation, getters/setters |
| 6 | OOP Paradigms — inheritance, polymorphism, abstract classes, interfaces |
| 7 | GUI & Event Handling — Swing basics, layouts, listeners |
| 8 | Collection API & Java 8 — Set, Map, streams, lambdas, Optional |

### CS 1103 — Programming 2

| Module | Topic |
|---|---|
| 1 | Exception & String Handling — try/catch, custom exceptions, regex |
| 2 | Packages — namespaces, access modifiers, project structure |
| 3 | Multithreading — threads, synchronization, Executors, Future |
| 4 | I/O & Applets — streams, NIO.2, serialization |
| 5 | JDBC — SQL, PreparedStatement, transactions, DAO pattern |
| 6 | Generic Programming — type parameters, bounds, wildcards, Comparable |
| 7 | Files & Networking — directory walking, sockets, TCP/UDP, HTTP |
| 8 | Advanced GUI — MVC, custom graphics, SwingWorker, JTable |

### Where to Go From Here

You now have a solid, professional foundation in Java. Paths you can explore next:

- **Spring Boot** — the industry-standard framework for building web applications and REST APIs in Java.
- **Android Development** — use your Java skills to build mobile apps.
- **JavaFX** — a modern replacement for Swing with CSS styling and FXML layouts.
- **Design Patterns** — Singleton, Factory, Observer, Strategy, and more.
- **Data Structures & Algorithms** — deepen your understanding of how collections work internally.
- **Testing** — JUnit 5 and Mockito for writing reliable, testable code.
- **Build Tools** — Maven and Gradle for managing dependencies and building projects professionally.

The skills you've built here — OOP, generics, concurrency, I/O, networking, databases, and GUI — are exactly what professional Java developers use every day. You're well prepared.

---

*CS 1103 Programming 2 (Java) — Module 8 of 8 — Course Complete*
