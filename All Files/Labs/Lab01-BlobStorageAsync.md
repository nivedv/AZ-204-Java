# Azure Blob Storage with Async Java APIs - Hands-on Workshop

This workshop guides you through building a Java application that uses the Azure Blob Storage asynchronous APIs. You'll learn how to perform common operations like creating containers, uploading, listing, and downloading blobs using reactive programming.

## Prerequisites

Before starting this workshop, ensure you have:

- [Visual Studio Code](https://code.visualstudio.com/) installed
- [Java Development Kit (JDK) 8 or later](https://www.oracle.com/java/technologies/javase-downloads.html)
- [Extension Pack for Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack) for VS Code
- An Azure account with an active subscription
- [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) (optional, but helpful)

## Step 1: Create an Azure Storage Account

1. Log in to the Azure portal (https://portal.azure.com/)
2. Click on "Create a resource"
3. Search for "Storage Account" and select it
4. Click "Create"
5. Fill in the following details:
   - **Subscription**: Select your subscription
   - **Resource Group**: Create a new one (e.g., "blob-storage-async-workshop")
   - **Storage account name**: Enter a unique name (e.g., "asyncblobstorage12345")
   - **Region**: Select the region closest to you
   - **Performance**: Standard
   - **Redundancy**: Locally-redundant storage (LRS)
6. Click "Review + create", then "Create"
7. Wait for deployment to complete, then click "Go to resource"

## Step 2: Get the Connection String

1. In your storage account, navigate to "Access keys" under "Security + networking"
2. Find "Connection string" and click "Show" to reveal it
3. Click the copy icon to copy the connection string
4. Save this connection string securely - you'll need it later

## Step 3: Create a Java Project in VS Code

1. Open VS Code
2. Press `Ctrl+Shift+P` (Windows/Linux) or `Cmd+Shift+P` (macOS) to open the Command Palette
3. Type "Java: Create Java Project" and select it
4. Select "Maven" as the build tool
5. Select "maven-archetype-quickstart" from the list of archetypes
6. For the Group Id, enter "com.example"
7. For the Artifact Id, enter "azure-blob-async-quickstart"
8. Select the latest version of the archetype
9. Choose a location to save your project
10. Wait for VS Code to create and set up the project

## Step 4: Configure Project Dependencies

1. Open the `pom.xml` file in your project
2. Replace the entire `<dependencies>` section with:

```xml
<dependencies>
    <!-- Azure Blob Storage -->
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-storage-blob</artifactId>
        <version>12.20.0</version>
    </dependency>
    <!-- Azure Identity for authentication -->
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-identity</artifactId>
        <version>1.7.1</version>
    </dependency>
    <!-- Project Reactor for async programming -->
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-core</artifactId>
        <version>3.5.0</version>
    </dependency>
    <!-- JUnit for testing -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

3. Add a build section after the dependencies to configure the Maven exec plugin:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <version>3.1.0</version>
            <configuration>
                <mainClass>com.example.AsyncBlobApp</mainClass>
            </configuration>
        </plugin>
    </plugins>
</build>
```

4. Save the file
5. VS Code will automatically update the Maven project configuration
6. You might see a notification in the bottom right asking to "Reload projects"; if so, click "Reload"

## Step 5: Write the Async Java Code

1. Navigate to `src/main/java/com/example/App.java`
2. Rename the file to `AsyncBlobApp.java` (right-click and select "Rename...")
3. Replace the entire content with the following async code:

```java
package com.example;

import com.azure.core.util.BinaryData;
import com.azure.storage.blob.BlobContainerAsyncClient;
import com.azure.storage.blob.BlobServiceAsyncClient;
import com.azure.storage.blob.BlobServiceClientBuilder;
import com.azure.storage.blob.models.BlobItem;
import com.azure.storage.blob.specialized.BlockBlobAsyncClient;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.time.Duration;
import java.util.Scanner;
import java.util.UUID;
import java.util.concurrent.CountDownLatch;

public class AsyncBlobApp {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Azure Blob Storage Async Quick Start");

        // Connection string from the Azure Portal
        String connectStr = System.getenv("AZURE_STORAGE_CONNECTION_STRING");
        
        if (connectStr == null || connectStr.isEmpty()) {
            System.out.println("Please set the AZURE_STORAGE_CONNECTION_STRING environment variable");
            return;
        }

        // Create a BlobServiceAsyncClient
        BlobServiceAsyncClient blobServiceAsyncClient = new BlobServiceClientBuilder()
            .connectionString(connectStr)
            .buildAsyncClient();

        // Create a unique name for the container
        String containerName = "asyncblobs" + UUID.randomUUID();

        // Create a local directory to hold blob data
        String localPath = "./data/";
        File localPathDir = new File(localPath);
        if (!localPathDir.exists()) {
            localPathDir.mkdirs();
        }

        // Create a local file in the ./data/ directory
        String fileName = "asyncblob" + UUID.randomUUID() + ".txt";
        String localFilePath = localPath + fileName;
        
        // Write text to the file
        try {
            Files.write(Paths.get(localFilePath), "Hello, Async World!".getBytes());
            System.out.println("Created local file: " + localFilePath);
        } catch (IOException e) {
            System.err.println("Error creating local file: " + e.getMessage());
            return;
        }

        // CountDownLatch to keep the application running until async operations complete
        CountDownLatch completionLatch = new CountDownLatch(1);

        // Create the container and then perform operations
        System.out.println("Creating container: " + containerName);
        
        blobServiceAsyncClient.createBlobContainer(containerName)
            .flatMap(containerResponse -> {
                System.out.println("Container created successfully: " + containerName);
                
                // Get a reference to the container
                BlobContainerAsyncClient containerAsyncClient = blobServiceAsyncClient.getBlobContainerAsyncClient(containerName);
                
                // Get a reference to a blob
                BlockBlobAsyncClient blobAsyncClient = containerAsyncClient.getBlobAsyncClient(fileName).getBlockBlobAsyncClient();
                
                System.out.println("\nUploading to Blob storage as blob:\n\t" + blobAsyncClient.getBlobUrl());
                
                // Upload the blob asynchronously from the file
                return Mono.fromCallable(() -> {
                    try {
                        return Files.readAllBytes(Paths.get(localFilePath));
                    } catch (IOException e) {
                        throw new RuntimeException("Failed to read file", e);
                    }
                }).flatMap(fileBytes -> 
                    blobAsyncClient.upload(BinaryData.fromBytes(fileBytes))
                ).thenReturn(containerAsyncClient);
            })
            .flatMap(containerAsyncClient -> {
                // List the blobs in the container
                System.out.println("\nListing blobs...");
                
                return containerAsyncClient.listBlobs()
                    .doOnNext(blobItem -> System.out.println("\t" + blobItem.getName()))
                    .collectList()
                    .thenReturn(containerAsyncClient);
            })
            .flatMap(containerAsyncClient -> {
                // Get a reference to the blob for download
                BlockBlobAsyncClient blobAsyncClient = containerAsyncClient.getBlobAsyncClient(fileName).getBlockBlobAsyncClient();
                
                // Download the blob to a local file
                String downloadFileName = fileName.replace(".txt", "DOWNLOAD.txt");
                String downloadFilePath = localPath + downloadFileName;
                
                System.out.println("\nDownloading blob to\n\t " + downloadFilePath);
                
                return blobAsyncClient.downloadContent()
                    .flatMap(response -> {
                        // Save the downloaded content to a file
                        return Mono.fromCallable(() -> {
                            try {
                                Files.write(Paths.get(downloadFilePath), response.toBytes());
                                return downloadFilePath;
                            } catch (IOException e) {
                                throw new RuntimeException("Failed to write to file", e);
                            }
                        });
                    })
                    .thenReturn(containerAsyncClient);
            })
            .doOnNext(containerAsyncClient -> {
                System.out.println("\nPress Enter to begin clean up");
                new Scanner(System.in).nextLine();
                
                System.out.println("Deleting blob container...");
            })
            .flatMap(containerAsyncClient -> containerAsyncClient.delete())
            .doOnNext(aVoid -> {
                System.out.println("Container deleted successfully");
                
                System.out.println("Deleting the local source and downloaded files...");
                try {
                    Files.deleteIfExists(Paths.get(localFilePath));
                    Files.deleteIfExists(Paths.get(localPath + fileName.replace(".txt", "DOWNLOAD.txt")));
                    
                    // Clean up local directory if empty
                    if (localPathDir.listFiles() != null && localPathDir.listFiles().length == 0) {
                        localPathDir.delete();
                    }
                } catch (IOException e) {
                    System.err.println("Error deleting local files: " + e.getMessage());
                }
                
                System.out.println("Done");
            })
            .doOnError(error -> {
                System.err.println("Error: " + error.getMessage());
                error.printStackTrace();
            })
            .doFinally(signalType -> completionLatch.countDown())
            .subscribe();

        // Wait for all operations to complete
        completionLatch.await();
    }
}
```

4. Save the file

## Step 6: Set Up the Connection String Environment Variable

### Windows PowerShell
```powershell
$env:AZURE_STORAGE_CONNECTION_STRING="your-connection-string-here"
```

### macOS/Linux Terminal
```bash
export AZURE_STORAGE_CONNECTION_STRING="your-connection-string-here"
```

Replace "your-connection-string-here" with the connection string you copied from the Azure portal.

## Step 7: Build and Run the Application

1. Open a terminal in VS Code (Terminal > New Terminal)
2. Build the project using Maven:
   ```bash
   mvn clean package
   ```
3. Run the application:
   ```bash
   mvn exec:java
   ```

## Step 8: Understand the Async Flow

Let's break down the key async elements in this code:

### 1. Reactive Stream Creation
The app creates a reactive stream starting with the container creation:
```java
blobServiceAsyncClient.createBlobContainer(containerName)
    .flatMap(containerResponse -> {
        // More operations here
    })
```

### 2. Chaining Operations with flatMap
Each operation is chained to the previous one using `flatMap`, which allows sequential execution:
```java
.flatMap(containerAsyncClient -> {
    // Next operation here
})
```

### 3. Converting Synchronous Operations to Asynchronous
When using sync APIs (like reading files), we wrap them in `Mono.fromCallable()`:
```java
Mono.fromCallable(() -> {
    try {
        return Files.readAllBytes(Paths.get(localFilePath));
    } catch (IOException e) {
        throw new RuntimeException("Failed to read file", e);
    }
})
```

### 4. Handling Events
Using handlers like `doOnNext`, `doOnError`, and `doFinally`:
```java
.doOnNext(aVoid -> {
    // Handle successful completion
})
.doOnError(error -> {
    // Handle errors
})
.doFinally(signalType -> {
    // Clean up resources
})
```

### 5. Subscribing to Start the Stream
Nothing happens until you subscribe:
```java
.subscribe();
```

### 6. Controlling Application Lifetime
Using a `CountDownLatch` to prevent the application from terminating before async operations complete:
```java
CountDownLatch completionLatch = new CountDownLatch(1);
// In doFinally:
.doFinally(signalType -> completionLatch.countDown())
// Wait for completion:
completionLatch.await();
```

## Step 9: Observe the Results

Watch the console output as the application:
1. Creates a container in your Azure Storage account
2. Creates a local file with "Hello, Async World!" text
3. Uploads the file to the container
4. Lists all blobs in the container
5. Downloads the blob to a local file with "DOWNLOAD" appended to the name
6. Waits for you to press Enter
7. Cleans up by deleting the container and local files

## Step 10: Verify in Azure Portal

1. Return to the Azure portal
2. Go to your storage account
3. Click on "Containers" under "Data storage"
4. You should see that the container has been deleted (if you pressed Enter in the application)
   - If you haven't completed the application run, you'll see the container created by your application

## Common Issues and Troubleshooting

### Connection String Issues
- Make sure the environment variable is set correctly
- Check for typos in the connection string
- Ensure you have the right permissions on your storage account

### Reactive Programming Issues
- If you're getting "Nothing was subscribed" errors, make sure you're calling `.subscribe()`
- If operations aren't executing, check that you're properly chaining with `flatMap` or `map`
- For debugging, add more `.doOnNext(item -> System.out.println("Debug: " + item))` calls

### CountDownLatch Issues
- If the application terminates too early, make sure the `countDown()` is called in the `doFinally` block
- If the application seems to hang, ensure the `countDown()` gets called even in error scenarios

## Key Differences from Synchronous APIs

1. **Client Classes**
   - Synchronous: `BlobServiceClient`, `BlobContainerClient`, `BlobClient`
   - Asynchronous: `BlobServiceAsyncClient`, `BlobContainerAsyncClient`, `BlockBlobAsyncClient`

2. **Return Types**
   - Synchronous: Direct return values (`void`, `String`, etc.)
   - Asynchronous: Wrapped in reactive types (`Mono<T>`, `Flux<T>`)

3. **Execution Flow**
   - Synchronous: Sequential, blocking execution
   - Asynchronous: Non-blocking, operations chain with operators like `flatMap`

4. **Error Handling**
   - Synchronous: Try/catch blocks
   - Asynchronous: `.doOnError()` and error propagation through the chain

5. **Resources**
   - Synchronous: Resources can block threads
   - Asynchronous: Better resource utilization, especially for I/O-bound operations

## Benefits of Async APIs

1. **Improved Throughput**: Process more requests without blocking threads
2. **Better Resource Utilization**: CPU doesn't wait idle during I/O operations
3. **Scalability**: Handle more concurrent operations with fewer threads
4. **Responsiveness**: Non-blocking operations keep your application responsive
5. **Composition**: Complex operations can be composed from simpler ones

## Next Steps

1. Try modifying the code to:
   - Upload multiple blobs in parallel using `Flux.merge()`
   - Implement retry logic using `.retry()` operators
   - Add timeout handling with `.timeout()` 
   - Use backpressure strategies for large files

2. Explore other async operations:
   - Setting and retrieving metadata
   - Using blob leases
   - Implementing conditional operations
   - Working with blob snapshots

3. Advanced topics:
   - Integration with Spring WebFlux for reactive web applications
   - Using Azure Event Grid with reactive programming
   - Implementing a reactive API with Azure Functions

## Additional Resources

- [Azure Blob Storage Documentation](https://docs.microsoft.com/en-us/azure/storage/blobs/)
- [Java SDK for Azure Blob Storage](https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/storage/azure-storage-blob)
- [Project Reactor Reference Guide](https://projectreactor.io/docs/core/release/reference/)
- [Reactive Streams Specification](https://www.reactive-streams.org/)
- [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) - a useful tool for managing your storage visually
- [VS Code Java Tutorial](https://code.visualstudio.com/docs/java/java-tutorial) - Learn more about Java development in VS Code