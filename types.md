# Types

Raku features a rich, flexible type system that combines static and dynamic  
typing capabilities. It includes built-in types for common data structures,  
support for custom types, gradual typing, and powerful type constraints.  
The type system enables both runtime safety and compile-time optimization  
while maintaining the flexibility needed for rapid development. This guide  
covers fundamental types, type checking, constraints, coercion, and advanced  
features that make Raku's type system unique and powerful.

## Basic type checking

Examining types of values using the WHAT method.  

```raku
my $number = 42;
my $text = 'hello';
my $flag = True;
my $decimal = 3.14;

say $number.WHAT;   # (Int)
say $text.WHAT;     # (Str)
say $flag.WHAT;     # (Bool)
say $decimal.WHAT;  # (Rat)
```

The `WHAT` method returns the type object of a value. Type objects are  
represented in parentheses and can be used for type comparisons and  
runtime type checking. Every value in Raku has a type.  

## Type identity comparison

Using type identity operators to check exact type matches.  

```raku
my $x = 12;
my $w = 'cup';

say 'integer' if $x.WHAT === Int;
say 'string' if $w.WHAT === Str;

# Alternative syntax
say 'number' if $x ~~ Int;
say 'text' if $w ~~ Str;
```

The `===` operator checks for type identity, while `~~` performs smart  
matching which includes type checking. Both are useful for different  
scenarios - `===` for exact type comparison, `~~` for more flexible  
type checking including inheritance relationships.  

## Type hierarchy checking

Testing inheritance relationships with the isa method.  

```raku
my @data = 'storm', 1, 1.3, True, 1/2;

for @data -> $e {
    say "integer" if $e.isa(Int);
    say "string" if $e.isa(Str);
    say "real" if $e.isa(Real);
    say "boolean" if $e.isa(Bool);
    say "rational" if $e.isa(Rat);
    say '-------';
}
```

The `isa` method checks if a value belongs to a type or any of its  
parent types in the inheritance hierarchy. This is useful for checking  
broader type categories like Numeric, Real, or Any.  

## Type names and introspection

Getting human-readable type names and detailed type information.  

```raku
my @vals = (1, 2, 3, 4);
my $word = "falcon";
my $n = 12;

say @vals.WHAT;     # (Array)
say $word.WHAT;     # (Str)
say $n.WHAT;        # (Int)

say @vals.^name;    # Array
say $word.^name;    # Str
say $n.^name;       # Int
```

The `^name` method provides the string representation of a type name  
without parentheses. The `^` operator accesses metaclass methods,  
enabling deeper introspection of type properties and capabilities.  

## Integer types

Working with integer values and their properties.  

```raku
my Int $count = 42;
my $auto = 100;  # Automatically typed as Int

say $count.WHAT;
say $auto.WHAT;

# Integer operations
my $sum = $count + $auto;
my $product = $count * 2;

say "Sum: $sum ({$sum.^name})";
say "Product: $product ({$product.^name})";
```

Integers in Raku are arbitrary precision by default. You can explicitly  
type variables with `Int` or let Raku infer the type automatically.  
Integer operations preserve the integer type when possible.  

## String types

String creation, manipulation, and type behavior.  

```raku
my Str $message = 'Hello, World!';
my $auto_str = "Raku";

say $message.WHAT;
say $auto_str.WHAT;

# String operations
my $combined = $message ~ " " ~ $auto_str;
my $uppercase = $message.uc;

say "Combined: $combined ({$combined.^name})";
say "Uppercase: $uppercase";
```

Strings are immutable sequences of characters. String operations create  
new string objects rather than modifying existing ones. Both single  
and double quotes create string literals with different interpolation  
behavior.  

## Boolean types

Boolean values and truthiness evaluation.  

```raku
my Bool $flag = True;
my $auto_bool = False;

say $flag.WHAT;
say $auto_bool.WHAT;

# Truthiness evaluation
say so True;    # True
say so False;   # False
say so "";      # False
say so " ";     # True
say so ();      # False
say so (1,);    # True
```

Raku has explicit `True` and `False` boolean values. The `so` operator  
converts any value to a boolean. Empty strings, empty lists, and  
undefined values are false; most other values are true.  

## Numeric types hierarchy

Understanding the numeric type hierarchy and relationships.  

```raku
my Int $integer = 42;
my Rat $rational = 1/3;
my Num $floating = 3.14e0;
my Complex $complex = 1+2i;

say $integer.isa(Real);     # True
say $rational.isa(Real);    # True
say $floating.isa(Real);    # True
say $complex.isa(Numeric);  # True

say $integer.isa(Numeric);  # True
say $rational ~~ Real;      # True
```

Raku's numeric hierarchy: `Numeric` is the root, `Real` excludes complex  
numbers, and specific types like `Int`, `Rat`, and `Num` provide exact  
semantics. This hierarchy enables polymorphic numeric operations.  

## Rational numbers

Working with exact fractional arithmetic using Rat types.  

```raku
my Rat $fraction = 1/3;
my $auto_rat = 22/7;  # Automatically Rat

say $fraction.WHAT;
say $auto_rat.WHAT;

say $fraction.nude;      # (1 3) - numerator and denominator
say $auto_rat.nude;      # (22 7)

# Rational arithmetic
my $sum = $fraction + $auto_rat;
say $sum;
say $sum.nude;
```

Rational numbers preserve exact fractional values without floating-point  
errors. The `nude` method returns the numerator and denominator. Rat  
arithmetic maintains precision for exact calculations.  

## Array types and element typing

Arrays and their relationship with contained element types.  

```raku
my @mixed = 1, 'hello', True, 3.14;
my Int @integers = 1, 2, 3, 4;
my Str @strings = 'a', 'b', 'c';

say @mixed.WHAT;      # (Array)
say @integers.WHAT;   # (Array[Int])
say @strings.WHAT;    # (Array[Str])

say @mixed[0].WHAT;   # (Int)
say @strings[0].WHAT; # (Str)
```

Arrays can be untyped (containing any elements) or typed (restricting  
element types). Typed arrays enforce type constraints on all elements,  
providing compile-time and runtime safety for homogeneous collections.  

## Hash types and value typing

Hashes with typed values and key constraints.  

```raku
my %mixed = name => 'Alice', age => 30, active => True;
my Int %scores = alice => 95, bob => 87, charlie => 92;
my Str %names = 1 => 'Alice', 2 => 'Bob', 3 => 'Charlie';

say %mixed.WHAT;    # (Hash)
say %scores.WHAT;   # (Hash[Int])
say %names.WHAT;    # (Hash[Str])

say %mixed<name>.WHAT;    # (Str)
say %scores<alice>.WHAT;  # (Int)
```

Hashes can constrain value types while keys are typically strings.  
Typed hashes ensure all values conform to the specified type, enabling  
safe operations on homogeneous data collections.  

## List types and immutability

Understanding List types and their immutable nature.  

```raku
my @array = 1, 2, 3, 4;
my $list = (1, 2, 3, 4);
my List $explicit = (5, 6, 7, 8);

say @array.WHAT;     # (Array)
say $list.WHAT;      # (List)
say $explicit.WHAT;  # (List)

# Lists are immutable
# $list[0] = 10;  # Error: Cannot modify immutable List
say $list[0];        # 1
```

Lists are immutable ordered collections, unlike arrays which are mutable.  
Parentheses create lists while square brackets or `@` sigils create  
arrays. Lists provide efficient read-only access to sequential data.  

## Range types and lazy evaluation

Working with Range objects and their lazy properties.  

```raku
my $range1 = 1..10;
my $range2 = 1^..^10;    # Exclusive range
my $range3 = 'a'..'z';

say $range1.WHAT;        # (Range)
say $range2.WHAT;        # (Range)
say $range3.WHAT;        # (Range)

say $range1.list;        # (1 2 3 4 5 6 7 8 9 10)
say $range2.list;        # (2 3 4 5 6 7 8 9)
say $range3[0..4];       # (a b c d e)
```

Ranges represent sequences between two endpoints. They're lazy by default,  
computing values only when accessed. Different range operators control  
endpoint inclusion: `..` (inclusive), `^..` (exclusive start),  
`..^` (exclusive end), `^..^` (both exclusive).  

## Type constraints with where clauses

Adding runtime validation to type constraints.  

```raku
my Int $positive where * > 0 = 42;
my Str $non_empty where *.chars > 0 = 'hello';
my Int $even where * %% 2 = 8;

sub validate-age(Int $age where 0 <= * <= 150) {
    "Age $age is valid";
}

say validate-age(25);
# validate-age(-5);  # Constraint violation
```

Where clauses add runtime constraints beyond basic type checking. The  
`*` placeholder represents the value being tested. Constraints are  
evaluated when values are assigned, providing domain-specific validation.  

## Subset types

Creating custom constrained types with subset declarations.  

```raku
subset PositiveInt of Int where * > 0;
subset Email of Str where /^ \w+ '@' \w+ '.' \w+ $/;
subset Grade of Int where 0 <= * <= 100;

my PositiveInt $count = 42;
my Email $contact = 'user@example.com';
my Grade $score = 95;

say $count.WHAT;     # (PositiveInt)
say $contact.WHAT;   # (Email)
say $score.WHAT;     # (Grade)
```

Subset types create named constraints that can be reused throughout  
your program. They inherit from their base type while adding specific  
validation rules, enabling domain-specific type safety.  

## Native types for performance

Using native types for better performance and memory efficiency.  

```raku
my int $native_int = 42;       # Native machine integer
my num $native_num = 3.14e0;   # Native floating point
my str $native_str = 'hello';   # Native string

say $native_int.WHAT;    # (int)
say $native_num.WHAT;    # (num)
say $native_str.WHAT;    # (str)

# Native operations are faster
my int @fast_array = 1, 2, 3, 4, 5;
say @fast_array.WHAT;    # (array[int])
```

Native types (lowercase names) map directly to machine representations  
for better performance. They have limited precision compared to  
arbitrary-precision types but offer significant speed advantages for  
numeric computations.  

## Enum types

Creating enumerated types with fixed sets of values.  

```raku
enum Color <Red Green Blue>;
enum Status (
    Pending => 1,
    Processing => 2,
    Complete => 3,
    Failed => 4
);

my Color $bg = Red;
my Status $current = Processing;

say $bg.WHAT;         # (Color)
say $current.WHAT;    # (Status)
say +$current;        # 2 (numeric value)
```

Enums create types with a fixed set of named values. They can auto-assign  
ordinal values or use explicit values. Enums provide type safety for  
categorical data and state machines.  

## Any and Mu types

Understanding the root types in Raku's type hierarchy.  

```raku
my $any_val = Any;
my $mu_val = Mu;

say $any_val.WHAT;     # (Any)
say $mu_val.WHAT;      # (Mu)

say 42.isa(Any);       # True
say 42.isa(Mu);        # True
say Any.isa(Mu);       # True

# Any is the default type
my $default;
say $default.WHAT;     # (Any)
```

`Mu` is the root of all types, while `Any` is the root of most normal  
types (excluding some special cases). Variables without explicit types  
default to `Any`. Understanding this hierarchy helps with polymorphic  
programming and type relationships.  

## Junction types

Working with superposition of values using junctions.  

```raku
my $any_junction = any(1, 2, 3);
my $all_junction = all(1, 2, 3);
my $one_junction = one(1, 2, 3);

say $any_junction.WHAT;    # (Junction)
say $all_junction.WHAT;    # (Junction)

# Junction comparisons
say 2 == $any_junction;    # True (2 matches any)
say 4 == $any_junction;    # False

my @values = 1, 2, 3, 4, 5;
say @values.grep(* == $any_junction);  # (1 2 3)
```

Junctions represent superpositions of values, enabling quantum-like  
operations. They're useful for complex conditional logic and pattern  
matching where you need to test against multiple values simultaneously.  

## Coercion and conversion

Converting between types using coercion methods.  

```raku
my $str_num = "42";
my $int_val = $str_num.Int;
my $rat_val = $str_num.Rat;

say $int_val.WHAT;     # (Int)
say $rat_val.WHAT;     # (Rat)

# Explicit coercion in signatures
sub process(Int() $value) {
    "Processed: $value";
}

say process("123");    # Automatically coerces string to Int
say process(45.67);    # Coerces Rat to Int
```

Raku provides coercion methods like `.Int`, `.Str`, `.Rat` for explicit  
type conversion. Coercion type constraints in signatures (like `Int()`)  
automatically convert compatible values to the required type.  

## Definedness and type checking

Working with defined and undefined values.  

```raku
my $defined = 42;
my $undefined;
my Int $typed_undef;

say $defined.defined;      # True
say $undefined.defined;    # False
say $typed_undef.defined;  # False

say $defined.WHAT;         # (Int)
say $undefined.WHAT;       # (Any)
say $typed_undef.WHAT;     # (Int)

# Type checks with definedness
say $defined ~~ Int:D;     # True (defined Int)
say $typed_undef ~~ Int:U; # True (undefined Int)
```

The `:D` and `:U` type smileys distinguish between defined and undefined  
values. This enables precise type checking that considers both type  
and definedness state, useful for optional parameters and validation.  

## Parametric types

Working with generic types that accept type parameters.  

```raku
my Array[Int] @matrix;
my Hash[Str] %config;
my Promise[Int] $future;

say @matrix.WHAT;      # (Array[Array[Int]])
say %config.WHAT;      # (Hash[Str])
say $future.WHAT;      # (Promise[Int])

# Creating parametric instances
@matrix = [1, 2, 3], [4, 5, 6];
%config = timeout => '30s', host => 'localhost';
```

Parametric types accept type parameters in square brackets, enabling  
generic programming with type safety. Common examples include  
`Array[T]`, `Hash[T]`, and `Promise[T]` where `T` is the parameter type.  

## Role types and composition

Understanding roles as types and their composition behavior.  

```raku
role Drawable {
    method draw() { "Drawing..." }
}

role Clickable {
    method click() { "Clicked!" }
}

class Button does Drawable does Clickable {
    has $.label;
}

my $btn = Button.new(label => 'OK');

say $btn.WHAT;              # (Button)
say $btn.isa(Drawable);     # True
say $btn.isa(Clickable);    # True
say $btn ~~ Drawable;       # True
```

Roles define reusable behavior that can be composed into classes.  
Objects that do roles are considered to be of those role types,  
enabling flexible polymorphism and multiple inheritance-like behavior.  

## Class types and inheritance

Working with class types and inheritance relationships.  

```raku
class Animal {
    has $.name;
    method speak() { "Some sound" }
}

class Dog is Animal {
    method speak() { "Woof!" }
}

my Animal $pet = Dog.new(name => 'Buddy');

say $pet.WHAT;           # (Dog)
say $pet.^name;          # Dog
say $pet.isa(Animal);    # True
say $pet.isa(Dog);       # True
say Animal ~~ $pet.WHAT; # False
```

Class inheritance creates is-a relationships in the type hierarchy.  
Subclass objects are instances of both their specific class and all  
parent classes, enabling polymorphic behavior and type compatibility.  

## Type captures and variables

Using type objects as first-class values and variables.  

```raku
my $int_type = Int;
my $str_type = Str;
my @types = Int, Str, Bool, Rat;

say $int_type.WHAT;      # (Int)
say $int_type.^name;     # Int

# Using type variables
sub create-default($type) {
    $type.new;
}

my $zero = create-default(Int);    # 0
my $empty = create-default(Str);   # ""

say $zero.WHAT;          # (Int)
say $empty.WHAT;         # (Str)
```

Type objects can be stored in variables and passed as arguments,  
enabling meta-programming and generic algorithms. Type objects have  
their own type (the type of Int is Int) and can be used to create  
instances dynamically.  

## Smart matching with types

Advanced pattern matching using type smart matching.  

```raku
my @values = 42, 'hello', True, 3.14, [1, 2, 3];

for @values -> $val {
    given $val {
        when Int     { say "$val is an integer" }
        when Str     { say "$val is a string" }
        when Bool    { say "$val is a boolean" }
        when Numeric { say "$val is numeric" }
        when Array   { say "$val is an array" }
        default      { say "$val is something else" }
    }
}
```

Smart matching with `~~` and `when` enables powerful type-based  
pattern matching. The `given`/`when` construct automatically uses  
smart matching, making type-based dispatch elegant and readable.  

## Type constraints in signatures

Advanced type constraint techniques for subroutine parameters.  

```raku
# Multiple type constraint
sub process(Int|Str $value) {
    "Processing: $value";
}

# Constraint with coercion
sub calculate(Numeric(Cool) $input) {
    $input * 2;
}

# Where clause with signature
sub fibonacci(Int $n where * >= 0) {
    $n <= 1 ?? $n !! fibonacci($n-1) + fibonacci($n-2);
}

say process(42);           # Works with Int
say process("text");       # Works with Str
say calculate("3.14");     # Coerces string to Numeric
say fibonacci(10);         # 55
```

Signature type constraints support unions (`|`), coercion (`()`), and  
where clauses for complex validation. These features enable precise  
parameter validation and automatic type conversion in subroutines.  

## Complex number types

Working with complex numbers and their mathematical operations.  

```raku
my Complex $complex1 = 1+2i;
my $complex2 = 3-4i;  # Automatically Complex
my $imaginary = 5i;

say $complex1.WHAT;     # (Complex)
say $complex2.WHAT;     # (Complex)
say $imaginary.WHAT;    # (Complex)

say $complex1.re;       # 1 (real part)
say $complex1.im;       # 2 (imaginary part)

my $sum = $complex1 + $complex2;
say $sum;               # 4-2i
```

Complex numbers support full mathematical operations and provide access  
to real and imaginary components. Raku handles complex arithmetic  
naturally, promoting real numbers to complex when needed.  

## Number types and precision

Understanding different numeric precisions and their use cases.  

```raku
my FatRat $high_precision = FatRat.new(22, 7);
my Num $float = 3.14159265359;
my $scientific = 6.022e23;

say $high_precision.WHAT;   # (FatRat)
say $float.WHAT;            # (Num)
say $scientific.WHAT;       # (Num)

# Precision comparison
my $pi_rat = 22/7;
my $pi_fatrat = FatRat.new(22, 7);
say $pi_rat.nude;           # (22 7)
say $pi_fatrat.nude;        # (22 7)
```

`FatRat` provides arbitrary precision rational arithmetic, while `Num`  
offers IEEE floating-point precision. Choose the appropriate type  
based on precision requirements versus performance needs.  

## Callable types and code objects

Understanding callable types and code references.  

```raku
my &square = sub ($x) { $x * $x };
my $doubler = { $_ * 2 };
my Callable $func = &say;

say &square.WHAT;    # (Sub)
say $doubler.WHAT;   # (Block)
say $func.WHAT;      # (Sub)

say &square(5);      # 25
say $doubler(3);     # 6

# Signature introspection
say &square.signature;
```

Callable types include `Sub`, `Block`, `Method`, and the general  
`Callable` type. Code objects are first-class values that can be  
stored, passed, and introspected for their signatures and behavior.  

## IO types and handles

File handles and IO-related type information.  

```raku
my $file_handle = $*IN;
my $out_handle = $*OUT;
my $err_handle = $*ERR;

say $file_handle.WHAT;  # (IO::Handle)
say $out_handle.WHAT;   # (IO::Handle)
say $err_handle.WHAT;   # (IO::Handle)

# IO::Path for file paths
my IO::Path $path = 'example.txt'.IO;
say $path.WHAT;         # (IO::Path)
```

IO types handle input/output operations. `IO::Handle` represents file  
handles, while `IO::Path` represents filesystem paths. These types  
provide methods for reading, writing, and path manipulation.  

## Date and time types

Working with temporal data types and their relationships.  

```raku
my DateTime $datetime = DateTime.now;
my Date $date = Date.today;
my Instant $instant = Instant.now;

say $datetime.WHAT;     # (DateTime)
say $date.WHAT;         # (Date)
say $instant.WHAT;      # (Instant)

# Duration calculations
my Duration $duration = Duration.new(60);
say $duration.WHAT;     # (Duration)

my $later = $datetime + $duration;
say $later.WHAT;        # (DateTime)
```

Temporal types handle dates, times, and durations. `DateTime` includes  
time zones, `Date` represents calendar dates, `Instant` represents  
precise moments, and `Duration` represents time spans.  

## Exception and failure types

Understanding error handling types in Raku.  

```raku
my Exception $error = Exception.new(message => 'Test error');
my $failure = fail 'Something went wrong';

say $error.WHAT;        # (Exception)
say $failure.WHAT;      # (Failure)

# Catching specific exception types
try {
    die "Division error";
    CATCH {
        when X::AdHoc { say "Caught ad-hoc exception: {.message}" }
        default { say "Unknown exception: {.^name}" }
    }
}
```

Exception types enable structured error handling. `Exception` is the  
base type, `Failure` represents soft failures, and specific exception  
types like `X::AdHoc` provide detailed error categorization.  

## Type equivalence and compatibility

Testing type equivalence and compatibility relationships.  

```raku
my Int $a = 42;
my Int $b = 84;
my Str $c = "hello";

# Value equivalence vs type equivalence
say $a eqv $b;          # False (different values)
say $a.WHAT eqv $b.WHAT;    # True (same type)
say $a.WHAT === $b.WHAT;    # True (identical type objects)

# Type compatibility
say Int ~~ Numeric;     # True (inheritance)
say Str ~~ Numeric;     # False (no relationship)

# Multiple dispatch compatibility
multi sub process(Int $x) { "Integer: $x" }
multi sub process(Str $x) { "String: $x" }

say process(42);        # "Integer: 42"
say process("text");    # "String: text"
```

Type equivalence uses `eqv` for deep comparison and `===` for identity.  
Type compatibility enables polymorphism through inheritance relationships  
and multiple dispatch based on parameter types.  

## Type system best practices

Guidelines for effective use of Raku's type system.  

```raku
# Use specific types for clarity
sub calculate-tax(Numeric $amount, Rat $rate --> Numeric) {
    $amount * $rate;
}

# Subset types for domain validation
subset EmailAddress of Str where /^ \w+ '@' \w+ '.' \w+ $/;
subset PositiveNumber of Numeric where * > 0;

# Gradual typing - start loose, add constraints
my $data = load-data();           # Any type initially
my Int @numbers = $data.list;     # Constrain when needed

# Use definedness checking
sub greet(Str:D $name) {          # Must be defined string
    "Hello, $name!";
}
```

Effective type usage includes: using specific types for documentation,  
creating subset types for domain constraints, applying gradual typing  
by adding constraints progressively, and using definedness checking  
to prevent null-related errors.  