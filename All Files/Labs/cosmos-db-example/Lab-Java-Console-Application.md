// Project structure
// ├── pom.xml
// ├── src
// │   └── main
// │       └── java
// │           └── com
// │               └── example
// │                   ├── App.java
// │                   └── model
// │                       └── Person.java
// └── README.md

// File: pom.xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>cosmos-db-java-demo</artifactId>
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

// File: src/main/java/com/example/App.java
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
    private static final String DATABASE_NAME = "YourDatabaseName";
    private static final String CONTAINER_NAME = "YourContainerName";

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

// File: src/main/java/com/example/model/Person.java
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

// File: README.md
# Azure Cosmos DB Java Console Application

This is a sample Java console application that demonstrates how to connect to and interact with Azure Cosmos DB using the Azure SDK for Java.

## Features

- Connect to Azure Cosmos DB using the Azure SDK for Java
- Query all items in a container
- Query items by ID
- Create new items
- Update existing items
- Delete items

## Prerequisites

- Java Development Kit (JDK) 11 or later
- Apache Maven
- An Azure account with an active subscription
- Azure Cosmos DB account

## Setup

1. Clone this repository:
   ```
   git clone https://github.com/yourusername/cosmos-db-java-demo.git
   cd cosmos-db-java-demo
   ```

2. Open the `src/main/java/com/example/App.java` file and replace the placeholder values with your actual Azure Cosmos DB details:
   ```java
   private static final String ENDPOINT = "https://your-account.documents.azure.com:443/";
   private static final String KEY = "YOUR_PRIMARY_KEY";
   private static final String DATABASE_NAME = "YourDatabaseName";
   private static final String CONTAINER_NAME = "YourContainerName";
   ```

3. Ensure your Cosmos DB container has a partition key that matches the ID field of the items or update the code to use your actual partition key.

## Build and Run

1. Build the application:
   ```
   mvn clean package
   ```

2. Run the application:
   ```
   java -jar target/cosmos-db-java-demo-1.0-SNAPSHOT-jar-with-dependencies.jar
   ```

## Application Structure

- `App.java`: Main application class with interactive menu and Cosmos DB operations
- `model/Person.java`: Model class representing documents in the Cosmos DB container

## Additional Resources

- [Azure Cosmos DB Documentation](https://docs.microsoft.com/azure/cosmos-db/)
- [Azure Cosmos DB Java SDK Documentation](https://docs.microsoft.com/java/api/overview/azure/cosmos-readme)

## License

This project is licensed under the MIT License - see the LICENSE file for details.