# Testing

This chapter covers 20 comprehensive testing examples in Raku designed to  
help you write robust and reliable test suites. These examples demonstrate  
essential testing concepts from basic assertions to advanced patterns,  
progressing from simple test cases to complex testing scenarios. Each  
example includes detailed explanations to help you understand testing  
best practices and apply them effectively in your Raku projects.  

## Basic test setup

Setting up a simple test file with the Test module.  

```raku
use Test;

# Test count planning (optional but recommended)
plan 3;

# Basic assertions
ok True, 'True value passes';
is 2 + 2, 4, 'Addition works correctly';
isnt 3, 4, 'Values are different';

done-testing;
```

The `Test` module provides all testing functionality. Use `plan` to specify  
expected test count, `ok` for boolean tests, `is` for equality, `isnt` for  
inequality, and `done-testing` to finalize the test run.  

## Testing functions

Verifying function behavior with different inputs and edge cases.  

```raku
use Test;

sub multiply($a, $b) { $a * $b }
sub factorial($n) {
    return 1 if $n <= 1;
    $n * factorial($n - 1);
}

# Test normal cases
is multiply(3, 4), 12, 'Multiplication works';
is factorial(5), 120, 'Factorial calculation correct';
is factorial(0), 1, 'Factorial of zero is one';

# Test edge cases
is multiply(0, 100), 0, 'Multiplication by zero';
is factorial(1), 1, 'Factorial of one';

done-testing;
```

Test functions with various inputs including normal cases, edge cases, and  
boundary conditions. Use descriptive test messages to explain what each  
test verifies. This helps identify failures quickly during development.  

## Exception testing

Testing that functions throw exceptions when expected.  

```raku
use Test;

sub divide($a, $b) {
    die "Division by zero" if $b == 0;
    $a / $b;
}

class CustomError is Exception {
    has $.code;
    method message() { "Error code: {$.code}" }
}

sub throw-custom($code) {
    die CustomError.new(code => $code);
}

# Test successful operations
is divide(10, 2), 5, 'Normal division works';

# Test exception throwing
throws-like { divide(10, 0) }, Exception, 
    'Division by zero throws exception';

throws-like { divide(10, 0) }, Exception,
    message => /zero/,
    'Exception message contains expected text';

# Test custom exceptions
throws-like { throw-custom(404) }, CustomError,
    'Custom exception type thrown correctly';

done-testing;
```

Use `throws-like` to verify exceptions are thrown with correct types and  
messages. Test both standard exceptions and custom exception classes.  
Include message pattern matching to verify error details.  

## Array and list testing

Testing array operations, transformations, and properties.  

```raku
use Test;

my @numbers = 1, 2, 3, 4, 5;
my @empty = ();
my @mixed = 'a', 1, True, 3.14;

# Test array properties
is @numbers.elems, 5, 'Array has correct element count';
ok @empty.is-empty, 'Empty array is empty';
isnt @mixed.is-empty, True, 'Mixed array is not empty';

# Test array access
is @numbers[0], 1, 'First element access';
is @numbers[*-1], 5, 'Last element access using *-1';

# Test array operations
is-deeply @numbers.reverse, [5, 4, 3, 2, 1], 'Array reversal';
is-deeply @numbers.sort, [1, 2, 3, 4, 5], 'Array sorting';

# Test array slicing
is-deeply @numbers[1..3], [2, 3, 4], 'Array slice works';

done-testing;
```

Use `is-deeply` for comparing complex data structures like arrays and  
hashes. Test array properties, access patterns, and common operations.  
Verify both content and structure match expectations.  

## Hash testing

Testing hash operations, key-value manipulation, and lookups.  

```raku
use Test;

my %person = name => 'Alice', age => 30, city => 'New York';
my %empty = ();

# Test hash properties
is %person.elems, 3, 'Hash has correct key count';
ok %empty.is-empty, 'Empty hash is empty';

# Test key existence
ok %person<name>:exists, 'Key exists in hash';
nok %person<height>:exists, 'Non-existent key check';

# Test value access
is %person<name>, 'Alice', 'Hash value access';
is %person<age>, 30, 'Numeric value in hash';

# Test hash operations
%person<height> = 165;
is %person<height>, 165, 'New key-value addition';

# Test key and value lists
is-deeply %person.keys.sort, ['age', 'city', 'height', 'name'],
    'Keys list correct';
ok 'Alice' ∈ %person.values, 'Value exists in hash';

done-testing;
```

Test hash operations including key existence checks with `:exists`,  
value access patterns, and dynamic key-value manipulation. Verify both  
keys and values collections match expectations.  

## Type testing

Verifying type constraints and type-based behavior.  

```raku
use Test;

sub typed-function(Int $number, Str $text --> Str) {
    "$text: $number";
}

class Person {
    has $.name;
    has $.age;
}

my $person = Person.new(name => 'Bob', age => 25);

# Test type checking
isa-ok $person, Person, 'Object is correct type';
isa-ok 42, Int, 'Integer type check';
isa-ok "hello", Str, 'String type check';

# Test type coercion
is typed-function(42, "Answer"), "Answer: 42", 'Typed function works';

# Test type failures
throws-like { typed-function("not-int", "test") }, X::TypeCheck,
    'Type constraint violation throws exception';

# Test type relationships
ok Int ~~ Numeric, 'Int is a Numeric type';
ok Str ~~ Cool, 'Str inherits from Cool';

done-testing;
```

Use `isa-ok` to verify object types and type relationships. Test type  
constraints in function signatures and verify type checking behavior.  
Smart matching with `~~` tests type inheritance and relationships.  

## Parametric testing

Running the same test logic with multiple data sets.  

```raku
use Test;

sub power($base, $exponent) { $base ** $exponent }

# Test data: [base, exponent, expected_result, description]
my @test-cases = (
    [2, 3, 8, 'Two to the power of three'],
    [5, 0, 1, 'Any number to power zero is one'],
    [1, 100, 1, 'One to any power is one'],
    [10, 2, 100, 'Ten squared is one hundred'],
    [3, 4, 81, 'Three to the fourth power'],
);

plan @test-cases.elems;

for @test-cases -> [$base, $exp, $expected, $desc] {
    is power($base, $exp), $expected, $desc;
}

done-testing;
```

Parametric testing allows testing the same logic with multiple inputs.  
Define test cases as data structures and iterate through them. This  
approach reduces code duplication and makes adding new test cases easy.  

## Test organization

Organizing tests into logical groups with subtests.  

```raku
use Test;

sub calculator($a, $b, $op) {
    given $op {
        when 'add' { $a + $b }
        when 'sub' { $a - $b }
        when 'mul' { $a * $b }
        when 'div' { 
            die "Division by zero" if $b == 0;
            $a / $b 
        }
        default { die "Unknown operation: $op" }
    }
}

# Group related tests together
subtest 'Addition operations' => {
    is calculator(5, 3, 'add'), 8, 'Positive addition';
    is calculator(-2, 7, 'add'), 5, 'Negative and positive';
    is calculator(0, 0, 'add'), 0, 'Zero addition';
};

subtest 'Division operations' => {
    is calculator(10, 2, 'div'), 5, 'Normal division';
    throws-like { calculator(10, 0, 'div') }, Exception,
        'Division by zero throws exception';
};

subtest 'Error conditions' => {
    throws-like { calculator(1, 2, 'invalid') }, Exception,
        'Invalid operation throws exception';
};

done-testing;
```

Use `subtest` to group related tests logically. This creates hierarchical  
test structure and improves test output readability. Each subtest can  
contain multiple assertions and maintains its own test context.  

## Mock testing

Testing with mock objects and stubbed dependencies.  

```raku
use Test;

role DatabaseRole {
    method get-user($id) { ... }
    method save-user($user) { ... }
}

class MockDatabase does DatabaseRole {
    has %.users = 1 => { name => 'Alice', age => 30 };
    
    method get-user($id) {
        %.users{$id} // Nil;
    }
    
    method save-user($user) {
        %.users{$user<id>} = $user;
        return True;
    }
}

class UserService {
    has $.db;
    
    method get-user-info($id) {
        my $user = $.db.get-user($id);
        return Nil unless $user;
        "User: {$user<name>}, Age: {$user<age>}";
    }
}

# Test with mock database
my $mock-db = MockDatabase.new;
my $service = UserService.new(db => $mock-db);

is $service.get-user-info(1), "User: Alice, Age: 30",
    'Service returns user info correctly';
is $service.get-user-info(999), Nil,
    'Service returns Nil for non-existent user';

# Test mock interaction
ok $mock-db.save-user({ id => 2, name => 'Bob', age => 25 }),
    'Mock saves user successfully';
is $service.get-user-info(2), "User: Bob, Age: 25",
    'Service returns newly saved user';

done-testing;
```

Mock objects replace real dependencies with controlled implementations.  
Use roles to define interfaces and mock classes that implement them.  
This enables testing components in isolation without external dependencies.  

## Performance testing

Testing performance characteristics and execution timing.  

```raku
use Test;

sub fibonacci-recursive($n) {
    return $n if $n <= 1;
    fibonacci-recursive($n - 1) + fibonacci-recursive($n - 2);
}

sub fibonacci-iterative($n) {
    return $n if $n <= 1;
    my ($a, $b) = 0, 1;
    for 2..$n {
        ($a, $b) = $b, $a + $b;
    }
    return $b;
}

# Test correctness first
is fibonacci-recursive(10), 55, 'Recursive fibonacci correct';
is fibonacci-iterative(10), 55, 'Iterative fibonacci correct';

# Performance timing tests
my $start-time = now;
my $result1 = fibonacci-recursive(25);
my $recursive-time = now - $start-time;

$start-time = now;
my $result2 = fibonacci-iterative(25);
my $iterative-time = now - $start-time;

# Verify results are the same
is $result1, $result2, 'Both algorithms produce same result';

# Performance comparison (iterative should be faster)
ok $iterative-time < $recursive-time, 
    'Iterative implementation is faster';

cmp-ok $iterative-time, '<=', 0.01,
    'Iterative solution completes quickly';

done-testing;
```

Performance testing verifies execution timing and resource usage. Use  
`now` to measure execution time and `cmp-ok` for threshold comparisons.  
Always test correctness before performance to ensure optimizations work.  

## Class testing

Testing class behavior, inheritance, and object interactions.  

```raku
use Test;

class Animal {
    has $.name;
    has $.species;
    
    method speak() { "Some animal sound" }
    method info() { "I am a {$.species} named {$.name}" }
}

class Dog is Animal {
    has $.breed;
    
    method speak() { "Woof!" }
    method fetch() { "Fetching the ball!" }
}

# Test class instantiation
my $animal = Animal.new(name => 'Generic', species => 'Unknown');
isa-ok $animal, Animal, 'Animal instance created';

# Test attribute access
is $animal.name, 'Generic', 'Animal name attribute';
is $animal.species, 'Unknown', 'Animal species attribute';

# Test method behavior
is $animal.speak(), "Some animal sound", 'Animal speak method';
like $animal.info(), /Generic/, 'Animal info contains name';

# Test inheritance
my $dog = Dog.new(name => 'Buddy', species => 'Canine', breed => 'Golden');
isa-ok $dog, Dog, 'Dog instance created';
isa-ok $dog, Animal, 'Dog inherits from Animal';

# Test method overriding
is $dog.speak(), "Woof!", 'Dog overrides speak method';
is $dog.fetch(), "Fetching the ball!", 'Dog has additional methods';

# Test inherited methods
like $dog.info(), /Buddy/, 'Dog inherits info method';

done-testing;
```

Test class instantiation, attribute access, method behavior, and  
inheritance relationships. Verify method overriding works correctly  
and inherited methods are accessible in derived classes.  

## Module testing

Testing custom modules with proper import and export behavior.  

```raku
# This would be in a separate file: lib/Calculator.rakumod
# unit module Calculator;
# 
# sub add($a, $b) is export { $a + $b }
# sub multiply($a, $b) is export { $a * $b }
# 
# our $VERSION = '1.0.0';

use Test;
# use lib 'lib';
# use Calculator;

# Since we can't create separate files, simulate module behavior
my module Calculator {
    our sub add($a, $b) is export { $a + $b }
    our sub multiply($a, $b) is export { $a * $b }
    our $VERSION = '1.0.0';
}

# Import functions into current namespace
my &add = &Calculator::add;
my &multiply = &Calculator::multiply;

# Test exported functions
is add(3, 4), 7, 'Module add function works';
is multiply(3, 4), 12, 'Module multiply function works';

# Test module variables
is Calculator::<$VERSION>, '1.0.0', 'Module version accessible';

# Test with different parameter types
is add(2.5, 1.5), 4, 'Module functions handle decimals';
is multiply(-2, 3), -6, 'Module functions handle negative numbers';

# Test edge cases
is add(0, 0), 0, 'Module handles zero values';
is multiply(1, 1000), 1000, 'Module handles large numbers';

done-testing;
```

Test module imports, exported functions, and module variables. Verify  
exported functions work correctly with different parameter types and  
edge cases. Test module-level constants and configuration values.  

## Regex testing

Testing regular expressions and pattern matching behavior.  

```raku
use Test;

# Test basic regex matching
ok "hello" ~~ /hello/, 'Basic string matching';
ok "Hello" ~~ /:i hello/, 'Case-insensitive matching';
nok "goodbye" ~~ /hello/, 'Non-matching string';

# Test capturing groups
my $text = "Date: 2023-12-25";
ok $text ~~ /(\d+) '-' (\d+) '-' (\d+)/, 'Date pattern matches';

if $text ~~ /(\d+) '-' (\d+) '-' (\d+)/ {
    is $0, '2023', 'Year captured correctly';
    is $1, '12', 'Month captured correctly';  
    is $2, '25', 'Day captured correctly';
}

# Test named captures
my $email = "user@example.com";
ok $email ~~ /$<name>=(\w+) '@' $<domain>=(\w+ '.' \w+)/, 
    'Email pattern with named captures';

if $email ~~ /$<name>=(\w+) '@' $<domain>=(\w+ '.' \w+)/ {
    is $<name>, 'user', 'Email name captured';
    is $<domain>, 'example.com', 'Email domain captured';
}

# Test quantifiers
ok "aaa" ~~ /a+/, 'One or more quantifier';
ok "aaa" ~~ /a{3}/, 'Exact count quantifier';
ok "ab" ~~ /a?b/, 'Zero or one quantifier';

done-testing;
```

Test regex patterns with different matching modes, capture groups, and  
quantifiers. Verify both anonymous and named captures work correctly.  
Test pattern matching against various input strings.  

## Asynchronous testing

Testing concurrent operations and promise-based code.  

```raku
use Test;

sub async-operation($delay, $value) {
    start {
        sleep $delay;
        $value * 2;
    }
}

sub failing-async-operation($delay) {
    start {
        sleep $delay;
        die "Async operation failed";
    }
}

# Test promise completion
my $promise = async-operation(0.1, 5);
await $promise;
is $promise.result, 10, 'Async operation completed correctly';

# Test multiple concurrent operations
my @promises = (1..3).map: { async-operation(0.1, $_) };
my @results = await @promises;
is-deeply @results, [2, 4, 6], 'Multiple async operations';

# Test promise chaining
my $chained = async-operation(0.1, 3).then: { .result + 1 };
is await($chained), 7, 'Promise chaining works';

# Test async exception handling
my $failing = failing-async-operation(0.1);
throws-like { await $failing }, Exception,
    message => /failed/,
    'Async exception propagates correctly';

# Test timeout behavior
my $slow = start { sleep 1; "done" };
throws-like { 
    await Promise.in(0.1).then: { $slow.break("timeout") }; 
    await $slow;
}, X::Promise::Broken,
    'Promise timeout handling';

done-testing;
```

Test asynchronous operations using promises and `start` blocks. Verify  
promise completion, concurrent execution, and exception handling in  
async contexts. Test timeout scenarios and promise chaining behavior.  

## Data validation testing

Testing input validation and data constraint enforcement.  

```raku
use Test;

subset PositiveInt of Int where * > 0;
subset ValidEmail of Str where /^ \w+ '@' \w+ '.' \w+ $/;

sub validate-user-data(%data) {
    die "Name required" unless %data<name>;
    die "Age must be positive" unless %data<age> ~~ PositiveInt;
    die "Invalid email format" unless %data<email> ~~ ValidEmail;
    return True;
}

class User {
    has Str $.name where *.chars > 0;
    has PositiveInt $.age;
    has ValidEmail $.email;
}

# Test successful validation
my %valid-data = name => 'Alice', age => 25, email => 'alice@test.com';
lives-ok { validate-user-data(%valid-data) }, 'Valid data passes';

# Test validation failures
throws-like { validate-user-data(%()) }, Exception,
    message => /Name/,
    'Missing name throws exception';

throws-like { validate-user-data(name => 'Bob', age => -5) }, Exception,
    message => /positive/,
    'Negative age throws exception';

throws-like { 
    validate-user-data(name => 'Bob', age => 25, email => 'invalid') 
}, Exception, message => /email/, 'Invalid email throws exception';

# Test type constraints in class
lives-ok { User.new(|%valid-data) }, 'Valid User object creation';

throws-like { User.new(name => '', age => 25, email => 'test@test.com') },
    X::TypeCheck, 'Empty name constraint violation';

done-testing;
```

Test data validation using subsets, where clauses, and type constraints.  
Verify both successful validation and various failure modes. Test  
constraint enforcement in class attributes and function parameters.  

## Test fixtures and setup

Managing test data and setup/teardown operations.  

```raku
use Test;

class TestDatabase {
    has @.users;
    
    method setup() {
        @.users = (
            { id => 1, name => 'Alice', active => True },
            { id => 2, name => 'Bob', active => False },
            { id => 3, name => 'Charlie', active => True },
        );
    }
    
    method teardown() {
        @.users = ();
    }
    
    method find-active-users() {
        @.users.grep({ .<active> });
    }
    
    method count-users() {
        @.users.elems;
    }
}

# Test with setup and teardown
my $db = TestDatabase.new;

subtest 'Database operations' => {
    # Setup
    $db.setup();
    
    # Tests
    is $db.count-users(), 3, 'Initial user count correct';
    
    my @active = $db.find-active-users();
    is @active.elems, 2, 'Correct number of active users';
    ok @active.all.<active>, 'All returned users are active';
    
    # Verify specific users
    my @names = @active.map(*<name>);
    ok 'Alice' ∈ @names, 'Alice is active';
    ok 'Charlie' ∈ @names, 'Charlie is active';
    nok 'Bob' ∈ @names, 'Bob is not in active list';
    
    # Teardown
    $db.teardown();
    is $db.count-users(), 0, 'Database cleaned up after tests';
};

done-testing;
```

Use setup and teardown methods to manage test state and fixtures.  
Create consistent test environments and ensure proper cleanup after  
tests complete. This prevents test interference and maintains isolation.  

## Integration testing

Testing component interactions and system-level behavior.  

```raku
use Test;

class Logger {
    has @.messages;
    method log($message) { @.messages.push($message) }
    method get-logs() { @.messages }
}

class EmailService {
    has $.logger;
    has @.sent-emails;
    
    method send($to, $subject, $body) {
        $.logger.log("Sending email to $to");
        @.sent-emails.push({ to => $to, subject => $subject, body => $body });
        return "email-{@.sent-emails.elems}";
    }
}

class UserRegistration {
    has $.email-service;
    has %.users;
    
    method register($name, $email) {
        return "User already exists" if %.users{$email}:exists;
        
        %.users{$email} = { name => $name, registered => now };
        my $email-id = $.email-service.send(
            $email, 
            "Welcome $name!", 
            "Thanks for registering with us."
        );
        
        return "Registration successful: $email-id";
    }
}

# Integration test setup
my $logger = Logger.new;
my $email-service = EmailService.new(logger => $logger);
my $registration = UserRegistration.new(email-service => $email-service);

# Test complete registration flow
my $result = $registration.register('Alice', 'alice@test.com');
like $result, /successful/, 'Registration completes successfully';
like $result, /email\-\d+/, 'Email ID returned in result';

# Verify integration effects
is $registration.users.elems, 1, 'User added to system';
ok $registration.users<alice@test.com>:exists, 'User email stored';

is $email-service.sent-emails.elems, 1, 'Welcome email sent';
is $logger.get-logs.elems, 1, 'Email sending logged';

# Test duplicate registration
my $duplicate = $registration.register('Alice Again', 'alice@test.com');
is $duplicate, "User already exists", 'Duplicate registration prevented';
is $registration.users.elems, 1, 'No additional user created';

done-testing;
```

Integration tests verify that multiple components work together correctly.  
Test complete workflows and verify side effects across component  
boundaries. Check that data flows properly between integrated systems.  

## Test reporting and diagnostics

Advanced test reporting with custom diagnostics and debugging.  

```raku
use Test;

sub complex-calculation($x, $y, $z) {
    my $step1 = $x ** 2 + $y ** 2;
    my $step2 = $step1 * $z;
    my $step3 = sqrt($step2);
    return $step3;
}

# Test with diagnostic output
subtest 'Complex calculation verification' => {
    my ($x, $y, $z) = (3, 4, 2);
    my $expected = 10;  # sqrt((3² + 4²) * 2) = sqrt(25 * 2) = sqrt(50) ≈ 7.07
    
    # Detailed diagnostic testing
    my $step1 = $x ** 2 + $y ** 2;
    is $step1, 25, "Step 1: x² + y² = $step1";
    
    my $step2 = $step1 * $z;
    is $step2, 50, "Step 2: step1 * z = $step2";
    
    my $actual = complex-calculation($x, $y, $z);
    diag "Input: x=$x, y=$y, z=$z";
    diag "Expected approximately: 7.07";
    diag "Actual result: $actual";
    
    # Approximate comparison for floating point
    ok abs($actual - 7.071) < 0.01, 'Result within tolerance';
};

# Test with TODO items
todo 'Feature not yet implemented', 2;
is unimplemented-feature(), 'expected', 'Future feature test';
ok False, 'This test will be marked as TODO';

# Test with conditional skipping
if $*DISTRO.name eq 'windows' {
    skip 'Unix-specific test', 1;
} else {
    ok True, 'Unix-specific functionality works';
}

# Test timing information
my $start = now;
my $result = complex-calculation(10, 10, 3);
my $duration = now - $start;
diag "Calculation took {$duration} seconds";
cmp-ok $duration, '<', 1, 'Calculation completes within time limit';

done-testing;
```

Use `diag` for diagnostic output, `todo` for planned features, and  
`skip` for conditional test exclusion. Provide detailed debugging  
information and performance metrics to aid in development and maintenance.  

## Property-based testing

Testing with randomly generated data to find edge cases.  

```raku
use Test;

sub is-sorted(@arr) {
    return True if @arr.elems <= 1;
    for 1..^@arr.elems {
        return False if @arr[$_ - 1] > @arr[$_];
    }
    True;
}

sub quicksort(@arr) {
    return @arr if @arr.elems <= 1;
    my $pivot = @arr[0];
    my @less = @arr[1..*].grep(* <= $pivot);
    my @greater = @arr[1..*].grep(* > $pivot);
    return (quicksort(@less), $pivot, quicksort(@greater)).flat;
}

# Property: sorted arrays should remain sorted
for 1..10 {
    my @random = (^100).pick(20);
    my @sorted = quicksort(@random);
    
    ok is-sorted(@sorted), "Random array {$_} sorted correctly";
    is @sorted.elems, @random.elems, "No elements lost in sorting";
    
    # Property: all original elements should be present
    for @random -> $elem {
        ok $elem ∈ @sorted, "Element $elem preserved in sorted array";
    }
}

# Property: sorting twice should be idempotent
my @test-array = 5, 2, 8, 1, 9, 3;
my @sorted-once = quicksort(@test-array);
my @sorted-twice = quicksort(@sorted-once);
is-deeply @sorted-once, @sorted-twice, 'Sorting is idempotent';

done-testing;
```

Property-based testing uses randomly generated inputs to verify that  
functions maintain certain properties across a wide range of inputs.  
This approach can discover edge cases that manual test cases might miss.  

## Benchmark testing

Comparing performance between different implementations.  

```raku
use Test;

sub sum-loop(@numbers) {
    my $total = 0;
    for @numbers -> $num {
        $total += $num;
    }
    $total;
}

sub sum-reduce(@numbers) {
    @numbers.reduce(* + *);
}

sub sum-fold(@numbers) {
    [+] @numbers;
}

# Test correctness first
my @test-data = 1..1000;
my $expected = 500500;  # Sum of 1 to 1000

is sum-loop(@test-data), $expected, 'Loop implementation correct';
is sum-reduce(@test-data), $expected, 'Reduce implementation correct';
is sum-fold(@test-data), $expected, 'Fold implementation correct';

# Benchmark different approaches
my @large-data = 1..10000;

my $start = now;
my $result1 = sum-loop(@large-data);
my $loop-time = now - $start;

$start = now;
my $result2 = sum-reduce(@large-data);
my $reduce-time = now - $start;

$start = now;
my $result3 = sum-fold(@large-data);
my $fold-time = now - $start;

# Verify all implementations produce same result
is $result1, $result2, 'Loop and reduce produce same result';
is $result2, $result3, 'Reduce and fold produce same result';

# Performance comparisons
diag "Loop time: {$loop-time}s";
diag "Reduce time: {$reduce-time}s";
diag "Fold time: {$fold-time}s";

# Test that all methods complete within reasonable time
cmp-ok $loop-time, '<', 1, 'Loop implementation fast enough';
cmp-ok $reduce-time, '<', 1, 'Reduce implementation fast enough';
cmp-ok $fold-time, '<', 1, 'Fold implementation fast enough';

# Memory usage test (simplified)
subtest 'Memory efficiency test' => {
    # Test with very large array to check memory usage
    my @huge = 1..100000;
    lives-ok { sum-fold(@huge) }, 'Handles large datasets without crashing';
};

done-testing;
```

Benchmark testing compares performance characteristics of different  
implementations. Always verify correctness before performance testing.  
Use timing measurements and memory usage tests to identify optimal  
solutions for different use cases and data sizes.