# Command Pattern

## Introduction

The Command pattern encapsulates a request as an object, allowing parameterization of clients with different requests, queuing or logging of requests, and support for undoable operations. It turns a request into a stand-alone object that contains all information about the request.

## Real-World Applications

- **Undo/redo in editors** – Each user action (typing, delete, formatting) is a `Command` object with `execute()` and `undo()` methods, stored in a history stack.
- **GUI button actions** – A `Button` control does not know what action it triggers; it simply invokes `execute()` on a command object passed to it.
- **Transaction processing** – Database transactions are commands that can be committed or rolled back.
- **Job scheduling** – Tasks are encapsulated as command objects that can be queued, scheduled, or executed immediately.
- **Remote control systems** – Each button on a universal remote stores a command object (TurnOn, TurnOff, VolumeUp, ChannelChange).

## Components

| Component | Description |
|-----------|-------------|
| **Command** | Declares an interface for executing an operation. |
| **ConcreteCommand** | Defines a binding between a `Receiver` object and an action; implements `execute()` by invoking the corresponding operations on `Receiver`. |
| **Receiver** | Knows how to perform the operations associated with carrying out a request. |
| **Invoker** | Asks the command to carry out the request. |
| **Client** | Creates a `ConcreteCommand` object and sets its receiver. |

## Code Example

### Problem

You are building a text editor with actions like copy, paste, cut, and undo. If each action is hardcoded into the menu items and toolbar buttons, adding a new action requires modifying every UI component. Furthermore, implementing undo/redo would require tracking state changes across all actions.

### Solution

The Command pattern encapsulates each action as a `Command` object with `execute()` and `undo()` methods. A command history stack stores executed commands. Undo pops the last command and calls `undo()`. New actions can be added by writing new command classes without touching UI code.

```java
// Command
interface Command {
    void execute();
    void undo();
}

// Receiver
class TextDocument {
    private StringBuilder text = new StringBuilder();
    private String lastClipboard = "";

    public void insert(String content, int position) {
        text.insert(position, content);
    }

    public void delete(int start, int end) {
        text.delete(start, end);
    }

    public String getText() { return text.toString(); }
}

// ConcreteCommand
class PasteCommand implements Command {
    private TextDocument doc;
    private String content;
    private int position;

    public PasteCommand(TextDocument doc, String content, int position) {
        this.doc = doc;
        this.content = content;
        this.position = position;
    }

    public void execute() {
        doc.insert(content, position);
    }

    public void undo() {
        doc.delete(position, position + content.length());
    }
}

// Invoker
class Editor {
    private Command command;
    private Stack<Command> history = new Stack<>();

    public void setCommand(Command command) {
        this.command = command;
    }

    public void execute() {
        command.execute();
        history.push(command);
    }

    public void undo() {
        if (!history.isEmpty()) {
            Command cmd = history.pop();
            cmd.undo();
        }
    }
}

// Client
public class Main {
    public static void main(String[] args) {
        TextDocument doc = new TextDocument();
        Editor editor = new Editor();

        Command paste = new PasteCommand(doc, "Hello World", 0);
        editor.setCommand(paste);
        editor.execute();
        System.out.println(doc.getText());

        editor.undo();
        System.out.println(doc.getText());
    }
}
```

## Advantages and Disadvantages

### Advantages
- **Decoupling** – The invoker is decoupled from the receiver; the invoker knows only the `Command` interface.
- **Undo/Redo Support** – Commands can store state and support reversible operations.
- **Queuing and Logging** – Commands can be queued for later execution or logged for auditing.
- **Composite Commands** – Commands can be composed into macros (via the Composite pattern).
- **Open/Closed Principle** – New commands can be added without modifying existing code.

### Disadvantages
- **Class Explosion** – Each individual action requires a separate command class, leading to many small classes.
- **Increased Memory** – Storing command history for undo can consume significant memory for complex operations.
- **Complexity for Simple Actions** – The pattern introduces overhead that may not be justified for simple, one-off actions.
