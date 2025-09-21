# Subroutines

Subroutines are reusable code blocks that encapsulate functionality and can  
accept parameters, return values, and be called multiple times throughout a  
program. In Raku, subroutines offer powerful features including multiple  
dispatch, named parameters, type constraints, and flexible argument handling.  
They form the foundation of modular programming, enabling code organization,  
testing, and maintainability. This chapter explores subroutine definition,  
parameter handling, return mechanisms, and advanced features that make Raku  
subroutines exceptionally expressive and versatile for diverse programming  
tasks from simple calculations to complex data transformations.  

## Basic definition

Simple subroutine definition and calling syntax.  

```raku
sub greet {
    say "Hello, World!";
}

sub add-numbers {
    5 + 3;
}

greet();
say add-numbers();
```

Subroutines are defined with the `sub` keyword followed by a name. The body  
contains the code to execute. Subroutines can be called with or without  
parentheses. The last expression in a subroutine becomes its return value  
automatically, eliminating the need for explicit return statements in simple  
cases.  

## Parameters

Accepting input values through subroutine parameters.  

```raku
sub greet-person($name) {
    say "Hello, $name!";
}

sub multiply($a, $b) {
    $a * $b;
}

sub describe-person($name, $age) {
    say "$name is $age years old";
}

greet-person("Alice");
say multiply(4, 7);
describe-person("Bob", 25);
```

Parameters are declared in parentheses after the subroutine name. Variables  
inside the parameter list become available within the subroutine body.  
Multiple parameters are separated by commas. Arguments are passed in the same  
order as parameters are declared.  

## Return values

Explicit control over what values subroutines return.  

```raku
sub square($n) {
    return $n * $n;
}

sub max($a, $b) {
    if $a > $b {
        return $a;
    } else {
        return $b;
    }
}

sub get-status() {
    return "active", "logged-in", time;
}

say square(5);
say max(10, 7);
my ($status, $login, $timestamp) = get-status();
say $status;
```

The `return` statement explicitly specifies return values and exits the  
subroutine immediately. Multiple values can be returned as a list and  
unpacked into multiple variables. Early returns enable conditional logic  
without nested structures.  

## Type constraints

Adding type information to parameters and return values.  

```raku
sub add-integers(Int $a, Int $b --> Int) {
    $a + $b;
}

sub format-name(Str $first, Str $last --> Str) {
    "$first $last";
}

sub calculate-area(Numeric $radius --> Numeric) {
    π * $radius ** 2;
}

say add-integers(5, 3);
say format-name("John", "Doe");
say calculate-area(2.5);
```

Type constraints ensure parameters match expected types and document the  
subroutine interface. The `-->` syntax specifies the return type. Type  
violations cause runtime errors, improving code reliability and making  
intentions explicit for maintainability.  

## Optional parameters

Providing default values for parameters.  

```raku
sub greet-with-title($name, $title = "Mr.") {
    say "$title $name";
}

sub create-range($start, $end = 10, $step = 1) {
    $start, $start + $step ... $end;
}

sub log-message($message, $level = "INFO", $timestamp = now) {
    say "[$timestamp] $level: $message";
}

greet-with-title("Smith");
greet-with-title("Jones", "Dr.");
say create-range(1);
say create-range(5, 15);
log-message("System started");
```

Default parameter values are specified with `=` after the parameter name.  
Optional parameters must come after required parameters. When arguments are  
omitted, default values are used automatically. This enables flexible APIs  
with sensible defaults while allowing customization when needed.  

## Named parameters

Using named parameters for clearer subroutine calls.  

```raku
sub create-user(:$name, :$email, :$age = 18) {
    say "Creating user: $name ($email), age $age";
}

sub connect-database(:$host, :$port = 5432, :$ssl = False) {
    say "Connecting to $host:$port (SSL: $ssl)";
}

sub format-currency(:$amount, :$currency = 'USD', :$precision = 2) {
    sprintf "%.{$precision}f %s", $amount, $currency;
}

create-user(name => "Alice", email => "alice@example.com");
create-user(:name<Bob>, :email<bob@test.com>, :age(25));
connect-database(:host<localhost>, :ssl);
say format-currency(:amount(123.456), :currency<EUR>, :precision(3));
```

Named parameters use the `:` prefix in the parameter declaration. They can be  
called using the `name => value` syntax or the `:name<value>` shorthand.  
Boolean parameters can use `:name` for true values. Named parameters provide  
self-documenting code and allow arguments in any order.  

## Slurpy parameters

Collecting variable numbers of arguments.  

```raku
sub sum-all(*@numbers) {
    [+] @numbers;
}

sub join-with-separator($separator, *@items) {
    @items.join($separator);
}

sub print-report($title, *%data) {
    say "=== $title ===";
    for %data.kv -> $key, $value {
        say "$key: $value";
    }
}

say sum-all(1, 2, 3, 4, 5);
say join-with-separator(", ", "apple", "banana", "cherry");
print-report("System Status", 
    :cpu<45%>, :memory<67%>, :disk<23%>);
```

Slurpy parameters use `*` prefix to collect remaining arguments. `*@array`  
collects positional arguments into an array. `*%hash` collects named  
arguments into a hash. This enables variadic functions that accept flexible  
argument counts while maintaining type safety.  

## Multiple dispatch

Creating multiple subroutines with the same name but different signatures.  

```raku
multi sub format(Int $n) {
    "Integer: $n";
}

multi sub format(Str $s) {
    "String: '$s'";
}

multi sub format(Bool $b) {
    "Boolean: " ~ ($b ?? "true" !! "false");
}

multi sub distance(Int $x, Int $y) {
    sqrt($x² + $y²);
}

multi sub distance(Int $x1, Int $y1, Int $x2, Int $y2) {
    sqrt(($x2-$x1)² + ($y2-$y1)²);
}

say format(42);
say format("hello");
say format(True);
say distance(3, 4);
say distance(1, 1, 4, 5);
```

Multi subroutines use the `multi` keyword to create multiple versions with  
different parameter signatures. Raku automatically selects the best matching  
candidate based on argument types and count. This enables polymorphic  
behavior and clean APIs that handle different input types naturally.  

## Recursion

Subroutines that call themselves to solve problems.  

```raku
sub factorial($n) {
    return 1 if $n <= 1;
    $n * factorial($n - 1);
}

sub fibonacci($n) {
    return $n if $n <= 1;
    fibonacci($n - 1) + fibonacci($n - 2);
}

sub count-down($n) {
    return if $n <= 0;
    say $n;
    count-down($n - 1);
}

say factorial(5);
say fibonacci(8);
count-down(5);
```

Recursive subroutines solve problems by breaking them into smaller instances  
of the same problem. Base cases prevent infinite recursion by providing  
stopping conditions. Recursive solutions are elegant for mathematical  
sequences, tree traversal, and divide-and-conquer algorithms.  

## Closures

Subroutines that capture variables from their surrounding scope.  

```raku
sub make-counter($start = 0) {
    my $count = $start;
    return sub {
        ++$count;
    };
}

sub make-adder($increment) {
    return sub ($value) {
        $value + $increment;
    };
}

my $counter1 = make-counter();
my $counter2 = make-counter(10);
my $add5 = make-adder(5);

say $counter1();  # 1
say $counter1();  # 2
say $counter2();  # 11
say $add5(3);     # 8
```

Closures are anonymous subroutines that capture variables from their  
enclosing scope. The captured variables remain accessible even after the  
outer subroutine returns. This enables creating specialized functions,  
implementing callbacks, and building functional programming patterns.  

## Subroutine references

Storing and passing subroutines as values.  

```raku
sub double($n) { $n * 2 }
sub triple($n) { $n * 3 }
sub apply-operation(@numbers, &operation) {
    @numbers.map(&operation);
}

my &my-func = &double;
my @operations = &double, &triple;

say my-func(5);
say apply-operation([1, 2, 3, 4], &double);
say apply-operation([1, 2, 3, 4], &triple);

for @operations -> &op {
    say op(10);
}
```

Subroutine references use the `&` sigil to refer to subroutines as first-  
class values. They can be stored in variables, passed as arguments, and  
stored in collections. This enables higher-order functions, strategy  
patterns, and functional programming techniques.  

## Anonymous subroutines

Creating unnamed subroutines for immediate use.  

```raku
my @numbers = 1, 2, 3, 4, 5;

my @doubled = @numbers.map(sub ($n) { $n * 2 });
my @filtered = @numbers.grep(sub ($n) { $n % 2 == 0 });

my $processor = sub ($text) {
    $text.uc.subst(/\s+/, '_', :global);
};

say @doubled;
say @filtered;
say $processor("hello world example");
```

Anonymous subroutines are defined without names using the `sub` keyword  
followed immediately by parameters and body. They're useful for short  
operations passed to higher-order functions like map and grep, or for  
creating specialized functions on demand.  

## Where clauses

Adding runtime constraints to parameters.  

```raku
sub positive-sqrt($n where $_ > 0) {
    sqrt($n);
}

sub grade-letter($score where 0 <= $_ <= 100) {
    given $score {
        when $_ >= 90 { 'A' }
        when $_ >= 80 { 'B' }
        when $_ >= 70 { 'C' }
        when $_ >= 60 { 'D' }
        default { 'F' }
    }
}

sub even-number($n where $_ %% 2) {
    "$n is even";
}

say positive-sqrt(16);
say grade-letter(85);
say even-number(42);
```

Where clauses add runtime constraints to parameters beyond type checking.  
The `$_` variable refers to the argument value. Constraints can check ranges,  
mathematical properties, or any boolean condition. Failed constraints throw  
exceptions, ensuring subroutines receive valid inputs.  

## Traits

Adding metadata and behavior to subroutines.  

```raku
sub cached-fibonacci($n) is cached {
    return $n if $n <= 1;
    cached-fibonacci($n - 1) + cached-fibonacci($n - 2);
}

sub log-calls() is rw {
    state $count = 0;
    ++$count;
}

sub inline-operation($a, $b) is inline {
    $a + $b * 2;
}

# Using ENTER/LEAVE phasers
sub traced-function($input) {
    ENTER { say "Entering with: $input" }
    LEAVE { say "Leaving function" }
    
    $input.uc;
}

say cached-fibonacci(10);
say log-calls();
say traced-function("hello");
```

Traits modify subroutine behavior and provide metadata. `is cached` enables  
automatic memoization. `is rw` allows the subroutine to be used as an lvalue.  
`is inline` suggests compiler optimization. Phasers like ENTER and LEAVE  
execute code at specific points in subroutine execution.  

## Captures

Advanced parameter handling with capture objects.  

```raku
sub flexible-call(&func, |args) {
    say "Calling function with capture";
    func(|args);
}

sub make-call($target, *@pos, *%named) {
    my $capture = \(@pos, %named);
    $target(|$capture);
}

sub example-target($a, $b, :$option = 'default') {
    say "a=$a, b=$b, option=$option";
}

flexible-call(&example-target, 1, 2, :option<custom>);
make-call(&example-target, 10, 20, :option<test>);

# Signature binding
sub process-args(|args) {
    my ($first, $second, *@rest) := args;
    say "First: $first, Second: $second, Rest: @rest[]";
}

process-args(1, 2, 3, 4, 5);
```

Captures represent argument lists as first-class objects using the `\`  
operator. The `|` operator flattens captures back into arguments. This  
enables meta-programming, argument forwarding, and flexible parameter  
handling patterns for framework and library development.  

## Nested subroutines

Defining subroutines inside other subroutines.  

```raku
sub outer-function($data) {
    sub validate($item) {
        $item.defined && $item !~~ '';
    }
    
    sub process($item) {
        $item.uc.subst(/\s+/, '_', :global);
    }
    
    sub format-result(@items) {
        @items.join(', ');
    }
    
    my @valid = $data.grep(&validate);
    my @processed = @valid.map(&process);
    format-result(@processed);
}

sub create-validator($min-length = 3) {
    sub inner-validator($text) {
        $text.chars >= $min-length;
    }
    return &inner-validator;
}

say outer-function(['hello', '', 'world', 'test']);
my &validator = create-validator(5);
say validator('short');
say validator('long enough');
```

Nested subroutines are defined inside other subroutines, creating local  
scope and encapsulation. They have access to the outer subroutine's  
variables and parameters. Inner subroutines can be returned as closures,  
enabling factory patterns and specialized function creation.  

## Error handling

Managing errors and exceptions in subroutines.  

```raku
sub safe-divide($a, $b) {
    die "Division by zero" if $b == 0;
    $a / $b;
}

sub parse-number($text) {
    return Int($text);
    CATCH {
        when X::Str::Numeric {
            return Nil;
        }
    }
}

sub with-fallback($operation, $fallback) {
    try {
        return $operation();
    }
    return $fallback;
}

try {
    say safe-divide(10, 2);
    say safe-divide(10, 0);
}

say parse-number("123") // "invalid";
say parse-number("abc") // "invalid";

say with-fallback({ die "error" }, "default");
```

Error handling uses `die` to throw exceptions and `try`/`CATCH` blocks to  
handle them. The `CATCH` block can match specific exception types. The `try`  
statement returns Nil on exceptions. This enables robust subroutines that  
handle edge cases gracefully.  

## Documentation

Adding documentation to subroutines using Pod6.  

```raku
#| Calculate the factorial of a positive integer
#| Returns the factorial as an integer value
sub documented-factorial(
    Int $n where $_ >= 0  #= The positive integer input
    --> Int               #= The factorial result
) {
    return 1 if $n <= 1;
    $n * documented-factorial($n - 1);
}

=begin pod

=head2 Advanced Calculator

This subroutine performs complex mathematical operations
with proper error handling and type validation.

=end pod

sub calculate(
    Numeric $a,           #= First operand
    Numeric $b,           #= Second operand
    Str $operation        #= Operation: 'add', 'multiply', 'divide'
    --> Numeric           #= Calculation result
) {
    given $operation {
        when 'add'      { $a + $b }
        when 'multiply' { $a * $b }
        when 'divide'   { 
            die "Division by zero" if $b == 0;
            $a / $b 
        }
        default { die "Unknown operation: $operation" }
    }
}
```

Documentation uses `#|` for subroutine descriptions and `#=` for parameter  
documentation. Pod6 blocks provide detailed documentation with formatting.  
This creates self-documenting code with built-in help accessible through  
introspection and documentation generation tools.  

## Higher-order functions

Subroutines that operate on other subroutines.  

```raku
sub compose(&f, &g) {
    return sub ($x) { f(g($x)) };
}

sub partial(&func, *@fixed-args) {
    return sub (*@remaining-args) {
        func(|@fixed-args, |@remaining-args);
    };
}

sub memoize(&func) {
    my %cache;
    return sub (*@args) {
        my $key = @args.join(',');
        return %cache{$key} if %cache{$key}:exists;
        %cache{$key} = func(|@args);
    };
}

my &add1 = * + 1;
my &times2 = * * 2;
my &composed = compose(&times2, &add1);

my &add-partial = partial(&[+], 10);
my &memoized-fib = memoize(&fibonacci);

say composed(5);      # (5 + 1) * 2 = 12
say add-partial(5);   # 10 + 5 = 15
say memoized-fib(10); # Cached fibonacci
```

Higher-order functions accept other functions as parameters or return new  
functions. Function composition combines operations. Partial application  
fixes some arguments. Memoization caches results. These patterns enable  
powerful abstractions and functional programming techniques.  

## Performance considerations

Optimizing subroutine performance and behavior.  

```raku
# Inlined for performance
sub fast-square($n) is inline {
    $n * $n;
}

# Cached expensive computation
sub expensive-calculation($n) is cached {
    sleep 0.1;  # Simulate expensive work
    $n ** 3 + $n ** 2 + $n;
}

# Using native types for speed
sub native-math(int $a, int $b --> int) {
    $a * $b + $a;
}

# Avoiding object creation
sub efficient-string-build(*@parts) {
    my str $result = '';
    for @parts -> $part {
        $result = $result ~ $part;
    }
    $result;
}

say fast-square(100);
say expensive-calculation(5);
say native-math(10, 20);
say efficient-string-build("Hello", " ", "World");
```

Performance optimization uses various techniques: `is inline` for compiler  
optimization, `is cached` for memoization, native types (int, str) for  
speed, and efficient algorithms to minimize object creation. Profile before  
optimizing and measure the impact of changes.  

## Subroutine signatures

Advanced signature features and constraints.  

```raku
# Complex signature with all features
sub advanced-signature(
    Int $required,                    # Required positional
    Str $optional = 'default',       # Optional with default
    Int :$named!,                     # Required named parameter
    Bool :$flag = False,              # Optional named with default
    *@slurpy,                         # Slurpy positional
    *%options                         # Slurpy named
    --> List                          # Return type constraint
) {
    return $required, $optional, $named, @slurpy, %options;
}

# Signature with where clauses
sub constrained(
    $number where Int|Rat,
    $text where *.chars > 0,
    :$count where * > 0 = 1
) {
    "$text " x $count ~ "($number)";
}

say advanced-signature(
    42, 'test', :named(100), :flag, 
    'extra1', 'extra2', :option1<value1>
);

say constrained(3.14, "Hello", :count(3));
```

Advanced signatures combine multiple features: required and optional  
parameters, named parameters with `!` for required, slurpy parameters,  
return type constraints, and where clauses for validation. This creates  
highly expressive and safe interfaces for complex subroutines.  

## Testing subroutines

Writing tests to verify subroutine behavior.  

```raku
use Test;

sub add($a, $b) { $a + $b }
sub divide($a, $b) { 
    die "Division by zero" if $b == 0;
    $a / $b 
}

# Basic tests
is add(2, 3), 5, 'Addition works correctly';
is add(-1, 1), 0, 'Addition with negative numbers';

# Testing exceptions
throws-like { divide(10, 0) }, Exception, 
    'Division by zero throws exception';

# Testing with different inputs
for (
    [1, 1, 2],
    [0, 5, 5],
    [-2, 2, 0]
) -> [$a, $b, $expected] {
    is add($a, $b), $expected, "add($a, $b) = $expected";
}

done-testing;
```

Testing uses the Test module with functions like `is` for equality checks  
and `throws-like` for exception testing. Parameterized tests verify multiple  
input combinations. Good tests cover normal cases, edge cases, and error  
conditions to ensure subroutine reliability.  

## State variables

Using state variables to maintain data between calls.  

```raku
sub next-id() {
    state $counter = 0;
    ++$counter;
}

sub session-manager(:$action, :$data) {
    state %sessions;
    state $session-count = 0;
    
    given $action {
        when 'create' {
            my $id = 'session_' ~ ++$session-count;
            %sessions{$id} = %( :created(now), |$data );
            return $id;
        }
        when 'get' {
            return %sessions{$data<id>};
        }
        when 'list' {
            return %sessions.keys;
        }
    }
}

say next-id();  # 1
say next-id();  # 2

my $id1 = session-manager(:action<create>, :data{ :user<alice> });
my $id2 = session-manager(:action<create>, :data{ :user<bob> });
say session-manager(:action<get>, :data{ :$id1 });
say session-manager(:action<list>);
```

State variables persist between subroutine calls, maintaining their values  
across invocations. They are initialized only once on first call. State  
variables enable stateful subroutines for counters, caches, and persistent  
data without global variables. Each subroutine maintains its own state  
space.  

## Functional operators

Creating custom operators as subroutines.  

```raku
# Custom infix operator
sub infix:<⊕>($a, $b) {
    ($a + $b) % 2;  # XOR for integers
}

# Custom prefix operator  
sub prefix:<±>($n) {
    ($n, -$n);
}

# Custom postfix operator
sub postfix:<!>($n) {
    [*] 1..$n;  # Factorial
}

# Custom circumfix operator
sub circumfix:<⟨ ⟩>(*@items) {
    @items.join(' → ');
}

# Custom postcircumfix operator
sub postcircumfix:<« »>($obj, $key) {
    $obj{$key} // 'undefined';
}

say 5 ⊕ 3;        # 0 (binary XOR)
say ±42;          # (42 -42)
say 5!;           # 120
say ⟨a b c⟩;      # a → b → c

my %data = a => 1, b => 2;
say %data«a»;     # 1
say %data«c»;     # undefined
```

Custom operators are defined as subroutines with special naming conventions.  
Infix operators go between operands, prefix before, postfix after, circumfix  
around, and postcircumfix after with delimiters. This enables domain-  
specific languages and mathematical notation within Raku programs.  

## Best practices

Guidelines for writing maintainable subroutines.  

```raku
# Good: Clear name and purpose
sub calculate-monthly-payment(
    Numeric $principal,
    Numeric $rate,
    Int $months
    --> Numeric
) {
    return $principal if $rate == 0;
    my $monthly-rate = $rate / 12;
    $principal * $monthly-rate / (1 - (1 + $monthly-rate) ** -$months);
}

# Good: Single responsibility
sub validate-email(Str $email --> Bool) {
    $email ~~ /^ \w+ '@' \w+ '.' \w+ $/;
}

sub format-currency(Numeric $amount, Str $currency = 'USD' --> Str) {
    sprintf "%.2f %s", $amount, $currency;
}

# Good: Error handling
sub safe-file-read(Str $filename --> Str) {
    CATCH {
        when X::IO { return "" }
    }
    $filename.IO.slurp;
}

# Good: Documentation and typing
#| Process user data with validation and formatting
sub process-user-data(
    Str $name where *.chars > 0,     #= User's full name
    Int $age where 0 < * < 150,      #= User's age in years
    :$format = 'standard'             #= Output format option
    --> Hash                          #= Processed user information
) {
    %(
        name => $name.tc,
        age => $age,
        category => $age < 18 ?? 'minor' !! 'adult'
    );
}
```

Best practices include: descriptive names that explain purpose, single  
responsibility principle, proper type annotations, comprehensive error  
handling, thorough documentation, and input validation. Keep subroutines  
focused, testable, and predictable for maintainable codebases.