# Classes

Classes in Raku provide a powerful object-oriented programming foundation,  
combining familiar OOP concepts with unique features like roles, multiple  
inheritance, and flexible attribute handling. Raku classes support  
encapsulation, inheritance, polymorphism, and composition, offering both  
traditional class-based patterns and modern approaches like mixins and  
traits. This comprehensive guide covers everything from basic class  
definition to advanced features like operator overloading, introspection,  
and performance optimization.

## Basic class definition

Creating a simple class with basic structure.  

```raku
class Person {
    has $.name;
    has $.age;
    
    method greet() {
        "Hello, I'm $.name and I'm $.age years old";
    }
}

my $person = Person.new(name => 'Alice', age => 30);
say $person.greet();
say $person.name;
```

Classes are defined with the `class` keyword. The `has` keyword declares  
attributes with `$.` for read-only public access. The `.new()` constructor  
automatically accepts named arguments for all public attributes.  

## Class attributes

Different types of attribute declarations and access levels.  

```raku
class Employee {
    has $.name;           # Public read-only
    has $.id is rw;      # Public read-write
    has $!salary;        # Private attribute
    has @.skills;        # Public array attribute
    has %.metadata;      # Public hash attribute
    
    method set-salary($amount) {
        $!salary = $amount;
    }
    
    method salary() {
        $!salary;
    }
}

my $emp = Employee.new(name => 'Bob', id => 123);
$emp.id = 456;  # Can modify because of 'is rw'
$emp.set-salary(50000);
say $emp.salary();
```

Attributes use different sigils and traits: `$.` for public read-only,  
`$!` for private, `is rw` for read-write access. Array and hash  
attributes use `@.` and `%.` respectively.  

## Constructor methods

Custom initialization with BUILD and new methods.  

```raku
class Rectangle {
    has $.width;
    has $.height;
    has $.area;
    
    submethod BUILD(:$!width, :$!height) {
        $!area = $!width * $!height;
    }
    
    method new($width, $height) {
        self.bless(:$width, :$height);
    }
}

my $rect1 = Rectangle.new(5, 10);
my $rect2 = Rectangle.new(width => 3, height => 4);
say "Area: {$rect1.area}";
```

The `BUILD` submethod runs during object construction. Custom `new`  
methods can provide alternative construction interfaces. The `bless`  
method creates the actual object instance.  

## Method definitions

Various ways to define and use methods in classes.  

```raku
class Calculator {
    has $.value = 0;
    
    method add($num) {
        Calculator.new(value => $.value + $num);
    }
    
    method multiply($num) {
        Calculator.new(value => $.value * $num);
    }
    
    method result() {
        $.value;
    }
    
    # Method with return type
    method divide(Numeric $num --> Numeric) {
        $.value / $num;
    }
}

my $calc = Calculator.new(value => 10);
my $result = $calc.add(5).multiply(2);
say $result.result();  # 30
```

Methods are defined with the `method` keyword. They can return new  
instances for immutable patterns. Type annotations with `-->` specify  
return types for better documentation and validation.  

## Private methods

Creating and using private methods within classes.  

```raku
class BankAccount {
    has $.balance is rw;
    has $!pin;
    
    submethod BUILD(:$!balance, :$!pin) { }
    
    method !validate-pin($entered-pin) {
        $!pin eq $entered-pin;
    }
    
    method withdraw($amount, $pin) {
        return "Invalid PIN" unless self!validate-pin($pin);
        return "Insufficient funds" if $amount > $.balance;
        
        $.balance -= $amount;
        "Withdrew $amount. Balance: $.balance";
    }
}

my $account = BankAccount.new(balance => 1000, pin => '1234');
say $account.withdraw(100, '1234');
```

Private methods use `!` instead of `.` in both definition and calling.  
They're only accessible within the class and provide internal  
functionality without exposing implementation details.  

## Class inheritance

Extending classes with the 'is' keyword.  

```raku
class Animal {
    has $.name;
    has $.species;
    
    method speak() {
        "The $.species named $.name makes a sound";
    }
}

class Dog is Animal {
    has $.breed;
    
    method speak() {
        "The $.breed dog $.name barks!";
    }
    
    method fetch() {
        "$.name is fetching the ball";
    }
}

my $dog = Dog.new(name => 'Buddy', species => 'Canine', breed => 'Golden Retriever');
say $dog.speak();
say $dog.fetch();
```

The `is` keyword creates inheritance relationships. Child classes  
inherit all attributes and methods from parents. Methods can be  
overridden to provide specialized behavior.  

## Method overriding

Customizing inherited methods and calling parent implementations.  

```raku
class Vehicle {
    has $.make;
    has $.model;
    
    method describe() {
        "This is a $.make $.model";
    }
    
    method start() {
        "Vehicle started";
    }
}

class Car is Vehicle {
    has $.doors;
    
    method describe() {
        nextsame ~ " with $.doors doors";
    }
    
    method start() {
        nextwith() ~ " - Engine running";
    }
}

my $car = Car.new(make => 'Toyota', model => 'Camry', doors => 4);
say $car.describe();
say $car.start();
```

Use `nextsame` to call the parent method with the same arguments, or  
`nextwith()` to call with no arguments. This enables extending rather  
than completely replacing parent functionality.  

## Multiple inheritance

Inheriting from multiple parent classes.  

```raku
class Flyable {
    method fly() {
        "Flying through the air";
    }
}

class Swimmable {
    method swim() {
        "Swimming through water";
    }
}

class Duck is Flyable is Swimmable {
    has $.name;
    
    method quack() {
        "$.name says quack!";
    }
}

my $duck = Duck.new(name => 'Donald');
say $duck.fly();
say $duck.swim();
say $duck.quack();
```

Multiple inheritance is achieved by chaining `is` keywords. The class  
inherits methods from all parent classes. Method resolution follows  
the order of inheritance declaration.  

## Roles and composition

Using roles for flexible code reuse and composition.  

```raku
role Printable {
    method print() {
        "Printing: {self.Str}";
    }
}

role Serializable {
    method to-json() {
        "{ json representation of {self.^name} }";
    }
}

class Document does Printable does Serializable {
    has $.title;
    has $.content;
    
    method Str() {
        "Document: $.title";
    }
}

my $doc = Document.new(title => 'Report', content => 'Data...');
say $doc.print();
say $doc.to-json();
```

Roles define reusable behavior that can be composed into classes using  
`does`. Unlike inheritance, roles are mixed in at compile time and  
provide a way to share code without rigid hierarchies.  

## Class variables and methods

Shared data and behavior across all instances.  

```raku
class Counter {
    my $global-count = 0;
    has $.local-count = 0;
    
    method BUILD() {
        $global-count++;
    }
    
    method get-global-count() {
        $global-count;
    }
    
    method increment() {
        Counter.new(local-count => $.local-count + 1);
    }
}

my $c1 = Counter.new();
my $c2 = Counter.new();
say $c1.get-global-count();  # 2
```

Class variables declared with `my` are shared across all instances.  
They maintain state at the class level rather than instance level,  
useful for counting instances or shared configuration.  

## Attribute accessors

Controlling attribute access with custom accessors.  

```raku
class Temperature {
    has $!celsius;
    
    method celsius() { $!celsius }
    
    method celsius($new-temp) {
        die "Temperature below absolute zero" if $new-temp < -273.15;
        $!celsius = $new-temp;
    }
    
    method fahrenheit() {
        $!celsius * 9/5 + 32;
    }
    
    method fahrenheit($f) {
        self.celsius(($f - 32) * 5/9);
    }
}

my $temp = Temperature.new();
$temp.celsius = 25;
say $temp.fahrenheit();  # 77
```

Custom accessors allow validation and computed properties. Define  
methods with the same name as attributes to control access and  
provide automatic conversions between different representations.  

## Attribute defaults and builders

Setting default values and computed initial values.  

```raku
class User {
    has $.username;
    has $.email;
    has $.id = $++;                    # Auto-incrementing default
    has $.created = DateTime.now;      # Computed default
    has $.display-name;
    
    submethod BUILD(:$!username, :$!email, :$display-name) {
        $!display-name = $display-name // $!username.tc;
    }
}

class Config {
    has $.debug = %*ENV<DEBUG> // False;
    has $.timeout = 30;
    has @.servers = ['localhost'];
}

my $user = User.new(username => 'john', email => 'john@example.com');
say $user.display-name;  # John
```

Attributes can have default values using `=` or be computed in `BUILD`.  
Defaults can be constants, expressions, or environment-dependent  
values, providing flexible initialization strategies.  

## Type constraints

Enforcing types and constraints on class attributes.  

```raku
class Product {
    has Str $.name where *.chars > 0;
    has Numeric $.price where * >= 0;
    has Int $.quantity where * >= 0 = 0;
    has DateTime $.created = DateTime.now;
    
    subset PositiveInt of Int where * > 0;
    has PositiveInt $.id;
}

class Email {
    subset EmailAddress of Str where /^ \w+ '@' \w+ '.' \w+ $/;
    has EmailAddress $.address;
    
    method new($address) {
        self.bless(:$address);
    }
}

my $product = Product.new(
    name => 'Widget', 
    price => 9.99, 
    id => 123
);
```

Type constraints ensure data integrity. Use built-in types like `Str`,  
`Int`, `Numeric`, and create custom subsets with `where` clauses for  
domain-specific validation rules.  

## Submethod handling

Using submethods for initialization and cleanup.  

```raku
class DatabaseConnection {
    has $.host;
    has $.port;
    has $!connection;
    
    submethod BUILD(:$!host, :$!port = 5432) {
        $!connection = "Connected to $!host:$!port";
        say "Establishing connection...";
    }
    
    submethod DESTROY() {
        say "Closing connection to $.host";
    }
    
    method query($sql) {
        "Executing: $sql on $!connection";
    }
}

{
    my $db = DatabaseConnection.new(host => 'localhost');
    say $db.query('SELECT * FROM users');
}  # DESTROY called here
```

Submethods like `BUILD` and `DESTROY` handle object lifecycle events.  
`BUILD` runs during construction for initialization, while `DESTROY`  
handles cleanup when objects go out of scope.  

## Method delegation

Delegating method calls to contained objects.  

```raku
class Engine {
    has $.horsepower;
    
    method start() { "Engine started" }
    method stop() { "Engine stopped" }
}

class Car {
    has Engine $.engine handles <start stop>;
    has $.make;
    has $.model;
    
    method describe() {
        "$.make $.model with {$.engine.horsepower}hp engine";
    }
}

my $car = Car.new(
    make => 'Ford', 
    model => 'Mustang',
    engine => Engine.new(horsepower => 450)
);

say $car.start();    # Delegated to engine
say $car.describe();
```

The `handles` trait automatically creates methods that delegate to  
contained objects. This enables composition patterns where complex  
objects delegate specific behaviors to their components.  

## Class introspection

Examining class structure and capabilities at runtime.  

```raku
class Person {
    has $.name;
    has $.age;
    
    method greet() { "Hello!" }
    method walk() { "Walking..." }
}

my $person = Person.new(name => 'Alice', age => 30);

# Class information
say $person.^name;                    # Person
say $person.^attributes;              # List of attributes
say $person.^methods;                 # List of methods
say $person.^parents;                 # Parent classes

# Check capabilities
say $person.^can('greet');            # True
say $person.^has_method('walk');      # True

# Attribute inspection
for $person.^attributes -> $attr {
    say "Attribute: {$attr.name}";
}
```

The `.^` meta-object protocol provides powerful introspection  
capabilities. Examine class hierarchy, available methods, attributes,  
and other metadata to build flexible, dynamic applications.  

## Abstract classes and methods

Creating abstract base classes with required implementations.  

```raku
class Shape {
    has $.color = 'black';
    
    method area() { ... }  # Abstract method
    method perimeter() { ... }  # Abstract method
    
    method describe() {
        "A $.color shape with area {self.area()} and perimeter {self.perimeter()}";
    }
}

class Circle is Shape {
    has $.radius;
    
    method area() {
        π * $.radius²;
    }
    
    method perimeter() {
        2 * π * $.radius;
    }
}

my $circle = Circle.new(radius => 5, color => 'blue');
say $circle.describe();
```

Use `...` (the yada-yada operator) to mark methods as abstract. Child  
classes must implement these methods. This enforces interface  
contracts while allowing shared code in the base class.  

## Mixins and traits

Runtime composition and class modification.  

```raku
role Timestamped {
    has $.timestamp = DateTime.now;
    
    method age() {
        DateTime.now - $.timestamp;
    }
}

role Auditable {
    has @.audit-log;
    
    method log-action($action) {
        @.audit-log.push: "{DateTime.now}: $action";
    }
}

class Document {
    has $.title;
    has $.content;
}

my $doc = Document.new(title => 'Report', content => 'Data');
$doc does Timestamped;
$doc does Auditable;

$doc.log-action('Created document');
say $doc.age();
```

Runtime mixins with `does` add roles to existing objects. This enables  
dynamic behavior modification and flexible object composition based  
on runtime conditions or user preferences.  

## Initialization and destruction

Managing object lifecycle with proper setup and cleanup.  

```raku
class FileProcessor {
    has $.filename;
    has $!file-handle;
    has @.processed-lines;
    
    submethod BUILD(:$!filename) {
        $!file-handle = open $!filename, :r;
        say "Opened file: $!filename";
    }
    
    method process-line($line) {
        @.processed-lines.push: $line.uc;
    }
    
    method finalize() {
        $!file-handle.close;
        say "Processed {@.processed-lines.elems} lines";
    }
    
    submethod DESTROY() {
        $!file-handle.close if $!file-handle;
        say "FileProcessor destroyed";
    }
}

{
    my $processor = FileProcessor.new(filename => 'data.txt');
    # ... process file
    $processor.finalize();
}
```

Proper initialization in `BUILD` and cleanup in `DESTROY` ensures  
resources are managed correctly. Always clean up file handles,  
network connections, and other resources in destruction methods.  

## Nested classes

Defining classes within other classes for encapsulation.  

```raku
class Library {
    has @.books;
    
    class Book {
        has $.title;
        has $.author;
        has $.isbn;
        
        method summary() {
            "'{$.title}' by $.author (ISBN: $.isbn)";
        }
    }
    
    method add-book($title, $author, $isbn) {
        @.books.push: Book.new(:$title, :$author, :$isbn);
    }
    
    method list-books() {
        @.books.map(*.summary);
    }
}

my $lib = Library.new();
$lib.add-book('1984', 'George Orwell', '978-0-452-28423-4');
say $lib.list-books();

# Access nested class
my $book = Library::Book.new(
    title => 'Brave New World',
    author => 'Aldous Huxley',
    isbn => '978-0-06-085052-4'
);
```

Nested classes provide logical grouping and namespace organization.  
Access them using the `::` operator. They're particularly useful for  
helper classes that are only relevant within a specific context.  

## Enums as classes

Creating enumerated types with class-like behavior.  

```raku
enum Status <Pending Processing Complete Failed>;

enum Priority (
    Low => 1,
    Medium => 5,
    High => 10,
    Critical => 20
);

class Task {
    has $.description;
    has Status $.status = Pending;
    has Priority $.priority = Medium;
    has DateTime $.created = DateTime.now;
    
    method set-status(Status $new-status) {
        $.status = $new-status;
        say "Task status changed to: {$new-status.Str}";
    }
    
    method is-urgent() {
        $.priority >= High;
    }
}

my $task = Task.new(description => 'Fix bug');
$task.set-status(Processing);
say "Urgent: {$task.is-urgent()}";
```

Enums create types with restricted values. They integrate seamlessly  
with classes, providing type safety and meaningful constants. Use  
them for state machines, configuration options, and categorical data.  

## Singleton pattern

Ensuring only one instance of a class exists.  

```raku
class Logger {
    my $instance;
    has @.log-entries;
    
    method new() {
        $instance //= self.bless();
    }
    
    method log($level, $message) {
        @.log-entries.push: "{DateTime.now}: [$level] $message";
    }
    
    method get-logs() {
        @.log-entries;
    }
    
    method clear() {
        @.log-entries = ();
    }
}

my $logger1 = Logger.new();
my $logger2 = Logger.new();

$logger1.log('INFO', 'Application started');
$logger2.log('ERROR', 'Something went wrong');

say $logger1.get-logs();  # Contains both messages
say $logger1 === $logger2;  # True - same instance
```

The singleton pattern ensures a single instance using a class variable  
and custom constructor. The `//=` operator creates the instance only  
if it doesn't exist, maintaining single-instance behavior.  

## Factory methods

Creating objects through factory patterns for flexible instantiation.  

```raku
class Animal {
    has $.name;
    has $.species;
    has $.sound;
    
    method make-sound() {
        "$.name the $.species says $.sound";
    }
}

class AnimalFactory {
    method create-dog($name) {
        Animal.new(:$name, species => 'Dog', sound => 'Woof');
    }
    
    method create-cat($name) {
        Animal.new(:$name, species => 'Cat', sound => 'Meow');
    }
    
    method create-animal($type, $name) {
        given $type {
            when 'dog' { self.create-dog($name) }
            when 'cat' { self.create-cat($name) }
            default { die "Unknown animal type: $type" }
        }
    }
}

my $factory = AnimalFactory.new();
my $dog = $factory.create-dog('Buddy');
my $cat = $factory.create-animal('cat', 'Whiskers');

say $dog.make-sound();
say $cat.make-sound();
```

Factory methods encapsulate object creation logic, providing  
meaningful constructor alternatives. They're especially useful when  
object creation involves complex logic or multiple initialization  
strategies.  

## Operator overloading

Defining custom operators for class instances.  

```raku
class Vector {
    has $.x;
    has $.y;
    
    method new($x, $y) {
        self.bless(:$x, :$y);
    }
    
    multi method infix:<+>(Vector $other) {
        Vector.new($.x + $other.x, $.y + $other.y);
    }
    
    multi method infix:<*>(Numeric $scalar) {
        Vector.new($.x * $scalar, $.y * $scalar);
    }
    
    method Str() {
        "($.x, $.y)";
    }
    
    method magnitude() {
        sqrt($.x² + $.y²);
    }
}

my $v1 = Vector.new(3, 4);
my $v2 = Vector.new(1, 2);
my $v3 = $v1 + $v2;
my $v4 = $v1 * 2;

say $v3;  # (4, 6)
say $v4;  # (6, 8)
```

Operator overloading uses `multi method` with operator names like  
`infix:<+>`. This enables natural mathematical notation for custom  
types, making code more readable and intuitive for domain objects.  

## Class serialization

Converting objects to and from serialized formats.  

```raku
role Serializable {
    method to-hash() {
        my %data;
        for self.^attributes -> $attr {
            my $name = $attr.name.substr(2);  # Remove '$!' or '$.'
            %data{$name} = $attr.get_value(self);
        }
        %data;
    }
    
    method from-hash(%data) {
        self.new(|%data);
    }
}

class Person does Serializable {
    has $.name;
    has $.age;
    has $.email;
    
    method to-json() {
        use JSON::Fast;
        to-json(self.to-hash());
    }
}

my $person = Person.new(name => 'Alice', age => 30, email => 'alice@example.com');
my %hash = $person.to-hash();
my $json = $person.to-json();

say %hash;
say $json;

# Reconstruction
my $restored = Person.new(|%hash);
say $restored.name;
```

Serialization roles provide automatic conversion between objects and  
data formats. Use introspection to automatically extract attribute  
values, enabling easy persistence and data exchange.  

## Class comparison and equality

Implementing comparison and equality operations.  

```raku
class Point {
    has $.x;
    has $.y;
    
    method WHICH() {
        "Point|{$.x}|{$.y}";
    }
    
    multi method infix:<==>(Point $other) {
        $.x == $other.x && $.y == $other.y;
    }
    
    multi method infix:«<»(Point $other) {
        self.distance-from-origin < $other.distance-from-origin;
    }
    
    method distance-from-origin() {
        sqrt($.x² + $.y²);
    }
    
    method Str() {
        "($.x, $.y)";
    }
}

my $p1 = Point.new(x => 3, y => 4);
my $p2 = Point.new(x => 3, y => 4);
my $p3 = Point.new(x => 1, y => 1);

say $p1 == $p2;     # True (same coordinates)
say $p1 === $p2;    # False (different objects)
say $p3 < $p1;      # True (closer to origin)

my %points = $p1 => 'first', $p2 => 'second';
say %points.keys;   # Uses WHICH for hash keys
```

The `WHICH` method provides object identity for hash keys and set  
membership. Comparison operators enable sorting and equality testing  
based on meaningful object properties rather than memory addresses.  

## Class documentation

Documenting classes with Pod and embedded documentation.  

```raku
#| A class representing a bank account with basic operations
class BankAccount {
    #| The current balance of the account
    has Numeric $.balance is rw = 0;
    
    #| The account number (read-only)
    has Str $.account-number;
    
    #| Deposit money into the account
    #| Returns the new balance
    method deposit(Numeric $amount where * > 0 --> Numeric) {
        $.balance += $amount;
    }
    
    #| Withdraw money from the account
    #| Returns True if successful, False if insufficient funds
    method withdraw(Numeric $amount where * > 0 --> Bool) {
        return False if $amount > $.balance;
        $.balance -= $amount;
        True;
    }
    
    #| Get account summary information
    method summary() {
        "Account {$.account-number}: Balance \${$.balance}";
    }
}

=begin pod

=head1 BankAccount

A simple bank account implementation for demonstration purposes.

=head2 Example Usage

    my $account = BankAccount.new(account-number => '12345');
    $account.deposit(100);
    $account.withdraw(25);
    say $account.summary();

=end pod
```

Use `#|` for single-line documentation and `=begin pod`/`=end pod`  
blocks for comprehensive documentation. Good documentation includes  
purpose, parameters, return values, and usage examples.  

## Class testing patterns

Writing effective tests for class behavior.  

```raku
use Test;

class Calculator {
    has $.value = 0;
    
    method add($n) { Calculator.new(value => $.value + $n) }
    method multiply($n) { Calculator.new(value => $.value * $n) }
    method result() { $.value }
}

# Test basic functionality
my $calc = Calculator.new(value => 10);
is $calc.result(), 10, 'Initial value set correctly';

# Test method chaining
my $result = $calc.add(5).multiply(2);
is $result.result(), 30, 'Method chaining works';

# Test immutability
my $original = Calculator.new(value => 5);
my $modified = $original.add(3);
is $original.result(), 5, 'Original object unchanged';
is $modified.result(), 8, 'New object has correct value';

# Test edge cases
my $zero-calc = Calculator.new();
is $zero-calc.result(), 0, 'Default value is zero';

done-testing;
```

Test classes thoroughly including construction, method behavior,  
immutability, edge cases, and error conditions. Use descriptive test  
names and test both positive and negative scenarios.  

## Performance considerations

Optimizing class design for better performance.  

```raku
# Efficient attribute access
class FastPoint {
    has num $.x;  # Native numeric type
    has num $.y;  # Native numeric type
    
    method distance-squared(FastPoint $other) {
        my num $dx = $.x - $other.x;
        my num $dy = $.y - $other.y;
        $dx * $dx + $dy * $dy;
    }
}

# Lazy evaluation for expensive computations
class DataProcessor {
    has $.data;
    has $!processed-data;
    
    method processed-data() {
        $!processed-data //= do {
            say "Computing expensive operation...";
            $.data.map(* * 2).cache;
        }
    }
}

# Object pooling for frequent creation
class ObjectPool {
    my @pool;
    
    method acquire() {
        @pool.pop // self.bless();
    }
    
    method release($obj) {
        @pool.push($obj) if @pool.elems < 100;
    }
}
```

Use native types (`num`, `int`) for better performance. Implement  
lazy evaluation with `//=` for expensive computations. Consider  
object pooling for frequently created/destroyed objects.  

## Advanced class features

Exploring sophisticated class capabilities and meta-programming.  

```raku
# Custom metamodel
class MetaClass is Metamodel::ClassHOW {
    method add-logging($obj, $name) {
        my $original-method = $obj.^lookup($name);
        my $wrapper = method (|args) {
            say "Calling $name with {args.perl}";
            my $result = $original-method(self, |args);
            say "Result: {$result.perl}";
            $result;
        };
        $obj.^add_method($name, $wrapper);
    }
}

# Trait for automatic validation
multi trait_mod:<is>(Attribute $attr, :$validated!) {
    $attr.set_build(-> $, $value {
        die "Validation failed for {$attr.name}" unless $validated($value);
        $value;
    });
}

class Person {
    has $.name is validated(*.chars > 0);
    has $.age is validated(0 <= * <= 150);
    has $.email is validated(* ~~ /@/);
}

# Proxy class for method interception
class MethodProxy {
    has $.target;
    
    method FALLBACK($name, |args) {
        say "Intercepting call to $name";
        $.target."$name"(|args);
    }
}

my $person = Person.new(name => 'Alice', age => 30, email => 'alice@example.com');
my $proxy = MethodProxy.new(target => $person);
# All method calls are intercepted
```

Advanced features include custom metamodels for behavior modification,  
traits for automatic attribute processing, and proxy classes for  
method interception. These enable powerful meta-programming patterns.