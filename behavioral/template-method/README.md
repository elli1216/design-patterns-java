# Template Method Pattern

## Introduction

The Template Method pattern defines the skeleton of an algorithm in a method, deferring some steps to subclasses. It lets subclasses redefine certain steps of an algorithm without changing the algorithm's structure.

## Real-World Applications

- **Framework initialization** – A web framework defines the startup process (load config → initialize DB → register routes → start server) as a template method; subclasses customize each step.
- **Data mining** – A `DataMiner` template defines the algorithm: open file → extract data → parse data → analyze → send report. Different file formats (CSV, PDF, XML) override the relevant steps.
- **Build processes (CI/CD)** – A build pipeline defines stages (checkout → compile → test → package → deploy) as a template; each project customizes the compile and test steps.
- **Game AI** – A template method defines the turn structure (collect resources → build units → move → attack). Different AI personalities override specific steps.
- **Unit testing frameworks (JUnit)** – The test lifecycle (setup → test → teardown) is a template; subclasses override `setUp()`, the test method, and `tearDown()`.

## Components

| Component | Description |
|-----------|-------------|
| **AbstractClass** | Defines abstract primitive operations that concrete subclasses define to implement steps of an algorithm; implements a template method that defines the skeleton of an algorithm. |
| **ConcreteClass** | Implements the primitive operations to carry out subclass-specific steps of the algorithm. |

## Code Example

### Problem

You are building a data mining tool that extracts information from different file formats (CSV, PDF, DOCX). The overall pipeline is the same: open the file, extract raw data, parse the data, analyze it, and generate a report. However, the extraction and parsing steps differ for each format. Copying the pipeline logic for each format leads to code duplication.

### Solution

The Template Method pattern places the invariant pipeline in a base class's `mineData()` method. The varying steps (`extractData()`, `parseData()`) are declared as abstract or hook methods that subclasses override.

```java
// AbstractClass
abstract class DataMiner {
    // Template method
    public final void mineData(String filePath) {
        openFile(filePath);
        String rawData = extractData(filePath);
        String parsedData = parseData(rawData);
        String analysis = analyze(parsedData);
        sendReport(analysis);
        closeFile();
    }

    private void openFile(String path) {
        System.out.println("Opening file: " + path);
    }

    protected abstract String extractData(String path);
    protected abstract String parseData(String data);

    private String analyze(String data) {
        System.out.println("Analyzing data...");
        return "Analysis result for: " + data;
    }

    private void sendReport(String report) {
        System.out.println("Sending report: " + report);
    }

    private void closeFile() {
        System.out.println("Closing file");
    }
}

// ConcreteClass
class CSVDataMiner extends DataMiner {
    protected String extractData(String path) {
        System.out.println("Extracting CSV data");
        return "raw,csv,data";
    }

    protected String parseData(String data) {
        System.out.println("Parsing CSV data");
        return data.replace(",", " | ");
    }
}

class PDFDataMiner extends DataMiner {
    protected String extractData(String path) {
        System.out.println("Extracting PDF data");
        return "raw pdf data";
    }

    protected String parseData(String data) {
        System.out.println("Parsing PDF data");
        return "Parsed: " + data;
    }
}

// Client
public class Main {
    public static void main(String[] args) {
        DataMiner csvMiner = new CSVDataMiner();
        csvMiner.mineData("data.csv");

        System.out.println();

        DataMiner pdfMiner = new PDFDataMiner();
        pdfMiner.mineData("report.pdf");
    }
}
```

## Advantages and Disadvantages

### Advantages
- **Code Reuse** – The invariant parts of an algorithm are written once in the base class, avoiding duplication.
- **Controlled Extension** – Subclasses override only specific steps, preventing them from altering the algorithm's core structure.
- **Inversion of Control** – The base class calls subclass methods (the Hollywood Principle: "Don't call us, we'll call you").
- **Consistent Structure** – All subclasses follow the same algorithm skeleton, ensuring consistency.

### Disadvantages
- **Limited Flexibility** – The template method is often declared `final`, preventing subclasses from overriding the overall algorithm structure.
- **Violation of Liskov Substitution** – If a subclass overrides a non-abstract hook method in a way that changes the algorithm's behavior, it may violate the Liskov Substitution Principle.
- **Base Class Bloat** – As more steps are added, the base class can become large and harder to maintain.
- **Debugging Difficulty** – The algorithm's execution flow crosses multiple classes, making debugging more complex.
