# Mediator Pattern

## Introduction

The Mediator pattern defines an object that encapsulates how a set of objects interact. It promotes loose coupling by keeping objects from referring to each other explicitly and lets you vary their interaction independently.

## Real-World Applications

- **Air traffic control** – Aircraft communicate through a control tower (mediator), not directly with each other. The tower manages landing queues, runway assignments, and collision avoidance.
- **Chat rooms** – Users in a chat room send messages to the room server (mediator), which distributes them to all other participants.
- **GUI dialog boxes** – A dialog mediator coordinates interactions between buttons, text fields, dropdowns, and sliders without them knowing about each other.
- **Smart home systems** – A central hub mediates between lights, thermostat, door locks, and sensors so they don't need direct connections.
- **Database triggers** – A mediator manages interactions between tables when data changes, ensuring referential integrity.

## Components

| Component | Description |
|-----------|-------------|
| **Mediator** | Declares the interface for communication with colleague objects. |
| **ConcreteMediator** | Implements cooperative behavior by coordinating colleague objects; knows and maintains references to colleagues. |
| **Colleague** | Each colleague object knows its mediator object; communicates with other colleagues only through the mediator. |

## Code Example

### Problem

You are building a chat application. If every user directly references every other user, the system becomes a tangled web of dependencies. Adding a new user means every existing user must be updated. Broadcasting a message requires iterating over all known users, violating the Open/Closed Principle.

### Solution

The Mediator pattern introduces a `ChatServer` that acts as the central hub. Users only know the server; they send messages to the server, which distributes them to all other users or to specific recipients. Adding a new user requires no changes to existing user classes.

```java
// Mediator
interface ChatServer {
    void sendMessage(String message, User sender);
    void registerUser(User user);
}

// ConcreteMediator
class ChatRoom implements ChatServer {
    private List<User> users = new ArrayList<>();

    public void registerUser(User user) {
        users.add(user);
        user.setChatServer(this);
    }

    public void sendMessage(String message, User sender) {
        for (User user : users) {
            if (user != sender) {
                user.receive(message, sender);
            }
        }
    }
}

// Colleague
abstract class User {
    protected String name;
    protected ChatServer chatServer;

    public User(String name) { this.name = name; }
    public void setChatServer(ChatServer server) { this.chatServer = server; }

    public abstract void send(String message);
    public abstract void receive(String message, User sender);

    public String getName() { return name; }
}

// ConcreteColleague
class ChatUser extends User {
    public ChatUser(String name) { super(name); }

    public void send(String message) {
        System.out.println(name + " sends: " + message);
        chatServer.sendMessage(message, this);
    }

    public void receive(String message, User sender) {
        System.out.println(name + " received from " + sender.getName() + ": " + message);
    }
}

// Client
public class Main {
    public static void main(String[] args) {
        ChatRoom chatRoom = new ChatRoom();

        User alice = new ChatUser("Alice");
        User bob = new ChatUser("Bob");
        User charlie = new ChatUser("Charlie");

        chatRoom.registerUser(alice);
        chatRoom.registerUser(bob);
        chatRoom.registerUser(charlie);

        alice.send("Hello everyone!");
    }
}
```

## Advantages and Disadvantages

### Advantages
- **Reduced Coupling** – Colleagues are loosely coupled; they interact only with the mediator, not with each other.
- **Centralized Control** – Interaction logic is centralized in one place, making it easier to understand, maintain, and modify.
- **Simplified Colleagues** – Individual colleague classes become simpler because they do not need to manage their own interactions.
- **Reusability** – Colleagues can be reused independently since they depend only on the mediator interface.

### Disadvantages
- **Mediator Complexity** – The mediator can become a "god object" that knows too much, accumulating all the interaction logic.
- **Performance Bottleneck** – All communication passes through the mediator, creating a potential single point of failure.
- **Increased Complexity** – Adding a mediator introduces an extra layer of indirection; for simple interactions, direct communication may be simpler.
- **Debugging Difficulty** – Tracing the flow of communication through the mediator can be more challenging than following direct references.
