# Memento Pattern

## Introduction

The Memento pattern captures and externalizes an object's internal state so that the object can be restored to this state later, without violating encapsulation. It enables undo/redo, rollback, and checkpoint functionality while keeping the object's internal details hidden.

## Real-World Applications

- **Undo operations in editors** – A text editor saves snapshots of the document state before each operation; Ctrl+Z restores the previous snapshot.
- **Game save points** – A game captures the player's state (health, inventory, position) as a memento; the player can reload from a saved checkpoint.
- **Database transactions** – The database captures the state before a transaction begins; if the transaction fails, the state is rolled back.
- **Version control systems** – Git commits are mementos of the repository state, allowing rollback to any previous commit.
- **Wizard dialogs** – A multi-step wizard saves the state after each step so the user can click "Back" without losing data.

## Components

| Component | Description |
|-----------|-------------|
| **Originator** | Creates a memento containing a snapshot of its current internal state; uses the memento to restore its state. |
| **Memento** | Stores the internal state of the `Originator` object; prevents objects other than the originator from accessing its contents. |
| **Caretaker** | Is responsible for the memento's safekeeping; never operates on or examines the contents of a memento. |

## Code Example

### Problem

You are building a text editor with an undo feature. The `TextDocument` class stores its state (content, cursor position, selection) as private fields. Exposing these fields to support undo would break encapsulation. You need a way to save and restore state without revealing internal details.

### Solution

The Memento pattern lets the `TextDocument` (originator) produce a snapshot of its state in a `DocumentMemento` object. The `History` (caretaker) stores these mementos without inspecting them. When the user requests undo, the caretaker returns the last memento, and the originator restores its state from it.

```java
// Memento
class DocumentMemento {
    private final String content;
    private final int cursorPosition;

    public DocumentMemento(String content, int cursorPosition) {
        this.content = content;
        this.cursorPosition = cursorPosition;
    }

    public String getContent() { return content; }
    public int getCursorPosition() { return cursorPosition; }
}

// Originator
class TextDocument {
    private StringBuilder content = new StringBuilder();
    private int cursorPosition = 0;

    public void write(String text) {
        content.insert(cursorPosition, text);
        cursorPosition += text.length();
    }

    public DocumentMemento save() {
        return new DocumentMemento(content.toString(), cursorPosition);
    }

    public void restore(DocumentMemento memento) {
        this.content = new StringBuilder(memento.getContent());
        this.cursorPosition = memento.getCursorPosition();
    }

    @Override
    public String toString() {
        return "Content: '" + content + "' | Cursor at: " + cursorPosition;
    }
}

// Caretaker
class History {
    private Stack<DocumentMemento> mementos = new Stack<>();

    public void push(DocumentMemento memento) {
        mementos.push(memento);
    }

    public DocumentMemento pop() {
        if (!mementos.isEmpty()) {
            return mementos.pop();
        }
        return null;
    }
}

// Client
public class Main {
    public static void main(String[] args) {
        TextDocument doc = new TextDocument();
        History history = new History();

        doc.write("Hello");
        history.push(doc.save());

        doc.write(" World");
        history.push(doc.save());

        doc.write("!!!");

        System.out.println("Before undo: " + doc);

        doc.restore(history.pop());
        System.out.println("After undo 1: " + doc);

        doc.restore(history.pop());
        System.out.println("After undo 2: " + doc);
    }
}
```

## Advantages and Disadvantages

### Advantages
- **Encapsulation Preservation** – The originator's internal state is saved and restored without exposing its private fields.
- **Simplified Originator** – The originator does not need to manage its own history; the caretaker handles it.
- **Undo/Redo Support** – Provides a clean mechanism for implementing undo, redo, and rollback.
- **Snapshot Flexibility** – Snapshots can be taken at any granularity and at any point in time.

### Disadvantages
- **Memory Consumption** – Storing many mementos can consume significant memory, especially if the originator's state is large.
- **Overhead** – Creating and managing mementos adds computational overhead.
- **Caretaker Responsibility** – The caretaker must manage the lifecycle of mementos, including deciding when to discard old ones.
- **No Incremental State** – The pattern typically saves full snapshots rather than deltas, which can be wasteful for small changes.
