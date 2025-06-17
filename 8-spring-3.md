Excellent. This video marks the shift from traditional XML to modern Java-based configuration. This is a critical topic for any current Spring developer. Here is the gist.

### Core Concept: Java-Based Configuration

Instead of defining your beans and their relationships in an XML file, you can do everything directly in Java code. This approach is generally preferred in modern applications because it is more powerful, type-safe, and easier to refactor.

### Key Annotations and Classes

1.  **`@Configuration`**:
    * This is a class-level annotation. You place it on a Java class to signal that its primary purpose is to be a source of bean definitions.
    * It is the direct replacement for the `<beans>` root tag in your `spring.xml` file.

2.  **`@Bean`**:
    * This is a method-level annotation. You place it on a method inside a `@Configuration` class.
    * It tells Spring that this method will return an object that should be registered as a bean in the Spring container.
    * It is the direct replacement for the `<bean>` tag in XML.

3.  **`AnnotationConfigApplicationContext`**:
    * To load your application using Java configuration, you must use this specific implementation of `ApplicationContext`.
    * You pass your `@Configuration` class to its constructor (e.g., `new AnnotationConfigApplicationContext(AppConfig.class)`).

### How It Works: A Step-by-Step Guide

1.  **Create a Configuration Class**: Make a new Java class (e.g., `AppConfig`) and annotate it with `@Configuration`.

2.  **Define Bean-Producing Methods**: Inside `AppConfig`, create methods for each bean you want Spring to manage. Annotate each of these methods with `@Bean`. The method should create and return an instance of your object.

    ```java
    @Configuration
    public class AppConfig {

        @Bean
        public Desktop desktop() {
            // We create the object, but Spring manages its lifecycle
            return new Desktop();
        }

        // We could define more beans here...
    }
    ```

3.  **Bootstrap the Application**: In your `main` method, switch from `ClassPathXmlApplicationContext` to `AnnotationConfigApplicationContext`.

    ```java
    // The new way to start the Spring container
    ApplicationContext context =
        new AnnotationConfigApplicationContext(AppConfig.class);

    // Retrieving the bean works the same as before
    Desktop myDesktop = context.getBean(Desktop.class);
    myDesktop.compile();
    ```

**Important Clarification**: Even though you are writing `return new Desktop();` yourself, you are **not** managing the object. The Spring container calls your `@Bean` method and takes control of the returned object, managing its entire lifecycle (scope, injection, etc.) just as it did with XML.

---

### 📝 Interview Q&A

**Q: What are the main advantages of Java-based configuration over XML?**
**A:** Java-based configuration is type-safe, which means the compiler can catch errors. It's also more refactor-friendly, as renaming a class or method will be automatically updated by an IDE. It allows for more complex, programmatic logic when defining beans, which is difficult to do in static XML.

**Q: What is the role of the `@Configuration` and `@Bean` annotations?**
**A:** `@Configuration` marks a class as a source of bean definitions for the Spring container. `@Bean` is used on methods within that class to declare that the method produces a bean that should be managed by Spring.

**Q: How do you initialize the Spring container when using Java-based configuration?**
**A:** You instantiate the `AnnotationConfigApplicationContext` class, passing your `@Configuration` class (or classes) as an argument to its constructor.

**Q: Inside a `@Bean` method, you write `return new MyObject()`. Does this mean you are breaking the Inversion of Control principle?**
**A:** No. While you are writing the instantiation logic, you are giving control to Spring. The Spring container is responsible for invoking that `@Bean` method at the appropriate time and managing the lifecycle of the object that is returned. You are simply providing a factory for the bean, not managing it yourself.

---

Excellent. This video dives into the specifics of naming and retrieving beans in Java configuration. Here's the core gist for your notes.

### Core Concept: Naming Beans in Java Configuration

When you define a bean using the `@Bean` annotation, it needs a name (or "ID") so you can refer to it. There are specific rules for how this name is determined.

#### 1. Default Naming Convention

If you don't specify a name for your bean, Spring uses a default naming strategy:

* **The default name of the bean is the name of the method annotated with `@Bean`**.

```java
@Configuration
public class AppConfig {

    @Bean
    public Desktop myDesktop() { // The bean's name will be "myDesktop"
        return new Desktop();
    }
}

// How to retrieve it:
Desktop dt = context.getBean("myDesktop", Desktop.class);
```

#### 2. Customizing the Bean Name

You can easily override the default name using the `name` attribute of the `@Bean` annotation.

```java
@Configuration
public class AppConfig {

    @Bean(name = "com2") // The bean's name is now "com2"
    public Desktop desktop() {
        return new Desktop();
    }
}

// How to retrieve it:
Desktop dt = context.getBean("com2", Desktop.class);
```

#### 3. Providing Multiple Names (Aliases)

You can assign multiple names to a single bean by providing a string array to the `name` attribute. The first name is the primary name, and the rest are aliases.

```java
@Configuration
public class AppConfig {
    
    // This single bean can be accessed by any of these three names
    @Bean(name = {"com2", "desktop1", "Beast"})
    public Desktop desktop() {
        return new Desktop();
    }
}

// All of these lines would retrieve the same bean instance:
Desktop d1 = context.getBean("com2", Desktop.class);
Desktop d2 = context.getBean("desktop1", Desktop.class);
Desktop d3 = context.getBean("Beast", Desktop.class);
```

---

### 📝 Interview Q&A

**Q: When using the `@Bean` annotation without any attributes, what is the default name of the bean that gets created?**
**A:** The default name of the bean is the name of the method itself. For example, if the method is `getDataSource()`, the bean name will be `getDataSource`.

**Q: How do you explicitly set a custom name for a bean in Java configuration?**
**A:** You use the `name` attribute of the `@Bean` annotation, like this: `@Bean(name = "customBeanName")`.

**Q: Can a single bean have multiple names? If so, how do you configure that?**
**A:** Yes, a bean can have multiple names, also known as aliases. You can configure this by passing a string array to the `name` attribute of the `@Bean` annotation, for example: `@Bean(name = {"primaryName", "aliasOne", "aliasTwo"})`.

**Q: You have a bean defined as `@Bean public Computer buildComputer()`. How would you retrieve this specific bean by name from the `ApplicationContext`?**
**A:** You would call `context.getBean("buildComputer", Computer.class)`.

---

Excellent. This video covers how to manage bean scopes, a fundamental concept, within the context of modern Java-based configuration. Here is the gist.

### Core Concept: Configuring Bean Scopes with `@Scope`

Just like in XML, you can control the lifecycle of your beans in Java configuration. The default scope for any bean defined with `@Bean` is **singleton**.

To change the scope, you use the **`@Scope`** annotation.

* **What it is**: A method-level annotation that you place directly on your `@Bean` method.
* **How it works**: You provide the name of the desired scope as a string value. The two most important scopes are `"singleton"` and `"prototype"`.

### How to Implement `prototype` Scope

To make a bean `prototype` (meaning a new instance is created every time it's requested), you simply add `@Scope("prototype")` to its bean definition method.

#### Before: Default Singleton Scope

Calling `getBean()` multiple times returns the **same** instance.

```java
@Configuration
public class AppConfig {
    @Bean
    // No @Scope annotation means it defaults to "singleton"
    public Desktop desktop() {
        return new Desktop();
    }
}
// Result: The constructor is called only ONCE.
```

#### After: Prototype Scope

Calling `getBean()` multiple times returns a **new, distinct** instance each time.

```java
@Configuration
public class AppConfig {
    @Bean
    @Scope("prototype") // Explicitly set the scope
    public Desktop desktop() {
        return new Desktop();
    }
}
// Result: The constructor is called every time getBean() is invoked.
```

Using `@Scope("prototype")` is the direct Java-based equivalent of using `<bean scope="prototype">` in XML.

---

### 📝 Interview Q&A

**Q: In Java-based configuration, what is the default scope of a bean defined with the `@Bean` annotation?**
**A:** The default scope is **singleton**.

**Q: How do you change the scope of a bean to `prototype` when using Java configuration?**
**A:** You annotate the `@Bean` method with the `@Scope("prototype")` annotation.

**Q: Where is the `@Scope` annotation placed?**
**A:** It is placed directly on the method that is annotated with `@Bean`.

**Q: If a bean is annotated with `@Scope("prototype")` and you call `context.getBean()` on it five times, how many objects are created?**
**A:** Five distinct objects are created—one for each call to `getBean()`.

---

Excellent. This two-part video covers the most common and important aspects of dependency injection and ambiguity resolution in modern Java-based configuration.

### Part 1: How to Wire Dependencies in Java Config

When one of your beans (e.g., `Alien`) depends on another (e.g., a `Computer`), you need to wire them together.

**The Best Practice: Use Method Parameters**

The cleanest and most powerful way to inject a dependency into a `@Bean` method is to declare it as a parameter of that method.

Spring's IoC container is smart. When it sees it needs to create the `alien` bean, it will notice the `alien()` method requires a `Computer` object. It will then automatically look for a bean of type `Computer` in the container and pass it in as an argument.

```java
@Configuration
public class AppConfig {

    @Bean
    public Alien alien(Computer computer) { // <-- Spring will inject a Computer bean here
        Alien alien = new Alien();
        alien.setAge(25);
        alien.setCom(computer); // <-- Use the injected dependency
        return alien;
    }

    @Bean
    public Desktop desktop() {
        return new Desktop();
    }
}
```
In modern Spring, `@Autowired` on the parameter is optional; Spring infers the dependency automatically.

### Part 2: Resolving Ambiguity (`@Qualifier` vs. `@Primary`)

A problem arises when you have **multiple beans of the same type**. If you have both a `Desktop` bean and a `Laptop` bean, and Spring is asked to inject a `Computer`, it doesn't know which one to choose and will fail with a `NoUniqueBeanDefinitionException`.

Here are the two primary ways to solve this:

#### 1. `@Qualifier`

* **What it does**: Specifies exactly **which bean to use by its name**.
* **Where it's used**: At the **injection point** (on the method parameter).
* **When to use it**: When the *consumer* (the `alien` method) needs to decide which specific implementation it wants.

```java
@Bean
public Alien alien(@Qualifier("desktop") Computer computer) {
    // ... logic ...
    // This will now specifically inject the bean named "desktop"
}

@Bean // This bean's name is "laptop"
public Laptop laptop() { return new Laptop(); }

@Bean // This bean's name is "desktop"
public Desktop desktop() { return new Desktop(); }
```

#### 2. `@Primary`

* **What it does**: Marks one bean definition as the **default choice** if multiple candidates are available.
* **Where it's used**: On the **`@Bean` definition** itself.
* **When to use it**: When one of the *implementations* (`Laptop` or `Desktop`) should be considered the main or default option.

```java
@Bean
public Alien alien(Computer computer) {
    // ... logic ...
    // Spring will now inject the Laptop bean because it's primary.
}

@Bean
@Primary // Mark Laptop as the primary choice
public Laptop laptop() { return new Laptop(); }

@Bean
public Desktop desktop() { return new Desktop(); }
```

---

### 📝 Interview Q&A

**Q: In Java config, if bean `A` depends on bean `B`, what is the recommended way to inject `B` into `A`'s definition?**
**A:** The recommended way is to declare bean `B` as a parameter in the `@Bean` method for bean `A`. Spring will automatically find a qualifying bean `B` in the context and pass it in when creating bean `A`.

**Q: What happens in Java config if you try to autowire a dependency by type, but there are multiple beans of that type available?**
**A:** The application context will fail to start, throwing a `NoUniqueBeanDefinitionException`. Spring cannot decide which of the multiple candidates to inject, so it reports an error.

**Q: Explain the difference between `@Qualifier` and `@Primary` and when you would use each.**
**A:** Both solve autowiring ambiguity.
* `@Qualifier` is used at the **injection point** to select a bean by its specific **name**. You use it when the *consuming* bean needs to make the choice.
* `@Primary` is used on a **bean definition** to mark it as the **default** candidate. You use it when you want one of the *implementations* to be the preferred option for all injections.

**Q: If a bean is marked `@Primary` but the injection point uses a `@Qualifier` annotation to specify a different bean, which one wins?**
**A:** `@Qualifier` always wins. It is a more explicit and direct instruction, so it overrides the default behavior provided by `@Primary`.

---

Of course. This video introduces a major shift towards the modern, automated way of configuring Spring, which is essential for understanding Spring Boot. Here is the gist.

### Core Concept: Component Scanning

So far, you have been **explicitly** telling Spring what beans to create, either through `<bean>` tags in XML or `@Bean` methods in a Java `@Configuration` class.

**Component Scanning** flips this around. Instead of a central configuration file listing all the beans, you mark your individual classes as Spring components and then tell Spring where to find them.

This is a more decentralized and automated approach that dramatically reduces the amount of configuration code you need to write.

### The Key Annotations

This new approach relies on two main annotations:

1.  **`@Component`**
    * **What it is**: A class-level annotation. It's a generic "stereotype" annotation that you put on any class you want Spring to manage.
    * **What it means**: It's like putting a sign on your class that says, "Hey Spring, please create an instance of me and manage me as a bean."

2.  **`@ComponentScan`**
    * **What it is**: An annotation you place on your main `@Configuration` class.
    * **What it does**: It tells Spring to **scan** a specific package (and all its sub-packages) to find any classes marked with `@Component` (or other stereotype annotations like `@Service`, `@Repository`, etc.). For every component it finds, it will automatically register a bean in the container.

### How It Works: The New Workflow

1.  **Mark Your Classes**: Go to your individual classes (`Alien`, `Laptop`, `Desktop`) and add the `@Component` annotation to each one.

    ```java
    @Component
    public class Alien {
      // ...
    }

    @Component
    public class Desktop implements Computer {
      // ...
    }
    ```

2.  **Enable Scanning**: In your `AppConfig` class, remove all the `@Bean` methods and add `@ComponentScan`, telling it which base package to scan.

    ```java
    @Configuration
    @ComponentScan(basePackages = "com.telusko")
    public class AppConfig {
        // This class can now be empty!
        // Spring will find the components automatically.
    }
    ```

### The New Problem: Injection

The video ends on a critical point. After setting this up, the application fails with a `NullPointerException`.

* **Why?** Component scanning is great at **creating** the beans (`Alien`, `Laptop`, `Desktop` objects all exist in the container). However, it does **not** automatically handle **injecting** them into each other. The `Alien` bean was created, but its `Computer` dependency was never injected, so the `com` field remains `null`.

This sets the stage for the next crucial annotation: `@Autowired`.

---

### 📝 Interview Q&A

**Q: What is the difference between defining beans with `@Bean` vs. using `@Component`?**
**A:** `@Bean` is an explicit declaration in a `@Configuration` class. You are manually telling Spring how to create the bean. `@Component` is an implicit declaration on the class itself. You mark the class and let Spring's component-scanning mechanism find it and create a bean automatically.

**Q: If you put `@Component` on your classes, what else is required for Spring to create the beans?**
**A:** You must add the `@ComponentScan` annotation to a `@Configuration` class and specify the base package(s) where your components are located. Without `@ComponentScan`, Spring won't know where to look for them.

**Q: What is a "stereotype" annotation in Spring?**
**A:** It's an annotation that marks a class for a specific role. `@Component` is the generic stereotype. More specific ones like `@Service` (for business logic), `@Repository` (for data access), and `@Controller` (for web layers) all derive from `@Component` and are treated the same way by component scanning.

**Q: After switching to `@ComponentScan`, your beans are created, but you get a `NullPointerException` when one bean tries to call a method on a dependency. What's the likely cause?**
**A:** The likely cause is that while the beans were created, they were not injected into each other. Component scanning handles bean discovery and instantiation, but you still need to use an annotation like `@Autowired` to handle the dependency injection (wiring).

---
Of course. This video provides the solution to the dependency injection problem that arose from using component scanning. Here is the essential gist.

### Core Concept: `@Autowired` for Injection

The previous lesson showed that `@ComponentScan` is great at *creating* beans, but it doesn't automatically *inject* them into each other. The solution is the **`@Autowired`** annotation.

* **What it does**: `@Autowired` tells the Spring container to find a suitable bean from the container and automatically inject it into the annotated field, constructor, or method.
* **Default Behavior**: By default, `@Autowired` works **by type**. It looks for a bean in the container that matches the data type of the field it's annotating.

### The Ambiguity Problem (Revisited)

When you autowire by type (e.g., `@Autowired private Computer com;`), a problem occurs if there are multiple beans that implement the `Computer` interface (`Laptop` and `Desktop`). Spring doesn't know which one to inject and will throw a `NoUniqueBeanDefinitionException`.

### Solutions to Autowiring Ambiguity

There are two primary ways to solve this when using component scanning:

#### 1. `@Qualifier("beanName")`

* **What it does**: Used alongside `@Autowired`, `@Qualifier` lets you specify exactly **which bean to inject by its name**.
* **How it Works**: You provide the name of the desired bean as a string. Spring will then look for the bean with that specific name, resolving the ambiguity.
* **Default Bean Names**: When using `@Component`, the default name of the bean is the class name in camelCase (e.g., `Laptop` class becomes bean `"laptop"`). You can override this by providing a name, like `@Component("com2")`.

```java
@Component
public class Alien {
    @Autowired
    @Qualifier("desktop") // Tells Spring: "Inject the bean named 'desktop'"
    private Computer com;
    //...
}
```

#### 2. `@Primary`

* **What it does**: Marks a specific component as the default choice.
* **How it Works**: You add the `@Primary` annotation to one of your component classes. If Spring finds multiple candidates for an autowired dependency, it will choose the one marked as `@Primary`.

```java
@Component
@Primary // If there's any confusion, choose Laptop by default.
public class Laptop implements Computer {
    //...
}
```

### The Three Types of Injection

You can place the `@Autowired` annotation in three places, each representing a different injection style:

1.  **Field Injection (Most Common)**: Directly on the private field. It's concise but can make testing more difficult.
    ```java
    @Autowired
    private Computer com;
    ```
2.  **Setter Injection (Good Practice)**: On the setter method. This is more flexible and easier to test.
    ```java
    @Autowired
    public void setCom(Computer com) {
        this.com = com;
    }
    ```
3.  **Constructor Injection (Often Recommended)**: On the class constructor. This ensures the bean has all its required dependencies as soon as it's created.
    ```java
    @Autowired
    public Alien(Computer com) {
        this.com = com;
    }
    ```

---

### 📝 Interview Q&A

**Q: What is the purpose of the `@Autowired` annotation?**
**A:** `@Autowired` is used to enable Spring's automatic dependency injection. It tells the Spring container to find a matching bean from the context and inject it into the annotated field, setter method, or constructor.

**Q: By default, does `@Autowired` inject dependencies by name or by type?**
**A:** By default, it injects **by type**.

**Q: You are autowiring an interface, but there are two implementation beans available. What are two common ways to resolve this ambiguity?**
**A:**
1.  Use the `@Qualifier("beanName")` annotation along with `@Autowired` to specify which bean to inject by its name.
2.  Add the `@Primary` annotation to one of the implementation classes to mark it as the default choice.

**Q: When using `@Component` on a class named `MySpecialService`, what will the default bean name be?**
**A:** The default bean name will be the camel-cased version of the class name: `"mySpecialService"`.

**Q: What are the three types of dependency injection, and where do you place the `@Autowired` annotation for each?**
**A:** 1. **Field Injection**: On the instance variable. 2. **Setter Injection**: On the setter method. 3. **Constructor Injection**: On the constructor.

---
Excellent. This video clarifies the second method for resolving autowiring ambiguity and, most importantly, establishes the priority between the two solutions. Here's the gist.

### Core Concept: `@Primary` Annotation

When you have multiple beans of the same type (like `Laptop` and `Desktop` both implementing `Computer`), `@Autowired` fails due to ambiguity. One way to solve this is with the **`@Primary`** annotation.

* **What it does**: `@Primary` marks one of the component classes as the "default" or "primary" candidate for injection.
* **Where it's used**: It's a class-level annotation, placed on the component you want to be the default choice.
* **How it works**: When Spring finds multiple beans that match the required type for an `@Autowired` field, it will inject the one that is marked with `@Primary`.

```java
@Component
@Primary // Now, Desktop is the default Computer
public class Desktop implements Computer {
    // ...
}

@Component
public class Laptop implements Computer {
    // ...
}
```

Now, a simple `@Autowired` on a `Computer` field will automatically receive the `Desktop` instance without any error.

### The Precedence Rule: `@Qualifier` vs. `@Primary`

This is the most critical takeaway. What happens if you use both annotations?

* `@Primary` is a general suggestion from the bean *provider*. It says, "If you don't have a specific preference, use me."
* `@Qualifier` is a specific request from the bean *consumer*. It says, "I don't care about the default; I want *this specific one*."

**The Rule: `@Qualifier` always has higher precedence than `@Primary`.**

A specific request at the injection point will always override the general default.

#### Example

```java
// Injection point in Alien class
@Autowired
@Qualifier("laptop") // This is a specific request for the "laptop" bean
private Computer com;

// ---------------------------------------------------

// Component classes
@Component
@Primary // This is a general default
public class Desktop implements Computer { /* ... */ }

@Component // This bean's name is "laptop"
public class Laptop implements Computer { /* ... */ }
```

**Result**: The `Laptop` bean will be injected, because the specific `@Qualifier` request overrides the `@Primary` default.

---

### 📝 Interview Q&A

**Q: What is the purpose of the `@Primary` annotation?**
**A:** `@Primary` is used to resolve autowiring ambiguity. When multiple beans of the same type exist, `@Primary` indicates which bean should be injected by default if no other specific qualifier is used.

**Q: Where do you place the `@Primary` annotation?**
**A:** You place it at the class level, directly on the `@Component` (or `@Service`, `@Repository`, etc.) that you want to designate as the primary choice.

**Q: You have two `DataSource` beans. One is marked `@Primary` for production, and the other is for testing. At an injection point, you use `@Qualifier` to ask for the testing `DataSource`. Which one gets injected?**
**A:** The testing `DataSource` will be injected. The `@Qualifier` annotation is a more specific instruction and always has higher precedence, overriding the default behavior set by `@Primary`.

**Q: When would you choose to use `@Primary` versus `@Qualifier`?**
**A:** You use `@Primary` when you have a clear, system-wide default implementation for an interface that should be used in most cases. You use `@Qualifier` for the exceptional cases where you need to inject a specific, non-default implementation.

---

Of course. This video covers two important annotations for fine-tuning your components. Here's the gist.

### `@Scope` Annotation with Components

You can control the scope of a bean discovered through component scanning by adding the **`@Scope`** annotation directly to the class. This works exactly like it does on `@Bean` methods.

* **Placement**: On the class, alongside `@Component`.
* **Usage**: Provide the desired scope as a string (`"singleton"` or `"prototype"`).

```java
@Component
@Scope("prototype") // This Alien bean will now be a prototype
public class Alien {
    // ...
}
```

---

### `@Value` Annotation for Property Injection

When using component scanning, you lose the ability to manually set properties like `age` in a central configuration class. The **`@Value`** annotation solves this by allowing you to inject simple, literal values directly into a bean's fields.

* **Placement**: Typically on the field you want to populate.
* **Usage**: You provide the value you want to inject as a string. Spring will automatically convert it to the correct type (like `int`).

```java
@Component
public class Alien {
    @Value("25") // Injects the value 25 into the age field
    private int age;
    // ...
}
```

#### Why use `@Value` over direct initialization?

While `@Value("25")` looks similar to `private int age = 25;`, `@Value` is far more powerful. Its primary purpose is to inject values from **external sources**, like a `.properties` file. This allows you to change configuration values without recompiling your code.

For example, you could later change the code to `@Value("${alien.default.age}")` to read the age from a property file, making your application much more flexible.

---

### 📝 Interview Q&A

**Q: When using `@Component` on a class, how do you change its scope from singleton to prototype?**
**A:** You add the `@Scope("prototype")` annotation directly to the class definition.

**Q: What is the purpose of the `@Value` annotation?**
**A:** The `@Value` annotation is used to inject simple literal values (like Strings, numbers, or booleans) into a bean's fields. Its main strength is injecting values from external configuration sources, like property files.

**Q: What is the key advantage of using `@Value` over directly initializing a field in your class?**
**A:** Direct initialization hardcodes the value into your compiled code. `@Value` allows you to **externalize configuration**, meaning the value can be read from a property file outside your application. This lets you change the configuration without modifying and recompiling the Java code.

**Q: Can you use `@Value` to inject a reference to another bean?**
**A:** No. `@Value` is for simple property/literal values. To inject a reference to another bean, you must use `@Autowired`.

---

