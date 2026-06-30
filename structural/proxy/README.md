# Proxy Pattern

## Introduction

The Proxy pattern provides a surrogate or placeholder for another object to control access to it. The proxy has the same interface as the real subject and can add indirection for purposes such as lazy loading, access control, logging, or caching.

## Real-World Applications

- **Virtual proxies (lazy loading)** – A high-resolution image proxy loads the actual image only when it is first displayed, saving memory and startup time.
- **Protection proxies (access control)** – A proxy checks user permissions before delegating to the real object, preventing unauthorized access.
- **Remote proxies (RPC)** – A proxy object on the client side represents a real object on a remote server; method calls are serialized and transmitted over the network.
- **Caching proxies** – A proxy caches results from an expensive operation and returns cached data for repeated identical requests.
- **Logging proxies** – A proxy logs every method call (including parameters and execution time) before delegating to the real object.

## Components

| Component | Description |
|-----------|-------------|
| **Subject** | Defines the common interface for the `RealSubject` and the `Proxy` so the proxy can be used anywhere the real subject is expected. |
| **RealSubject** | The real object that the proxy represents. |
| **Proxy** | Maintains a reference to the `RealSubject` and controls access to it; implements the same interface as the `Subject`. |

## Code Example

### Problem

You are building a document viewer that displays large images. Loading every image when the document opens consumes too much memory and slows down startup. You need a way to defer image loading until the image is actually visible on screen, without changing the code that renders images.

### Solution

A `ImageProxy` implements the same interface as the real `HighResImage`. The proxy does not load the image until `display()` is called. The client code remains unchanged, working with the `Image` interface throughout.

```java
// Subject
interface Image {
    void display();
    long getSize();
}

// RealSubject
class HighResImage implements Image {
    private String filename;
    private long size;

    public HighResImage(String filename) {
        this.filename = filename;
        this.size = loadFromDisk();
    }

    private long loadFromDisk() {
        System.out.println("Loading " + filename + " from disk...");
        return 10_000_000; // Simulated size in bytes
    }

    public void display() {
        System.out.println("Displaying " + filename);
    }

    public long getSize() { return size; }
}

// Proxy
class ImageProxy implements Image {
    private String filename;
    private HighResImage realImage;
    private long cachedSize;

    public ImageProxy(String filename) {
        this.filename = filename;
        this.cachedSize = -1;
    }

    public void display() {
        if (realImage == null) {
            realImage = new HighResImage(filename);
        }
        realImage.display();
    }

    public long getSize() {
        if (realImage != null) {
            return realImage.getSize();
        }
        if (cachedSize == -1) {
            // Read file size metadata without loading the image
            cachedSize = queryFileSize(filename);
        }
        return cachedSize;
    }

    private long queryFileSize(String filename) {
        System.out.println("Querying size metadata for " + filename);
        return 10_000_000;
    }
}

// Client
public class Main {
    public static void main(String[] args) {
        Image image = new ImageProxy("sunset.png");
        // Image is not loaded yet
        System.out.println("Size: " + image.getSize() + " bytes");
        // Image is loaded only now
        image.display();
    }
}
```

## Advantages and Disadvantages

### Advantages
- **Lazy Loading** – Virtual proxies defer expensive object creation until it is actually needed.
- **Access Control** – Protection proxies can enforce permissions without modifying the real subject.
- **Logging & Auditing** – Proxies transparently add logging, monitoring, or instrumentation around method calls.
- **Caching** – Proxies can cache results and improve performance for repeated requests.
- **Remote Access** – Remote proxies handle network communication, making remote objects feel local.

### Disadvantages
- **Increased Complexity** – Adds an extra layer of indirection that can make the system harder to debug.
- **Performance Overhead** – Each method call goes through the proxy, adding indirection and potential latency.
- **Proxy and Subject Inconsistency** – The proxy must stay in sync with the real subject's interface; any change requires updating both.
- **Delayed Errors** – Lazy loading defers errors (e.g., missing file, no permissions) to the first time the object is actually used, which may be far from the initialization code.
