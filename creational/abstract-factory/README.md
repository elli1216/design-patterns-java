# Abstract Factory Pattern

## Introduction

The Abstract Factory pattern provides an interface for creating families of related or dependent objects without specifying their concrete classes. It is essentially a factory of factories, ensuring that products from the same family are used together.

## Real-World Applications

- **Cross-platform GUI toolkits** – A `GUIFactory` produces buttons, checkboxes, and scrollbars that match the look-and-feel of Windows, macOS, or Linux.
- **Furniture store** – A `FurnitureFactory` creates matching sets of chairs, sofas, and coffee tables (Victorian, Modern, Art Deco).
- **Database migration tools** – A `QueryFactory` generates SQL dialects specific to different database engines (MySQL, PostgreSQL, Oracle).
- **Game development** – A `FantasyGameFactory` creates elves, orcs, and castles, while a `SciFiGameFactory` creates aliens, robots, and space stations.

## Components

| Component | Description |
|-----------|-------------|
| **AbstractFactory** | Declares an interface for operations that create abstract product objects. |
| **ConcreteFactory** | Implements the operations to create concrete product objects. |
| **AbstractProduct** | Declares an interface for a type of product object. |
| **ConcreteProduct** | Defines a product object to be created by the corresponding concrete factory; implements the `AbstractProduct` interface. |
| **Client** | Uses only interfaces declared by `AbstractFactory` and `AbstractProduct` classes. |

## Code Example

### Problem

You are building a furniture shop simulator. The shop needs to produce matching sets of furniture – chairs, sofas, and coffee tables. Each set shares a style (Victorian, Modern, Art Deco). Hardcoding every combination of style and furniture leads to an explosion of classes and makes it nearly impossible to introduce a new style without touching many files.

### Solution

The Abstract Factory pattern provides a `FurnitureFactory` interface with methods to create each piece of furniture. Each style (Victorian, Modern) gets its own factory implementation. The client receives a factory and uses it to produce a consistent family of products.

```java
// AbstractProduct
interface Chair {
    void sitOn();
}
interface Sofa {
    void lieOn();
}

// ConcreteProduct
class VictorianChair implements Chair {
    public void sitOn() { System.out.println("Victorian chair"); }
}
class ModernChair implements Chair {
    public void sitOn() { System.out.println("Modern chair"); }
}
class VictorianSofa implements Sofa {
    public void lieOn() { System.out.println("Victorian sofa"); }
}
class ModernSofa implements Sofa {
    public void lieOn() { System.out.println("Modern sofa"); }
}

// AbstractFactory
interface FurnitureFactory {
    Chair createChair();
    Sofa createSofa();
}

// ConcreteFactory
class VictorianFurnitureFactory implements FurnitureFactory {
    public Chair createChair() { return new VictorianChair(); }
    public Sofa createSofa()   { return new VictorianSofa(); }
}

class ModernFurnitureFactory implements FurnitureFactory {
    public Chair createChair() { return new ModernChair(); }
    public Sofa createSofa()   { return new ModernSofa(); }
}

// Client
public class Main {
    public static void main(String[] args) {
        FurnitureFactory factory = new VictorianFurnitureFactory();
        Chair chair = factory.createChair();
        Sofa sofa = factory.createSofa();
        chair.sitOn();
        sofa.lieOn();
    }
}
```

## Advantages and Disadvantages

### Advantages
- **Family Consistency** – Ensures products from the same family are used together, avoiding style mismatches.
- **Open/Closed Principle** – New product families can be added without modifying existing client code.
- **Single Responsibility Principle** – Product creation logic is isolated in concrete factory classes.
- **Loose Coupling** – Client code depends only on abstract interfaces, not concrete implementations.

### Disadvantages
- **Complexity** – Many new interfaces and classes are introduced, which can overcomplicate simple scenarios.
- **Rigid Product Structure** – Adding a new product type (e.g., a new piece of furniture) requires changes to the `AbstractFactory` interface and every `ConcreteFactory` – a violation of the Open/Closed Principle.
- **Overkill for Simple Needs** – If the application only needs a single product family, the simpler Factory Method pattern is more appropriate.
