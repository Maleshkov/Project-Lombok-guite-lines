# Project-Lombok-guite-lines
Project Lombok guite lines

1. Story about repetitive code – What is the most important to every project? And what is the necessary useless think?
2. Maven introduction - build automation tool
3. Introduction of Project Lombok
* What is Project Lombok? – tool that helps you to eliminate needless code?
*	How it works – The way it works is by plugging into your build process and autogenerating Java bytecode into your .class files as per a number of project annotations you introduce in your code
4.	Getters/Setters, Constructors
*	What is the most repetitive code in your project? 
*	Spring boot necessary constructors, getters and setters. 
*	IDE’s can generate Getters/Setters, Constructors, but they stay in your source code (git).	


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
