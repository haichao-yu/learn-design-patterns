# learn-design-patterns
Some notes of "Design Patterns in Java" by Dmitri Nesteruk in Udemy

## Index

### SOLID Design Principles:
- Single Responsibility Principle
- Open-Closed Principle
- Liskov Substitution Principle
- Interface Segregation Principle
- Dependency Inversion Principle

### Creational Design Patterns:
- Builder
- Factories (Factory Method and Abstract Factory)
- Prototype
- Singleton

### Structrural Design Patterns
- Adapter
- Bridge
- Composite
- Decorator
- Façade
- Flyweight
- Proxy

### Behavioral Design Patterns
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
- A builder is a separate component for building an object.
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
- A factory is a component responsible solely for the wholesale (not piecewise) creation of objects.
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
<summary>Cartesian-Product Duplication</summary>

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