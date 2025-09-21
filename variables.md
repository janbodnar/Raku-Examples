# Variables

Variables are fundamental building blocks in Raku programming that store and  
manipulate data. Raku uses sigils to distinguish between different variable  
types: scalars ($), arrays (@), and hashes (%). Variables can be typed or  
untyped, mutable or immutable, and support various scoping mechanisms. This  
chapter covers the essential concepts of variable declaration, assignment,  
binding, scoping, and advanced features that make Raku variables powerful  
and flexible for modern programming needs.

## Basic scalar variables

Simple variable declaration and assignment with the scalar sigil.  

```raku
my $name = 'Alice';
my $age = 30;
my $height = 5.6;
my $active = True;

say $name;
say $age;
say $height;
say $active;
```

Scalar variables use the `$` sigil and can hold any single value including  
strings, numbers, booleans, and objects. The `my` keyword declares a  
lexically scoped variable that exists within its current block.  

## Variable assignment vs binding

Understanding the difference between assignment and binding operations.  

```raku
# Assignment with =
my $value = 42;
say $value;

$value = 84;  # Can be reassigned
say $value;

# Binding with :=
my $reference := $value;
say $reference;

$value = 168;
say $reference;  # Shows 168, bound to same container

# Constant binding
my $constant := 'immutable';
# $constant = 'changed';  # Would cause error
```

Assignment (`=`) stores a value in a container that can be changed later.  
Binding (`:=`) creates an alias to the same container, so changes to either  
variable affect both. Binding to literals creates read-only references.  

## Array variables

Creating and working with array variables using the @ sigil.  

```raku
my @numbers = 1, 2, 3, 4, 5;
my @words = <apple banana cherry>;
my @mixed = 'text', 42, True, 3.14;

say @numbers;
say @words;
say @mixed;

# Array operations
@numbers.push(6);
say @numbers.elems;
say @numbers[0];
```

Array variables use the `@` sigil and store ordered collections of values.  
They are mutable by default and can contain elements of different types.  
The angle bracket notation `< >` creates word lists for easy initialization.  

## Hash variables

Creating and manipulating hash variables with the % sigil.  

```raku
my %person = (
    'name' => 'Bob',
    'age' => 25,
    'city' => 'New York'
);

my %colors = :red<#FF0000>, :green<#00FF00>, :blue<#0000FF>;

say %person;
say %person<name>;
say %colors<red>;

%person<occupation> = 'Developer';
say %person;
```

Hash variables use the `%` sigil and store key-value pairs for associative  
data access. The fat arrow `=>` and colon pair `:key<value>` syntaxes both  
create hash entries. Angle brackets provide convenient key access.  

## Variable interpolation

Embedding variables within string literals for dynamic text generation.  

```raku
my $name = 'Charlie';
my $age = 28;
my @hobbies = <reading coding hiking>;

say "$name is $age years old";
say "Name: $name, Age: $age";
say "Hobbies: @hobbies[]";  # Array interpolation with []
say "Next year $name will be {$age + 1}";  # Expression interpolation
```

String interpolation allows variables and expressions to be embedded directly  
in double-quoted strings. Arrays need `[]` to interpolate their contents.  
Curly braces `{}` enable complex expression evaluation within strings.  

## Typed variables

Declaring variables with specific type constraints for safety.  

```raku
my Int $count = 10;
my Str $message = 'Hello';
my Bool $flag = True;
my Num $pi = 3.14159;

say $count;
say $message;
say $flag;
say $pi;

# Type checking at assignment
# $count = 'text';  # Would cause type error
```

Type constraints ensure variables only accept values of specific types.  
Common types include `Int`, `Str`, `Bool`, `Num`, and `Rat`. Type  
violations are caught at runtime, providing safety and clarity.  

## Variable scoping

Different scoping mechanisms for controlling variable visibility.  

```raku
# Lexical scoping with my
my $lexical = 'block scope';

# Package scoping with our
our $package = 'package scope';

# State variables persist between calls
sub counter() {
    state $count = 0;
    ++$count;
}

say counter();  # 1
say counter();  # 2
say counter();  # 3
```

The `my` keyword creates lexically scoped variables visible only in their  
block. The `our` keyword creates package-scoped variables shared across  
modules. State variables maintain their value between function calls.  

## Default values and conditional assignment

Setting default values and conditional assignment operators.  

```raku
my $username;
$username //= 'guest';  # Assign if undefined
say $username;

my $counter = 0;
$counter ||= 1;  # Assign if false
say $counter;

my $config = %();
$config<timeout> //= 30;  # Default hash value
say $config;
```

The `//=` operator assigns only if the variable is undefined (Nil). The  
`||=` operator assigns only if the variable is false. These operators  
provide elegant ways to set default values without explicit conditions.  

## Variable aliasing

Creating aliases and references to existing variables.  

```raku
my @original = 1, 2, 3, 4, 5;
my @alias := @original;

@alias.push(6);
say @original;  # Shows [1 2 3 4 5 6]

# Aliasing hash elements
my %data = :a(1), :b(2), :c(3);
my $ref := %data<a>;
$ref = 10;
say %data<a>;  # Shows 10
```

Variable aliasing with `:=` creates references to the same underlying  
container. Changes through either the original variable or its alias  
affect the same storage location, enabling powerful data manipulation.  

## Constant variables

Creating immutable variables that cannot be changed after initialization.  

```raku
constant PI = 3.14159;
constant MAX_SIZE = 1000;
constant VERSION = 'v1.0.0';

say PI;
say MAX_SIZE;
say VERSION;

# Constants can be computed at compile time
constant SECONDS_PER_DAY = 24 * 60 * 60;
say SECONDS_PER_DAY;
```

Constants are declared with the `constant` keyword and cannot be modified  
after creation. They are evaluated at compile time when possible and  
provide a way to define unchanging values throughout the program.  

## Anonymous variables

Using anonymous variables for temporary or placeholder values.  

```raku
# Anonymous scalar
say $++;  # Auto-incrementing anonymous variable

# Anonymous array indexing
my @data = 10, 20, 30, 40, 50;
say @data[$++];  # Uses anonymous counter

# Anonymous variables in loops
for 1..5 {
    say "Iteration: $_";  # $_ is the topic variable
}

# Multiple anonymous variables
(1, 2, 3).map: { $^a * $^b * $^c };  # $^a, $^b, $^c are placeholders
```

Anonymous variables like `$_` (topic) and `$^a`, `$^b` (placeholders) provide  
convenient ways to work with temporary values. Auto-incrementing `$++`  
creates unique anonymous variables for counters and iterators.  

## Variable containers and values

Understanding the difference between containers and their values.  

```raku
my $container = 42;
say $container.VAR.^name;  # Shows container type
say $container.WHAT.^name;  # Shows value type

# Container assignment vs value assignment
my $a = 10;
my $b := $a;  # Bind to container
my $c = $a;   # Copy value

$a = 20;
say $b;  # Shows 20 (same container)
say $c;  # Shows 10 (different container)
```

Variables are containers that hold values. The `.VAR` method introspects  
the container while `.WHAT` examines the value. Understanding this  
distinction helps explain assignment versus binding behavior.  

## Dynamic variables

Using dynamic scoping and twigils for special variable access.  

```raku
# Dynamic variable with *
my $*DYNAMIC = 'outer value';

sub inner() {
    say $*DYNAMIC;  # Accesses dynamically scoped variable
}

sub middle() {
    my $*DYNAMIC = 'middle value';  # Shadows outer dynamic
    inner();
}

middle();  # Prints 'middle value'
inner();   # Prints 'outer value'
```

Dynamic variables use the `*` twigil and follow dynamic scoping rules,  
where the most recent assignment in the call stack is used. This enables  
context-dependent behavior across function calls.  

## Variable introspection

Examining variable properties and metadata at runtime.  

```raku
my Int $typed = 42;
my $untyped = 'hello';
my @array = 1, 2, 3;

say $typed.VAR.^name;     # Scalar container type
say $typed.WHAT.^name;    # Value type (Int)
say $untyped.WHAT.^name;  # Value type (Str)
say @array.WHAT.^name;    # Array type

# Check if variable is defined
say $typed.defined;       # True
say Nil.defined;          # False
```

Introspection methods reveal variable and value types at runtime. The  
`.VAR` method examines containers, `.WHAT` examines values, and  
`.defined` checks if a variable contains a meaningful value.  

## Variable destruction and cleanup

Understanding variable lifecycle and cleanup mechanisms.  

```raku
{
    my $temp = 'temporary value';
    say $temp;
}  # $temp goes out of scope here

# Variables with cleanup behavior
my $resource = open 'data.txt', :w;
$resource.say('Hello');
$resource.close;  # Explicit cleanup

# DESTROY method for custom cleanup
class Resource {
    method DESTROY() {
        say 'Resource cleaned up';
    }
}

{
    my $obj = Resource.new;
}  # DESTROY called when $obj goes out of scope
```

Variables are automatically cleaned up when they go out of scope. Objects  
can define DESTROY methods for custom cleanup behavior. Proper resource  
management ensures memory and system resources are released promptly.  

## Complex variable patterns

Advanced variable usage patterns for sophisticated data manipulation.  

```raku
# Multiple assignment
my ($x, $y, $z) = 1, 2, 3;
say "$x, $y, $z";

# Array slicing assignment
my @numbers = 1..10;
my ($first, $second, *@rest) = @numbers;
say "First: $first, Second: $second, Rest: @rest[]";

# Hash destructuring
my %person = :name<Alice>, :age(30), :city<Boston>;
my (:$name, :$age) := %person;
say "Name: $name, Age: $age";
```

Raku supports sophisticated variable patterns including multiple assignment,  
array destructuring with slurpy parameters (`*@rest`), and hash  
destructuring with named parameters. These patterns enable elegant  
data manipulation and extraction.  

## Variable performance considerations

Understanding performance implications of different variable patterns.  

```raku
# Efficient: Direct assignment
my @fast = 1..1000;

# Less efficient: Multiple operations
my @slow;
for 1..1000 -> $n {
    @slow.push($n);
}

# Container vs value semantics
my @original = 1, 2, 3;
my @copy = @original;       # Copies values
my @reference := @original; # References container

# Memory usage comparison
say @copy.elems;      # Same elements
say @reference.elems; # Same elements, shared storage
```

Variable performance depends on assignment patterns, container semantics,  
and memory usage. Direct assignment and container references are more  
efficient than incremental operations and value copying for large data.  

## Variable best practices

Guidelines for effective variable usage in Raku programming.  

```raku
# Use meaningful names
my $user_count = 50;           # Clear intent
my $temperature_celsius = 25;  # Explicit units

# Prefer typed variables for clarity
my Int $retry_count = 3;
my Str $error_message = 'Failed to connect';

# Use constants for fixed values
constant DEFAULT_TIMEOUT = 30;
constant MAX_RETRIES = 5;

# Scope variables appropriately
{
    my $local_temp = compute_value();
    process($local_temp);
}  # $local_temp not accessible outside
```

Good variable practices include descriptive naming, appropriate typing,  
using constants for fixed values, and proper scoping to limit variable  
lifetime and accessibility. These practices improve code readability  
and maintainability.  

## Variable constraints and validation

Using subset types and where clauses to validate variable contents.  

```raku
# Subset types for validation
subset PositiveInt of Int where * > 0;
subset Email of Str where * ~~ /\w+ '@' \w+ '.' \w+/;

my PositiveInt $age = 25;
my Email $contact = 'user@example.com';

say $age;
say $contact;

# Custom validation with where clauses
my $grade where 0 <= * <= 100 = 85;
my @even_numbers where .all %% 2 = 2, 4, 6, 8;

say $grade;
say @even_numbers;
```

Subset types create constrained versions of existing types with validation  
rules. The `where` clause specifies conditions that values must satisfy.  
These constraints are checked at assignment time, providing runtime safety.  

## Variable traits and attributes

Using traits to modify variable behavior and add metadata.  

```raku
# Variable traits for special behavior
my $counter is rw = 0;  # Explicitly read-write
my $readonly is readonly = 'constant';

# Default trait for automatic initialization
my @buffer is default([]);
say @buffer;  # Shows empty array

# Custom traits can be defined
subset NonEmpty of Str where *.chars > 0;
my NonEmpty $message is required;  # Must be assigned

# Variables with custom attributes
class Config {
    has $.database is required;
    has $.timeout is rw = 30;
    has $.debug is readonly = False;
}

my $config = Config.new(database => 'mydb');
say $config.database;
say $config.timeout;
```

Traits modify variable behavior: `is rw` makes variables explicitly mutable,  
`is readonly` prevents modification, `is default` provides automatic  
initialization. Custom traits and attributes enable sophisticated variable  
configuration and validation in classes and roles.  