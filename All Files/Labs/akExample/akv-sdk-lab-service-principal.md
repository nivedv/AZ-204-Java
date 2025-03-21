# Azure Key Vault Java SDK Lab: Hands-On Guide with VS Code
# Using Application Registration (Service Principal) Authentication with Azure Key Vault

This step-by-step guide will walk you through setting up a Java project in Visual Studio Code to interact with Azure Key Vault using the Azure SDK for Java. You'll learn how to create, retrieve, update, and delete keys in Azure Key Vault.

## Prerequisites

- [Visual Studio Code](https://code.visualstudio.com/download)
- [JDK 11 or later](https://adoptopenjdk.net/)
- [Maven](https://maven.apache.org/download.cgi)
- [VS Code Java Extension Pack](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
- An Azure account with an active subscription

## Lab Setup

This guide explains how to authenticate to Azure Key Vault using an application registration (service principal) in your Java application instead of using interactive authentication methods.

## Why Use Service Principal Authentication?

- **Non-interactive scenarios**: Ideal for automated processes, CI/CD pipelines, and background services
- **Limited permissions**: You can restrict the service principal to only the permissions it needs
- **Better for production**: More suitable for production environments than user-based authentication

### Step 1: Sign in to Azure

Open a terminal and run:

```bash
az login
```

Follow the prompts to authenticate with your Azure account.

### Step 2: Create an Azure Key Vault

1. Create a resource group:

```bash
az group create --name keyvault-lab-rg --location eastus
```

2. Create a Key Vault:

```bash
az keyvault create --name kv-java-lab-[your-unique-suffix] --resource-group keyvault-lab-rg --location eastus
```

Note: Replace `[your-unique-suffix]` with a unique string to ensure a globally unique vault name.

## Step 3: Create an App Registration in Azure

1. Sign in to the [Azure Portal](https://portal.azure.com)
2. Navigate to **Microsoft Entra ID** (previously Azure Active Directory)
3. Select **App registrations** from the left menu
4. Click **+ New registration**
5. Enter the following details:
   - **Name**: `KeyVaultJavaClient` (or your preferred name)
   - **Supported account types**: Choose "Accounts in this organizational directory only"
   - **Redirect URI**: Leave blank (not needed for this scenario)
6. Click **Register**

## Step 4: Create a Client Secret

1. In your newly created app registration, go to **Certificates & secrets** in the left menu
2. Under **Client secrets**, click **+ New client secret**
3. Enter a description like "KeyVault Access"
4. Select an expiration period (e.g., 1 year, 2 years)
5. Click **Add**
6. **IMPORTANT**: Copy the **Value** of the secret immediately and save it securely. You won't be able to view it again after leaving this page.

## Step 5: Grant Key Vault Access to Your Service Principal

1. Go to your Key Vault resource in the Azure Portal
2. Select **Access policies** from the left menu
3. Click **+ Add Access Policy**
4. Configure the following:
   - **Key permissions**: Select appropriate permissions (e.g., Get, List, Create, Delete, Update)
   - **Secret permissions**: Select as needed if you'll be working with secrets
   - **Certificate permissions**: Select as needed if you'll be working with certificates
5. Under **Select principal**, click **None selected**
6. Search for the name of your app registration (e.g., "KeyVaultJavaClient")
7. Select it and click **Select**
8. Click **Add** to add the access policy
9. Don't forget to click **Save** at the top to apply the changes

## Step 6: Note Your Application (Client) ID and Tenant ID

1. Go back to your app registration in Microsoft Entra ID
2. On the Overview page, copy the following values:
   - **Application (client) ID**
   - **Directory (tenant) ID**

### Step 7: Set Up a Java Project in VS Code

1. Open VS Code

2. Open the Command Palette (Ctrl+Shift+P or Cmd+Shift+P on macOS) and run:
   ```
   Maven: Create Maven Project
   ```

3. Select "maven-archetype-quickstart" archetype

4. Choose the latest version (typically at the top of the list)

5. Enter the following details when prompted:
   - Group ID: `com.keyvault.demo`
   - Artifact ID: `keyvault-demo`
   - Version: Press Enter to accept the default (1.0-SNAPSHOT)
   - Package: Press Enter to accept the default (same as Group ID)

6. Choose a directory where you want to create the project

7. VS Code will generate the Maven project and open it

### Step 8: Configure Project Dependencies

1. Open the `pom.xml` file

2. Replace the contents with the following XML:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.keyvault.demo</groupId>
  <artifactId>keyvault-demo</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>keyvault-demo</name>
  <url>http://maven.apache.org</url>
  
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
  </properties>
  
  <dependencies>
    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-security-keyvault-keys</artifactId>
      <version>4.5.1</version>
    </dependency>
    <dependency>
      <groupId>com.azure</groupId>
      <artifactId>azure-identity</artifactId>
      <version>1.7.0</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-simple</artifactId>
      <version>1.7.36</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.10.1</version>
        <configuration>
          <source>11</source>
          <target>11</target>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>3.1.0</version>
        <configuration>
          <mainClass>com.keyvault.demo.App</mainClass>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

3. Save the file and wait for Maven to download the dependencies

## Lab Exercises

### Exercise 1: Create a Key Vault Client

1. Create a new Java class file:
   - Navigate to src/main/java/com/keyvault/demo
   - Create a new file called `KeyVaultOperations.java`

2. Add the following code to the file:

```java
package com.keyvault.demo;

import com.azure.identity.ClientSecretCredentialBuilder;
import com.azure.security.keyvault.keys.KeyClient;
import com.azure.security.keyvault.keys.KeyClientBuilder;
import com.azure.security.keyvault.keys.models.KeyVaultKey;
import com.azure.security.keyvault.keys.models.KeyType;

public class KeyVaultOperations {
    
    private static KeyClient keyClient;
    
    public static void main(String[] args) {
        // Replace with your Key Vault URL (e.g., https://kv-java-lab-12345.vault.azure.net/)
        String keyVaultUrl = "https://kv-java-lab-[your-unique-suffix].vault.azure.net/";
        
        System.out.println("Connecting to Key Vault: " + keyVaultUrl);
        
         // Service Principal credentials from App Registration 
        // you have obtained from Azure Portal
        String clientId = "YOUR-APPLICATION-CLIENT-ID";
        String clientSecret = "YOUR-CLIENT-SECRET";
        String tenantId = "YOUR-TENANT-ID";
        // Create a KeyClient using ClientSecretCredential
        keyClient = new KeyClientBuilder()
            .vaultUrl(keyVaultUrl)
            .credential(new ClientSecretCredentialBuilder()
                .clientId(clientId)
                .clientSecret(clientSecret)
                .tenantId(tenantId)
                .build())
            .buildClient();
            
        System.out.println("Successfully connected to Key Vault");
        
        // Test the connection by listing keys
        System.out.println("\nListing keys to verify connection:");
        keyClient.listPropertiesOfKeys().forEach(key -> {
            System.out.println("Key name: " + key.getName());
        });
            
    }
}
```

3. Update the `App.java` file to call your new class:

```java
package com.keyvault.demo;

public class App {
    public static void main(String[] args) {
        KeyVaultOperations.main(args);
    }
}
```

4. Replace `[your-unique-suffix]` in the `keyVaultUrl` with the suffix you used when creating your Key Vault.

5. Run the application:
   - Open a terminal in VS Code (Terminal > New Terminal)
   - Run the following command:
   ```bash
   mvn compile exec:java
   ```

6. Verify output showing successful connection to Key Vault

### Exercise 2: Create a Key in Key Vault

1. Update the `KeyVaultOperations.java` file:

```java
package com.keyvault.demo;

import com.azure.identity.DefaultAzureCredentialBuilder;
import com.azure.security.keyvault.keys.KeyClient;
import com.azure.security.keyvault.keys.KeyClientBuilder;
import com.azure.security.keyvault.keys.models.KeyVaultKey;
import com.azure.security.keyvault.keys.models.KeyType;
import com.azure.security.keyvault.keys.models.CreateRsaKeyOptions;

public class KeyVaultOperations {
    
    private static KeyClient keyClient;
    
    public static void main(String[] args) {
        // Replace with your Key Vault URL
        String keyVaultUrl = "https://kv-java-lab-[your-unique-suffix].vault.azure.net/";
        
        System.out.println("Connecting to Key Vault: " + keyVaultUrl);
        
        // Create a KeyClient using DefaultAzureCredential
        keyClient = new KeyClientBuilder()
            .vaultUrl(keyVaultUrl)
            .credential(new DefaultAzureCredentialBuilder().build())
            .buildClient();
            
        System.out.println("Successfully connected to Key Vault");
        
        // Create a key
        createKey("myRsaKey");
    }
    
    private static void createKey(String keyName) {
        System.out.println("\nCreating key: " + keyName);
        
        try {
            // Create an RSA key with 2048 bits
            CreateRsaKeyOptions options = new CreateRsaKeyOptions(keyName)
                .setKeySize(2048);
                
            KeyVaultKey key = keyClient.createRsaKey(options);
            
            System.out.println("Key created with name: " + key.getName());
            System.out.println("Key ID: " + key.getId());
            System.out.println("Key type: " + key.getKeyType());
        } catch (Exception e) {
            System.out.println("Key may already exist: " + e.getMessage());
        }
    }
}
```

2. Run the application:

```bash
mvn compile exec:java
```

3. Verify output showing the key was created successfully

### Exercise 3: Retrieve and List Keys

1. Update the `KeyVaultOperations.java` file:

```java
package com.keyvault.demo;

import com.azure.identity.DefaultAzureCredentialBuilder;
import com.azure.security.keyvault.keys.KeyClient;
import com.azure.security.keyvault.keys.KeyClientBuilder;
import com.azure.security.keyvault.keys.models.KeyVaultKey;
import com.azure.security.keyvault.keys.models.KeyType;
import com.azure.security.keyvault.keys.models.CreateRsaKeyOptions;
import com.azure.security.keyvault.keys.models.KeyProperties;

public class KeyVaultOperations {
    
    private static KeyClient keyClient;
    
    public static void main(String[] args) {
        // Replace with your Key Vault URL
        String keyVaultUrl = "https://kv-java-lab-[your-unique-suffix].vault.azure.net/";
        
        System.out.println("Connecting to Key Vault: " + keyVaultUrl);
        
        // Create a KeyClient using DefaultAzureCredential
        keyClient = new KeyClientBuilder()
            .vaultUrl(keyVaultUrl)
            .credential(new DefaultAzureCredentialBuilder().build())
            .buildClient();
            
        System.out.println("Successfully connected to Key Vault");
        
        // Create a key
        createKey("myRsaKey");
        
        // Retrieve the key
        retrieveKey("myRsaKey");
        
        // List all keys
        listKeys();
    }
    
    private static void createKey(String keyName) {
        System.out.println("\nCreating key: " + keyName);
        
        try {
            // Create an RSA key with 2048 bits
            CreateRsaKeyOptions options = new CreateRsaKeyOptions()
                .setKeySize(2048)
                .setName(keyName);
                
            KeyVaultKey key = keyClient.createRsaKey(options);
            
            System.out.println("Key created with name: " + key.getName());
            System.out.println("Key ID: " + key.getId());
            System.out.println("Key type: " + key.getKeyType());
        } catch (Exception e) {
            System.out.println("Key may already exist: " + e.getMessage());
        }
    }
    
    private static void retrieveKey(String keyName) {
        System.out.println("\nRetrieving key: " + keyName);
        
        KeyVaultKey key = keyClient.getKey(keyName);
        
        System.out.println("Key retrieved with name: " + key.getName());
        System.out.println("Key ID: " + key.getId());
        System.out.println("Key type: " + key.getKeyType());
    }
    
    private static void listKeys() {
        System.out.println("\nListing all keys:");
        
        // List all keys in the Key Vault
        for (KeyProperties keyProperties : keyClient.listPropertiesOfKeys()) {
            System.out.println("Key name: " + keyProperties.getName());
            System.out.println("Key ID: " + keyProperties.getId());
            System.out.println("Enabled: " + keyProperties.isEnabled());
            System.out.println("---");
        }
    }
}
```

2. Run the application:

```bash
mvn compile exec:java
```

3. Verify output showing keys retrieved and listed

### Exercise 4: Update a Key's Properties

1. Update the `KeyVaultOperations.java` file:

```java
package com.keyvault.demo;

import com.azure.identity.DefaultAzureCredentialBuilder;
import com.azure.security.keyvault.keys.KeyClient;
import com.azure.security.keyvault.keys.KeyClientBuilder;
import com.azure.security.keyvault.keys.models.KeyVaultKey;
import com.azure.security.keyvault.keys.models.KeyType;
import com.azure.security.keyvault.keys.models.CreateRsaKeyOptions;
import com.azure.security.keyvault.keys.models.KeyProperties;
import com.azure.security.keyvault.keys.models.KeyOperation;

import java.util.Arrays;
import java.util.List;

public class KeyVaultOperations {
    
    private static KeyClient keyClient;
    
    public static void main(String[] args) {
        // Replace with your Key Vault URL
        String keyVaultUrl = "https://kv-java-lab-[your-unique-suffix].vault.azure.net/";
        
        System.out.println("Connecting to Key Vault: " + keyVaultUrl);
        
        // Create a KeyClient using DefaultAzureCredential
        keyClient = new KeyClientBuilder()
            .vaultUrl(keyVaultUrl)
            .credential(new DefaultAzureCredentialBuilder().build())
            .buildClient();
            
        System.out.println("Successfully connected to Key Vault");
        
        String keyName = "myRsaKey";
        
        // Create a key
        createKey(keyName);
        
        // Retrieve the key
        retrieveKey(keyName);
        
        // Update the key
        updateKey(keyName);
        
        // Retrieve the updated key
        retrieveKey(keyName);
        
        // List all keys
        listKeys();
    }
    
    private static void createKey(String keyName) {
        System.out.println("\nCreating key: " + keyName);
        
        try {
            // Create an RSA key with 2048 bits
            CreateRsaKeyOptions options = new CreateRsaKeyOptions()
                .setKeySize(2048)
                .setName(keyName);
                
            KeyVaultKey key = keyClient.createRsaKey(options);
            
            System.out.println("Key created with name: " + key.getName());
            System.out.println("Key ID: " + key.getId());
            System.out.println("Key type: " + key.getKeyType());
        } catch (Exception e) {
            System.out.println("Key may already exist: " + e.getMessage());
        }
    }
    
    private static void retrieveKey(String keyName) {
        System.out.println("\nRetrieving key: " + keyName);
        
        KeyVaultKey key = keyClient.getKey(keyName);
        
        System.out.println("Key retrieved with name: " + key.getName());
        System.out.println("Key ID: " + key.getId());
        System.out.println("Key type: " + key.getKeyType());
        System.out.println("Key operations: " + key.getKeyOperations());
    }
    
    private static void updateKey(String keyName) {
        System.out.println("\nUpdating key: " + keyName);
        
        // First get the current key
        KeyVaultKey key = keyClient.getKey(keyName);
        
        // Define the key operations
        List<KeyOperation> keyOperations = Arrays.asList(
            KeyOperation.ENCRYPT,
            KeyOperation.DECRYPT,
            KeyOperation.SIGN,
            KeyOperation.VERIFY
        );
        
        // Update the key properties
        key.getProperties().setExpiresOn(null); // Remove expiration if set
        KeyVaultKey updatedKey = keyClient.updateKeyProperties(
            key.getProperties().setEnabled(true);
        );
        
        System.out.println("Key updated with name: " + updatedKey.getName());
        System.out.println("Key enabled: " + updatedKey.getProperties().isEnabled());
        System.out.println("Key operations: " + updatedKey.getKeyOperations());
    }
    
    private static void listKeys() {
        System.out.println("\nListing all keys:");
        
        // List all keys in the Key Vault
        for (KeyProperties keyProperties : keyClient.listPropertiesOfKeys()) {
            System.out.println("Key name: " + keyProperties.getName());
            System.out.println("Key ID: " + keyProperties.getId());
            System.out.println("Enabled: " + keyProperties.isEnabled());
            System.out.println("---");
        }
    }
}
```

2. Run the application:

```bash
mvn compile exec:java
```

3. Verify output showing key properties updated

### Exercise 5: Create Different Key Types

1. Update the `KeyVaultOperations.java` file:

```java
package com.keyvault.demo;

import com.azure.identity.DefaultAzureCredentialBuilder;
import com.azure.security.keyvault.keys.KeyClient;
import com.azure.security.keyvault.keys.KeyClientBuilder;
import com.azure.security.keyvault.keys.models.KeyVaultKey;
import com.azure.security.keyvault.keys.models.KeyType;
import com.azure.security.keyvault.keys.models.CreateRsaKeyOptions;
import com.azure.security.keyvault.keys.models.CreateEcKeyOptions;
import com.azure.security.keyvault.keys.models.KeyCurveName;
import com.azure.security.keyvault.keys.models.KeyProperties;

public class KeyVaultOperations {
    
    private static KeyClient keyClient;
    
    public static void main(String[] args) {
        // Replace with your Key Vault URL
        String keyVaultUrl = "https://kv-java-lab-[your-unique-suffix].vault.azure.net/";
        
        System.out.println("Connecting to Key Vault: " + keyVaultUrl);
        
        // Create a KeyClient using DefaultAzureCredential
        keyClient = new KeyClientBuilder()
            .vaultUrl(keyVaultUrl)
            .credential(new DefaultAzureCredentialBuilder().build())
            .buildClient();
            
        System.out.println("Successfully connected to Key Vault");
        
        // Create different key types
        createRsaKey("myRsaKey");
        createEcKey("myEcKey");
        createOctKey("myOctKey");
        
        // List all keys
        listKeys();
    }
    
    private static void createRsaKey(String keyName) {
        System.out.println("\nCreating RSA key: " + keyName);
        
        try {
            // Create an RSA key with 2048 bits
            CreateRsaKeyOptions options = new CreateRsaKeyOptions()
                .setKeySize(2048)
                .setName(keyName);
                
            KeyVaultKey key = keyClient.createRsaKey(options);
            
            System.out.println("RSA Key created with name: " + key.getName());
            System.out.println("Key ID: " + key.getId());
            System.out.println("Key type: " + key.getKeyType());
        } catch (Exception e) {
            System.out.println("Key may already exist: " + e.getMessage());
        }
    }
    
    private static void createEcKey(String keyName) {
        System.out.println("\nCreating EC key: " + keyName);
        
        try {
            // Create an EC key with P-256 curve
            CreateEcKeyOptions options = new CreateEcKeyOptions()
                .setCurveName(KeyCurveName.P_256)
                .setName(keyName);
                
            KeyVaultKey key = keyClient.createEcKey(options);
            
            System.out.println("EC Key created with name: " + key.getName());
            System.out.println("Key ID: " + key.getId());
            System.out.println("Key type: " + key.getKeyType());
        } catch (Exception e) {
            System.out.println("Key may already exist: " + e.getMessage());
        }
    }
    
    private static void createOctKey(String keyName) {
        System.out.println("\nCreating OCT key: " + keyName);
        
        try {
            // Create an OCT key (symmetric key)
            KeyVaultKey key = keyClient.createKey(keyName, KeyType.OCT_HSM);
            
            System.out.println("OCT Key created with name: " + key.getName());
            System.out.println("Key ID: " + key.getId());
            System.out.println("Key type: " + key.getKeyType());
        } catch (Exception e) {
            System.out.println("Key may already exist: " + e.getMessage());
        }
    }
    
    private static void listKeys() {
        System.out.println("\nListing all keys:");
        
        // List all keys in the Key Vault
        for (KeyProperties keyProperties : keyClient.listPropertiesOfKeys()) {
            System.out.println("Key name: " + keyProperties.getName());
            System.out.println("Key ID: " + keyProperties.getId());
            System.out.println("Enabled: " + keyProperties.isEnabled());
            
            // Get the full key to see its type
            KeyVaultKey key = keyClient.getKey(keyProperties.getName());
            System.out.println("Key type: " + key.getKeyType());
            System.out.println("---");
        }
    }
}
```

2. Run the application:

```bash
mvn compile exec:java
```

3. Verify output showing different key types created

### Exercise 6: Delete Keys and Clean Up

1. Update the `KeyVaultOperations.java` file to add delete functionality:

```java
package com.keyvault.demo;

import com.azure.identity.DefaultAzureCredentialBuilder;
import com.azure.security.keyvault.keys.KeyClient;
import com.azure.security.keyvault.keys.KeyClientBuilder;
import com.azure.security.keyvault.keys.models.KeyVaultKey;
import com.azure.security.keyvault.keys.models.KeyType;
import com.azure.security.keyvault.keys.models.CreateRsaKeyOptions;
import com.azure.security.keyvault.keys.models.CreateEcKeyOptions;
import com.azure.security.keyvault.keys.models.KeyCurveName;
import com.azure.security.keyvault.keys.models.KeyProperties;
import com.azure.security.keyvault.keys.models.DeletedKey;

public class KeyVaultOperations {
    
    private static KeyClient keyClient;
    
    public static void main(String[] args) {
        // Replace with your Key Vault URL
        String keyVaultUrl = "https://kv-java-lab-[your-unique-suffix].vault.azure.net/";
        
        System.out.println("Connecting to Key Vault: " + keyVaultUrl);
        
        // Create a KeyClient using DefaultAzureCredential
        keyClient = new KeyClientBuilder()
            .vaultUrl(keyVaultUrl)
            .credential(new DefaultAzureCredentialBuilder().build())
            .buildClient();
            
        System.out.println("Successfully connected to Key Vault");
        
        // List keys before deletion
        System.out.println("\nKeys before deletion:");
        listKeys();
        
        // Delete the keys we created
        deleteKey("myRsaKey");
        deleteKey("myEcKey");
        deleteKey("myOctKey");
        
        System.out.println("\nKey deletion operations initiated. Keys will be soft-deleted.");
    }
    
    private static void listKeys() {
        // List all keys in the Key Vault
        for (KeyProperties keyProperties : keyClient.listPropertiesOfKeys()) {
            System.out.println("Key name: " + keyProperties.getName());
            System.out.println("Key ID: " + keyProperties.getId());
            System.out.println("---");
        }
    }
    
    private static void deleteKey(String keyName) {
        System.out.println("\nDeleting key: " + keyName);
        
        try {
            // Start the deletion
            DeletedKey deletedKey = keyClient.beginDeleteKey(keyName).poll().getValue();
            
            System.out.println("Key deletion initiated for: " + deletedKey.getName());
            System.out.println("Recovery ID: " + deletedKey.getRecoveryId());
            System.out.println("Scheduled purge date: " + deletedKey.getScheduledPurgeDate());
            System.out.println("Deletion date: " + deletedKey.getDeletedOn());
        } catch (Exception e) {
            System.out.println("Error deleting key: " + e.getMessage());
        }
    }
}
```

2. Run the application one final time:

```bash
mvn compile exec:java
```

3. Verify output showing keys being deleted

## Advanced Exercise: Backup and Restore Keys

1. Create a new file called `KeyVaultBackupRestore.java` in the same directory:

```java
package com.keyvault.demo;

import com.azure.identity.DefaultAzureCredentialBuilder;
import com.azure.security.keyvault.keys.KeyClient;
import com.azure.security.keyvault.keys.KeyClientBuilder;
import com.azure.security.keyvault.keys.models.KeyVaultKey;
import com.azure.security.keyvault.keys.models.CreateRsaKeyOptions;

import java.io.File;
import java.io.FileOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;

public class KeyVaultBackupRestore {
    
    private static KeyClient keyClient;
    
    public static void main(String[] args) {
        // Replace with your Key Vault URL
        String keyVaultUrl = "https://kv-java-lab-[your-unique-suffix].vault.azure.net/";
        
        System.out.println("Connecting to Key Vault: " + keyVaultUrl);
        
        // Create a KeyClient using DefaultAzureCredential
        keyClient = new KeyClientBuilder()
            .vaultUrl(keyVaultUrl)
            .credential(new DefaultAzureCredentialBuilder().build())
            .buildClient();
            
        System.out.println("Successfully connected to Key Vault");
        
        String keyName = "backupTestKey";
        
        // Create a key
        createKey(keyName);
        
        // Backup the key
        backupKey(keyName);
        
        // Delete the key
        System.out.println("\nDeleting key so we can test restore...");
        keyClient.beginDeleteKey(keyName).poll();
        
        // Wait for deletion to complete (in real applications, use polling)
        try {
            System.out.println("Waiting for deletion to complete...");
            Thread.sleep(20000); // 20 seconds
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        // Restore the key from backup
        restoreKey(keyName);
    }
    
    private static void createKey(String keyName) {
        System.out.println("\nCreating key: " + keyName);
        
        try {
            // Create an RSA key with 2048 bits
            CreateRsaKeyOptions options = new CreateRsaKeyOptions()
                .setKeySize(2048)
                .setName(keyName);
                
            KeyVaultKey key = keyClient.createRsaKey(options);
            
            System.out.println("Key created with name: " + key.getName());
            System.out.println("Key ID: " + key.getId());
            System.out.println("Key type: " + key.getKeyType());
        } catch (Exception e) {
            System.out.println("Key may already exist: " + e.getMessage());
        }
    }
    
    private static void backupKey(String keyName) {
        System.out.println("\nBacking up key: " + keyName);
        
        try {
            // Get the key backup
            byte[] backup = keyClient.backupKey(keyName);
            
            // Save the backup to a file
            String backupFileName = keyName + ".backup";
            try (FileOutputStream fos = new FileOutputStream(backupFileName)) {
                fos.write(backup);
            }
            
            System.out.println("Key backup saved to " + backupFileName);
        } catch (Exception e) {
            System.out.println("Failed to backup key: " + e.getMessage());
        }
    }
    
    private static void restoreKey(String keyName) {
        System.out.println("\nRestoring key from backup file: " + keyName + ".backup");
        
        try {
            // Read the backup file
            String backupFileName = keyName + ".backup";
            byte[] backup = Files.readAllBytes(Paths.get(backupFileName));
            
            // Restore the key
            KeyVaultKey restoredKey = keyClient.restoreKeyBackup(backup);
            
            System.out.println("Key restored with name: " + restoredKey.getName());
            System.out.println("Key ID: " + restoredKey.getId());
            System.out.println("Key type: " + restoredKey.getKeyType());
        } catch (IOException e) {
            System.out.println("Failed to read backup file: " + e.getMessage());
        } catch (Exception e) {
            System.out.println("Failed to restore key: " + e.getMessage());
        }
    }
}
```

2. Update the `App.java` file to call this new class:

```java
package com.keyvault.demo;

public class App {
    public static void main(String[] args) {
        // KeyVaultOperations.main(args);
        KeyVaultBackupRestore.main(args);
    }
}
```

3. Run the application:

```bash
mvn compile exec:java
```