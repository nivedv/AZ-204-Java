# Azure Blob Storage Java Examples

This repository contains example projects demonstrating how to use Azure Blob Storage with Java. Both synchronous and asynchronous approaches are included to help you understand the different programming models and choose the one that best fits your needs.

## 📋 Examples Included

- **Synchronous API Example**: Traditional approach using the Azure SDK for Java
- **Asynchronous API Example**: Modern reactive approach using Project Reactor with the Azure SDK

## 🚀 Getting Started

### Prerequisites

- Java Development Kit (JDK) 8 or later
- Maven
- An Azure account with an active subscription
- An Azure Storage Account
- Visual Studio Code (recommended) with Java extensions

### Setting Up

1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/azure-blob-java-examples.git
   cd azure-blob-java-examples
   ```

2. Set your Azure Storage connection string as an environment variable:

   **Windows (PowerShell)**:
   ```powershell
   $env:AZURE_STORAGE_CONNECTION_STRING="your-connection-string-here"
   ```

   **macOS/Linux**:
   ```bash
   export AZURE_STORAGE_CONNECTION_STRING="your-connection-string-here"
   ```

3. Navigate to either the sync or async example directory:
   ```bash
   cd sync-example
   # OR
   cd async-example
   ```

4. Build the project:
   ```bash
   mvn clean package
   ```

5. Run the example:
   ```bash
   mvn exec:java
   ```

## 📚 Project Structure

### Synchronous Example

```
sync-example/
├── pom.xml
└── src/
    └── main/
        └── java/
            └── com/
                └── example/
                    └── App.java
```

### Asynchronous Example

```
async-example/
├── pom.xml
└── src/
    └── main/
        └── java/
            └── com/
                └── example/
                    └── AsyncBlobApp.java
```

## 📝 What You'll Learn

### Synchronous API

- Creating storage containers
- Uploading blobs to Azure
- Listing blobs in a container
- Downloading blobs
- Deleting containers and blobs
- Basic error handling

### Asynchronous API

- Working with reactive streams in Azure
- Non-blocking I/O operations
- Chaining asynchronous operations
- Error handling in reactive streams
- Managing application flow with reactive programming
- Performance benefits of asynchronous programming

## 🔄 Key Differences Between Sync and Async APIs

| Feature | Synchronous | Asynchronous |
|---------|-------------|--------------|
| Programming Model | Sequential, blocking | Reactive, non-blocking |
| Client Classes | `BlobServiceClient` | `BlobServiceAsyncClient` |
| Return Types | Direct values | `Mono<T>` / `Flux<T>` |
| Error Handling | Try/catch blocks | `doOnError()` handlers |
| Resource Utilization | Potentially blocking threads | Better thread utilization |
| Learning Curve | Simpler to understand | Steeper learning curve |

## 💡 Common Operations

Both examples demonstrate these core operations:

1. **Creating a container**
2. **Uploading a blob**
3. **Listing blobs in a container**
4. **Downloading a blob**
5. **Cleaning up resources**

## 📖 Additional Resources

- [Azure Blob Storage Documentation](https://docs.microsoft.com/en-us/azure/storage/blobs/)
- [Java SDK for Azure Blob Storage](https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/storage/azure-storage-blob)
- [Project Reactor](https://projectreactor.io/) (for async example)
- [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/)

## ⚠️ Important Notes

- The examples use environment variables for security. Never hard-code your connection strings.
- For production use, consider using Azure Key Vault for storing secrets.
- In production code, implement more robust error handling and logging.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.
