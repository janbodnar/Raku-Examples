# Pattern Matching

Pattern matching is a powerful feature in Raku that allows you to test values  
against patterns using the smart match operator (`~~`). Raku's pattern  
matching works with types, ranges, regular expressions, code blocks, and  
custom patterns, making it versatile for data validation, control flow, and  
text processing. This comprehensive guide progresses from basic matching to  
advanced techniques for sophisticated pattern-based programming.  

## Basic smart matching

Testing simple values with the smart match operator.  

```raku
my $value = 42;
my $text = 'Hello';

say $value ~~ 42;        # True
say $text ~~ 'Hello';    # True
say $value ~~ 100;       # False
say $text ~~ 'World';    # False
```

The smart match operator (`~~`) compares values for equality and returns  
True or False. This is the foundation of pattern matching, providing a  
consistent interface for testing values against various patterns.  

## String pattern matching

Matching strings with different comparison techniques.  

```raku
my $name = 'Alice';
my @names = <Alice Bob Charlie>;

say $name ~~ 'Alice';      # True (exact match)
say $name ~~ @names;       # True (found in array)
say $name ~~ /^A/;         # True (regex match)
say $name ~~ *.chars > 3;  # True (length check)
```

String pattern matching supports exact equality, array membership, regular  
expressions, and custom predicates. Each pattern type provides different  
matching semantics for flexible text processing.  

## Type pattern matching

Testing values against type constraints.  

```raku
my $number = 42;
my $text = 'Hello';
my @array = [1, 2, 3];

say $number ~~ Int;      # True
say $text ~~ Str;        # True
say @array ~~ Array;     # True
say $number ~~ Str;      # False
```

Type pattern matching uses the type hierarchy to determine compatibility.  
This enables runtime type checking and polymorphic behavior based on  
actual object types rather than just variable declarations.  

## Numeric range matching

Using ranges for numeric pattern matching.  

```raku
my $score = 85;
my $temperature = -5;

say $score ~~ 0..100;        # True (0 to 100)
say $score ~~ 90..100;       # False (90 to 100)
say $temperature ~~ ^0;      # True (less than 0)
say $score ~~ 80^..^90;      # True (80 < x < 90)
```

Range patterns test numeric inclusion using various range operators.  
Exclusive ranges (`^`) and infinite ranges provide precise numeric  
boundary testing for validation and classification.  

## Whatever star patterns

Creating dynamic patterns with whatever star expressions.  

```raku
my $word = 'programming';
my $count = 15;

say $word ~~ *.chars > 8;       # True (length > 8)
say $count ~~ * %% 3;           # True (divisible by 3)
say $word ~~ *.contains('gram'); # True (contains substring)
say $count ~~ * < 10;           # False (less than 10)
```

Whatever star (`*`) creates lambda expressions for custom matching logic.  
These anonymous functions enable complex predicates that can test any  
property or relationship of the matched value.  

## Regular expression patterns

Advanced text matching with regular expressions.  

```raku
my $email = 'user@example.com';
my $phone = '555-123-4567';
my $code = 'ABC123';

say $email ~~ /\w+ '@' \w+ '.' \w+/;    # True (email format)
say $phone ~~ /\d**3 '-' \d**3 '-' \d**4/; # True (phone format)
say $code ~~ /^<[A..Z]>**3 \d**3$/;     # True (code format)
say $email ~~ /^\d+$/;                  # False (not all digits)
```

Regular expression patterns provide sophisticated text matching with  
character classes, quantifiers, and anchors. Raku's regex syntax supports  
both traditional and advanced pattern matching features.  

## Array pattern matching

Testing arrays against various patterns and conditions.  

```raku
my @numbers = [1, 2, 3, 4, 5];
my @mixed = ['a', 42, True];
my @empty = [];

say @numbers ~~ Array;           # True (is array)
say @numbers ~~ *.elems == 5;    # True (has 5 elements)
say @mixed ~~ *.grep(Int);       # True (contains integers)
say @empty ~~ *.so;              # False (empty array)
```

Array pattern matching tests structural properties like type, length,  
and content characteristics. Whatever star patterns enable complex  
array validation using any available array method.  

## Hash pattern matching

Matching hash structures and their contents.  

```raku
my %person = name => 'Alice', age => 30, city => 'NYC';
my %empty = ();

say %person ~~ Hash;                    # True (is hash)
say %person ~~ *.keys.contains('name'); # True (has name key)
say %person ~~ *.<age> > 25;            # True (age > 25)
say %empty ~~ *.so;                     # False (empty hash)
```

Hash pattern matching examines structure, keys, and values using various  
techniques. Hash subscript notation and whatever star patterns enable  
sophisticated data structure validation.  

## Junction patterns

Using junctions for multiple pattern matching.  

```raku
my $value = 42;
my $letter = 'B';

say $value ~~ any(40, 41, 42, 43);      # True (any match)
say $letter ~~ all('A'..'Z');           # True (all conditions)
say $value ~~ one(40, 42, 44);          # True (exactly one)
say $letter ~~ none('a'..'z');          # True (none match)
```

Junction patterns test values against multiple alternatives simultaneously.  
The `any`, `all`, `one`, and `none` junctions provide different logical  
combinations for complex matching scenarios.  

## Given-when pattern matching

Using given-when for complex pattern-based control flow.  

```raku
sub classify-value($val) {
    given $val {
        when Int         { "Integer: $val" }
        when Str         { "String: $val" }
        when 0..100      { "Percentage: $val%" }
        when *.chars > 10 { "Long text: $val" }
        default          { "Other: $val" }
    }
}

say classify-value(42);        # Integer: 42
say classify-value('Hello');   # String: Hello
say classify-value(75);        # Percentage: 75%
```

Given-when constructs enable pattern-based dispatch similar to switch  
statements but with Raku's powerful pattern matching. Each when clause  
uses smart matching for flexible condition testing.  

## Regex capture patterns

Extracting data using regex patterns with captures.  

```raku
my $log = '2023-12-25 ERROR Connection timeout';
my $url = 'https://example.com:8080/path';

if $log ~~ /:s (\d**4 '-' \d**2 '-' \d**2) (\w+) (.+)/ {
    say "Date: $0, Level: $1, Message: $2";
}

if $url ~~ m{(\w+) '://' (\w+) ':' (\d+) '/' (.+)} {
    say "Protocol: $0, Host: $1, Port: $2, Path: $3";
}
```

Regex capture patterns extract structured data from text using parentheses.  
Captured groups are available as numbered variables (`$0`, `$1`, etc.)  
for processing matched substrings.  

## Named capture patterns

Using named captures for more readable pattern matching.  

```raku
my $person = 'John Doe, 35, Engineer';
my $record = 'ID:123 NAME:Alice ROLE:Admin';

if $person ~~ /:s $<name>=(\w+ \s+ \w+) ',' \s* $<age>=(\d+) ',' \s* $<job>=(.+)/ {
    say "Name: $<name>, Age: $<age>, Job: $<job>";
}

if $record ~~ /:s 'ID:' $<id>=(\d+) \s+ 'NAME:' $<name>=(\w+) \s+ 'ROLE:' $<role>=(\w+)/ {
    say "ID: $<id>, Name: $<name>, Role: $<role>";
}
```

Named captures provide descriptive labels for extracted data using  
`$<name>=()` syntax. This makes pattern matching more self-documenting  
and easier to maintain in complex text processing scenarios.  

## Multi-level pattern matching

Nested pattern matching for complex data structures.  

```raku
my @data = (
    { type => 'user', info => { name => 'Alice', age => 30 } },
    { type => 'admin', info => { name => 'Bob', privileges => ['read', 'write'] } },
    { type => 'guest', info => { session => 'temp123' } }
);

for @data -> $item {
    given $item {
        when .<type> eq 'user' && .<info><age> > 25 {
            say "Adult user: {$item<info><name>}";
        }
        when .<type> eq 'admin' && .<info><privileges>.grep('write') {
            say "Admin with write access: {$item<info><name>}";
        }
        when .<type> eq 'guest' {
            say "Guest session: {$item<info><session>}";
        }
    }
}
```

Multi-level pattern matching combines hash key access with smart matching  
to handle nested data structures. This enables sophisticated data processing  
patterns for complex object hierarchies.  

## Code block patterns

Using executable code as patterns for dynamic matching.  

```raku
my @values = 42, 'hello', [1, 2, 3], { x => 10 };

sub is-even($n) { $n ~~ Int && $n %% 2 }
sub is-short-string($s) { $s ~~ Str && $s.chars < 10 }
sub is-small-array($a) { $a ~~ Array && $a.elems < 5 }

for @values -> $val {
    given $val {
        when &is-even        { say "$val is an even number" }
        when &is-short-string { say "$val is a short string" }
        when &is-small-array { say "$val is a small array" }
        default              { say "$val doesn't match patterns" }
    }
}
```

Code block patterns use callable objects as matching predicates. Functions  
can encapsulate complex logic and be reused across different pattern  
matching contexts for modular and testable code.  

## Signature patterns

Pattern matching with function signatures for parameter validation.  

```raku
multi process-data(Int $n where * > 0) {
    "Processing positive integer: $n";
}

multi process-data(Str $s where *.chars > 0) {
    "Processing non-empty string: $s";
}

multi process-data(@arr where *.elems > 0) {
    "Processing non-empty array with {+@arr} elements";
}

multi process-data($unknown) {
    "Cannot process: {$unknown.WHAT.^name}";
}

say process-data(42);           # Processing positive integer: 42
say process-data('hello');      # Processing non-empty string: hello
say process-data([1, 2, 3]);    # Processing non-empty array with 3 elements
say process-data(-5);           # Cannot process: Int
```

Signature patterns combine multi-dispatch with where clauses for powerful  
parameter validation. This enables type-safe function overloading with  
custom constraints and automatic pattern-based dispatch.  

## Custom pattern classes

Creating reusable pattern classes for domain-specific matching.  

```raku
class EmailPattern {
    method ACCEPTS($email) {
        $email ~~ /^ \w+ '@' \w+ '.' \w+ $/;
    }
}

class RangePattern {
    has $.min;
    has $.max;
    
    method ACCEPTS($value) {
        $value ~~ Numeric && $.min <= $value <= $.max;
    }
}

my $email-checker = EmailPattern.new;
my $score-range = RangePattern.new(min => 0, max => 100);

say 'valid@email.com' ~~ $email-checker;    # True
say 'invalid-email' ~~ $email-checker;      # False
say 85 ~~ $score-range;                     # True
say 150 ~~ $score-range;                    # False
```

Custom pattern classes implement the `ACCEPTS` method to define specialized  
matching behavior. This enables creation of reusable, parameterized patterns  
for domain-specific validation and complex matching logic.  

## Grammars for advanced patterns

Using grammars for complex structured pattern matching.  

```raku
grammar LogEntry {
    token TOP { <timestamp> \s+ <level> \s+ <message> }
    token timestamp { \d**4 '-' \d**2 '-' \d**2 \s+ \d**2 ':' \d**2 ':' \d**2 }
    token level { 'DEBUG' | 'INFO' | 'WARN' | 'ERROR' }
    token message { .+ }
}

grammar JSONValue {
    rule TOP { <value> }
    rule value { <string> | <number> | <object> | <array> | <boolean> | <null> }
    token string { '"' <-["]>* '"' }
    token number { '-'? \d+ ['.' \d+]? }
    rule object { '{' [<string> ':' <value>]* % ',' '}' }
    rule array { '[' [<value>]* % ',' ']' }
    token boolean { 'true' | 'false' }
    token null { 'null' }
}

my $log = '2023-12-25 14:30:15 ERROR Database connection failed';
my $json = '{"name": "Alice", "age": 30}';

say $log ~~ LogEntry;     # True
say $json ~~ JSONValue;   # True

if LogEntry.parse($log) -> $parsed {
    say "Level: {$parsed<level>}";
    say "Message: {$parsed<message>}";
}
```

Grammars provide the most sophisticated pattern matching capability in Raku,  
enabling parsing of complex structured text. They combine multiple patterns  
into hierarchical rules for language processing and data extraction.  

## Set and bag pattern matching

Matching against sets and bags for collection operations.  

```raku
my $letter = 'A';
my @vowels = <A E I O U>;
my %grades = A => 90, B => 80, C => 70;

my $vowel-set = set(@vowels);
my $grade-keys = set(%grades.keys);

say $letter ~~ $vowel-set;           # True (A is in vowel set)
say 'F' ~~ $vowel-set;               # False (F not in vowel set)
say 'B' ~~ $grade-keys;              # True (B is a grade key)
say @vowels ~~ *.Set ⊆ $vowel-set;   # True (subset relationship)
```

Set pattern matching uses mathematical set operations for membership  
testing and subset relationships. This enables efficient collection  
comparisons and data validation using set theory concepts.  

## File and path pattern matching

Pattern matching for file system operations and path validation.  

```raku
my $file = 'document.pdf';
my $path = '/home/user/documents/report.txt';
my $image = 'photo.jpg';

say $file ~~ /\.pdf$/;                    # True (PDF extension)
say $path ~~ /^ '/' .+ '/' \w+ '.' \w+ $/; # True (absolute path)
say $image ~~ /\.(jpg|png|gif)$/;         # True (image format)
say $file ~~ *.IO.e;                      # Check if file exists
```

File pattern matching validates paths, extensions, and file properties.  
Combining regex patterns with IO methods enables comprehensive file  
system validation and path manipulation.  

## Performance-optimized patterns

Efficient pattern matching techniques for high-performance scenarios.  

```raku
my regex fast-email { \w+ '@' \w+ '.' \w+ }
my regex fast-number { '-'? \d+ ['.' \d+]? }

my @test-data = 'user@test.com', '123.45', 'invalid', '-67.89';

for @test-data -> $item {
    given $item {
        when fast-email  { say "$item is valid email" }
        when fast-number { say "$item is valid number" }
        default          { say "$item is invalid" }
    }
}

# Compiled patterns for repeated use
my $email-pattern = rx/^ \w+ '@' \w+ '.' \w+ $/;
my $phone-pattern = rx/^ \d**3 '-' \d**3 '-' \d**4 $/;

say 'test@example.com' ~~ $email-pattern;  # True
say '555-123-4567' ~~ $phone-pattern;      # True
```

Performance-optimized patterns use compiled regex objects and named  
patterns for repeated matching operations. This reduces compilation  
overhead and improves execution speed in performance-critical code.  