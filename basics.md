# Basics

This chapter covers 30 fundamental Raku examples designed to help you get  
started with the language quickly. These examples demonstrate the most  
common Raku syntax and idioms, progressing from simple concepts to more  
sophisticated features. Each example includes clear explanations to help  
you understand the underlying concepts and apply them in your own code.  

## Hello World

The traditional first program in any language.  

```raku
say "Hello, World!";
```

The `say` function outputs text followed by a newline. It's one of the  
most commonly used functions in Raku for displaying output.  

## Variables with sigils

Raku uses sigils to indicate variable container types.  

```raku
my $name = "Alice";      # Scalar variable
my @colors = "red", "green", "blue";  # Array variable  
my %person = name => "Bob", age => 25;  # Hash variable

say $name;
say @colors[1];          # green
say %person<name>;       # Bob
```

Sigils (`$`, `@`, `%`) make variable types visually distinct and help  
the compiler optimize code. Scalars hold single values, arrays hold  
ordered lists, and hashes hold key-value pairs.  

## Basic data types

Raku's fundamental data types for different kinds of values.  

```raku
my $text = "Hello";      # Str
my $number = 42;         # Int
my $decimal = 3.14;      # Rat (rational number)
my $flag = True;         # Bool
my $nothing = Nil;       # Nil (absence of value)

say $text.WHAT;          # (Str)
say $number.WHAT;        # (Int)
say $decimal.WHAT;       # (Rat)
```

The `.WHAT` method shows the type of any value. Raku has rich type  
system but allows you to start without explicit type declarations.  

## Array creation and access

Different ways to create and access array elements.  

```raku
my @fruits = "apple", "banana", "cherry";
my @numbers = 1, 2, 3, 4, 5;
my @mixed = "text", 42, True;
my @range = 1..10;

say @fruits[0];          # apple
say @numbers[2];         # 3
say @range[*-1];         # 10 (last element)
say @mixed.elems;        # 3 (number of elements)
```

Arrays can contain any type of values and are created by listing  
elements separated by commas. Access elements using square brackets  
with zero-based indexing.  

## Hash creation and access

Creating and working with key-value data structures.  

```raku
my %scores = Alice => 95, Bob => 87, Carol => 92;
my %config = :host<localhost>, :port(8080), :ssl;

say %scores<Alice>;      # 95
say %scores{'Bob'};      # 87
say %config<host>;       # localhost
say %config.keys;        # (ssl host port)
```

Hashes store key-value pairs and can be created using the fat arrow  
`=>` operator or colon notation. Access values using angle brackets  
`<>` or curly braces `{}`.  

## Arithmetic operations

Basic mathematical operations with numbers.  

```raku
my $a = 10;
my $b = 3;

say $a + $b;             # 13 (addition)
say $a - $b;             # 7 (subtraction)
say $a * $b;             # 30 (multiplication)
say $a / $b;             # 3.333333 (division)
say $a div $b;           # 3 (integer division)
say $a % $b;             # 1 (modulo)
say $a ** $b;            # 1000 (exponentiation)
```

Raku provides all standard arithmetic operators. Division `/` returns  
rational numbers by default, while `div` performs integer division.  
The `**` operator is used for exponentiation.  

## String operations

Common operations for working with text.  

```raku
my $str1 = "Hello";
my $str2 = "World";

say $str1 ~ " " ~ $str2;      # Hello World (concatenation)
say "$str1 $str2";            # Hello World (interpolation)
say $str1 x 3;                # HelloHelloHello (repetition)
say $str1.chars;              # 5 (character count)
say $str1.uc;                 # HELLO (uppercase)
say $str2.lc;                 # world (lowercase)
```

The `~` operator concatenates strings, `x` repeats strings, and  
methods like `.uc` and `.lc` transform case. String interpolation  
with `"$variable"` is often more readable than concatenation.  

## Comparison operators

Comparing values for equality and ordering.  

```raku
my $x = 10;
my $y = 20;
my $str = "hello";

say $x == $y;            # False (numeric equality)
say $x != $y;            # True (numeric inequality)  
say $x < $y;             # True (less than)
say $x > $y;             # False (greater than)
say $str eq "hello";     # True (string equality)
say $str ne "world";     # True (string inequality)
```

Use `==` and `!=` for numeric comparisons, `eq` and `ne` for string  
comparisons. The `<`, `>`, `<=`, `>=` operators work for both  
numbers and strings (lexicographic order).  

## Logical operators

Combining boolean expressions with logical operations.  

```raku
my $a = True;
my $b = False;
my $x = 10;

say ($a and $b);         # False
say ($a or $b);          # True  
say not $a;              # False
say $a && $b;            # False (alternative syntax)
say $a || $b;            # True (alternative syntax)
say $x > 5 && $x < 15;   # True (combining conditions)
```

Logical operators `and`/`&&`, `or`/`||`, and `not`/`!` combine  
boolean values. The word forms have lower precedence than symbolic  
forms, useful for control flow.  

## Conditional statements

Making decisions in your code with if/elsif/else.  

```raku
my $score = 85;

if $score >= 90 {
    say "Excellent!";
} elsif $score >= 70 {
    say "Good job!";
} else {
    say "Keep trying!";
}

# Postfix conditional
say "High score!" if $score > 80;
```

Conditional statements control program flow based on boolean  
expressions. Postfix conditionals provide a concise way to execute  
single statements conditionally.  

## For loops

Iterating over sequences and collections.  

```raku
# Loop over range
for 1..5 -> $i {
    say "Number: $i";
}

# Loop over array
my @colors = "red", "green", "blue";
for @colors -> $color {
    say "Color: $color";
}

# Loop with index
for @colors.kv -> $index, $value {
    say "$index: $value";
}
```

For loops use the arrow syntax `->` to declare loop variables.  
The `.kv` method provides both keys (indices) and values for  
simultaneous iteration.  

## While loops

Repeating code while a condition is true.  

```raku
my $count = 0;
while $count < 5 {
    say "Count: $count";
    $count++;
}

# Until loop (opposite of while)
my $num = 10;
until $num <= 0 {
    say $num;
    $num -= 2;
}
```

While loops continue executing while the condition is true. Until  
loops continue while the condition is false. Both check the condition  
before each iteration.  

## Loop statement

The general-purpose loop construct with initialization, condition, and increment.  

```raku
loop (my $i = 0; $i < 5; $i++) {
    say "Iteration: $i";
}

# Infinite loop with explicit exit
loop {
    my $input = prompt("Enter 'quit' to exit: ");
    last if $input eq 'quit';
    say "You entered: $input";
}
```

The `loop` statement provides C-style loop syntax with three parts:  
initialization, condition, and increment. Use `last` to exit loops  
early and `next` to skip to the next iteration.  

## Ranges

Generating sequences of consecutive values.  

```raku
my $range1 = 1..10;      # 1 to 10 inclusive
my $range2 = 1^..^10;    # 1 to 10 exclusive
my $range3 = 'a'..'z';   # letters a to z

say $range1.list;        # (1 2 3 4 5 6 7 8 9 10)
say $range2.list;        # (2 3 4 5 6 7 8 9)
say $range3[0..2];       # (a b c)

for 1..3 -> $n { say $n }  # Use directly in loops
```

Ranges represent sequences between two endpoints. Use `..` for  
inclusive ranges, `^..` to exclude the start, `..^` to exclude  
the end, and `^..^` to exclude both endpoints.  

## Basic subroutines

Creating reusable functions to organize code.  

```raku
sub greet($name) {
    return "Hello, $name!";
}

sub add($a, $b) {
    $a + $b;    # implicit return
}

sub say-hello {
    say "Hello from subroutine!";
}

say greet("Alice");      # Hello, Alice!
say add(5, 3);           # 8
say-hello;               # Hello from subroutine!
```

Subroutines encapsulate reusable code. Parameters are listed in  
parentheses, and the last expression is automatically returned.  
Explicit `return` statements are optional.  

## Pattern matching with given/when

Elegant alternative to long if/elsif chains.  

```raku
my $day = "Monday";

given $day {
    when "Monday"    { say "Start of work week" }
    when "Friday"    { say "TGIF!" }
    when "Saturday" | "Sunday" { say "Weekend!" }
    default          { say "Regular day" }
}

# Pattern matching with numbers
my $num = 42;
given $num {
    when 0..10   { say "Small number" }
    when 11..100 { say "Medium number" }
    default      { say "Large number" }
}
```

The `given/when` construct provides powerful pattern matching.  
Each `when` clause can match values, ranges, types, or complex  
patterns using smart matching.  

## Regular expressions basics

Pattern matching and text processing with regex.  

```raku
my $text = "The year 2023 was great";

# Basic matching
if $text ~~ /\d+/ {
    say "Found numbers: $/";
}

# Capture groups
if $text ~~ /year \s+ (\d+)/ {
    say "Year: $0";          # 2023
}

# Substitution
my $new_text = $text.subst(/\d+/, "2024");
say $new_text;               # The year 2024 was great
```

Regular expressions use the `~~` operator for matching. Captures are  
stored in `$/` (full match) and `$0`, `$1`, etc. (groups). The  
`.subst` method replaces matched patterns.  

## File reading

Reading content from files safely.  

```raku
# Read entire file
if "/etc/hostname".IO.f {
    my $content = "/etc/hostname".IO.slurp;
    say "Hostname: $content.chomp()";
}

# Read line by line
if "/etc/passwd".IO.f {
    for "/etc/passwd".IO.lines[0..2] -> $line {
        say "Line: $line";
    }
}

# Handle missing files
try {
    my $data = "nonexistent.txt".IO.slurp;
    CATCH {
        default { say "File not found or error reading" }
    }
}
```

The `.IO` method creates file objects with methods like `.slurp`  
(read all), `.lines` (read lines), and `.f` (file exists check).  
Use `try/CATCH` for error handling.  

## Exception handling

Managing errors gracefully with try/CATCH blocks.  

```raku
# Basic exception handling
try {
    my $result = 10 / 0;
    CATCH {
        when X::Numeric::DivideByZero {
            say "Cannot divide by zero!";
        }
        default {
            say "Unknown error: $_";
        }
    }
}

# Multiple exception types
try {
    die "Custom error";
    CATCH {
        when X::AdHoc { say "Custom error caught" }
        default { say "Other error: $_.message()" }
    }
}
```

The `try/CATCH` construct handles exceptions. Different exception  
types can be caught with specific `when` clauses. Use `die` to  
throw custom exceptions.  

## Basic classes

Creating simple classes with attributes and methods.  

```raku
class Person {
    has $.name;
    has $.age;
    
    method greet {
        say "Hi, I'm $.name and I'm $.age years old";
    }
}

my $person = Person.new(name => "Alice", age => 30);
$person.greet;           # Hi, I'm Alice and I'm 30 years old
say $person.name;        # Alice
```

Classes are defined with the `class` keyword. Attributes use `has`  
with the `$.` twigil for read-only public access. Methods are  
defined with the `method` keyword.  

## Method calls

Different ways to call methods on objects.  

```raku
my $text = "hello world";

# Standard method calls
say $text.uc;            # HELLO WORLD
say $text.chars;         # 11
say $text.words.elems;   # 2

# Method chaining  
say $text.uc.split(' ').join('-');  # HELLO-WORLD

# Indirect method calls
say uc($text);           # HELLO WORLD (function style)
```

Methods can be called with dot notation, chained together for  
fluent interfaces, or used as functions. Chaining allows concise  
transformation pipelines.  

## Array operations

Essential operations for working with arrays.  

```raku
my @numbers = 1, 3, 2, 5, 4;

say @numbers.sort;       # (1 2 3 4 5)
say @numbers.reverse;    # (4 5 2 3 1)
say @numbers.elems;      # 5 (count)
say @numbers.sum;        # 15
say @numbers.max;        # 5
say @numbers.min;        # 1

@numbers.push(6);        # Add to end
@numbers.unshift(0);     # Add to beginning
say @numbers;            # [0 1 3 2 5 4 6]
```

Arrays have many built-in methods for sorting, counting, finding  
extrema, and modifying contents. These methods make array  
manipulation concise and readable.  

## Hash operations

Working with hash data structures efficiently.  

```raku
my %scores = Alice => 95, Bob => 87, Carol => 92;

say %scores.keys;        # (Alice Bob Carol)
say %scores.values;      # (95 87 92)
say %scores.pairs;       # (Alice => 95 Bob => 87 Carol => 92)

%scores<David> = 88;     # Add new key-value pair
%scores<Alice>:delete;   # Remove a key

say %scores.elems;       # 3 (count of pairs)
```

Hashes provide methods to access keys, values, and pairs. Adding  
and removing elements is straightforward with assignment and the  
`:delete` adverb.  

## String interpolation

Embedding expressions within strings for dynamic content.  

```raku
my $name = "Alice";
my $age = 30;
my @hobbies = "reading", "coding", "hiking";

say "My name is $name";
say "I am $age years old";
say "Next year I'll be {$age + 1}";
say "I have {+@hobbies} hobbies";
say "My hobbies: {@hobbies.join(', ')}";
```

String interpolation allows variables and expressions within double  
quotes. Use curly braces `{}` for complex expressions or to  
disambiguate variable boundaries.  

## Functional programming basics

Using map, grep, and sort for data transformation.  

```raku
my @numbers = 1..10;

# Transform with map
my @doubled = @numbers.map: * * 2;
say @doubled;            # [2 4 6 8 10 12 14 16 18 20]

# Filter with grep
my @evens = @numbers.grep: * %% 2;
say @evens;              # [2 4 6 8 10]

# Sort with custom criteria
my @words = "apple", "Banana", "cherry";
my @sorted = @words.sort: *.lc;
say @sorted;             # [apple Banana cherry]
```

Functional methods like `map`, `grep`, and `sort` transform data  
without modifying original arrays. The whatever star `*` creates  
anonymous functions for concise operations.  

## Type constraints

Adding type safety with explicit type declarations.  

```raku
my Int $count = 10;
my Str $message = "Hello";
my Bool $active = True;
my Rat $price = 19.99;

# Function with typed parameters
sub calculate-tax(Rat $amount, Rat $rate --> Rat) {
    $amount * $rate;
}

say calculate-tax(100.0, 0.08);  # 8

# This would cause a type error:
# $count = "not a number";
```

Type constraints ensure variables only accept values of specific  
types. Function signatures can specify parameter and return types  
for additional safety and documentation.  

## Multiple dispatch

Defining functions that behave differently based on parameter types.  

```raku
multi sub process(Int $n) {
    say "Processing integer: $n";
}

multi sub process(Str $s) {
    say "Processing string: $s";
}

multi sub process(@arr) {
    say "Processing array with {+@arr} elements";
}

process(42);             # Processing integer: 42
process("hello");        # Processing string: hello
process([1, 2, 3]);      # Processing array with 3 elements
```

Multiple dispatch allows the same function name to have different  
implementations based on the types or values of arguments. Raku  
automatically selects the best matching candidate.  

## Slurpy parameters

Functions that accept variable numbers of arguments.  

```raku
sub sum(*@numbers) {
    @numbers.sum;
}

sub format-message($template, *@args) {
    sprintf($template, |@args);
}

say sum(1, 2, 3, 4, 5);           # 15
say sum();                        # 0
say format-message("Hello %s, you have %d messages", "Alice", 5);
```

Slurpy parameters use `*` to collect remaining arguments into an  
array. This allows functions to accept flexible numbers of  
arguments while maintaining type safety.  

## Named parameters

Functions with labeled arguments for clarity and flexibility.  

```raku
sub create-user(:$name!, :$age = 18, :$active = True) {
    say "User: $name, Age: $age, Active: $active";
}

sub connect(:$host = "localhost", :$port = 8080, :$ssl = False) {
    say "Connecting to $host:$port (SSL: $ssl)";
}

create-user(name => "Alice", age => 25);
connect(host => "example.com", ssl => True);
connect();  # Uses all defaults
```

Named parameters use `:$name` syntax. The `!` makes parameters  
required, and `=` provides default values. Named parameters can  
be passed in any order.  

## Command line arguments

Accessing and processing arguments passed to your program.  

```raku
sub MAIN($filename, :$verbose = False, :$count = 1) {
    say "Processing file: $filename";
    say "Verbose mode: $verbose" if $verbose;
    say "Count: $count";
}

# Usage examples:
# script.raku myfile.txt
# script.raku myfile.txt --verbose
# script.raku myfile.txt --count=5 --verbose
```

The `MAIN` subroutine automatically processes command line arguments.  
Positional parameters become required arguments, and named parameters  
become optional flags. Raku generates help text automatically.
