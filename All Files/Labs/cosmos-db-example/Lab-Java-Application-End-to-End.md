# Azure Cosmos DB Java Lab with VS Code

This lab guide will walk you through creating a Java console application that connects to Azure Cosmos DB using Visual Studio Code. By the end of this lab, you'll have a working application that can perform CRUD operations (Create, Read, Update, Delete) on a Cosmos DB database.

## Prerequisites

Before starting this lab, ensure you have the following:

1. **Visual Studio Code** installed on your computer
   - Download from: [https://code.visualstudio.com/](https://code.visualstudio.com/)

2. **Java Development Kit (JDK)** version 11 or later
   - Download from: [https://adoptium.net/](https://adoptium.net/)

3. **Maven** for dependency management
   - Download from: [https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)

4. **VS Code Extensions**
   - Extension Pack for Java
   - Maven for Java

5. **Azure Cosmos DB account**
   - You'll need an active Azure account with a Cosmos DB instance

## Part 1: Set Up Your Development Environment

### 1. Install Required VS Code Extensions

1. Open VS Code
2. Click on the Extensions icon in the Activity Bar (or press `Ctrl+Shift+X`)
3. Search for and install the following extensions:
   - "Extension Pack for Java" by Microsoft
   - "Maven for Java" by Microsoft

### 2. Verify Java and Maven Installation

1. Open a terminal in VS Code by selecting `Terminal > New Terminal` from the menu
2. Check that Java is installed correctly:
   ```
   java -version
   ```
   You should see output showing your Java version.

3. Check that Maven is installed correctly:
   ```
   mvn -version
   ```
   You should see output showing your Maven version.

## Part 2: Set Up Your Azure Cosmos DB Account

### 1. Create a Cosmos DB Account (If You Don't Have One)

1. Sign in to the [Azure Portal](https://portal.azure.com/)
2. Click "Create a resource" > "Databases" > "Azure Cosmos DB"
3. Select "Azure Cosmos DB for NoSQL" (formerly "SQL API")
4. Fill in the required details:
   - Subscription: Select your Azure subscription
   - Resource Group: Create a new one or use an existing one
   - Account Name: Enter a unique name
   - Location: Choose a location close to you
   - Capacity mode: Choose "Provisioned throughput" or "Serverless" (for testing, Serverless is cost-effective)
5. Click "Review + create", then "Create"
6. Wait for the deployment to complete

### 2. Create a Database and Container

1. Once your Cosmos DB account is created, navigate to it in the Azure Portal
2. Select "Data Explorer" from the left menu
3. Click "New Database"
   - Database id: `DemoDatabase`
   - Throughput: Select "Manual" and set to the minimum (400 RU/s) for testing
   - Click "OK"
4. Select your new database, click "New Container"
   - Container id: `Persons`
   - Partition key: `/id` (for simplicity in this lab)
   - Click "OK"

### 3. Retrieve Connection Information

1. In your Cosmos DB account, select "Keys" from the left menu
2. Note down the following information:
   - URI
   - Primary Key
   
You'll need these to connect your application to Cosmos DB.

## Part 3: Create a New Java Maven Project in VS Code

### 1. Create a New Maven Project

1. Open VS Code
2. Open the Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P` on Mac)
3. Type "Java: Create Java Project" and select it
4. Select "Maven" from the project type options
5. Select "maven-archetype-quickstart" from the archetype options
6. Enter the following information when prompted:
   - Group Id: `com.example`
   - Artifact Id: `cosmosdb-demo`
   - Version: Press Enter to accept the default (1.0-SNAPSHOT)
   - Package name: Press Enter to accept the default (com.example)
7. Select a location to save your project
8. Wait for the project to be created and opened in VS Code

### 2. Configure Maven Dependencies

1. Open the `pom.xml` file in your project
2. Replace the entire content with the following:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>cosmosdb-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- Azure Cosmos DB SDK -->
        <dependency>
            <groupId>com.azure</groupId>
            <artifactId>azure-cosmos</artifactId>
            <version>4.53.1</version>
        </dependency>
        
        <!-- Logging dependencies -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.36</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.36</version>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Netty native dependencies for improved performance -->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-tcnative-boringssl-static</artifactId>
            <version>2.0.61.Final</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Maven Compiler Plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
            
            <!-- Maven Assembly Plugin for creating an executable JAR with dependencies -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.6.0</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                        <configuration>
                            <archive>
                                <manifest>
                                    <mainClass>com.example.App</mainClass>
                                </manifest>
                            </archive>
                            <descriptorRefs>
                                <descriptorRef>jar-with-dependencies</descriptorRef>
                            </descriptorRefs>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

3. Save the file (`Ctrl+S` or `Cmd+S` on Mac)
4. VS Code should automatically download the dependencies (this may take a few minutes)

## Part 4: Create the Model Class

1. In VS Code's Explorer view (left sidebar), right-click on the `src/main/java/com/example` folder
2. Select "New Folder" and name it `model`
3. Right-click on the new `model` folder and select "New File"
4. Name the file `Person.java`
5. Add the following code to the `Person.java` file:

```java
package com.example.model;

/**
 * Model class for a Person document in Cosmos DB.
 * This class represents the structure of documents in the container.
 */
public class Person {
    private String id;
    private String name;
    private int age;
    private String email;
    
    // Default constructor required for JSON deserialization
    public Person() {
    }
    
    // Constructor with parameters
    public Person(String id, String name, int age, String email) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.email = email;
    }
    
    // Getters and setters
    public String getId() {
        return id;
    }
    
    public void setId(String id) {
        this.id = id;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public int getAge() {
        return age;
    }
    
    public void setAge(int age) {
        this.age = age;
    }
    
    public String getEmail() {
        return email;
    }
    
    public void setEmail(String email) {
        this.email = email;
    }
    
    @Override
    public String toString() {
        return "Person{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", email='" + email + '\'' +
                '}';
    }
}
```

6. Save the file

## Part 5: Create the Main Application Class

1. Open the `App.java` file in the `src/main/java/com/example` folder
2. Replace its content with the following code:

```java
package com.example;

import com.azure.cosmos.ConsistencyLevel;
import com.azure.cosmos.CosmosClient;
import com.azure.cosmos.CosmosClientBuilder;
import com.azure.cosmos.CosmosContainer;
import com.azure.cosmos.CosmosDatabase;
import com.azure.cosmos.models.CosmosItemRequestOptions;
import com.azure.cosmos.models.CosmosQueryRequestOptions;
import com.azure.cosmos.models.PartitionKey;
import com.azure.cosmos.util.CosmosPagedIterable;
import com.example.model.Person;

import java.util.Scanner;
import java.util.UUID;

public class App {
    // Replace these with your actual Cosmos DB details
    private static final String ENDPOINT = "https://your-account.documents.azure.com:443/";
    private static final String KEY = "YOUR_PRIMARY_KEY";
    private static final String DATABASE_NAME = "DemoDatabase";
    private static final String CONTAINER_NAME = "Persons";

    private static CosmosClient cosmosClient;
    private static CosmosDatabase database;
    private static CosmosContainer container;
    private static Scanner scanner;

    public static void main(String[] args) {
        System.out.println("Azure Cosmos DB Demo Application");
        System.out.println("--------------------------------");

        scanner = new Scanner(System.in);
        
        try {
            // Initialize the Cosmos client
            initializeCosmosClient();
            
            // Show menu and handle user input
            boolean exit = false;
            while (!exit) {
                displayMenu();
                int choice = getUserChoice();
                
                switch (choice) {
                    case 1:
                        queryAllItems();
                        break;
                    case 2:
                        queryItemById();
                        break;
                    case 3:
                        createNewItem();
                        break;
                    case 4:
                        updateItem();
                        break;
                    case 5:
                        deleteItem();
                        break;
                    case 6:
                        exit = true;
                        break;
                    default:
                        System.out.println("Invalid option. Please try again.");
                }
                
                System.out.println("\nPress Enter to continue...");
                scanner.nextLine();
            }
            
        } catch (Exception e) {
            System.err.println("Error occurred: " + e.getMessage());
            e.printStackTrace();
        } finally {
            // Close the client
            if (cosmosClient != null) {
                cosmosClient.close();
                System.out.println("Cosmos DB client closed.");
            }
            
            if (scanner != null) {
                scanner.close();
            }
        }
        
        System.out.println("Application terminated. Goodbye!");
    }

    private static void initializeCosmosClient() {
        System.out.println("Connecting to Azure Cosmos DB...");
        
        cosmosClient = new CosmosClientBuilder()
            .endpoint(ENDPOINT)
            .key(KEY)
            .consistencyLevel(ConsistencyLevel.EVENTUAL)
            .buildClient();
        
        database = cosmosClient.getDatabase(DATABASE_NAME);
        container = database.getContainer(CONTAINER_NAME);
        
        System.out.println("Connected successfully to Azure Cosmos DB!");
        System.out.println("Database: " + DATABASE_NAME);
        System.out.println("Container: " + CONTAINER_NAME);
    }

    private static void displayMenu() {
        System.out.println("\n========= MENU =========");
        System.out.println("1. Query all items");
        System.out.println("2. Query item by ID");
        System.out.println("3. Create new item");
        System.out.println("4. Update an item");
        System.out.println("5. Delete an item");
        System.out.println("6. Exit");
        System.out.println("========================");
        System.out.print("Enter your choice (1-6): ");
    }

    private static int getUserChoice() {
        try {
            int choice = Integer.parseInt(scanner.nextLine());
            return choice;
        } catch (NumberFormatException e) {
            return -1; // Invalid choice
        }
    }

    private static void queryAllItems() {
        System.out.println("\n--- Querying All Items ---");
        
        String query = "SELECT * FROM c";
        CosmosPagedIterable<Person> items = container.queryItems(query, new CosmosQueryRequestOptions(), Person.class);
        
        System.out.println("Query: " + query);
        System.out.println("\nResults:");
        
        int count = 0;
        for (Person item : items) {
            System.out.println(item);
            count++;
        }
        
        if (count == 0) {
            System.out.println("No items found in the container.");
        } else {
            System.out.println("\nTotal items retrieved: " + count);
        }
    }

    private static void queryItemById() {
        System.out.println("\n--- Query Item by ID ---");
        System.out.print("Enter the item ID: ");
        String id = scanner.nextLine();
        
        try {
            // In a real application, you would need to know the partition key
            // For this example, we're assuming ID is also the partition key
            Person person = container.readItem(id, new PartitionKey(id), Person.class).getItem();
            System.out.println("\nItem found:");
            System.out.println(person);
        } catch (Exception e) {
            System.out.println("Error: Item not found or other error occurred.");
            System.out.println("Details: " + e.getMessage());
        }
    }

    private static void createNewItem() {
        System.out.println("\n--- Create New Item ---");
        
        Person newPerson = new Person();
        newPerson.setId(UUID.randomUUID().toString());
        
        System.out.print("Enter name: ");
        newPerson.setName(scanner.nextLine());
        
        System.out.print("Enter age: ");
        try {
            newPerson.setAge(Integer.parseInt(scanner.nextLine()));
        } catch (NumberFormatException e) {
            System.out.println("Invalid age. Setting default age of 0.");
            newPerson.setAge(0);
        }
        
        System.out.print("Enter email: ");
        newPerson.setEmail(scanner.nextLine());
        
        try {
            container.createItem(newPerson);
            System.out.println("\nItem created successfully!");
            System.out.println("Generated ID: " + newPerson.getId());
        } catch (Exception e) {
            System.out.println("Error creating item: " + e.getMessage());
        }
    }

    private static void updateItem() {
        System.out.println("\n--- Update Item ---");
        System.out.print("Enter the ID of the item to update: ");
        String id = scanner.nextLine();
        
        try {
            // Retrieve the item first
            Person person = container.readItem(id, new PartitionKey(id), Person.class).getItem();
            
            System.out.println("\nCurrent item details:");
            System.out.println(person);
            
            System.out.println("\nEnter new values (or press Enter to keep current values):");
            
            System.out.print("Name [" + person.getName() + "]: ");
            String name = scanner.nextLine();
            if (!name.isEmpty()) {
                person.setName(name);
            }
            
            System.out.print("Age [" + person.getAge() + "]: ");
            String ageStr = scanner.nextLine();
            if (!ageStr.isEmpty()) {
                try {
                    person.setAge(Integer.parseInt(ageStr));
                } catch (NumberFormatException e) {
                    System.out.println("Invalid age input. Age not updated.");
                }
            }
            
            System.out.print("Email [" + person.getEmail() + "]: ");
            String email = scanner.nextLine();
            if (!email.isEmpty()) {
                person.setEmail(email);
            }
            
            // Update the item in Cosmos DB
            container.replaceItem(person, person.getId(), new PartitionKey(person.getId()), new CosmosItemRequestOptions());
            
            System.out.println("\nItem updated successfully!");
        } catch (Exception e) {
            System.out.println("Error updating item: " + e.getMessage());
        }
    }

    private static void deleteItem() {
        System.out.println("\n--- Delete Item ---");
        System.out.print("Enter the ID of the item to delete: ");
        String id = scanner.nextLine();
        
        try {
            // In a real application, you would need to know the partition key
            // For this example, we're assuming ID is also the partition key
            container.deleteItem(id, new PartitionKey(id), new CosmosItemRequestOptions());
            System.out.println("\nItem deleted successfully!");
        } catch (Exception e) {
            System.out.println("Error deleting item: " + e.getMessage());
        }
    }
}
```

3. Update the Cosmos DB connection details:
   - Replace `ENDPOINT` with your Cosmos DB URI
   - Replace `KEY` with your Cosmos DB Primary Key
   - Verify the `DATABASE_NAME` and `CONTAINER_NAME` match what you created

4. Save the file

## Part 6: Build and Run the Application

### 1. Build the Project

1. Right-click on the `pom.xml` file in the Explorer view
2. Select "Run Maven Command"
3. Choose "clean package" from the list of Maven lifecycle phases
4. VS Code will open a terminal and run the Maven build command
5. Wait for the build to complete (you should see a "BUILD SUCCESS" message)

### 2. Run the Application

1. In VS Code's Explorer view, navigate to the `target` folder
2. You should see a file named `cosmosdb-demo-1.0-SNAPSHOT-jar-with-dependencies.jar`
3. Right-click on this file
4. Select "Run Java"

Alternatively, you can run the application from the terminal:
```
java -jar target/cosmosdb-demo-1.0-SNAPSHOT-jar-with-dependencies.jar
```

### 3. Test the Application

1. The application will display a menu with options
2. Try creating some new Person records (option 3)
3. Query all items to see your created records (option 1)
4. Try updating a record (option 4)
5. Try deleting a record (option 5)
6. When finished, select option 6 to exit

## Part 7: Troubleshooting Common Issues

### Connection Issues

If you encounter connection errors:

1. **Verify your endpoint and key**:
   - Double-check the URI and Primary Key from the Azure Portal
   - Make sure there are no trailing spaces when you copied the values

2. **Check your network**:
   - Ensure your machine can connect to Azure
   - Check if you need to configure network settings or a proxy

3. **Firewall settings**:
   - In the Azure Portal, go to your Cosmos DB account
   - Select "Networking" from the left menu
   - Ensure "Allow access from all networks" is selected, or add your IP address to the allowed list

### Build Issues

If Maven build fails:

1. **Check Maven installation**:
   - Verify Maven is correctly installed and in your PATH
   - Try running `mvn -version` in the terminal

2. **Check dependency availability**:
   - Ensure you have internet access to download dependencies
   - Check if your organization has a firewall blocking Maven repositories

3. **JDK compatibility**:
   - Ensure you're using JDK 11 or later
   - Verify the `JAVA_HOME` environment variable is set correctly

### Runtime Issues

If the application runs but doesn't work correctly:

1. **Verify database and container names**:
   - Check if the database and container names in your code match exactly what you created in Azure
   - These are case-sensitive

2. **Check partition key**:
   - The code assumes the partition key is the same as the `id` field
   - If you used a different partition key, update the code accordingly

3. **Permission issues**:
   - Ensure your Cosmos DB key has sufficient permissions to perform all operations

## Conclusion

Congratulations! You have successfully created a Java console application that connects to Azure Cosmos DB using Visual Studio Code. This application demonstrates the basic CRUD operations that you can perform with Cosmos DB.

### Next Steps

To further enhance your application, consider:

1. Adding more sophisticated error handling
2. Implementing pagination for large result sets
3. Adding more complex query capabilities
4. Creating a graphical user interface
5. Using the Cosmos DB Change Feed to react to changes in the database

## Additional Resources

- [Azure Cosmos DB Documentation](https://docs.microsoft.com/azure/cosmos-db/)
- [Azure Cosmos DB Java SDK Documentation](https://docs.microsoft.com/java/api/overview/azure/cosmos-readme)
- [VS Code Java Development Guide](https://code.visualstudio.com/docs/java/java-tutorial)
- [Maven Documentation](https://maven.apache.org/guides/index.html)