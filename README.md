# learn-design-patterns
Some notes of "Design Patterns in Java" by Dmitri Nesteruk in Udemy

## Index

### SOLID Design Principles:
- Single Responsibility Principle
- Open-Closed Principle
- Liskov Substitution Principle
- Interface Segregation Principle
- Dependency Inversion Principle

### Creational Design Patterns: Deal with the creation (construction) of objects
- Builder
- Factories (Factory Method and Abstract Factory)
- Prototype
- Singleton

### Structrural Design Patterns: Deal with the structure (e.g., class members)
- Adapter
- Bridge
- Composite
- Decorator
- Façade
- Flyweight
- Proxy

### Behavioral Design Patterns: Deal with different problems (no central theme)
- Chain of Responsibility
- Command
- Interpreter
- Iterator
- Mediator
- Memento
- Null Object
- Observer
- State
- Strategy
- Template Method
- Visitor

## SOLID Design Principles

### Single Responsibility Principle (SRP)
- A class should only have one reason to change.
- *Separation of concerns* - different classes handling different, independent tasks/problems.

<details>
<summary>How to break it?</summary>

```java
import java.io.File;
import java.io.PrintStream;
import java.net.URL;
import java.util.ArrayList;
import java.util.List;

class Journal {
    private final List<String> entries = new ArrayList<>();

    private static int count = 0;

    public void addEntry(String text) {
        entries.add("" + (++count) + ": " + text);
    }

    public void removeEntry(int index) {
        entries.remove(index);
    }

    @Override
    public String toString() {
        return String.join(System.lineSeparator(), entries);
    }

    // here we break SRP
    public void save(String filename) throws Exception {
        try (PrintStream out = new PrintStream(filename)) {
            out.println(toString());
        }
    }

    public void load(String filename) {}
    
    public void load(URL url) {}
}
```

</details>

<details>
<summary>How to fix it?</summary>

```java
import java.io.File;
import java.io.PrintStream;
import java.net.URL;
import java.util.ArrayList;
import java.util.List;

class Journal {
    private final List<String> entries = new ArrayList<>();

    private static int count = 0;

    public void addEntry(String text) {
        entries.add("" + (++count) + ": " + text);
    }

    public void removeEntry(int index) {
        entries.remove(index);
    }

    @Override
    public String toString() {
        return String.join(System.lineSeparator(), entries);
    }
}

// handles the responsibility of persisting objects
class Persistence {
    public void saveToFile(Journal journal, String filename, boolean overwrite) throws Exception {
        if (overwrite || new File(filename).exists()) {
            try (PrintStream out = new PrintStream(filename)) {
                out.println(journal.toString());
            }
        }
    }

    public void load(Journal journal, String filename) {}

    public void load(Journal journal, URL url) {}
}
```

</details>

### Open-Closed Principle (OCP)
- Classes should be open for extension but closed for modification.

<details>
<summary>How to break it?</summary>

```java
import java.util.List;
import java.util.stream.Stream;

enum Color {
    RED, GREEN, BLUE
}

enum Size {
    SMALL, MEDIUM, LARGE, HUGE
}

class Product {
    public String name;
    public Color color;
    public Size size;

    public Product(String name, Color color, Size size) {
        this.name = name;
        this.color = color;
        this.size = size;
    }
}

class ProductFilter {
    public Stream<Product> filterByColor(List<Product> products, Color color) {
        return products.stream().filter(p -> p.color == color);
    }

    public Stream<Product> filterBySize(List<Product> products, Size size) {
        return products.stream().filter(p -> p.size == size);
    }

    public Stream<Product> filterBySizeAndColor(List<Product> products, Size size, Color color) {
        return products.stream().filter(p -> p.size == size && p.color == color);
    }

    // ...

    // State space explosion: 2^n-1.
    // Suppose we have 3 criteria, we need 7 methods! If we add more criteria, we need more!
}
```

</details>

<details>
<summary>How to fix it?</summary>

```java
import java.util.List;
import java.util.stream.Stream;

enum Color {
    RED, GREEN, BLUE
}

enum Size {
    SMALL, MEDIUM, LARGE, HUGE
}

class Product {
    public String name;
    public Color color;
    public Size size;

    public Product(String name, Color color, Size size) {
        this.name = name;
        this.color = color;
        this.size = size;
    }
}

// we introduce two new interfaces that are open for extension
interface Specification<T> {
    boolean isSatisfied(T item);
}

interface Filter<T> {
    Stream<T> filter(List<T> items, Specification<T> spec);
}

class ColorSpecification implements Specification<Product> {
    private Color color;

    public ColorSpecification(Color color) {
        this.color = color;
    }

    @Override
    public boolean isSatisfied(Product p) {
        return p.color == color;
    }
}

class SizeSpecification implements Specification<Product> {
    private Size size;

    public SizeSpecification(Size size) {
        this.size = size;
    }

    @Override
    public boolean isSatisfied(Product p) {
        return p.size == size;
    }
}

class AndSpecification<T> implements Specification<T> {
    private Specification<T> first, second;

    public AndSpecification(Specification<T> first, Specification<T> second) {
        this.first = first;
        this.second = second;
    }

    @Override
    public boolean isSatisfied(T item) {
        return first.isSatisfied(item) && second.isSatisfied(item);
    }

}

class BetterProductFilter implements Filter<Product> {
    @Override
    public Stream<Product> filter(List<Product> items, Specification<Product> spec) {
        return items.stream().filter(p -> spec.isSatisfied(p));
    }
}
```

</details>

### Liskov Substitution Principle (LSP)
- You should be able to substitute a base type for a subtype.

<details>
<summary>How to break it?</summary>

```java
class Rectangle {
    protected int width, height;

    public Rectangle() {}

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    public int getWidth() {
        return width;
    }

    public void setWidth(int width) {
        this.width = width;
    }

    public int getHeight() {
        return height;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width*height;
    }

    @Override
    public String toString() {
        return "Rectangle{width=" + width + ", height=" + height + '}';
    }
}

class Square extends Rectangle {
    public Square() {}

    public Square(int side) {
        width = height = side;
    }

    @Override
    public void setWidth(int width) {
        super.setWidth(width);
        super.setHeight(width);
    }

    @Override
    public void setHeight(int height) {
        super.setHeight(height);
        super.setWidth(height);
    }
}

class Demo {
    static void useIt(Rectangle r) {
        int width = r.getWidth();
        r.setHeight(10);
        System.out.println("Expected area of " + (width*10) + ", got " + r.getArea() + ".");
    }

    public static void main(String[] args) {
        Rectangle rc = new Rectangle(2, 3);
        useIt(rc);  // "Expected area of 20, got 20."

        Rectangle sq = new Square(2);
        useIt(sq);  // "Expected area of 20, got 100."
    }
}
```

</details>

<details>
<summary>How to fix it?</summary>

```java
class Rectangle {
    protected int width, height;

    public Rectangle() {}

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    public int getWidth() {
        return width;
    }

    public void setWidth(int width) {
        this.width = width;
    }

    public int getHeight() {
        return height;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width*height;
    }

    @Override
    public String toString() {
        return "Rectangle{width=" + width + ", height=" + height + '}';
    }
}

class RectangleFactory {
    public static Rectangle newRectangle(int width, int height) {
        return new Rectangle(width, height);
    }

    public static Rectangle newSquare(int side) {
        return new Rectangle(side, side);
    }
}
```

</details>

### Interface Segregation Principle (ISP)
- Don't put too much into an interface; Split into separate interfaces.
- *YAGNI* - You Ain't Going to Need It.

<details>
<summary>How to break it?</summary>

```java
class Document {}

interface Machine {
    void print(Document d);
    void fax(Document d) throws Exception;
    void scan(Document d) throws Exception;
}

// ok if you need a multifunction machine
class MultiFunctionPrinter implements Machine {
    public void print(Document d) {
        //
    }

    public void fax(Document d) {
        //
    }

    public void scan(Document d) {
        //
    }
}

// not ok if old-fashioned printer has only print function
class OldFashionedPrinter implements Machine
{
    public void print(Document d) {
        // yep
    }

    public void fax(Document d) throws Exception {
        throw new Exception();
    }

    public void scan(Document d) throws Exception {
        throw new Exception();
    }
}
```

</details>

<details>
<summary>How to fix it?</summary>

```java
class Document {}

interface Printer {
    void print(Document d) throws Exception;
}

interface Scanner {
    void scan(Document d) throws Exception;
}

interface Faxer {
    void fax(Document d) throws Exception;
}

class JustAPrinter implements Printer {
    @Override
    public void print(Document d) throws Exception {
    }
}

class Photocopier implements Printer, Scanner {
    @Override
    public void print(Document d) throws Exception {
    }

    @Override
    public void scan(Document d) throws Exception {
    }
}

interface MultiFunctionDevice extends Printer, Scanner {}

class MultiFunctionMachine implements MultiFunctionDevice {
    // compose this out of several modules
    private Printer printer;
    private Scanner scanner;

    public MultiFunctionMachine(Printer printer, Scanner scanner) {
        this.printer = printer;
        this.scanner = scanner;
    }

    public void print(Document d) throws Exception {
        printer.print(d);
    }

    public void scan(Document d) throws Exception {
        scanner.scan(d);
    }
}
```

</details>

### Dependency Inversion Principle (DIP)
- High-level modules should not depend on low-level modules. Both should depend on abstraction.
- Abstractions should not depend on details. Details should depend on abstractions.

<details>
<summary>How to break it?</summary>

```java
import java.util.ArrayList;
import java.util.List;

class Triplet<T0, T1, T2> {
    private T0 t0;
    private T1 t1;
    private T2 t2;

    public Triplet(T0 t0, T1 t1, T2 t2) {
        this.t0 = t0;
        this.t1 = t1;
        this.t2 = t2;
    }

    public T0 getValue0() { return t0; }
    public T1 getValue1() { return t1; }
    public T2 getValue2() { return t2; }
}


enum Relationship {
    PARENT,
    CHILD,
    SIBLING
}

class Person {
    public String name;

    public Person(String name) {
        this.name = name;
    }
}

class Relationships {  // low-level (related to data storage, no business logic)
    private List<Triplet<Person, Relationship, Person>> r;

    public Relationships() {
        this.r = new ArrayList<>();
    }

    // major problem: exposed internal storage implementation as a public getter for everyone to access
    public List<Triplet<Person, Relationship, Person>> getRelationships() {
        return r;
    }

    public void addParentAndChild(Person parent, Person child) {
        r.add(new Triplet<>(parent, Relationship.PARENT, child));
        r.add(new Triplet<>(child, Relationship.CHILD, parent));
    }
}

class Research {  // high-level
    public Research() {}

    public void doResearch(Relationships relationships) {
        List<Triplet<Person, Relationship, Person>> r = relationships.getRelationships();
        r.stream()
         .filter(x -> x.getValue0().name.equals("John") && x.getValue1() == Relationship.PARENT)
         .forEach(ch -> System.out.println("John has a child called " + ch.getValue2().name));
    }
}

class Demo {
    public static void main(String[] args) {
        Person parent = new Person("John");
        Person child1 = new Person("Chris");
        Person child2 = new Person("Matt");

        Relationships relationships = new Relationships();
        relationships.addParentAndChild(parent, child1);
        relationships.addParentAndChild(parent, child2);

        Research research = new Research();
        research.doResearch(relationships);
    }
}
```

</details>

<details>
<summary>How to fix it?</summary>

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Objects;
import java.util.stream.Collectors;

class Triplet<T0, T1, T2> {
    private T0 t0;
    private T1 t1;
    private T2 t2;

    public Triplet(T0 t0, T1 t1, T2 t2) {
        this.t0 = t0;
        this.t1 = t1;
        this.t2 = t2;
    }

    public T0 getValue0() { return t0; }
    public T1 getValue1() { return t1; }
    public T2 getValue2() { return t2; }
}


enum Relationship {
    PARENT,
    CHILD,
    SIBLING
}

class Person {
    public String name;

    public Person(String name) {
        this.name = name;
    }
}

interface RelationshipBrowser {
    List<Person> findAllChildrenOf(String name);
}

class Relationships implements RelationshipBrowser {  // low-level (related to data storage, no business logic)
    private List<Triplet<Person, Relationship, Person>> r;

    public Relationships() {
        this.r = new ArrayList<>();
    }

    public void addParentAndChild(Person parent, Person child) {
        r.add(new Triplet<>(parent, Relationship.PARENT, child));
        r.add(new Triplet<>(child, Relationship.CHILD, parent));
    }

    @Override
    public List<Person> findAllChildrenOf(String name) {
        return r.stream()
                .filter(x -> Objects.equals(x.getValue0().name, name) && x.getValue1() == Relationship.PARENT)
                .map(Triplet::getValue2)
                .collect(Collectors.toList());
    }
}

class Research {  // high-level
    public Research() {}

    public void doResearch(RelationshipBrowser browser) {
        List<Person> children = browser.findAllChildrenOf("John");
        for (Person child : children) {
            System.out.println("John has a child called " + child.name);
        }
    }
}

class Demo {
    public static void main(String[] args) {
        Person parent = new Person("John");
        Person child1 = new Person("Chris");
        Person child2 = new Person("Matt");

        Relationships relationships = new Relationships();
        relationships.addParentAndChild(parent, child1);
        relationships.addParentAndChild(parent, child2);

        Research research = new Research();
        research.doResearch(relationships);
    }
}
```

</details>

## Builder
- When piecewise object construction is complicated, you can provide an API fo doing it succinctly.
- Builder is a separate component for building an object.
- You can either give builder a constructor or return it via a static function.
- To make builder fluent, return `this`.
- Different facets of an object can be built with different builders working in tandem via a base class.

<details>
<summary>Fluent Builder</summary>

```java
import java.util.*;

class MyField {
    private String name, type;

    public MyField(String name, String type) {
        this.name = name;
        this.type = type;
    }

    @Override
    public String toString() {
        return String.format("public %s %s;", type, name);
    }
}

class MyClass {
    private String className;
    private List<MyField> fields;

    public MyClass(String className) {
        this.className = className;
        this.fields = new ArrayList<>();
    }

    public void addField(MyField field) {
        fields.add(field);
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("public class ").append(className).append(" ").append("{").append(System.lineSeparator());
        for (MyField field : fields) {
            sb.append("    ").append(field).append(System.lineSeparator());
        }
        sb.append("}").append(System.lineSeparator());
        return sb.toString();
    }
}

class CodeBuilder {
    private MyClass myClass;

    public CodeBuilder(String className) {
        myClass = new MyClass(className);
    }

    public CodeBuilder addField(String name, String type) {  // fluent
        myClass.addField(new MyField(name, type));
        return this;
    }

    @Override
    public String toString() {
        return myClass.toString();
    }
}

class FluentBuilderDemo {
    public static void main(String[] args) {
        CodeBuilder cb = new CodeBuilder("Person")
                .addField("name", "String")
                .addField("age", "int");
        System.out.println(cb);
    }
}
```

</details>

<details>
<summary>Fluent Builder Inheritance with Recursive Generics</summary>

```java
class Person {
    public String name;
    public String position;

    @Override
    public String toString() {
        return "Person{name='" + name + "', position='" + position + "'}";
    }
}

class PersonBuilder<SELF extends PersonBuilder<SELF>> {
    protected Person person;

    public PersonBuilder() {
         person = new Person();
    }

    public SELF withName(String name) {
        person.name = name;
        // critical to return SELF here
        return self();
    }

    protected SELF self() {
        // unchecked cast, but actually safe
        // proof: try sticking a non-PersonBuilder as SELF parameter, it won't work!
        return (SELF) this;
    }

    public Person build() {
        return person;
    }
}

class EmployeeBuilder extends PersonBuilder<EmployeeBuilder> {
    public EmployeeBuilder worksAs(String position) {
        person.position = position;
        return self();
    }

    @Override
    protected EmployeeBuilder self() {
        return this;
    }
}

class RecursiveGenericsDemo {
    public static void main(String[] args) {
        PersonBuilder eb = new EmployeeBuilder()
                .withName("Haichao")
                .worksAs("Software Engineer");
        System.out.println(eb.build());
    }
}
```

</details>

<details>
<summary>Faceted Builder</summary>

```java
class Person {
    // address
    public String streetAddress, postcode, city;

    // employment
    public String companyName, position;
    public int annualIncome;

    @Override
    public String toString() {
        return "Person{" +
                "streetAddress='" + streetAddress + '\'' +
                ", postcode='" + postcode + '\'' +
                ", city='" + city + '\'' +
                ", companyName='" + companyName + '\'' +
                ", position='" + position + '\'' +
                ", annualIncome=" + annualIncome +
                '}';
    }
}

class PersonBuilder {
    protected Person person;

    public PersonBuilder() {
        person = new Person();
    }

    public PersonAddressBuilder lives() {
        return new PersonAddressBuilder(person);
    }

    public PersonJobBuilder works() {
        return new PersonJobBuilder(person);
    }

    public Person build() {
        return person;
    }
}

class PersonAddressBuilder extends PersonBuilder {
    public PersonAddressBuilder(Person person) {
        this.person = person;
    }

    public PersonAddressBuilder at(String streetAddress) {
        person.streetAddress = streetAddress;
        return this;
    }

    public PersonAddressBuilder withPostcode(String postcode) {
        person.postcode = postcode;
        return this;
    }

    public PersonAddressBuilder in(String city) {
        person.city = city;
        return this;
    }
}

class PersonJobBuilder extends PersonBuilder {
    public PersonJobBuilder(Person person) {
        this.person = person;
    }

    public PersonJobBuilder at(String companyName) {
        person.companyName = companyName;
        return this;
    }

    public PersonJobBuilder asA(String position) {
        person.position = position;
        return this;
    }

    public PersonJobBuilder earning(int annualIncome) {
        person.annualIncome = annualIncome;
        return this;
    }
}

class BuilderFacetsDemo {
    public static void main(String[] args) {
        PersonBuilder pb = new PersonBuilder();
        Person person = pb
                .lives()
                .at("123 London Road")
                .in("London")
                .withPostcode("SW12BC")
                .works()
                .at("Fabrikam")
                .asA("Engineer")
                .earning(123000)
                .build();
        System.out.println(person);
    }
}
```

</details>

### Java Recursive Generics
Refer to https://www.sitepoint.com/self-types-with-javas-generics/

## Factory
- Factory is a component responsible solely for the wholesale (not piecewise) creation of objects.
- A factory method is a static method that creates objects.
- A factory class can take care of object creation.
- A factory can be external or reside inside the object as an inner class.
- Hierarchies of factories can be used to create related objects.

<details>
<summary>Factory Method</summary>

```java
enum CoordinateSystem {
    CARTESIAN,
    POLAR
}

class Point {
    private double x, y;

    public Point(double a, double b, CoordinateSystem cs) {  // constructor can become ugly
        switch (cs) {
            case CARTESIAN:
                this.x = a;
                this.y = b;
                break;
            case POLAR:
                this.x = a * Math.cos(b);
                this.y = a * Math.sin(b);
                break;
        }
    }

    private Point(double x, double y) {  // private constructor
        this.x = x;
        this.y = y;
    }

    // factory methods
    public static Point newCartesianPoint(double x, double y) {
        return new Point(x,y);
    }

    public static Point newPolarPoint(double rho, double theta) {
        return new Point(rho * Math.cos(theta), rho * Math.sin(theta));
    }
}
```

</details>

<details>
<summary>Factory Class</summary>

```java
class Point {
    private double x, y;

    protected Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public static class Factory {  // reside inside the object as an inner class
        public static Point newCartesianPoint(double x, double y) {
            return new Point(x,y);
        }

        public static Point newPolarPoint(double rho, double theta) {
            return new Point(rho * Math.cos(theta), rho * Math.sin(theta));
        }
    }
}

class PointFactory {  // external
    public static Point newCartesianPoint(double x, double y) {
        return new Point(x,y);
    }

    public static Point newPolarPoint(double rho, double theta) {
        return new Point(rho * Math.cos(theta), rho * Math.sin(theta));
    }
}
```

</details>

<details>
<summary>Abstract Factory</summary>

```java
import javafx.util.Pair;
import org.reflections.Reflections;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.lang.reflect.Type;
import java.util.*;

interface HotDrink {
    void consume();
}

class Tea implements HotDrink {
    @Override
    public void consume() {
        System.out.println("This tea is nice but I'd prefer it with milk.");
    }
}

class Coffee implements HotDrink {
    @Override
    public void consume() {
        System.out.println("This coffee is delicious.");
    }
}

interface HotDrinkFactory {
    HotDrink prepare(int amount);
}

class TeaFactory implements HotDrinkFactory {
    @Override
    public HotDrink prepare(int amount) {
        System.out.println("Put in tea bag, boil water, pour " + amount + "ml, add lemon, enjoy!");
        return new Tea();
    }
}

class CoffeeFactory implements HotDrinkFactory {
    @Override
    public HotDrink prepare(int amount) {
        System.out.println("Grind some beans, boil water, pour " + amount + " ml, add cream and sugar, enjoy!");
        return new Coffee();
    }
}

class HotDrinkMachine {

    private List<Pair<String, HotDrinkFactory>> namedFactories = new ArrayList<>();

    public HotDrinkMachine() throws Exception {
        // find all implementors of HotDrinkFactory
        Set<Class<? extends HotDrinkFactory>> types = new Reflections("").getSubTypesOf(HotDrinkFactory.class);
        for (Class<? extends HotDrinkFactory> type : types) {
            namedFactories.add(new Pair<>(
                    type.getSimpleName().replace("Factory", ""),
                    type.getDeclaredConstructor().newInstance()
            ));
        }
    }

    public HotDrink makeDrink() throws IOException {
        System.out.println("Available drinks");
        for (int index = 0; index < namedFactories.size(); ++index) {
            Pair<String, HotDrinkFactory> item = namedFactories.get(index);
            System.out.println("" + index + ": " + item.getKey());
        }

        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        while (true) {
            String s;
            int i, amount;
            if ((s = reader.readLine()) != null && (i = Integer.parseInt(s)) >= 0 && i < namedFactories.size()) {
                System.out.println("Specify amount: ");
                s = reader.readLine();
                if (s != null && (amount = Integer.parseInt(s)) > 0) {
                    return namedFactories.get(i).getValue().prepare(amount);
                }
            }
            System.out.println("Incorrect input, try again.");
        }
    }
}
```

</details>

## Prototype
- Prototype is a partially or fully initialized object that you copy (clone) and make use of.
- Clone the prototype:
  * Implement your own deep copy functionality;
  * Serialize and deserialize;

<details>
<summary>Don't use cloneable</summary>

```java
class Address implements Cloneable {  // Cloneable is a marker interface
    private String streetName;
    private int houseNumber;

    public Address(String streetName, int houseNumber) {
        this.streetName = streetName;
        this.houseNumber = houseNumber;
    }

    @Override
    public Object clone() {  // base class clone() is protected
        return new Address(streetName, houseNumber);  // deep copy
    }
}

class Person implements Cloneable {
    private String [] names;  // first name, middle name, last name
    private Address address;

    public Person(String[] names, Address address) {
        this.names = names;
        this.address = address;
    }

    @Override
    public Object clone() {
        return new Person(names.clone(), (Address) address.clone());  // deep copy
    }
}
```

</details>

<details>
<summary>Copy Constructor</summary>

```java
class Address {
    private String streetAddress, city, country;

    public Address(String streetAddress, String city, String country) {
        this.streetAddress = streetAddress;
        this.city = city;
        this.country = country;
    }

    public Address(Address other) {  // copy constructor
        this(other.streetAddress, other.city, other.country);
    }
}

class Employee {
    private String name;
    private Address address;

    public Employee(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    public Employee(Employee other) {  // copy constructor
        name = other.name;
        address = new Address(other.address);
    }
}
```

</details>

<details>
<summary>Copy Through Serialization</summary>

```java
import java.io.Serializable;
import org.apache.commons.lang3.SerializationUtils;

class Foo implements Serializable {
    public int stuff;
    public String whatever;

    public Foo(int stuff, String whatever) {
        this.stuff = stuff;
        this.whatever = whatever;
    }

    @Override
    public String toString() {
        return "Foo{stuff=" + stuff + ", whatever='" + whatever + '}';
    }
}

class CopyThroughSerializationDemo {
    public static void main(String[] args) {
        Foo foo = new Foo(42, "life");

        // use apache commons!
        Foo foo2 = SerializationUtils.roundtrip(foo);  // roundtrip: serialize and deserialize

        foo2.whatever = "xyz";

        System.out.println(foo);
        System.out.println(foo2);
    }
}
```

</details>

## Singleton
- Singleton is a component which is instantiated only once.
- Singletons are difficult to test.
- Follow the Dependency Inversion Principle: Instead of depending on a concrete implementation of a singleton, consider depending on an abstraction (e.g., an interface).
- Consider defining singleton lifetime in a Dependency Injection container.

<details>
<summary>Basic Singleton</summary>

```java
class BasicSingleton implements Serializable {
    
    // You cannot new this class, however:
    // - Instance can be created deliberately (reflection);
    // - Instance can be created accidentally (serialization);
    private BasicSingleton() {
        System.out.println("Initializing a basic singleton");
    }

    private static final BasicSingleton INSTANCE = new BasicSingleton();

    public static BasicSingleton getInstance() {
        return INSTANCE;
    }

    // Required for correct serialization (readResolve is used for _replacing_ the object read from the stream)
    protected Object readResolve() {
        return INSTANCE;
    }
}
```

</details>

<details>
<summary>Static Block Singleton</summary>

```java
class StaticBlockSingleton {

    private static StaticBlockSingleton instance;

    private StaticBlockSingleton() {
        System.out.println("Initializing a Static Block Singleton");
    }

    //static block initialization for exception handling
    static {
        try {
            instance = new StaticBlockSingleton();
        } catch(Exception e) {
            throw new RuntimeException("Exception occured in creating singleton instance");
        }
    }

    public static StaticBlockSingleton getInstance() {
        return instance;
    }
}
```

</details>

<details>
<summary>Singleton with Laziness and Thread Safety</summary>

```java
class LazySingleton {

    private static LazySingleton instance;

    private LazySingleton() {
        System.out.println("Initializing a singleton with laziness and thread safety");
    }

    // Correct but possibly expensive multi-threaded version
    public static synchronized LazySingleton getInstance() {
        if (instance == null) {
            instance = new LazySingleton();
        }
        return instance;
    }

    // Double-Checked Locking (https://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java)
    public static LazySingleton getInstance_DoubleCheckedLocking() {
        if (instance == null) {
            synchronized (LazySingleton.class) {
                if (instance == null) {
                    instance = new LazySingleton();
                }
            }
        }
        return instance;
    }
}
```

</details>

<details>
<summary>Bill Pugh Singleton (Inner Static Singleton)</summary>

```java
class BillPughSingleton {

    /**
     * This is called the initialization-on-demand holder idiom.
     * In Java, encapsulating classes do not automatically initialize inner classes.
     * So the inner class only gets initialized by getInstance().
     * Then again, class initialization is guaranteed to be sequential in Java, so the JVM implicitly renders it thread-safe.
     */

    private BillPughSingleton() {
        System.out.println("Initializing an Bill Pugh Singleton");
    }

    public static class Impl {
        private static final BillPughSingleton INSTANCE = new BillPughSingleton();
    }

    public static BillPughSingleton getInstance() {
        return Impl.INSTANCE;
    }
}
```

</details>

<details>
<summary>Enum-based Singleton</summary>

```java
enum EnumBasedSingleton {

    INSTANCE;

    public static void doSomething(){
        //do something
    }
}
```

</details>

<details>
<summary>Multiton</summary>

```java
import java.util.HashMap;

enum Subsystem {
    PRIMARY,
    AUXILIARY,
    FALLBACK
}

class Printer {  // Multiton contains a finite set of instances
    private static int instanceCount = 0;

    private Printer() {
        instanceCount++;
        System.out.println("A total of " + instanceCount + " instances created so far.");
    }

    private static HashMap<Subsystem, Printer> instances = new HashMap<>();

    public static Printer get(Subsystem ss) {
        if (instances.containsKey(ss)) {
            return instances.get(ss);
        }

        Printer instance = new Printer();
        instances.put(ss, instance);
        return instance;
    }
}
```

</details>

## Adaptor
- Adaptor is a construct which adapts an existing interface X to conform to the required interface Y.
- Intermediate representations can pile up: use caching and other optimizations.

<details>
<summary>Adaptor without Cache</summary>

```java
import java.util.ArrayList;

class Point {
    public int x, y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

class Line {
    public Point start, end;

    public Line(Point start, Point end) {
        this.start = start;
        this.end = end;
    }
}

class LineToPointAdapter extends ArrayList<Point> {
    public LineToPointAdapter(Line line) {
        int left = Math.min(line.start.x, line.end.x);
        int right = Math.max(line.start.x, line.end.x);
        int top = Math.min(line.start.y, line.end.y);
        int bottom = Math.max(line.start.y, line.end.y);
        int dx = right - left;
        int dy = bottom - top;

        // For simplicity, here we only consider lines which are vertical and horizontal
        if (dx == 0) {
            for (int y = top; y <= bottom; ++y) {
                add(new Point(left, y));
            }
        }
        else if (dy == 0) {
            for (int x = left; x <= right; ++x) {
                add(new Point(x, top));
            }
        }
    }
}
```

</details>

<details>
<summary>Adaptor with Cache</summary>

```java
import java.util.*;
import java.util.function.Consumer;

class Point {
    public int x, y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        // implementation omitted
    }

    @Override
    public int hashCode() {
        // implementation omitted
    }
}

class Line {
    public Point start, end;

    public Line(Point start, Point end) {
        this.start = start;
        this.end = end;
    }

    @Override
    public boolean equals(Object o) {
        // implementation omitted
    }

    @Override
    public int hashCode() {
        // implementation omitted
    }
}

class LineToPointAdapter implements Iterable<Point> {
    private static Map<Line, List<Point>> cache = new HashMap<>();
    
    private Line line;

    public LineToPointAdapter(Line line) {
        if (cache.get(line) != null) {
            return;
        }

        this.line = line;
        List<Point> points = new ArrayList<>();

        int left = Math.min(line.start.x, line.end.x);
        int right = Math.max(line.start.x, line.end.x);
        int top = Math.min(line.start.y, line.end.y);
        int bottom = Math.max(line.start.y, line.end.y);
        int dx = right - left;
        int dy = bottom - top;

        if (dx == 0) {
            for (int y = top; y <= bottom; ++y) {
                points.add(new Point(left, y));
            }
        }
        else if (dy == 0) {
            for (int x = left; x <= right; ++x) {
                points.add(new Point(x, top));
            }
        }

        cache.put(line, points);
    }

    @Override
    public Iterator<Point> iterator() {
        return cache.get(line).iterator();
    }

    @Override
    public void forEach(Consumer<? super Point> action) {
        cache.get(line).forEach(action);
    }

    @Override
    public Spliterator<Point> spliterator() {
        return cache.get(line).spliterator();
    }
}
```

</details>

## Bridge
- Bridge is a mechanism that decouples abstraction (hierarchy) from implementation (hierarchy).

<details>
<summary>Cartesian-Product Duplication (Not Good)</summary>

```java
/**
 * Shape -> Triangle, Shape
 * Rendering -> Vector, Raster
 */

abstract class Shape {
    public abstract String getName();
}

class Triangle extends Shape {
    @Override
    public String getName() {
        return "Triangle";
    }
}

class Square extends Shape {
    @Override
    public String getName() {
        return "Square";
    }
}

/**
 * Cartesian product complexity explosion: 2 * 2 = 4
 */
class VectorTriangle extends Triangle {
    public void draw() {
        System.out.println(String.format("Drawing %s as lines", getName()));
    }
}

class RasterTriangle extends Triangle {
    public void draw() {
        System.out.println(String.format("Drawing %s as pixels", getName()));
    }
}

class VectorSquare extends Square {
    public void draw() {
        System.out.println(String.format("Drawing %s as lines", getName()));
    }
}

class RasterSquare extends Square {
    public void draw() {
        System.out.println(String.format("Drawing %s as pixels", getName()));
    }
}
```
</details>

<details>
<summary>Bridge Pattern</summary>

```java
interface Renderer {
    public String whatToRenderAs();
}

class VectorRenderer implements Renderer {
    @Override
    public String whatToRenderAs() {
        return "lines";
    }
}

class RasterRenderer implements Renderer {
    @Override
    public String whatToRenderAs() {
        return "pixels";
    }
}

abstract class Shape {
    private Renderer renderer;
    public String name;

    public Shape(Renderer renderer) {
        this.renderer = renderer;
    }

    public void draw() {
        System.out.println(String.format("Drawing %s as %s", name, renderer.whatToRenderAs()));
    }
}

class Triangle extends Shape {
    public Triangle(Renderer renderer) {
        super(renderer);
        name = "Triangle";
    }
}

class Square extends Shape {
    public Square(Renderer renderer) {
        super(renderer);
        name = "Square";
    }
}

class Demo {
    public static void main(String[] args) {
        Shape vectorTriangle = new Triangle(new VectorRenderer());
        vectorTriangle.draw();
        Shape rasterTriangle = new Triangle(new RasterRenderer());
        rasterTriangle.draw();
        Shape vectorSquare = new Square(new VectorRenderer());
        vectorSquare.draw();
        Shape rasterSquare = new Square(new RasterRenderer());
        rasterSquare.draw();
    }
}
```
</details>

## Composition
- Composition is a mechanism for treating individual (scalar) objects and compositions of objects in a uniform manner.

<details>
<summary>Composition Pattern: Neural Networks</summary>

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.Iterator;
import java.util.List;
import java.util.Spliterator;
import java.util.function.Consumer;

interface SomeNeurons extends Iterable<Neuron> {
    default void connectTo(SomeNeurons other) {
        if (this == other) return;

        for (Neuron from : this) {
            for (Neuron to : other) {
                from.out.add(to);
                to.in.add(from);
            }
        }
    }
}

// Object
class Neuron implements SomeNeurons {

    public List<Neuron> in, out;

    public Neuron() {
        this.in = new ArrayList<>();
        this.out = new ArrayList<>();
    }

    @Override
    public Iterator<Neuron> iterator() {
        return Collections.singleton(this).iterator();
    }

    @Override
    public void forEach(Consumer<? super Neuron> action) {
        action.accept(this);
    }

    @Override
    public Spliterator<Neuron> spliterator() {
        return Collections.singleton(this).spliterator();
    }
}

// Composition of objects (neuron layer contains multiple neuron)
class NeuronLayer extends ArrayList<Neuron> implements SomeNeurons {
}

class NeuralNetworksDemo {
    public static void main(String[] args) {
        Neuron neuron = new Neuron();
        Neuron neuron2 = new Neuron();
        NeuronLayer layer = new NeuronLayer();
        NeuronLayer layer2 = new NeuronLayer();

        // Composition Pattern: object and composition of objects are treated in a uniform manner
        neuron.connectTo(neuron2);
        neuron.connectTo(layer);
        layer.connectTo(neuron);
        layer.connectTo(layer2);
    }
}
```

</details>

## Decorator
- Decorator facilitates the addition of behaviors to individual objects without inheriting from them.
- A decorator keeps the reference to the decorated object(s).
- It may or may not forward calls: IDE (i.e., IntelliJ) can generate delegated members.

<details>
<summary>Dynamic Decorator</summary>

```java
// The interface Coffee defines the functionality of Coffee implemented by decorator
interface Coffee {
    public double getCost(); // Returns the cost of the coffee
    public String getIngredients(); // Returns the ingredients of the coffee
}

// Extension of a simple coffee without any extra ingredients
class SimpleCoffee implements Coffee {
    @Override
    public double getCost() {
        return 1;
    }

    @Override
    public String getIngredients() {
        return "Coffee";
    }
}

// Abstract decorator class - note that it implements Coffee interface
abstract class CoffeeDecorator implements Coffee {
    private final Coffee decoratedCoffee;

    public CoffeeDecorator(Coffee c) {
        this.decoratedCoffee = c;
    }

    @Override
    public double getCost() { // Implementing methods of the interface
        return decoratedCoffee.getCost();
    }

    @Override
    public String getIngredients() {
        return decoratedCoffee.getIngredients();
    }
}

// Decorator WithMilk mixes milk into coffee.
// Note it extends CoffeeDecorator.
class WithMilk extends CoffeeDecorator {
    public WithMilk(Coffee c) {
        super(c);
    }

    @Override
    public double getCost() { // Overriding methods defined in the abstract superclass
        return super.getCost() + 0.5;
    }

    @Override
    public String getIngredients() {
        return super.getIngredients() + ", Milk";
    }
}

// Decorator WithSprinkles mixes sprinkles onto coffee.
// Note it extends CoffeeDecorator.
class WithSprinkles extends CoffeeDecorator {
    public WithSprinkles(Coffee c) {
        super(c);
    }

    @Override
    public double getCost() {
        return super.getCost() + 0.2;
    }

    @Override
    public String getIngredients() {
        return super.getIngredients() + ", Sprinkles";
    }
}

class Demo {
    public static void printInfo(Coffee c) {
        System.out.println("Cost: " + c.getCost() + "; Ingredients: " + c.getIngredients());
    }

    public static void main(String[] args) {
        Coffee c = new SimpleCoffee();
        printInfo(c);

        c = new WithMilk(c);
        printInfo(c);

        c = new WithSprinkles(c);
        printInfo(c);
    }
}
```

</details>

## Façade
- The Façade design pattern is all about providing a simple, easy to understand user interface over a large and sophisticated body of code.
- Build a Façade to provide simplified APIs over a set of classes.
- May wish to (optionally) expose internals through the Façade.
- May allow users to "escalate" to use more complex APIs if they need to.

<details>
<summary>Façade Example</summary>

```java
/* Complex parts */

class CPU {
    public void freeze() { ... }
    public void jump(long position) { ... }
    public void execute() { ... }
}

class HardDrive {
    public byte[] read(long lba, int size) { ... }
}

class Memory {
    public void load(long position, byte[] data) { ... }
}

/* Facade */

class ComputerFacade {
    private final CPU processor;
    private final Memory ram;
    private final HardDrive hd;

    public ComputerFacade() {
        this.processor = new CPU();
        this.ram = new Memory();
        this.hd = new HardDrive();
    }

    public void start() {
        processor.freeze();
        ram.load(BOOT_ADDRESS, hd.read(BOOT_SECTOR, SECTOR_SIZE));
        processor.jump(BOOT_ADDRESS);
        processor.execute();
    }
}

/* Client */

class You {
    public static void main(String[] args) {
        ComputerFacade computer = new ComputerFacade();
        computer.start();
    }
}
```

</details>

## Flyweight
- Flyweight is a space optimization technique that lets us use less memory by storing externally the data associated with similar objects.

<details>
<summary>Flyweight Example</summary>

```java
import java.util.ArrayList;
import java.util.WeakHashMap;

class CoffeeFlavour {
    private final String name;
    private static final WeakHashMap<String, CoffeeFlavour> CACHE = new WeakHashMap<>();

    // only intern() can call this constructor
    private CoffeeFlavour(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return name;
    }

    public static CoffeeFlavour intern(String name) {
        synchronized (CACHE) {
            return CACHE.computeIfAbsent(name, CoffeeFlavour::new);
        }
    }

    public static int flavoursInCache() {
        synchronized (CACHE) {
            return CACHE.size();
        }
    }
}

@FunctionalInterface
interface Order {
    void serve();

    static Order of(String flavourName, int tableNumber) {
        CoffeeFlavour flavour = CoffeeFlavour.intern(flavourName);
        return () -> System.out.println("Serving " + flavour + " to table " + tableNumber);
    }
}

class CoffeeShop {
    private final ArrayList<Order> orders = new ArrayList<>();

    public void takeOrder(String flavour, int tableNumber) {
        orders.add(Order.of(flavour, tableNumber));
    }

    public void service() {
        orders.forEach(Order::serve);
    }
}

class FlyweightExample {
    public static void main(String[] args) {
        CoffeeShop shop = new CoffeeShop();
        shop.takeOrder("Cappuccino", 2);
        shop.takeOrder("Frappe", 1);
        shop.takeOrder("Espresso", 1);
        shop.takeOrder("Frappe", 897);
        shop.takeOrder("Cappuccino", 97);
        shop.takeOrder("Frappe", 3);
        shop.takeOrder("Espresso", 3);
        shop.takeOrder("Cappuccino", 3);
        shop.takeOrder("Espresso", 96);
        shop.takeOrder("Frappe", 552);
        shop.takeOrder("Cappuccino", 121);
        shop.takeOrder("Espresso", 121);

        shop.service();
        System.out.println("CoffeeFlavor objects in cache: " + CoffeeFlavour.flavoursInCache());
    }
}
```

</details>

## Proxy
- Proxy is a class that functions as an interface to a particular resource. That resource may be remote, expensive to construct, or may require logging or some other added functionality.
- A proxy has the same interface as the underlying object.
- To create a proxy, you can simply replicate the existing interface of an object.
- It allows to add relevant functionality to the redefined member functions.

<details>
<summary>Protection Proxy</summary>

```java
// Protection Proxy controls access to particular resource while offering the same API.

interface Drivable {
    void drive();
}

class Car implements Drivable {
    protected Driver driver;

    public Car(Driver driver) {
        this.driver = driver;
    }

    @Override
    public void drive() {
        System.out.println("Car being driven");
    }
}

class Driver {
    private int age;

    public Driver(int age) {
        this.age = age;
    }

    public int getAge() {
        return age;
    }
}

// CarProxy behaves every way as a car but it actually verify that the driver is old enough to drive.
class CarProxy extends Car {
    public CarProxy(Driver driver) {
        super(driver);
    }

    @Override
    public void drive() {
        if (driver.getAge() > 16) {
            super.drive();
        }
        else {
            System.out.println("Driver is too young");
        }
    }
}

class Demo {
    public static void main(String[] args) {
        Car car = new CarProxy(new Driver(12));
        car.drive();
    }
}
```

</details>

<details>
<summary>Property Proxy</summary>

```java
// Property Proxy is an idea of replacing a field with something which forces you to perform some kind of checks (e.g., logging).

class Property<T> {
    private T value;

    public Property(T value) {
        this.value = value;
    }

    public T getValue() {
        // You can do logging here.
        return value;
    }

    public void setValue(T value) {
        // You can do logging here.
        this.value = value;
    }
}

class Creature {
    private Property<Integer> agility;

    public Creature(int agility) {
        this.agility = new Property<>(agility);
    }

    public int getAgility() {
        return agility.getValue();
    }

    public void setAgility(int agility) {
        this.agility.setValue(agility);
    }
}
```

</details>

<details>
<summary>Dynamic Proxy for Logging</summary>

```java
// Dynamic Proxy is a proxy which is constructed at runtime as opposed to compile time.

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.HashMap;
import java.util.Map;

interface Human {
    void walk();
    void talk();
}

class Person implements Human {
    @Override
    public void walk() {
        System.out.println("I am walking.");
    }

    @Override
    public void talk() {
        System.out.println("I am talking.");
    }
}

class LoggingHandler implements InvocationHandler {
    private Object target;
    private Map<String, Integer> calls;

    public LoggingHandler(Object target) {
        this.target = target;
        calls = new HashMap<>();
    }
    
    // intercept the invocation of every single method
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        // before invocation you can perform additional process
        String name = method.getName();

        if (name.contains("toString")) {
            return calls.toString();
        }

        calls.put(name, calls.getOrDefault(name, 0) + 1);

        // actual invocation
        return method.invoke(target, args);
    }
}

class Demo {
    @SuppressWarnings("unchecked")
    public static <T> T withLogging(T target, Class<T> myClass) {
        return (T) Proxy.newProxyInstance(
                myClass.getClassLoader(),
                new Class<?>[] { myClass },
                new LoggingHandler(target)
        );
    }

    public static void main(String[] args) {
        Person person = new Person();
        Human logged = withLogging(person, Human.class);
        logged.talk();
        logged.walk();
        logged.walk();
        System.out.println(logged);
    }
}
```

</details>

## Chain of Responsibility
- A chain of components who all get a chance to process a command or a query, optionally having default processing implementation and an ability to terminate the processing chain.
- Command-Query Separation: every method should either be a command that performs an action, or a query that returns data to the caller, but not both.

<details>
<summary>Method Chain</summary>

```java
class Creature {
    public String name;
    public int offense, defense;

    public Creature(String name, int offense, int defense) {
        this.name = name;
        this.offense = offense;
        this.defense = defense;
    }

    @Override
    public String toString() {
        return "Creature{name='" + name + "', offense=" + offense + ", defense=" + defense + "}";
    }
}

class CreatureModifier {
    protected Creature creature;
    protected CreatureModifier next;

    public CreatureModifier(Creature creature) {
        this.creature = creature;
    }

    public void add(CreatureModifier cm) {
        if (next != null) {
            next.add(cm);
        }
        else {
            next = cm;
        }
    }

    public void handle() {
        if (next != null) {
            next.handle();
        }
    }
}

class DoubleOffenseModifier extends CreatureModifier {
    public DoubleOffenseModifier(Creature creature) {
        super(creature);
    }

    @Override
    public void handle() {
        creature.offense *= 2;
        super.handle();
    }
}

class IncreaseDefenseModifier extends CreatureModifier {
    public IncreaseDefenseModifier(Creature creature) {
        super(creature);
    }

    @Override
    public void handle() {
        creature.defense += 3;
        super.handle();
    }
}

class NoBonusesModifier extends CreatureModifier {
    public NoBonusesModifier(Creature creature) {
        super(creature);
    }

    @Override
    public void handle() {
        // do nothing
    }
}

class Demo {
    public static void main(String[] args) {
        Creature goblin = new Creature("goblin", 2, 2);
        System.out.println(goblin);

        CreatureModifier root = new CreatureModifier(goblin);  // dummy root
        // root.add(new NoBonusesModifier(goblin));
        root.add(new DoubleOffenseModifier(goblin));
        root.add(new IncreaseDefenseModifier(goblin));
        root.handle();
        System.out.println(goblin);
    }
}
```

</details>

<details>
<summary>Broker Chain (Hard to understand)</summary>

```java
import java.util.HashMap;
import java.util.Map;
import java.util.function.Consumer;

class Event<Args> {
    private int index;
    private Map<Integer, Consumer<Args>> handlers;

    public Event() {
        index = 0;
        handlers = new HashMap<>();
    }

    public int subscribe(Consumer<Args> handler) {
        int i = index;
        handlers.put(index++, handler);
        return i;
    }

    public void unsubscribe(int key) {
        handlers.remove(key);
    }

    public void fire(Args args) {
        for (Consumer<Args> handler : handlers.values()) {
            handler.accept(args);
        }
    }
}

enum Property {
    OFFENSE, DEFENSE
}

class Query {
    public String creatureName;
    public Property property;
    public int value;

    public Query(String creatureName, Property property, int value) {
        this.creatureName = creatureName;
        this.property = property;
        this.value = value;
    }
}

class Game {  // mediator
    public Event<Query> queries;

    public Game() {
        queries = new Event<>();
    }
}

class Creature {
    private Game game;
    public String name;
    private int baseOffense, baseDefense;

    public Creature(Game game, String name, int baseOffense, int baseDefense) {
        this.game = game;
        this.baseOffense = baseOffense;
        this.baseDefense = baseDefense;
        this.name = name;
    }

    int getOffense() {
        Query q = new Query(name, Property.OFFENSE, baseOffense);
        game.queries.fire(q);
        return q.value;
    }

    int getDefense() {
        Query q = new Query(name, Property.DEFENSE, baseDefense);
        game.queries.fire(q);
        return q.value;
    }

    @Override
    public String toString() {
        return "Creature{name='" + name + "', offense=" + getOffense() + ", defense=" + getDefense() + "}";
    }
}

class CreatureModifier {  // protected, not private!
    protected Game game;
    protected Creature creature;

    public CreatureModifier(Game game, Creature creature) {
        this.game = game;
        this.creature = creature;
    }
}

class IncreasedDefenseModifier extends CreatureModifier {

    public IncreasedDefenseModifier(Game game, Creature creature) {
        super(game, creature);

        game.queries.subscribe(q -> {
            if (q.creatureName.equals(creature.name) && q.property == Property.DEFENSE) {
                q.value += 3;
            }
        });
    }
}

class DoubleOffenseModifier extends CreatureModifier implements AutoCloseable {

    private int token;

    public DoubleOffenseModifier(Game game, Creature creature) {
        super(game, creature);

        token = game.queries.subscribe(q -> {
            if (q.creatureName.equals(creature.name) && q.property == Property.OFFENSE) {
                q.value *= 2;
            }
        });
    }

    @Override
    public void close() /*throws Exception*/ {
        game.queries.unsubscribe(token);
    }
}

class BrokerChainDemo {
    public static void main(String[] args) {
        Game game = new Game();
        Creature goblin = new Creature(game, "Strong Goblin", 2, 2);

        System.out.println(goblin);

        // modifiers can be piled up
        IncreasedDefenseModifier icm = new IncreasedDefenseModifier(game, goblin);
        System.out.println(goblin);

        try (DoubleOffenseModifier dom = new DoubleOffenseModifier(game, goblin)) {
            System.out.println(goblin);
        }
        System.out.println(goblin);
    }
}
```

</details>

## Command
- Command is an object which represents an instruction to perform a particular action. It contains all the information necessary for the action to be taken.

<details>
<summary>Command: Undo Operation</summary>

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

class BankAccount {
    private int balance;
    private final int overdraftLimit = -500;

    public void deposit(int amount) {
        balance += amount;
    }

    public boolean withdraw(int amount) {
        if (balance - amount >= overdraftLimit) {
            balance -= amount;
            return true;
        }
        return false;  // cannot withdraw
    }

    @Override
    public String toString() {
        return "BankAccount{balance=" + balance + "}";
    }
}

enum Action {
    DEPOSIT, WITHDRAW
}

interface Command {
    void call();
    void undo();
}

class BankAccountCommand implements Command {

    private BankAccount bankAccount;
    private Action action;
    private int amount;

    private boolean isCommandSucceeded;

    public BankAccountCommand(BankAccount bankAccount, Action action, int amount) {
        this.bankAccount = bankAccount;
        this.action = action;
        this.amount = amount;
    }

    @Override
    public void call() {
        switch (action) {
            case DEPOSIT:
                bankAccount.deposit(amount);
                isCommandSucceeded = true;
                break;
            case WITHDRAW:
                isCommandSucceeded = bankAccount.withdraw(amount);
                break;
        }
    }

    @Override
    public void undo() {  // assume it's only executed after call()
        if (!isCommandSucceeded) {
            return;
        }

        switch (action) {
            case DEPOSIT:
                bankAccount.withdraw(amount);
                break;
            case WITHDRAW:
                bankAccount.deposit(amount);
                break;
        }
    }
}

class Demo {
    public static void main(String[] args) {
        BankAccount bankAccount = new BankAccount();
        System.out.println(bankAccount);

        List<Command> commands = new ArrayList<>();
        commands.add(new BankAccountCommand(bankAccount, Action.DEPOSIT, 100));
        commands.add(new BankAccountCommand(bankAccount, Action.WITHDRAW, 1000));

        for (Command c : commands) {
            c.call();
            System.out.println(bankAccount);
        }

        Collections.reverse(commands);

        for (Command c : commands) {
            c.undo();
            System.out.println(bankAccount);
        }
    }
}
```

</details>

## Interpreter
- Interpreter is a component that processes structured text data by turning it into separate lexical token (lexing) and then interpreting sequences of said tokens (parsing).

## Iterator
- Iterator is an object that facilitates the traversal of a data structure.
- Iterator cannot be recursive (no coroutines).
- Iterator implements `Iterator<T>`; Iterable object implements `Iterable<T>`.

<details>
<summary>Binary Tree Inorder Traversal</summary>

```java
import java.util.Iterator;
import java.util.Spliterator;
import java.util.function.Consumer;

class Node<T> {
    public T value;
    public Node<T> left, right, parent;

    public Node(T value) {
        this.value = value;
    }

    public Node(T value, Node<T> left, Node<T> right) {
        this.value = value;
        this.left = left;
        this.right = right;

        left.parent = this;
        right.parent = this;
    }
}

class InOrderIterator<T> implements Iterator<T> {
    private Node<T> current;
    private boolean yieldedStart;

    public InOrderIterator(Node<T> root) {
        this.current = root;
        while (current.left != null) {
            current = current.left;
        }
    }

    private boolean hasRightmostParent(Node<T> node) {
        if (node.parent == null) {
            return false;
        }
        else {
            return node == node.parent.left || hasRightmostParent(node.parent);
        }
    }

    @Override
    public boolean hasNext() {
        return current.right != null || hasRightmostParent(current);
    }

    @Override
    public T next() {
        if (!yieldedStart) {
            yieldedStart = true;
            return current.value;
        }

        if (current.right != null) {
            current = current.right;
            while (current.left != null) {
                current = current.left;
            }
            return current.value;
        }
        else {
            Node<T> p = current.parent;
            while (p != null && p.right == current) {
                current = p;
                p = p.parent;
            }
            current = p;
            return current.value;
        }
    }
}

class BinaryTree<T> implements Iterable<T> {
    private Node<T> root;

    public BinaryTree(Node<T> root) {
        this.root = root;
    }

    @Override
    public Iterator<T> iterator() {
        return new InOrderIterator<>(root);
    }

    @Override
    public void forEach(Consumer<? super T> action) {
        for (T item : this) {
            action.accept(item);
        }
    }

    @Override
    public Spliterator<T> spliterator() {
        return null;
    }
}

class Demo {
    public static void main(String [] args) {
        //   1
        //  / \
        // 2   3
        Node<Integer> root = new Node<>(1, new Node<>(2), new Node<>(3));

        InOrderIterator<Integer> it = new InOrderIterator<>(root);
        while (it.hasNext()) {
            System.out.print("" + it.next() + ",");
        }
        System.out.println();

        BinaryTree<Integer> tree = new BinaryTree<>(root);
        for (int n : tree) {
            System.out.print("" + n + ",");
        }
        System.out.println();
    }
}
```

</details>

## Mediator
- Mediator is a component that facilitates communication between other components without them necessarily being aware of each other or having direct (reference) access to each other.
- Mediator has functions the components can call.
- Components have functions the mediator can call.
- Event processing libriries (e.g., [Reactive Extensions](https://github.com/ReactiveX/RxJava)) make communication easier to implement.

<details>
<summary>Chat Room</summary>

```java
import java.util.ArrayList;
import java.util.List;

class Person {
    public String name;  // assume name is unique
    public ChatRoom room;
    private List<String> chatLog = new ArrayList<>();

    public Person(String name) {
        this.name = name;
    }

    public void receive(String sender, String message) {
        String s = sender + ": '" + message + "'";
        System.out.println("[" + name + "'s chat session] " + s);
        chatLog.add(s);
    }

    public void say(String message) {
        room.broadcast(name, message);
    }

    public void privateMessage(String who, String message) {
        room.message(name, who, message);
    }
}

class ChatRoom {  // mediator
    private List<Person> persons = new ArrayList<>();

    public void broadcast(String source, String message) {
        for (Person person : persons) {
            if (!person.name.equals(source)) {
                person.receive(source, message);
            }
        }
    }

    public void join(Person p) {
        String joinMsg = p.name + " joins the chat";
        broadcast("room", joinMsg);

        p.room = this;
        persons.add(p);
    }

    public void message(String source, String destination, String message) {
        persons.stream()
                .filter(p -> p.name.equals(destination))
                .findFirst()
                .ifPresent(person -> person.receive(source, message));
    }
}

class ChatRoomDemo {
    public static void main(String[] args) {
        ChatRoom room = new ChatRoom();

        Person john = new Person("John");
        Person jane = new Person("Jane");

        room.join(john); // no message here
        room.join(jane);

        john.say("hi room");
        jane.say("oh, hey john");

        Person simon = new Person("Simon");
        room.join(simon);
        simon.say("hi everyone!");

        jane.privateMessage("Simon", "glad you could join us!");
    }
}
```

</details>

<details>
<summary>Reactive Extensions Event Broker</summary>

```java
import io.reactivex.Observable;
import io.reactivex.Observer;

import java.util.ArrayList;
import java.util.List;

class EventBroker extends Observable<Integer> {
    private List<Observer<? super Integer>> observers = new ArrayList<>();

    @Override
    protected void subscribeActual(Observer<? super Integer> observer) {
        observers.add(observer);
    }

    public void publish(int n) {
        for (Observer<? super Integer> o : observers) {
            o.onNext(n);
        }
    }
}

class FootballPlayer {
    private int goalsScored = 0;
    private EventBroker broker;
    public String name;

    public FootballPlayer(EventBroker broker, String name) {
        this.broker = broker;
        this.name = name;
    }

    public void score() {
        broker.publish(++goalsScored);
    }
}

class FootballCoach {
    public FootballCoach(EventBroker broker) {
        broker.subscribe(i -> {
            System.out.println("Hey, you scored " + i + " goals!");
        });
    }
}

class RxEventBrokerDemo {
    public static void main(String [] args) {
        EventBroker broker = new EventBroker();
        FootballPlayer player = new FootballPlayer(broker, "jones");
        FootballCoach coach = new FootballCoach(broker);

        player.score();
        player.score();
        player.score();
    }
}
```

</details>

## Memento
- Memento is a token/handle representing the system state.
- It allows you to roll back to the state when the token was generated.
- It may or may not directly expose state information.
- Memento can be used to implement undo/redo operations.
- Memento should be inmutable.

<details>
<summary>Memento Example</summary>

```java
class Memento {
    private int balance;

    public Memento(int balance) {
        this.balance = balance;
    }

    public int getBalance() {
        return balance;
    }
}

class BankAccount {
    private int balance;

    public BankAccount(int balance) {
        this.balance = balance;
    }

    public Memento deposit(int amount) {
        balance += amount;
        return new Memento(balance);
    }

    public void restore(Memento m) {
        balance = m.getBalance();
    }

    @Override
    public String toString() {
        return "BankAccount{balance=" + balance + "}";
    }
}

class MementoDemo {
    public static void main(String[] args) {
        BankAccount ba = new BankAccount(100);
        Memento m1 = ba.deposit(50); // 150
        Memento m2 = ba.deposit(25); // 175
        System.out.println(ba);

        // restore to m1
        ba.restore(m1);
        System.out.println(ba);

        // restore to m2
        ba.restore(m2);
        System.out.println(ba);
    }
}
```

</details>

## Null Object
- Null object is a no-op object that conforms to the required interface and satisfies a dependency requirement of some other object, but it doesn't really do anything.

<details>
<summary>Null Object</summary>

```java
interface Log {
    void info(String msg);
    void warn(String msg);
}

class ConsoleLog implements Log {

    @Override
    public void info(String msg) {
        System.out.println("INFO: " + msg);
    }

    @Override
    public void warn(String msg) {
        System.out.println("WARN: " + msg);
    }
}

class BankAccount {
    private int balance;

    private Log log;

    public BankAccount(Log log) {
        this.log = log;
    }

    public void deposit(int amount) {
        balance += amount;

        // check for null everywhere? --- not good
        if (log != null) {
            log.info("Deposited " + amount + ", balance is now " + balance);
        }
    }

    public void withdraw(int amount) {
        if (balance >= amount) {
            balance -= amount;
            if (log != null) {
                log.info("Withdrew " + amount + ", we have " + balance + " left");
            }
        }
        else {
            if (log != null) {
                log.warn("Could not withdraw " + amount + " because balance is only " + balance);
            }
        }
    }
}

final class NullLog implements Log {

    @Override
    public void info(String msg) { }

    @Override
    public void warn(String msg) { }
}

class NullObjectDemo {
    public static void main(String[] args) {
        // ConsoleLog log = new ConsoleLog();
        // Log log = null;
        NullLog log = new NullLog();

        BankAccount ba = new BankAccount(log);
        ba.deposit(100);
        ba.withdraw(200);
    }
}
```

</details>

<details>
<summary>Dynamic Null Object</summary>

```java
import java.lang.reflect.Proxy;


interface Log {
    void info(String msg);
    void warn(String msg);
}

class BankAccount {
    private int balance;

    private Log log;

    public BankAccount(Log log) {
        this.log = log;
    }

    public void deposit(int amount) {
        balance += amount;

        // check for null everywhere? --- not good
        if (log != null) {
            log.info("Deposited " + amount + ", balance is now " + balance);
        }
    }

    public void withdraw(int amount) {
        if (balance >= amount) {
            balance -= amount;
            if (log != null) {
                log.info("Withdrew " + amount + ", we have " + balance + " left");
            }
        }
        else {
            if (log != null) {
                log.warn("Could not withdraw " + amount + " because balance is only " + balance);
            }
        }
    }
}

class NullObjectDemo {

    @SuppressWarnings("unchecked")
    public static <T> T noOp(Class<T> itf) {
        return (T) Proxy.newProxyInstance(
                itf.getClassLoader(),
                new Class<?>[]{itf},
                (proxy, method, args) -> {
                    if (method.getReturnType().equals(Void.TYPE)) {
                        return null;
                    }
                    else {
                        return method.getReturnType().getConstructor().newInstance();
                    }
                });
    }

    public static void main(String[] args) {
        Log log = noOp(Log.class);

        BankAccount ba = new BankAccount(log);
        ba.deposit(100);
        ba.withdraw(200);
    }
}
```

</details>