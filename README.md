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