# Observer Pattern

## Introduction

The Observer pattern defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically. It is also known as publish-subscribe or event subscription.

## Real-World Applications

- **Event handling in GUIs** – A button's click event notifies registered listeners (observers) when clicked.
- **Stock market tickers** – A stock exchange (subject) publishes price updates; multiple trading applications (observers) react to the changes.
- **Newsletters / RSS feeds** – A publisher sends new content to all subscribers; each subscriber receives updates automatically.
- **Distributed cache invalidation** – A primary database notifies cache nodes when data changes so they can invalidate stale entries.
- **Model-View-Controller (MVC)** – The model notifies views when data changes, and each view re-renders accordingly.

## Components

| Component | Description |
|-----------|-------------|
| **Subject** | Knows its observers; provides an interface for attaching and detaching observer objects. |
| **Observer** | Defines an updating interface for objects that should be notified of changes in a subject. |
| **ConcreteSubject** | Stores the state of interest to `ConcreteObserver` objects; sends notification to its observers when its state changes. |
| **ConcreteObserver** | Maintains a reference to a `ConcreteSubject` object; implements the `Observer` updating interface to keep its state consistent with the subject's. |

## Code Example

### Problem

You are building a weather station that collects temperature, humidity, and pressure readings. Multiple display devices (current conditions, statistics, forecast) must show updated data whenever the measurements change. If you hardcode the display updates in the weather station, adding a new display requires modifying the weather station class.

### Solution

The Observer pattern makes the `WeatherStation` the subject and the displays the observers. The weather station maintains a list of observers and notifies them all when readings change. New display types can be added by implementing the `Observer` interface without touching the weather station.

```java
// Observer
interface Observer {
    void update(float temperature, float humidity, float pressure);
}

// Subject
interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

// ConcreteSubject
class WeatherStation implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private float temperature, humidity, pressure;

    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        notifyObservers();
    }

    public void attach(Observer observer) { observers.add(observer); }
    public void detach(Observer observer) { observers.remove(observer); }

    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(temperature, humidity, pressure);
        }
    }
}

// ConcreteObserver
class CurrentConditionsDisplay implements Observer {
    public void update(float temperature, float humidity, float pressure) {
        System.out.println("Current: " + temperature + "°C, " + humidity + "% humidity");
    }
}

class ForecastDisplay implements Observer {
    public void update(float temperature, float humidity, float pressure) {
        if (pressure > 1013) {
            System.out.println("Forecast: Improving weather");
        } else {
            System.out.println("Forecast: Rain expected");
        }
    }
}

// Client
public class Main {
    public static void main(String[] args) {
        WeatherStation station = new WeatherStation();

        station.attach(new CurrentConditionsDisplay());
        station.attach(new ForecastDisplay());

        station.setMeasurements(25.2f, 65f, 1012f);
        station.setMeasurements(22.8f, 70f, 1008f);
    }
}
```

## Advantages and Disadvantages

### Advantages
- **Loose Coupling** – The subject knows only that observers implement a specific interface; it does not know their concrete classes.
- **Broadcast Communication** – A single state change can notify multiple observers automatically.
- **Open/Closed Principle** – New observers can be added without modifying the subject.
- **Dynamic Relationships** – Observers can be attached or detached at runtime.

### Disadvantages
- **Unexpected Updates** – Observers are notified in no specific order, which can cause unexpected side effects.
- **Memory Leaks** – If observers are not properly detached, the subject retains references that prevent garbage collection.
- **Notification Overhead** – Every state change triggers notifications to all observers, even if only a subset needs to react.
- **Cascading Updates** – Updates can trigger further updates in other subjects, leading to complex chains of notifications.
- **Stale Data** – An observer may receive a notification but query the subject after another change has already occurred.
