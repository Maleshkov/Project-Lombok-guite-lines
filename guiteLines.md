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


