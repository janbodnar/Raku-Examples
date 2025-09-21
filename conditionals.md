# Conditionals

Conditionals control the flow of program execution based on boolean expressions.  
Raku provides several conditional constructs including if/elsif/else, unless,  
given-when, and postfix conditionals. These constructs allow for both simple  
and complex decision-making in your programs, with powerful pattern matching  
and smart matching capabilities that make conditional logic expressive and  
concise.

## Basic if statement

The simplest conditional construct in Raku.  

```raku
my $temperature = 25;

if $temperature > 20 {
    say 'It is warm outside';
}

if $temperature < 0 {
    say 'It is freezing';
}
```

The `if` statement executes its block when the condition evaluates to true.  
Parentheses around the condition are optional. The block is enclosed in  
curly braces and executed only when the condition is met.  

## if-elsif-else chains

Multiple conditional branches for different scenarios.  

```raku
my $score = 85;

if $score >= 90 {
    say 'Grade: A';
} elsif $score >= 80 {
    say 'Grade: B';
} elsif $score >= 70 {
    say 'Grade: C';
} elsif $score >= 60 {
    say 'Grade: D';
} else {
    say 'Grade: F';
}
```

The `elsif` keyword allows chaining multiple conditions. Only the first  
matching condition executes. The `else` clause handles all remaining cases.  
Conditions are evaluated in order from top to bottom.  

## unless statement

Negative conditional logic for clearer intent.  

```raku
my $account_balance = 50;

unless $account_balance < 0 {
    say 'Account is not overdrawn';
}

my $user_logged_in = False;

unless $user_logged_in {
    say 'Please log in to continue';
}
```

The `unless` statement is equivalent to `if not` and executes when the  
condition is false. This makes code more readable when checking for  
negative conditions or absence of something.  

## Postfix conditionals

Compact conditional syntax for simple statements.  

```raku
my $age = 17;
my $has_license = True;

say 'You can drive' if $age >= 16 and $has_license;
say 'Access denied' unless $age >= 18;

my $debug_mode = True;
say "Debug: Variable value is $age" if $debug_mode;
```

Postfix conditionals place the condition after the statement. They're  
perfect for simple conditional actions and guard clauses. Both `if` and  
`unless` can be used in postfix form.  

## Ternary operator

Conditional expressions for value selection.  

```raku
my $weather = 'sunny';
my $clothing = $weather eq 'sunny' ?? 'shorts' !! 'pants';
say $clothing;

my $number = -5;
my $sign = $number > 0 ?? 'positive' !! 
           $number < 0 ?? 'negative' !! 'zero';
say $sign;
```

The ternary operator `??!!` provides a concise way to choose between two  
values based on a condition. It can be chained for multiple conditions,  
creating readable conditional assignments.  

## given-when (switch)

Pattern matching for multiple discrete values.  

```raku
my $day = 'Wednesday';

given $day {
    when 'Monday' | 'Tuesday' | 'Wednesday' | 'Thursday' | 'Friday' {
        say 'Weekday';
    }
    when 'Saturday' | 'Sunday' {
        say 'Weekend';
    }
    default {
        say 'Unknown day';
    }
}
```

The `given-when` construct is Raku's switch statement. It uses smart  
matching to compare values. The pipe `|` operator creates junctions  
for matching multiple values in a single `when` clause.  

## Conditional assignment

Assigning values only when certain conditions are met.  

```raku
my $username;
$username //= 'guest';  # assign if undefined
say $username;

my $counter = 0;
$counter ||= 1;  # assign if false
say $counter;

my @items = ();
@items[0] //= 'default item';  # assign if element is undefined
say @items;
```

The `//=` operator assigns only if the variable is undefined (Nil).  
The `||=` operator assigns only if the variable is false. These  
operators provide concise ways to set default values.  

## Boolean operators

Combining multiple conditions with logical operators.  

```raku
my $temperature = 25;
my $humidity = 60;
my $wind_speed = 10;

if $temperature > 20 and $humidity < 70 {
    say 'Perfect weather for outdoor activities';
}

if $wind_speed > 50 or $temperature < -10 {
    say 'Stay indoors';
}

if not ($temperature > 35 and $humidity > 80) {
    say 'Not extremely uncomfortable';
}
```

Raku provides `and`, `or`, and `not` operators for combining conditions.  
These have different precedence than `&&`, `||`, and `!`. The word forms  
are generally preferred for readability.  

## Comparison operators

Different ways to compare values in conditions.  

```raku
my $x = 10;
my $y = 20;
my $name = 'Alice';

say 'Equal' if $x == $y;
say 'Not equal' if $x != $y;
say 'Less than' if $x < $y;
say 'Greater than or equal' if $x >= $y;

say 'String equal' if $name eq 'Alice';
say 'String not equal' if $name ne 'Bob';
say 'String less than' if $name lt 'Bob';
```

Raku has separate operators for numeric (`==`, `!=`, `<`, `<=`, `>`, `>=`)  
and string (`eq`, `ne`, `lt`, `le`, `gt`, `ge`) comparisons. This  
prevents common errors and makes intent clearer.  

## Pattern matching with conditionals

Using smart matching for complex conditions.  

```raku
my $value = 42;

if $value ~~ 1..100 {
    say 'Value is between 1 and 100';
}

my $text = 'Hello World';

if $text ~~ /Hello/ {
    say 'Text contains Hello';
}

my @valid_codes = <A1 B2 C3>;
my $code = 'B2';

if $code ~~ @valid_codes {
    say 'Valid code';
}
```

The smart match operator `~~` enables powerful pattern matching. It  
works with ranges, regular expressions, arrays, and many other types,  
making conditionals more expressive and concise.  

## Multiple conditions

Handling complex decision trees with multiple variables.  

```raku
my $user_type = 'premium';
my $account_age = 365;
my $purchase_amount = 150;

if $user_type eq 'premium' and $account_age > 30 {
    if $purchase_amount > 100 {
        say 'Apply premium discount and free shipping';
    } else {
        say 'Apply premium discount only';
    }
} elsif $user_type eq 'regular' and $account_age > 90 {
    say 'Apply loyalty discount';
} else {
    say 'No discount available';
}
```

Complex business logic often requires multiple nested or chained  
conditions. Structure them clearly with proper indentation and  
consider breaking complex logic into separate functions.  

## Nested conditionals

Conditionals within conditionals for detailed logic.  

```raku
my $weather = 'rainy';
my $temperature = 15;
my $have_umbrella = True;

if $weather eq 'rainy' {
    if $have_umbrella {
        say 'You can go out with your umbrella';
    } else {
        say 'Better stay inside or get wet';
    }
} elsif $weather eq 'sunny' {
    if $temperature > 25 {
        say 'Perfect day for the beach';
    } else {
        say 'Nice day for a walk';
    }
}
```

Nested conditionals handle hierarchical decision making. Keep nesting  
levels reasonable for readability. Consider using early returns or  
separate functions for deeply nested logic.  

## Conditional loops

Combining conditionals with iteration constructs.  

```raku
my @numbers = 1, 2, 3, 4, 5, 6, 7, 8, 9, 10;

for @numbers -> $num {
    next if $num %% 2;  # skip even numbers
    last if $num > 7;   # stop at numbers greater than 7
    say $num;
}

my $count = 0;
while $count < 5 {
    say "Count: $count" if $count %% 2 == 0;
    $count++;
}
```

The `next` and `last` keywords control loop flow based on conditions.  
`next` skips to the next iteration, `last` exits the loop entirely.  
These provide fine-grained control over iteration.  

## Smart matching

Advanced pattern matching with various types.  

```raku
my $input = 'Hello123';

given $input {
    when Int { say 'It is an integer' }
    when Str { say 'It is a string' }
    when /\d+/ { say 'Contains digits' }
    when *.chars > 10 { say 'Long string' }
    default { say 'Something else' }
}

my $number = 42;
say 'Even' if $number ~~ * %% 2;
say 'In range' if $number ~~ 1..100;
```

Smart matching works with types, regular expressions, ranges, and  
callable expressions. The `*` in `* %% 2` creates a lambda that  
tests if a number is even. This makes conditions very expressive.  

## Type-based conditionals

Making decisions based on variable types.  

```raku
my $data = 42;

if $data.WHAT === Int {
    say 'Processing integer: ', $data + 10;
} elsif $data.WHAT === Str {
    say 'Processing string: ', $data.uc;
} elsif $data.WHAT === Array {
    say 'Processing array: ', $data.elems, ' elements';
}

sub process-value($value) {
    given $value {
        when Int { $value * 2 }
        when Str { $value.uc }
        when Array { $value.sort }
        default { $value }
    }
}

say process-value(5);
say process-value('hello');
```

Type checking with `.WHAT` or smart matching enables polymorphic  
behavior. The `given-when` construct with type matching provides  
clean type-based dispatch without complex if-elsif chains.  

## Exception handling with conditionals

Using conditionals for error handling and validation.  

```raku
my $filename = 'data.txt';

if $filename.IO.e {
    say 'File exists, proceeding with processing';
} else {
    say 'Error: File not found';
    exit 1;
}

sub divide($a, $b) {
    return Nil if $b == 0;
    return $a / $b;
}

my $result = divide(10, 2);
say $result.defined ?? "Result: $result" !! "Division by zero!";
```

Conditionals can handle error conditions gracefully. Check for error  
states early and handle them appropriately. Use `.defined` to test  
if a value is valid after operations that might fail.  

## Functional conditionals

Using conditionals in functional programming contexts.  

```raku
my @numbers = 1, 2, 3, 4, 5, 6, 7, 8, 9, 10;

my @evens = @numbers.grep(* %% 2);
say @evens;

my @processed = @numbers.map(-> $n { 
    $n > 5 ?? $n * 2 !! $n 
});
say @processed;

my @categories = @numbers.map({
    when $_ < 3 { 'small' }
    when $_ < 7 { 'medium' }
    default { 'large' }
});
say @categories;
```

Conditionals integrate smoothly with functional methods like `.grep`  
and `.map`. The `when` statement can be used inside blocks for  
pattern-based transformations, creating expressive data processing  
pipelines.  

## Conditional method calls

Calling methods based on conditions and object states.  

```raku
my $user = {
    name => 'Alice',
    role => 'admin',
    active => True
};

$user<role> eq 'admin' && say 'Admin access granted';
$user<active> || say 'Account is suspended';

my @items = <apple banana cherry>;
@items.push('date') if @items.elems < 5;
@items.pop if @items.elems > 3;

say @items;

# Method chaining with conditionals
my $text = 'Hello World';
my $result = $text
    .lc
    .subst(' ', '_')
    .substr(0, $text.chars > 10 ?? 10 !! $text.chars);
say $result;
```

Conditional method calls help maintain object state and perform  
actions only when appropriate. The `&&` and `||` operators can  
execute expressions conditionally in a compact form.  

## Complex boolean expressions

Combining multiple operators and conditions effectively.  

```raku
my $age = 25;
my $income = 50000;
my $credit_score = 750;
my $employment_years = 3;

# Complex loan approval logic
my $approved = ($age >= 21 and $age <= 65) and
               ($income >= 30000 or $credit_score > 700) and
               ($employment_years >= 2 or $credit_score > 800);

say $approved ?? 'Loan approved' !! 'Loan denied';

# Using parentheses for clarity
my $eligible_for_discount = 
    ($age >= 65 or $age <= 18) and
    (not $income > 100000) and
    ($credit_score >= 600);

say $eligible_for_discount ?? 'Discount available' !! 'No discount';
```

Complex boolean expressions require careful use of parentheses and  
logical operators. Break long expressions across lines for readability  
and group related conditions to make the logic clear.  

## Advanced pattern matching

Sophisticated matching with custom patterns and constraints.  

```raku
my @data = (
    { type => 'user', age => 25, role => 'admin' },
    { type => 'user', age => 30, role => 'user' },
    { type => 'system', priority => 'high' },
    { type => 'error', code => 404 }
);

for @data -> $item {
    given $item {
        when .<type> eq 'user' && .<role> eq 'admin' {
            say "Admin user, age: {$item<age>}";
        }
        when .<type> eq 'user' && .<age> > 25 {
            say "Regular user over 25";
        }
        when .<type> eq 'system' && .<priority> eq 'high' {
            say "High priority system event";
        }
        when .<type> eq 'error' {
            say "Error with code: {$item<code>}";
        }
        default {
            say "Unknown item type";
        }
    }
}
```

Advanced pattern matching combines smart matching with hash key  
access and multiple conditions. This creates powerful data processing  
patterns that can handle complex structured data elegantly.
