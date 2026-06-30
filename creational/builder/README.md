# Builder Pattern

## Introduction

The Builder pattern separates the construction of a complex object from its representation, allowing the same construction process to create different representations. It is especially useful when an object requires many optional parameters or a multi-step initialization.

## Real-World Applications

- **StringBuilder / StringBuffer** – Java's own `StringBuilder` lets you progressively build a string without creating intermediate immutable objects.
- **Meal builders** – A fast-food restaurant builds a meal with customizable components (main, side, drink, toy) while keeping the assembly steps consistent.
- **Query builders** – SQL query builders (e.g., jOOQ, Hibernate Criteria) use a fluent API to construct queries step by step.
- **Document generation** – HTML or PDF documents are built by adding headers, paragraphs, tables, and images one section at a time.
- **Vehicle manufacturing** – A car is assembled step-by-step (engine, wheels, seats, paint) and the same process can produce a sports car, an SUV, or a truck.

## Components

| Component | Description |
|-----------|-------------|
| **Builder** | Declares the abstract interface for creating parts of a product object. |
| **ConcreteBuilder** | Implements the `Builder` interface; constructs and assembles parts of the product; keeps track of the product it creates; provides a method to retrieve the result. |
| **Director** | Constructs an object using the `Builder` interface. |
| **Product** | Represents the complex object under construction. |

## Code Example

### Problem

You are building a computer customization tool. A `Computer` has many attributes (CPU, RAM, storage, GPU, cooling system, etc.). Some are required, others are optional. Using a telescoping constructor pattern leads to a nightmare of overloaded constructors, and using JavaBeans style (setters) allows incomplete or inconsistent objects.

### Solution

The Builder pattern provides a `ComputerBuilder` that lets clients set only the attributes they care about using a fluent interface. Once all desired attributes are configured, `build()` produces the final immutable `Computer` object.

```java
// Product
class Computer {
    private final String cpu;
    private final int ramGB;
    private final int storageGB;
    private final boolean hasGPU;
    private final String cooling;

    public Computer(String cpu, int ramGB, int storageGB, boolean hasGPU, String cooling) {
        this.cpu = cpu;
        this.ramGB = ramGB;
        this.storageGB = storageGB;
        this.hasGPU = hasGPU;
        this.cooling = cooling;
    }

    @Override
    public String toString() {
        return "Computer [CPU=" + cpu + ", RAM=" + ramGB + "GB, Storage=" + storageGB + "GB]";
    }
}

// Builder
class ComputerBuilder {
    private String cpu;
    private int ramGB;
    private int storageGB;
    private boolean hasGPU;
    private String cooling;

    public ComputerBuilder setCPU(String cpu) {
        this.cpu = cpu;
        return this;
    }

    public ComputerBuilder setRAM(int ramGB) {
        this.ramGB = ramGB;
        return this;
    }

    public ComputerBuilder setStorage(int storageGB) {
        this.storageGB = storageGB;
        return this;
    }

    public ComputerBuilder setGPU(boolean hasGPU) {
        this.hasGPU = hasGPU;
        return this;
    }

    public ComputerBuilder setCooling(String cooling) {
        this.cooling = cooling;
        return this;
    }

    public Computer build() {
        return new Computer(cpu, ramGB, storageGB, hasGPU, cooling);
    }
}

// Client
public class Main {
    public static void main(String[] args) {
        Computer computer = new ComputerBuilder()
                .setCPU("Intel i7")
                .setRAM(16)
                .setStorage(512)
                .setGPU(true)
                .setCooling("Liquid")
                .build();
        System.out.println(computer);
    }
}
```

## Advantages and Disadvantages

### Advantages
- **Step-by-Step Construction** – Complex objects are built in clearly defined stages, improving readability.
- **Reusable Construction Process** – The same builder can produce different representations of the product.
- **Immutability** – The final product can be immutable, as all attributes are set through the builder before `build()` is called.
- **Fluent Interface** – Method chaining produces readable, expressive client code.
- **Encapsulation** – The product's internal structure is hidden from the client.

### Disadvantages
- **Code Duplication** – The builder often duplicates fields from the product class.
- **Overhead** – An extra builder class is needed, increasing the overall number of classes.
- **Rigid Product Structure** – Adding a new field to the product requires updating both the product class and its builder.
- **Not Suitable for Simple Objects** – The pattern is overkill when an object has only a few fields or no complex construction logic.
