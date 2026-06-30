# State Pattern

## Introduction

The State pattern allows an object to alter its behavior when its internal state changes. The object appears to change its class. It encapsulates state-specific behavior into separate classes and delegates the state-dependent behavior to the current state object.

## Real-World Applications

- **Vending machines** – A vending machine behaves differently in the "Idle", "HasMoney", "Dispensing", and "OutOfStock" states. Each state defines the behavior for inserting money, selecting a product, and dispensing.
- **Traffic lights** – A traffic light cycles through Green → Yellow → Red states, with each state determining the duration and next transition.
- **Document workflow** – A document transitions through Draft → Review → Approved → Published states; each state defines which actions are allowed (e.g., editing is allowed in Draft but not in Published).
- **Media players** – A music player in Playing, Paused, or Stopped state responds differently to play/pause and stop buttons.
- **TCP connections** – A TCP socket transitions through Established, Listening, Closed states, with different behaviors for sending and receiving data.

## Components

| Component | Description |
|-----------|-------------|
| **Context** | Defines the interface of interest to clients; maintains an instance of a `ConcreteState` subclass that defines the current state. |
| **State** | Declares an interface for encapsulating the behavior associated with a particular state of the `Context`. |
| **ConcreteState** | Each subclass implements a behavior associated with a state of the `Context`. |

## Code Example

### Problem

You are building a vending machine. Its behavior depends on its current state: Idle, HasMoney, Dispensing, or OutOfStock. Using a single class with conditional checks (`if (state == IDLE) ... else if (state == HAS_MONEY) ...`) leads to complex, error-prone code that is hard to extend when a new state is added.

### Solution

The State pattern encapsulates each state's behavior in a separate class. The `VendingMachine` context holds a reference to the current state object. When the state changes, the context swaps its state object. Adding a new state requires only a new class, not modifying existing conditionals.

```java
// State
interface VendingMachineState {
    void insertCoin();
    void selectProduct();
    void dispense();
}

// ConcreteState
class IdleState implements VendingMachineState {
    private VendingMachine machine;

    public IdleState(VendingMachine machine) { this.machine = machine; }

    public void insertCoin() {
        System.out.println("Coin inserted");
        machine.setState(machine.getHasMoneyState());
    }

    public void selectProduct() {
        System.out.println("Insert coin first");
    }

    public void dispense() {
        System.out.println("Insert coin first");
    }
}

class HasMoneyState implements VendingMachineState {
    private VendingMachine machine;

    public HasMoneyState(VendingMachine machine) { this.machine = machine; }

    public void insertCoin() {
        System.out.println("Already have coin");
    }

    public void selectProduct() {
        System.out.println("Product selected");
        machine.setState(machine.getDispensingState());
    }

    public void dispense() {
        System.out.println("Select a product first");
    }
}

class DispensingState implements VendingMachineState {
    private VendingMachine machine;

    public DispensingState(VendingMachine machine) { this.machine = machine; }

    public void insertCoin() { System.out.println("Please wait, dispensing"); }
    public void selectProduct() { System.out.println("Please wait, dispensing"); }

    public void dispense() {
        System.out.println("Dispensing product...");
        machine.setState(machine.getIdleState());
    }
}

// Context
class VendingMachine {
    private VendingMachineState idleState;
    private VendingMachineState hasMoneyState;
    private VendingMachineState dispensingState;
    private VendingMachineState currentState;

    public VendingMachine() {
        idleState = new IdleState(this);
        hasMoneyState = new HasMoneyState(this);
        dispensingState = new DispensingState(this);
        currentState = idleState;
    }

    public void setState(VendingMachineState state) { this.currentState = state; }
    public VendingMachineState getIdleState() { return idleState; }
    public VendingMachineState getHasMoneyState() { return hasMoneyState; }
    public VendingMachineState getDispensingState() { return dispensingState; }

    public void insertCoin() { currentState.insertCoin(); }
    public void selectProduct() { currentState.selectProduct(); }
    public void dispense() { currentState.dispense(); }
}

// Client
public class Main {
    public static void main(String[] args) {
        VendingMachine machine = new VendingMachine();
        machine.selectProduct();   // Insert coin first
        machine.insertCoin();      // Coin inserted
        machine.insertCoin();      // Already have coin
        machine.selectProduct();   // Product selected
        machine.dispense();        // Dispensing product...
    }
}
```

## Advantages and Disadvantages

### Advantages
- **Single Responsibility Principle** – State-specific behavior is organized into separate classes, keeping each class focused.
- **Open/Closed Principle** – New states can be added without modifying existing state classes or the context.
- **Eliminates Conditionals** – Replaces complex `if/else` or `switch` statements with polymorphic dispatch.
- **State Transitions are Explicit** – Each state defines its valid transitions, making the state machine clear and self-documenting.

### Disadvantages
- **Class Explosion** – Each state becomes a separate class, increasing the total number of classes.
- **Context Complexity** – The context may need to expose its state objects, potentially breaking encapsulation.
- **State Object Management** – The pattern often creates a state object for each state; managing their lifecycle (singleton vs. per-transition) adds complexity.
- **Overkill for Simple States** – For a small number of states with simple transitions, the pattern adds unnecessary overhead.
