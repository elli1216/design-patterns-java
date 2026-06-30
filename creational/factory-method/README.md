# Factory Method Pattern

## Introduction

The Factory Method pattern defines an interface for creating an object but lets subclasses decide which class to instantiate. It lets a class defer instantiation to subclasses, promoting loose coupling by eliminating the need to bind application-specific classes into the code.

## Real-World Applications

- **Logging frameworks** – A `LoggerFactory` produces different loggers (FileLogger, ConsoleLogger, DatabaseLogger) depending on configuration.
- **Document editors** – An application framework provides `DocumentFactory`; each document type (TextDocument, Spreadsheet, Drawing) has its own factory.
- **Database connectivity** – A `ConnectionFactory` returns the appropriate database connection based on the driver (MySQL, PostgreSQL, Oracle).
- **UI component libraries** – A `ButtonFactory` creates platform-specific buttons (WindowsButton, MacButton, LinuxButton).

## Components

| Component | Description |
|-----------|-------------|
| **Product** | Defines the interface of objects the factory method creates. |
| **ConcreteProduct** | Implements the `Product` interface. |
| **Creator** | Declares the factory method that returns a `Product` object. May also define a default implementation. |
| **ConcreteCreator** | Overrides the factory method to return an instance of a `ConcreteProduct`. |

## Code Example

### Problem

You are building a logistics management application. Initially it supports only road transport via trucks, but you know the business will soon request sea, air, and rail transport. Hardcoding every transport type into the core logic would make the system rigid and difficult to extend.

### Solution

The Factory Method pattern lets you decouple the client code from specific transport classes. A `Logistics` class declares a factory method `createTransport()`. Subclasses like `RoadLogistics` and `SeaLogistics` override it to instantiate the appropriate transport.

```java
// Product
interface Transport {
    void deliver();
}

// ConcreteProduct
class Truck implements Transport {
    public void deliver() {
        System.out.println("Delivering by land in a truck.");
    }
}

class Ship implements Transport {
    public void deliver() {
        System.out.println("Delivering by sea in a ship.");
    }
}

// Creator
abstract class Logistics {
    public void planDelivery() {
        Transport t = createTransport();
        t.deliver();
    }
    protected abstract Transport createTransport();
}

// ConcreteCreator
class RoadLogistics extends Logistics {
    protected Transport createTransport() {
        return new Truck();
    }
}

class SeaLogistics extends Logistics {
    protected Transport createTransport() {
        return new Ship();
    }
}

// Client
public class Main {
    public static void main(String[] args) {
        Logistics logistics = new RoadLogistics();
        logistics.planDelivery();
    }
}
```

## Advantages and Disadvantages

### Advantages
- **Single Responsibility Principle** – Product creation code is moved into one place, making the code easier to support.
- **Open/Closed Principle** – New product types can be introduced without breaking existing client code.
- **Loose Coupling** – Client code works with the abstract `Product` interface, not concrete implementations.
- **Extensibility** – Adding a new product requires only a new `ConcreteProduct` and a new `ConcreteCreator`.

### Disadvantages
- **Complexity** – The pattern introduces additional classes and interfaces, which can make the code harder to understand.
- **Subclassing Required** – The pattern relies on inheritance; refactoring an existing class to use factory methods can be invasive.
- **Parallel Class Hierarchy** – Each new product often requires a corresponding creator subclass, leading to many small classes.
