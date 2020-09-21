# Introduction to Project Lombok
Repetitive code – What is the most important to every project? And what is the necessary useless thing?

## What is Project Lombok?
Lombok is a tool that helps you to eliminate needless code. It is a java library that automatically plugs into your editor and build tools, spicing up your java.
Never write another getter or equals method again, with one annotation your class has a fully featured builder, automate your logging variables, and much more.

## How it works
(need more explanetion) The way it works is by plugging into your build process and autogenerating Java bytecode into your .class files as per a number of project annotations you introduce in your code

# Usage

##	Getters/Setters, Constructors
*	IDE’s can generate Getters/Setters, Constructors, but they stay in your source code (git).	
*	Spring boot necessary constructors, getters and setters. 
Let's consider this class we want to use as a JPA entity as an example:

````

@Entity
public class User implements Serializable {
 
    private @Id Long id; // will be set when persisting
 
    private String firstName;
    private String lastName;
    private int age;
 
    public User() {
    }
 
    public User(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }
 
    // getters and setters: ~30 extra lines of code
}

````

This is a rather simple class, but still consider if we added the extra code for getters and setters we'd end up with a definition where we would have more boilerplate zero-value code than the relevant business information: a User has first and last names, and age.

Let us now Lombok-ize this class:

````
@Entity
@Getter @Setter @NoArgsConstructor // <--- THIS is it
public class User implements Serializable {
 
    private @Id Long id; // will be set when persisting
 
    private String firstName;
    private String lastName;
    private int age;
 
    public User(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }
}
````

By adding the @Getter and @Setter annotations we told Lombok to, well, generate these for all the fields of the class. @NoArgsConstructor will lead to an empty constructor generation.
Note this is the whole class code.
If you further add attributes (properties) to your User class, the same will happen: you applied the annotations to the type itself so they will mind all fields by default.
What if you wanted to refine the visibility of some properties? For example, I like to keep my entities' id field modifiers package or protected visible because they are expected to be read but not explicitly set by application code. Just use a finer grained @Setter for this particular field:

````
private @Id @Setter(AccessLevel.PROTECTED) Long id;
````

## Value Classes/DTO's
There are many situations in which we want to define a data type with the sole purpose of representing complex “values” or as “Data Transfer Objects”, most of the time in the form of immutable data structures we build once and never want to change.
We design a class to represent a successful login operation. We want all fields to be non-null and objects be immutable so that we can thread-safely access its properties:
````
public class LoginResult {
 
    private final Instant loginTs;
 
    private final String authToken;
    private final Duration tokenValidity;
    
    private final URL tokenRefreshUrl;
 
    // constructor taking every field and checking nulls
 
    // read-only accessor, not necessarily as get*() form
}
````
````
@RequiredArgsConstructor
@Accessors(fluent = true) @Getter
public class LoginResult {
 
    private final @NonNull Instant loginTs;
 
    private final @NonNull String authToken;
    private final @NonNull Duration tokenValidity;
    
    private final @NonNull URL tokenRefreshUrl;
 
}
````
Just add the @RequiredArgsConstructor annotation and you'd get a constructor for all the final fields int the class, just as you declared them. Adding @NonNull to attributes makes our constructor check for nullability and throw NullPointerExceptions accordingly. This would also happen if the fields were non-final and we added @Setter for them.
Don't you want boring old get*() form for your properties? Because we added @Accessors(fluent=true) in this example “getters” would have the same method name as the properties: getAuthToken() simply becomes authToken().
This “fluent” form would apply to non-final fields for attribute setters and as well allow for chained calls:
````
// Imagine fields were no longer final now
return new LoginResult()
  .loginTs(Instant.now())
  .authToken("asdasd")
  . // and so on
````
## Core Java Boilerplate
Another situation in which we end up writing code we need to maintain is when generating toString(), equals() and hashCode() methods. IDEs try to help with templates for autogenerating these in terms of our class attributes.

We can automate this by means of other Lombok class-level annotations:

* @ToString: will generate a toString() method including all class attributes. No need to write one ourselves and maintain it as we enrich our data model.
* @EqualsAndHashCode: will generate both equals() and hashCode() methods by default considering all relevant fields, and according to very well though semantics.
These generators ship very handy configuration options. For example, if your annotated classes take part of a hierarchy you can just use the callSuper=true parameter and parent results will be considered when generating the method's code.
More on this: say we had our User JPA entity example include a reference to events associated to this user:
````
@OneToMany(mappedBy = "user")
private List<UserEvent> events;
````
We wouldn't like to have the whole list of events dumped whenever we call the toString() method of our User, just because we used the @ToString annotation. No problem: just parameterize it like this: 
````
@ToString(exclude = {“events”})
````
and that won't happen. This is also helpful to avoid circular references if, for example, UserEvents had a reference to a User.
For the LoginResult example, we may want to define equality and hash code calculation just in terms of the token itself and not the other final attributes in our class. Then, simply write something like 
````
@EqualsAndHashCode(of = {“authToken”}).
````
Bonus: if you liked the features from the annotations we've reviewed so far you may want to examine @Data and @Value annotations as they behave as if a set of them had been applied to our classes. After all, these discussed usages are very commonly put together in many cases.

## The Builder Pattern
````
@Builder
public class ApiClientConfiguration {
 
    // ... everything else remains the same
 
}
````
Leaving the class definition above as such (no declare constructors nor setters + @Builder) we can end up using it as:
````
ApiClientConfiguration config = 
    ApiClientConfiguration.builder()
        .host("api.server.com")
        .port(443)
        .useHttps(true)
        .connectTimeout(15_000L)
        .readTimeout(5_000L)
        .username("myusername")
        .password("secret")
    .build();
````
## Checked Exceptions Burden
````
public String resourceAsString() {
    try (InputStream is = this.getClass().getResourceAsStream("sure_in_my_jar.txt")) {
        BufferedReader br = new BufferedReader(new InputStreamReader(is, "UTF-8"));
        return br.lines().collect(Collectors.joining("\n"));
    } catch (IOException | UnsupportedCharsetException ex) {
        // If this ever happens, then its a bug.
        throw new RuntimeException(ex); <--- encapsulate into a Runtime ex.
    }
}
````
If you want to avoid this code patterns because the compiler won't be otherwise happy (and, after all, you know the checked errors cannot happen), use the aptly named @SneakyThrows:
````
@SneakyThrows
public String resourceAsString() {
    try (InputStream is = this.getClass().getResourceAsStream("sure_in_my_jar.txt")) {
        BufferedReader br = new BufferedReader(new InputStreamReader(is, "UTF-8"));
        return br.lines().collect(Collectors.joining("\n"));
    } 
}
````
## Ensure Your Resources Are Released
````
@Cleanup InputStream is = this.getClass().getResourceAsStream("res.txt");
@Cleanup("dispose") JFrame mainFrame = new JFrame("Main Window");
````
## Annotate Your Class to Get a Logger
````
public class ApiClientConfiguration {
 
    private static Logger LOG = LoggerFactory.getLogger(ApiClientConfiguration.class);
 
    // LOG.debug(), LOG.info(), ...
 
}
````
````
@Slf4j // or: @Log @CommonsLog @Log4j @Log4j2 @XSlf4j
public class ApiClientConfiguration {
 
    // log.debug(), log.info(), ...
 
}
````

