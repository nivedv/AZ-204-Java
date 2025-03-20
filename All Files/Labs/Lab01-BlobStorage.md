# Azure Blob Storage with Java - Hands-on Workshop

## Prerequisites

Before starting this workshop, ensure you have:

- [Visual Studio Code](https://code.visualstudio.com/) installed
- [Java Development Kit (JDK) 8 or later](https://www.oracle.com/java/technologies/javase-downloads.html)
- [Maven](https://maven.apache.org/download.cgi) installed
- An Azure account with an active subscription
- Azure CLI installed

## Step 1: Create an Azure Storage Account

1. Log in to the Azure portal (https://portal.azure.com/)
2. Click on "Create a resource"
3. Search for "Storage Account" and select it
4. Click "Create"
5. Fill in the following details:
   - **Subscription**: Select your subscription
   - **Resource Group**: Use your exisitng Resource Group 
   - **Storage account name**: Enter a unique name 
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

## Step 3: Set Up Your Java Project in VS Code

1. Open VS Code
2. Create a new folder for your project: `azure-blob-java-quickstart`
3. Open the folder in VS Code
4. Create a new file called `pom.xml` in the root of your project
5. Paste the following content into `pom.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>azure-blob-quickstart</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.azure</groupId>
            <artifactId>azure-storage-blob</artifactId>
            <version>12.20.0</version>
        </dependency>
        <dependency>
            <groupId>com.azure</groupId>
            <artifactId>azure-identity</artifactId>
            <version>1.7.1</version>
        </dependency>
    </dependencies>
    <build>
  <plugins>
    <plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>exec-maven-plugin</artifactId>
      <version>3.1.0</version>
      <configuration>
        <mainClass>com.example.AzureBlobApp</mainClass>
      </configuration>
    </plugin>
  </plugins>
</build>
</project>
```

6. Create the directory structure for your Java application:
   - Create a `src/main/java/com/example` folder hierarchy
   - Inside the `example` folder, create a new file named `AzureBlobApp.java`

## Step 4: Write the Java Code

1. Open `AzureBlobApp.java` and paste the following code:

```java
package com.example;

import com.azure.storage.blob.*;
import com.azure.storage.blob.models.*;
import java.io.*;
import java.util.*;

public class AzureBlobApp {
    public static void main(String[] args) throws IOException {
        System.out.println("Azure Blob Storage Quick Start");

        // Connection string from the Azure Portal
        String connectStr = System.getenv("AZURE_STORAGE_CONNECTION_STRING");
        
        if (connectStr == null || connectStr.isEmpty()) {
            System.out.println("Please set the AZURE_STORAGE_CONNECTION_STRING environment variable");
            return;
        }

        // Create a BlobServiceClient
        BlobServiceClient blobServiceClient = new BlobServiceClientBuilder()
            .connectionString(connectStr)
            .buildClient();

        // Create a unique name for the container
        String containerName = "quickstartblobs" + UUID.randomUUID();

        // Create the container
        BlobContainerClient containerClient = blobServiceClient.createBlobContainer(containerName);
        System.out.println("Container created: " + containerName);

        // Create a local directory to hold blob data
        String localPath = "./data/";
        File localPathDir = new File(localPath);
        if (!localPathDir.exists()) {
            localPathDir.mkdirs();
        }

        // Create a local file in the ./data/ directory
        String fileName = "quickstart" + UUID.randomUUID() + ".txt";
        File localFile = new File(localPath + fileName);

        // Write text to the file
        FileWriter writer = new FileWriter(localFile, true);
        writer.write("Hello, World!");
        writer.close();

        // Get a reference to a blob
        BlobClient blobClient = containerClient.getBlobClient(fileName);

        System.out.println("\nUploading to Blob storage as blob:\n\t" + blobClient.getBlobUrl());

        // Upload the blob
        blobClient.uploadFromFile(localPath + fileName);

        // List the blobs in the container
        System.out.println("\nListing blobs...");
        for (BlobItem blobItem : containerClient.listBlobs()) {
            System.out.println("\t" + blobItem.getName());
        }

        // Download the blob to a local file
        // Append the string "DOWNLOAD" before the .txt extension
        String downloadFileName = fileName.replace(".txt", "DOWNLOAD.txt");
        
        System.out.println("\nDownloading blob to\n\t " + localPath + downloadFileName);
        
        blobClient.downloadToFile(localPath + downloadFileName);

        System.out.println("\nPress the Enter key to begin clean up");
        System.console().readLine();

        System.out.println("Deleting blob container...");
        containerClient.delete();

        System.out.println("Deleting the local source and downloaded files...");
        localFile.delete();
        new File(localPath + downloadFileName).delete();
        
        // Clean up local directory
        if (localPathDir.listFiles().length == 0) {
            localPathDir.delete();
        }

        System.out.println("Done");
    }
}
```

2. Save the file

## Step 5: Set Up the Connection String Environment Variable

### Windows PowerShell
```powershell
# Replace the "your-connection-string-here" with the connection string you copied earlier in Step2
$env:AZURE_STORAGE_CONNECTION_STRING="your-connection-string-here"
```

### macOS/Linux Terminal
```bash
# Replace the "your-connection-string-here" with the connection string you copied earlier in Step2
export AZURE_STORAGE_CONNECTION_STRING="your-connection-string-here"
```

Replace "your-connection-string-here" with the connection string you copied from the Azure portal.

## Step 6: Build and Run the Application

1. Open a terminal in VS Code (Terminal > New Terminal)
2. Build the project using Maven:
   ```bash
   mvn clean package
   ```
3. Run the application:
   ```bash
   mvn exec:java
   ```

## Step 7: Observe the Results

Watch the console output as the application:
1. Creates a container in your Azure Storage account
2. Creates a local file with "Hello, World!" text
3. Uploads the file to the container
4. Lists all blobs in the container
5. Downloads the blob to a local file with "DOWNLOAD" appended to the name
6. Waits for you to press Enter
7. Cleans up by deleting the container and local files

## Step 8: Verify in Azure Portal

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

### Build Issues
- Make sure Maven is installed correctly
- Verify that your pom.xml has no errors
- Check that your Java version matches the one specified in pom.xml

### Runtime Issues
- If you get a NullPointerException when the program waits for input, try running from a regular command prompt rather than the integrated VS Code terminal, or modify the code to use Scanner for input instead of System.console()

## Next Steps

Now that you've completed the basic operations with Azure Blob Storage, consider:

1. Modifying the code to handle different file types (images, PDFs, etc.)
2. Adding error handling for more robust applications
3. Implementing authentication with Azure Active Directory instead of connection strings
4. Setting metadata and properties on blobs
5. Implementing blob leases for concurrency control
6. Using the async APIs for better performance in real applications

## Additional Resources

- [Azure Blob Storage Documentation](https://docs.microsoft.com/en-us/azure/storage/blobs/)
- [Java SDK for Azure Blob Storage](https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/storage/azure-storage-blob)
- [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) - a useful tool for managing your storage visually