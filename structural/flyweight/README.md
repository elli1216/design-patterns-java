# Flyweight Pattern

## Introduction

The Flyweight pattern minimizes memory usage by sharing as much data as possible with similar objects. It is used when a large number of objects are needed and many of them share common intrinsic state that can be stored externally and reused.

## Real-World Applications

- **Text editors** – Each character in a document does not store its font, size, and style individually. Instead, characters reference shared flyweight objects that represent character formatting.
- **Game development** – Thousands of trees, bullets, or particles in a game world share intrinsic properties (mesh, texture, color) while extrinsic properties (position, velocity) are computed per instance.
- **String interning** – Java's string pool ensures that identical string literals share the same underlying `char[]` data, reducing memory for repeated strings.
- **Database connection pooling** – Connection objects are reused rather than created and destroyed for each request.
- **Cache systems** – Frequently requested data objects are cached as flyweights and shared across requests.

## Components

| Component | Description |
|-----------|-------------|
| **Flyweight** | Declares an interface through which flyweights can receive and act on extrinsic state. |
| **ConcreteFlyweight** | Implements the `Flyweight` interface and stores intrinsic state (shared). |
| **UnsharedConcreteFlyweight** | Not all flyweight subclasses need to be shared; unshared flyweights can have children. |
| **FlyweightFactory** | Creates and manages flyweight objects; ensures that flyweights are shared properly. |
| **Client** | Maintains references to flyweights and computes or stores extrinsic state. |

## Code Example

### Problem

You are building a game with thousands of trees scattered across a map. Each `Tree` object stores its type (name, color, texture), position (x, y), and health. The type data (name, color, texture) is identical for every tree of the same species. Storing this duplicate data in every tree instance wastes a significant amount of memory.

### Solution

The Flyweight pattern splits tree data into intrinsic state (tree type: name, color, texture) stored in a shared flyweight, and extrinsic state (position, health) that the client maintains per tree. A `TreeFactory` ensures each tree type is created only once and shared across all trees of that species.

```java
// Flyweight (intrinsic state)
class TreeType {
    private String name;
    private String color;
    private String texture;

    public TreeType(String name, String color, String texture) {
        this.name = name;
        this.color = color;
        this.texture = texture;
    }

    public void display(int x, int y) {
        System.out.println(name + " tree at (" + x + "," + y + ") - " + color + ", " + texture);
    }
}

// FlyweightFactory
class TreeFactory {
    private static Map<String, TreeType> treeTypes = new HashMap<>();

    public static TreeType getTreeType(String name, String color, String texture) {
        String key = name + "|" + color + "|" + texture;
        if (!treeTypes.containsKey(key)) {
            treeTypes.put(key, new TreeType(name, color, texture));
        }
        return treeTypes.get(key);
    }
}

// Context (extrinsic state)
class Tree {
    private int x, y;
    private TreeType type;

    public Tree(int x, int y, TreeType type) {
        this.x = x;
        this.y = y;
        this.type = type;
    }

    public void display() {
        type.display(x, y);
    }
}

// Client
public class Main {
    public static void main(String[] args) {
        List<Tree> forest = new ArrayList<>();

        forest.add(new Tree(10, 20, TreeFactory.getTreeType("Oak", "Green", "Rough")));
        forest.add(new Tree(30, 40, TreeFactory.getTreeType("Pine", "Dark Green", "Smooth")));
        forest.add(new Tree(50, 60, TreeFactory.getTreeType("Oak", "Green", "Rough"))); // Reuses Oak

        for (Tree tree : forest) {
            tree.display();
        }
    }
}
```

## Advantages and Disadvantages

### Advantages
- **Reduced Memory Footprint** – Dramatically reduces memory consumption by sharing intrinsic state among many objects.
- **Improved Performance** – Less memory allocation means reduced GC pressure and better cache locality.
- **Scalability** – Enables the creation of thousands or millions of fine-grained objects that would otherwise exhaust memory.

### Disadvantages
- **Complexity** – Separating intrinsic and extrinsic state adds design complexity and makes the code harder to follow.
- **Performance Trade-off** – Managing extrinsic state requires additional computation (lookups, context passing) that may offset some gains.
- **Thread Safety** – Shared flyweight objects must be immutable or carefully synchronized to avoid race conditions.
- **Premature Optimization** – The pattern is often overkill; profiling should confirm that memory is actually a bottleneck before applying Flyweight.
