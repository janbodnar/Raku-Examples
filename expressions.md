# Expressions

Expressions are combinations of values, variables, operators, and function calls  
that evaluate to a single value. Raku provides a rich expression system with  
powerful operators, method chaining, pattern matching, and functional programming  
constructs. From simple arithmetic to complex data transformations, expressions  
form the building blocks of Raku programs. This chapter explores the diverse  
types of expressions available in Raku, progressing from basic literals to  
advanced meta-programming constructs.

## Literal expressions

Basic values that evaluate to themselves.  

```raku
say 42;
say 3.14159;
say 'hello world';
say True;
say Nil;
```

Literals are the simplest expressions - numbers, strings, booleans, and special  
values like `Nil`. Integer literals can be written in decimal, hexadecimal  
(`0xFF`), octal (`0o77`), or binary (`0b1010`) formats.  

## Variable expressions

Accessing stored values through variable names.  

```raku
my $name = 'Alice';
my $age = 30;
my @colors = <red green blue>;

say $name;
say $age;
say @colors;
```

Variable expressions retrieve values from memory locations. The sigil (`$`, `@`,  
`%`) indicates the variable type and determines how the value is accessed and  
interpreted in different contexts.  

## Arithmetic expressions

Mathematical operations on numeric values.  

```raku
my $a = 10;
my $b = 3;

say $a + $b;    # Addition
say $a - $b;    # Subtraction
say $a * $b;    # Multiplication
say $a / $b;    # Division
say $a % $b;    # Modulo
say $a ** $b;   # Exponentiation
```

Arithmetic expressions combine numeric values using mathematical operators.  
Raku handles integer and floating-point arithmetic automatically, promoting  
types as needed to preserve precision.  

## String expressions

Operations that create or manipulate text data.  

```raku
my $first = 'Hello';
my $second = 'World';

say $first ~ ' ' ~ $second;           # Concatenation
say "$first $second";                 # Interpolation
say $first.uc;                        # Method call
say $first x 3;                       # Repetition
```

String expressions use concatenation (`~`), interpolation within double quotes,  
method calls, and repetition operators. Interpolation allows embedding variables  
and expressions directly within string literals.  

## Comparison expressions

Evaluating relationships between values.  

```raku
my $x = 5;
my $y = 10;

say $x == $y;     # Numeric equality
say $x != $y;     # Numeric inequality
say $x < $y;      # Less than
say $x > $y;      # Greater than
say $x <= $y;     # Less than or equal
say $x >= $y;     # Greater than or equal
```

Comparison expressions return boolean values (`True` or `False`) based on  
relationships between operands. Raku provides separate operators for numeric  
and string comparisons (`eq`, `ne`, `lt`, `gt`, `le`, `ge`).  

## Logical expressions

Combining boolean values with logical operators.  

```raku
my $a = True;
my $b = False;

say $a && $b;     # Logical AND
say $a || $b;     # Logical OR
say !$a;          # Logical NOT
say $a ^^ $b;     # Logical XOR
say $a and $b;    # Low-precedence AND
say $a or $b;     # Low-precedence OR
```

Logical expressions combine boolean values using AND, OR, NOT, and XOR operators.  
Raku provides both high-precedence symbolic operators and low-precedence word  
operators for different parsing contexts.  

## Assignment expressions

Storing values in variables and containers.  

```raku
my $count;
$count = 0;               # Basic assignment
$count += 5;              # Addition assignment
$count *= 2;              # Multiplication assignment
$count //= 10;            # Defined-or assignment

my @items = ();
@items[0] = 'first';      # Array element assignment
```

Assignment expressions store values in variables and modify existing data.  
Compound assignment operators combine arithmetic operations with assignment.  
The defined-or operator (`//=`) assigns only if the variable is undefined.  

## Method call expressions

Invoking methods on objects to perform operations.  

```raku
my $text = 'Hello World';
my @numbers = 1, 2, 3, 4, 5;

say $text.uc;             # Convert to uppercase
say $text.chars;          # Get character count
say @numbers.sum;         # Calculate sum
say @numbers.max;         # Find maximum
say @numbers.reverse;     # Reverse order
```

Method call expressions invoke operations on objects using dot notation.  
Methods can transform data, calculate results, or return information about  
the object's state without modifying it.  

## Range expressions

Creating sequences of consecutive values.  

```raku
say 1..10;                # Inclusive range
say 1^..10;               # Exclusive start
say 1..^10;               # Exclusive end
say 1^..^10;              # Exclusive both ends
say 'a'..'z';             # Character range
```

Range expressions create sequences using the range operator (`..`). The caret  
(`^`) excludes endpoints from the range. Ranges work with numbers, characters,  
and any type that supports succession.  

## Array indexing expressions

Accessing elements from arrays and lists.  

```raku
my @fruits = <apple banana cherry date>;

say @fruits[0];           # First element
say @fruits[1];           # Second element
say @fruits[*-1];         # Last element
say @fruits[0, 2];        # Multiple elements
say @fruits[1..3];        # Range slice
```

Array indexing expressions access elements using square brackets. The whatever  
star (`*`) represents the array length, allowing negative indexing. Multiple  
indices and ranges create slices of the original array.  

## Hash indexing expressions

Retrieving values from associative arrays.  

```raku
my %person = name => 'Alice', age => 30, city => 'Boston';

say %person<name>;        # Angle bracket access
say %person{'age'};       # Curly brace access
say %person<name age>;    # Multiple keys
```

Hash indexing expressions retrieve values using keys. Angle brackets provide  
shorthand notation for string keys, while curly braces allow computed keys  
and variable interpolation.  

## Conditional expressions

Selecting values based on boolean conditions.  

```raku
my $age = 25;
my $status = $age >= 18 ?? 'adult' !! 'minor';
say $status;

my $result = do if $age > 21 {
    'can drink alcohol'
} else {
    'cannot drink alcohol'
};
say $result;
```

The ternary operator (`??` `!!`) provides inline conditional selection. The  
`do` keyword converts statements into expressions, allowing complex conditional  
logic to return values for assignment.  

## List construction expressions

Creating collections of values.  

```raku
my $single = (42,);       # Single-element list
my $multiple = (1, 2, 3, 4, 5);
my $mixed = (1, 'hello', True, 3.14);
my $nested = ((1, 2), (3, 4), (5, 6));

say $single;
say $multiple;
say $mixed;
say $nested;
```

Parentheses create list expressions containing multiple values. Lists can hold  
mixed types and nested structures. A trailing comma distinguishes single-element  
lists from simple grouping parentheses.  

## Hash construction expressions

Building associative data structures.  

```raku
my $empty = ();
my $colors = (red => '#FF0000', green => '#00FF00', blue => '#0000FF');
my $mixed = (name => 'John', age => 30, active => True);
my $nested = (person => (name => 'Alice', age => 25));

say $colors;
say $mixed;
say $nested;
```

Hash construction uses key-value pairs connected by the fat arrow (`=>`).  
Hashes can contain mixed value types and nested structures, providing flexible  
data organization capabilities.  

## String interpolation expressions

Embedding computed values within strings.  

```raku
my $name = 'World';
my $count = 42;
my @items = <apple banana cherry>;

say "Hello, $name!";
say "Count: $count";
say "Items: @items[]";
say "Square: {$count ** 2}";
say "Time: {DateTime.now}";
```

String interpolation embeds variables and expressions within double-quoted  
strings. Arrays need empty brackets (`[]`) for interpolation. Curly braces  
allow complex expressions and method calls.  

## Pattern matching expressions

Testing values against patterns.  

```raku
my $text = 'abc123def';
my $number = 42;

say $text ~~ /\d+/;       # Regex match
say $number ~~ Int;       # Type match
say $number ~~ 1..100;    # Range match
say $text ~~ *.chars > 5; # Code match
```

The smart match operator (`~~`) tests values against patterns including regular  
expressions, types, ranges, and callable expressions. This provides powerful  
and flexible pattern matching capabilities.  

## Functional expressions

Transforming data with higher-order functions.  

```raku
my @numbers = 1, 2, 3, 4, 5;

say @numbers.map(* ** 2);         # Square each element
say @numbers.grep(* %% 2);        # Filter even numbers
say @numbers.reduce({ $^a + $^b }); # Sum all elements
say @numbers.sort;                # Sort in ascending order
```

Functional expressions apply operations to collections without explicit loops.  
Methods like `map`, `grep`, and `reduce` enable data transformation pipelines  
using lambda expressions and whatever stars.  

## Pipeline expressions

Chaining operations in sequence.  

```raku
my @words = <the quick brown fox jumps over lazy dog>;

my $result = @words
    ==> grep(*.chars > 3)
    ==> map(*.uc)
    ==> sort()
    ==> join(' ');

say $result;
```

Pipeline expressions use the feed operator (`==>`) to pass results from one  
operation to the next. This creates readable data transformation chains that  
process collections step by step.  

## Whatever star expressions

Creating lambda functions with placeholder syntax.  

```raku
my @numbers = 1, 2, 3, 4, 5;

say @numbers.map(* + 10);     # Add 10 to each
say @numbers.grep(* > 3);     # Filter greater than 3
say @numbers.sort(* cmp *);   # Sort with comparison
say @numbers.map(* ** * );    # Power of self
```

The whatever star (`*`) creates concise lambda functions. Each star becomes  
a parameter in the generated function. This provides elegant syntax for  
simple transformations and predicates.  

## Reduction expressions

Aggregating collections into single values.  

```raku
my @values = 1, 2, 3, 4, 5;

say [+] @values;          # Sum
say [*] @values;          # Product
say [max] @values;        # Maximum
say [~] <a b c d e>;      # Concatenation
say [||] (True, False, True); # Logical OR
```

Reduction expressions apply operators across all elements using square bracket  
notation. This provides concise syntax for common aggregation operations like  
sum, product, maximum, and logical combinations.  

## Junction expressions

Creating quantum-like superpositions of values.  

```raku
my $any_color = any(<red green blue>);
my $all_positive = all(1, 2, 3, 4, 5);

say 'red' eq $any_color;      # True
say 5 < $all_positive;        # False
say $all_positive > 0;        # True
```

Junctions create special values that represent multiple possibilities  
simultaneously. Operations on junctions test all contained values, enabling  
elegant multi-value comparisons and logical operations.  

## Coercion expressions

Converting values between different types.  

```raku
my $text = '42';
my $number = 3.14159;

say +$text;               # Numeric coercion
say ~$number;             # String coercion
say ?$text;               # Boolean coercion
say $text.Int;            # Explicit Int conversion
say $number.Str;          # Explicit Str conversion
```

Coercion expressions convert values between types using prefix operators or  
explicit method calls. Numeric (`+`), string (`~`), and boolean (`?`) coercion  
operators provide convenient type conversion.  

## Meta-operator expressions

Modifying operator behavior with meta-operators.  

```raku
my @a = 1, 2, 3;
my @b = 4, 5, 6;
my @numbers = 1, 2, 3, 4, 5;

say @a >>+<< @b;          # Hyperoperator addition
say @numbers».abs;        # Hyper method call
say [+] @numbers;         # Reduction operator
say @a Z+ @b;             # Zip operator
```

Meta-operators modify existing operators to work with lists and arrays.  
Hyperoperators (`>><<`) apply operations element-wise, while zip operators  
(`Z`) combine corresponding elements from multiple lists.  

## Anonymous function expressions

Creating unnamed functions for immediate use.  

```raku
my @numbers = 1, 2, 3, 4, 5;

say @numbers.map({ $_ * 2 });
say @numbers.grep(-> $n { $n %% 2 });
say @numbers.sort({ $^a <=> $^b });

my $adder = { $^a + $^b };
say $adder(3, 4);
```

Anonymous functions provide inline code blocks for transformations and  
predicates. The topic variable (`$_`), arrow syntax (`->`), and placeholder  
variables (`$^a`, `$^b`) offer different parameter binding styles.  

## Type constraint expressions

Restricting values to specific types or conditions.  

```raku
subset PositiveInt of Int where * > 0;
subset EmailStr of Str where /^ \w+ '@' \w+ '.' \w+ $/;

my PositiveInt $count = 5;
my EmailStr $email = 'user@example.com';

say $count.^name;
say $email ~~ EmailStr;
```

Type constraints create specialized types with validation rules. The `subset`  
keyword defines constrained types using `where` clauses that test values  
against conditions or patterns.  

## Sequence expressions

Generating infinite or finite sequences.  

```raku
say (1, 2, 4 ... 64);         # Geometric sequence
say (1, 1, * + * ... *)[^10]; # Fibonacci sequence
say ('a' ... 'z')[5..10];     # Letter sequence slice
say (1, 3, 5 ... *)[^10];     # Infinite odd numbers
```

Sequence expressions use the sequence operator (`...`) to generate values  
based on patterns. The whatever star (`*`) creates infinite sequences, while  
specific endpoints create finite sequences.  

## Postfix expressions

Applying operations after the primary expression.  

```raku
my $number = 5;
my @items = <a b c>;

say $number++;            # Post-increment
say ++$number;            # Pre-increment
say @items.elems;         # Method call
say "Value: $number" if $number > 3;  # Postfix conditional
```

Postfix expressions modify values after evaluation or apply conditional  
execution. Increment/decrement operators, method calls, and postfix  
conditionals provide concise expression syntax.  

## Binding expressions

Creating aliases and references to data.  

```raku
my $original = 42;
my $alias := $original;

$alias = 100;
say $original;            # 100

my @source = 1, 2, 3;
my @view := @source[1..2];
say @view;                # [2 3]
```

Binding expressions create aliases using the binding operator (`:=`) instead  
of copying values. Changes through the alias affect the original data,  
enabling efficient data sharing and modification.  

## Signature expressions

Defining parameter patterns for functions.  

```raku
sub greet($name, $age = 25, :$formal = False) {
    my $greeting = $formal ?? "Good day" !! "Hi";
    "$greeting, $name! You are $age years old.";
}

say greet('Alice');
say greet('Bob', 30);
say greet('Carol', :formal);
```

Signature expressions define how functions accept parameters with optional  
values, named parameters, and type constraints. This enables flexible and  
self-documenting function interfaces.  

## Exception expressions

Handling errors with try-catch expressions.  

```raku
my $result = try {
    my $value = prompt("Enter a number: ");
    $value.Int / 0;
    CATCH {
        when X::AdHoc { "Division by zero!" }
        default { "Unknown error: $_" }
    }
}

say $result // "Operation failed";
```

Exception expressions handle errors gracefully using `try` blocks and `CATCH`  
handlers. The result is `Nil` if an exception occurs, allowing fallback  
values with the defined-or operator.  

## Parallel expressions

Executing operations concurrently.  

```raku
my @urls = <https://raku.org https://perl.org>;

my @promises = @urls.map: -> $url {
    start {
        say "Fetching $url";
        # Simulated work
        sleep 1;
        "Content from $url";
    }
};

say await @promises;
```

Parallel expressions use `start` to create promises for concurrent execution.  
The `await` keyword waits for all promises to complete, enabling efficient  
parallel processing of independent operations.  

## Reactive expressions

Creating data streams and event handling.  

```raku
my $supplier = Supplier.new;
my $supply = $supplier.Supply;

$supply.tap(-> $value {
    say "Received: $value";
});

$supplier.emit(1);
$supplier.emit(2);
$supplier.emit(3);
$supplier.done;
```

Reactive expressions create data streams using supplies and suppliers.  
The `tap` method attaches handlers to process emitted values, enabling  
event-driven and reactive programming patterns.  

## Meta-programming expressions

Manipulating code at compile and runtime.  

```raku
my $method-name = 'uc';
my $text = 'hello';

say $text."$method-name"();   # Dynamic method call

BEGIN {
    say "Compile time: {DateTime.now}";
}

say "Runtime: {DateTime.now}";
```

Meta-programming expressions manipulate code structure and execution.  
Dynamic method calls, compile-time execution with `BEGIN`, and code  
generation enable powerful meta-programming capabilities.  

## Chained comparison expressions

Performing multiple comparisons in sequence.  

```raku
my $age = 25;
my $score = 85;

say 18 <= $age <= 65;         # Age range check
say 0 <= $score <= 100;       # Score validation
say 'a' le 'm' le 'z';        # String range
say 1 < 5 < 10 < 20;          # Multiple comparisons
```

Chained comparison expressions test multiple conditions simultaneously using  
consecutive comparison operators. All comparisons must be true for the entire  
expression to evaluate as true, enabling elegant range checking.  

## Hyperoperator expressions

Applying operations across data structures automatically.  

```raku
my @numbers = 1, 2, 3, 4, 5;
my @letters = <a b c d e>;

say @numbers »**» 2;          # Square all elements
say @letters».uc;             # Uppercase all strings
say @numbers >>+>> @numbers;  # Add arrays element-wise
say @numbers >>max<< @numbers.reverse; # Element-wise maximum
```

Hyperoperator expressions use `>>op<<` or `»op«` syntax to automatically  
apply operations across all elements of data structures. This provides  
vectorized operations similar to those found in mathematical languages.