# Adapter Pattern

## Introduction

The Adapter pattern allows objects with incompatible interfaces to collaborate. It acts as a wrapper that converts the interface of a class into another interface the client expects, enabling classes to work together that could not otherwise due to incompatible interfaces.

## Real-World Applications

- **Power adapters** – A physical power adapter lets a device with a US plug work in a European socket.
- **Legacy system integration** – A new system communicates with an old legacy system through an adapter that translates modern API calls into legacy protocol calls.
- **Third-party library integration** – A wrapper adapts a third-party library's interface to match the interface expected by the application.
- **Data format converters** – An adapter converts data from XML to JSON or CSV so that different components can process it.

## Components

| Component | Description |
|-----------|-------------|
| **Target** | Defines the domain-specific interface that the client uses. |
| **Adapter** | Adapts the interface of the `Adaptee` to the `Target` interface. |
| **Adaptee** | Defines an existing interface that needs adapting. |
| **Client** | Collaborates with objects conforming to the `Target` interface. |

## Code Example

### Problem

You have an existing `JSONAnalyzer` class that processes data only in JSON format. A new business requirement demands analyzing XML data as well. The XML library provides a perfectly good XML parser, but its interface (`parseXML()`) is incompatible with the JSON analyzer's expected interface (`processJSON()`). Modifying the existing `JSONAnalyzer` or the XML library is not feasible.

### Solution

Create an `XMLAdapter` that implements the `Target` interface (`processJSON()`) and internally delegates to the XML library's `parseXML()` method. The client (`JSONAnalyzer`) works with the adapter as if it were a JSON processor.

```java
// Target interface (expected by the client)
interface DataProcessor {
    void processData(String data);
}

// Adaptee (incompatible interface)
class XMLParser {
    public void parseXML(String data) {
        System.out.println("Parsing XML data: " + data);
    }
}

// Adapter
class XMLAdapter implements DataProcessor {
    private XMLParser xmlParser;

    public XMLAdapter(XMLParser xmlParser) {
        this.xmlParser = xmlParser;
    }

    public void processData(String data) {
        // Convert JSON-style call to XML parsing
        xmlParser.parseXML(data);
    }
}

// Client
class JSONAnalyzer {
    public void analyze(DataProcessor processor, String data) {
        System.out.println("Analyzing data...");
        processor.processData(data);
    }
}

public class Main {
    public static void main(String[] args) {
        JSONAnalyzer analyzer = new JSONAnalyzer();
        XMLParser xmlParser = new XMLParser();
        DataProcessor adapter = new XMLAdapter(xmlParser);

        analyzer.analyze(adapter, "<data><item>example</item></data>");
    }
}
```

## Advantages and Disadvantages

### Advantages
- **Single Responsibility Principle** – The adapter handles the interface conversion, keeping the business logic clean.
- **Open/Closed Principle** – New adapters can be introduced without modifying existing client code.
- **Reusability** – Existing classes (Adaptees) can be reused even if their interfaces do not match the system's needs.
- **Decoupling** – The client and the adaptee are completely decoupled.

### Disadvantages
- **Complexity** – Adds an extra layer of indirection, which can make the system harder to understand.
- **Performance Overhead** – Every call passes through the adapter, introducing a small runtime cost.
- **Over-Engineering** – If the interface mismatch is minor, a simple refactor of the adaptee may be more appropriate.
