# Error Handling

Error handling in Raku provides robust mechanisms for managing exceptional  
conditions, recovering from failures, and maintaining program stability.  
Raku's exception system includes built-in exception types, custom exception  
classes, flexible catching mechanisms, and defensive programming patterns.  
This comprehensive guide covers everything from basic try/catch blocks to  
advanced error recovery strategies, enabling developers to write resilient  
applications that gracefully handle unexpected situations and provide  
meaningful feedback to users and systems.  

## Basic try/catch

Fundamental exception handling with try and CATCH blocks.  

```raku
# Basic exception handling
try {
    my $result = 10 / 0;
    say "Result: $result";
}

say "Program continues after error";

# With CATCH block
try {
    die "Something went wrong";
    CATCH {
        default { say "Caught exception: {.message}" }
    }
}

say "Execution continues";
```

The `try` block executes code that might fail. If an exception occurs,  
execution jumps to the `CATCH` block (if present) or returns Nil. The  
`default` case in CATCH handles any exception type. This provides basic  
fault tolerance and prevents program crashes.  

## Exception types

Catching specific exception types for targeted error handling.  

```raku
try {
    my $text = "not-a-number";
    my $num = Int($text);
    say $num;
    CATCH {
        when X::Str::Numeric {
            say "Invalid number format: {.source}";
        }
        when X::AdHoc {
            say "General error: {.payload}";
        }
        default {
            say "Unknown error: {.^name}";
        }
    }
}

# Multiple operations with different exception types
try {
    my $file = "nonexistent.txt".IO.slurp;
    CATCH {
        when X::IO {
            say "File error: {.message}";
        }
    }
}
```

Raku provides specific exception types for different error conditions.  
`X::Str::Numeric` for number parsing errors, `X::IO` for file operations,  
and `X::AdHoc` for general exceptions. Matching specific types enables  
appropriate error responses and recovery strategies.  

## Die statement

Throwing exceptions explicitly with die.  

```raku
sub validate-age($age) {
    die "Age cannot be negative" if $age < 0;
    die "Age must be reasonable" if $age > 150;
    return $age;
}

sub safe-operation($value) {
    die "Invalid input" unless $value.defined;
    
    if $value < 0 {
        die X::OutOfRange.new(
            what => 'value',
            got => $value,
            range => '0..Inf'
        );
    }
    
    return $value * 2;
}

try {
    validate-age(-5);
    CATCH {
        default { say "Validation failed: {.message}" }
    }
}
```

The `die` statement throws exceptions immediately. It can take simple  
strings or exception objects. Using specific exception types with  
parameters provides detailed error information for debugging and  
user feedback.  

## Custom exceptions

Creating custom exception classes for specific error conditions.  

```raku
class ValidationError is Exception {
    has $.field;
    has $.value;
    has $.reason;
    
    method message() {
        "Validation failed for field '{$.field}': " ~
        "'{$.value}' - {$.reason}";
    }
}

class NetworkError is Exception {
    has $.host;
    has $.port;
    has $.timeout;
    
    method message() {
        "Failed to connect to {$.host}:{$.port} " ~
        "(timeout: {$.timeout}s)";
    }
}

sub validate-email($email) {
    die ValidationError.new(
        field => 'email',
        value => $email,
        reason => 'missing @ symbol'
    ) unless $email.contains('@');
    
    return $email;
}

try {
    validate-email("invalid-email");
    CATCH {
        when ValidationError {
            say "Input error: {.message}";
            say "Field: {.field}, Value: {.value}";
        }
    }
}
```

Custom exception classes extend the base Exception class and can include  
specific attributes for error context. The `message` method provides  
formatted error descriptions. This enables precise error categorization  
and detailed debugging information.  

## Exception propagation

Understanding how exceptions propagate through call stacks.  

```raku
sub level-three() {
    die "Error at level 3";
}

sub level-two() {
    level-three();
    say "This won't execute";
}

sub level-one() {
    level-two();
    say "This won't execute either";
}

try {
    level-one();
    CATCH {
        default {
            say "Caught at top level: {.message}";
            say "Exception type: {.^name}";
        }
    }
}

# Partial catching with re-throwing
sub process-data($data) {
    try {
        my $result = risky-operation($data);
        return $result;
        CATCH {
            when X::IO {
                say "Logging I/O error: {.message}";
                die;  # Re-throw the exception
            }
        }
    }
}

sub risky-operation($data) {
    die X::IO.new(message => "File not accessible");
}

try {
    process-data("test");
    CATCH {
        when X::IO { say "Handled at higher level" }
    }
}
```

Exceptions propagate up the call stack until caught. Intermediate  
functions can catch, log, and re-throw exceptions using `die` without  
arguments. This enables layered error handling with logging and  
recovery at appropriate levels.  

## Failure objects

Working with Failure objects for soft exception handling.  

```raku
sub safe-divide($a, $b) {
    return Failure.new("Division by zero") if $b == 0;
    return $a / $b;
}

sub parse-safely($text) {
    my $result = try { Int($text) };
    return $result // Failure.new("Invalid number format");
}

# Using failure objects
my $result1 = safe-divide(10, 0);
if $result1 ~~ Failure {
    say "Operation failed: {$result1.exception.message}";
} else {
    say "Result: $result1";
}

my $result2 = parse-safely("abc");
say $result2.defined ?? "Parsed: $result2" !! "Parse failed";

# Failure objects throw when used
try {
    my $bad = safe-divide(5, 0);
    say $bad + 1;  # This will throw
    CATCH {
        default { say "Failure converted to exception" }
    }
}
```

Failure objects represent soft exceptions that don't immediately throw.  
They convert to exceptions when used in operations. This provides  
flexible error handling where callers can check for failures or let  
them propagate naturally.  

## File I/O error handling

Handling common file and I/O operation errors.  

```raku
sub safe-file-read($filename) {
    try {
        return $filename.IO.slurp;
        CATCH {
            when X::IO::DoesNotExist {
                say "File '$filename' not found";
                return "";
            }
            when X::IO::NoReadPermission {
                say "No permission to read '$filename'";
                return "";
            }
            when X::IO {
                say "I/O error reading '$filename': {.message}";
                return "";
            }
        }
    }
}

sub safe-file-write($filename, $content) {
    try {
        $filename.IO.spurt($content);
        return True;
        CATCH {
            when X::IO::NoWritePermission {
                say "No write permission for '$filename'";
                return False;
            }
            when X::IO {
                say "Write error: {.message}";
                return False;
            }
        }
    }
}

# Usage examples
my $content = safe-file-read("config.txt");
say $content ?? "File content: $content" !! "Using defaults";

my $success = safe-file-write("/tmp/test.txt", "Hello World");
say $success ?? "File written successfully" !! "Write failed";
```

File operations commonly fail due to missing files, permissions, or  
system issues. Specific I/O exception types enable appropriate responses:  
using defaults for missing files, requesting permissions, or providing  
alternative storage locations.  

## Network error handling

Managing network-related exceptions and timeouts.  

```raku
sub fetch-url($url, :$timeout = 30) {
    try {
        # Simulated network operation
        die X::IO::Socket::TimedOut.new if rand > 0.7;
        die X::AdHoc.new(payload => "Connection refused") if rand > 0.8;
        
        return "Response from $url";
        
        CATCH {
            when X::IO::Socket::TimedOut {
                say "Timeout connecting to $url after {$timeout}s";
                return Nil;
            }
            when X::AdHoc {
                say "Network error: {.payload}";
                return Nil;
            }
        }
    }
}

sub resilient-fetch($url, :$retries = 3) {
    for 1..$retries -> $attempt {
        my $result = fetch-url($url);
        return $result if $result.defined;
        
        say "Attempt $attempt failed, retrying..." if $attempt < $retries;
        sleep 1;
    }
    
    die "Failed to fetch $url after $retries attempts";
}

try {
    my $data = resilient-fetch("http://example.com");
    say "Received: $data";
    CATCH {
        default { say "All retry attempts failed: {.message}" }
    }
}
```

Network operations require robust error handling for timeouts, connection  
failures, and server errors. Retry mechanisms with exponential backoff  
improve reliability. Different exception types enable specific recovery  
strategies for various network conditions.  

## Type constraint violations

Handling type-related errors and validation failures.  

```raku
subset PositiveInt of Int where * > 0;
subset EmailStr of Str where /^ \w+ '@' \w+ '.' \w+ $/;

sub process-user-data(PositiveInt $age, EmailStr $email) {
    say "Processing user: age $age, email $email";
}

sub safe-type-conversion($value, $type) {
    try {
        given $type {
            when 'Int' { return Int($value) }
            when 'Num' { return Num($value) }
            when 'Str' { return Str($value) }
            default { die "Unknown type: $type" }
        }
        CATCH {
            when X::Str::Numeric {
                say "Cannot convert '$value' to number";
                return Nil;
            }
            when X::TypeCheck {
                say "Type constraint violation: {.message}";
                return Nil;
            }
        }
    }
}

# Usage with error handling
try {
    process-user-data(-5, "invalid@email");
    CATCH {
        when X::TypeCheck::Binding {
            say "Parameter validation failed: {.message}";
        }
    }
}

my $converted = safe-type-conversion("abc", "Int");
say $converted.defined ?? "Converted: $converted" !! "Conversion failed";
```

Type constraints and subset types provide compile-time and runtime  
validation. `X::TypeCheck` exceptions occur when values don't meet  
type requirements. Safe conversion functions handle type errors  
gracefully and provide fallback values.  

## Exception handling in classes

Implementing error handling within class methods.  

```raku
class BankAccount {
    has $.balance is rw = 0;
    has $.account-number;
    
    method withdraw($amount) {
        die "Invalid amount" if $amount <= 0;
        die "Insufficient funds" if $amount > $.balance;
        
        $.balance -= $amount;
        return $.balance;
    }
    
    method deposit($amount) {
        die "Invalid amount" if $amount <= 0;
        $.balance += $amount;
        return $.balance;
    }
    
    method transfer($target, $amount) {
        try {
            self.withdraw($amount);
            $target.deposit($amount);
            return True;
            CATCH {
                default {
                    say "Transfer failed: {.message}";
                    return False;
                }
            }
        }
    }
}

my $account1 = BankAccount.new(balance => 100, account-number => "123");
my $account2 = BankAccount.new(account-number => "456");

try {
    $account1.withdraw(150);
    CATCH {
        default { say "Withdrawal error: {.message}" }
    }
}

my $success = $account1.transfer($account2, 50);
say $success ?? "Transfer completed" !! "Transfer failed";
```

Class methods should validate inputs and throw meaningful exceptions  
for invalid operations. Internal error handling can provide transaction  
safety and rollback capabilities. Clear exception messages help users  
understand business rule violations.  

## Defensive programming

Implementing defensive coding practices for robust applications.  

```raku
sub robust-calculator($operation, $a, $b) {
    # Input validation
    return Failure.new("Invalid operation") 
        unless $operation ~~ any(<add sub mul div>);
    
    return Failure.new("Non-numeric input") 
        unless $a ~~ Numeric && $b ~~ Numeric;
    
    given $operation {
        when 'add' { $a + $b }
        when 'sub' { $a - $b }
        when 'mul' { $a * $b }
        when 'div' {
            return Failure.new("Division by zero") if $b == 0;
            $a / $b;
        }
    }
}

sub process-array(@data) {
    return [] unless @data.elems > 0;
    
    my @result;
    for @data -> $item {
        next unless $item.defined;
        
        try {
            @result.push: $item.Str;
            CATCH {
                when X::Method::NotFound {
                    say "Warning: Cannot stringify {$item.^name}";
                    @result.push: "N/A";
                }
            }
        }
    }
    
    return @result;
}

# Safe usage patterns
my $calc_result = robust-calculator('div', 10, 2);
if $calc_result ~~ Failure {
    say "Calculation failed: {$calc_result.exception.message}";
} else {
    say "Result: $calc_result";
}

my @processed = process-array([1, "text", Nil, DateTime.now]);
say "Processed: {@processed.join(', ')}";
```

Defensive programming validates inputs, handles edge cases, and provides  
fallback behaviors. Early validation prevents cascading errors. Using  
Failure objects allows callers to choose error handling strategies  
while maintaining program flow.  

## Module error patterns

Error handling strategies within modules and libraries.  

```raku
# file: lib/DataProcessor.rakumod
unit module DataProcessor;

class ProcessingError is Exception is export {
    has $.stage;
    has $.data;
    has $.reason;
    
    method message() {
        "Processing failed at stage '{$.stage}': {$.reason}";
    }
}

sub process-data($data) is export {
    try {
        validate-input($data);
        my $cleaned = clean-data($data);
        my $transformed = transform-data($cleaned);
        return $transformed;
        
        CATCH {
            when ProcessingError {
                .rethrow;  # Re-throw module exceptions
            }
            default {
                die ProcessingError.new(
                    stage => 'unknown',
                    data => $data,
                    reason => .message
                );
            }
        }
    }
}

sub validate-input($data) {
    die ProcessingError.new(
        stage => 'validation',
        data => $data,
        reason => 'data is undefined'
    ) unless $data.defined;
}

sub clean-data($data) {
    # Processing logic here
    return $data;
}

sub transform-data($data) {
    # Transformation logic here
    return $data;
}

# Usage in main program
use DataProcessor;

try {
    my $result = process-data(Nil);
    say "Processed: $result";
    CATCH {
        when ProcessingError {
            say "Module error: {.message}";
            say "Failed stage: {.stage}";
        }
    }
}
```

Modules should define specific exception types for their domain.  
Convert internal exceptions to module-specific ones for consistent  
error interfaces. Export exception classes so users can catch them  
specifically. Provide detailed context in error messages.  

## Async error handling

Managing errors in concurrent and asynchronous operations.  

```raku
sub async-operation($id) {
    start {
        sleep 1;  # Simulate work
        die "Random failure" if rand > 0.7;
        return "Result for $id";
    }
}

sub parallel-processing(@tasks) {
    my @promises = @tasks.map: -> $task {
        start {
            try {
                return process-task($task);
                CATCH {
                    default {
                        say "Task $task failed: {.message}";
                        return Nil;
                    }
                }
            }
        }
    }
    
    return await Promise.allof(@promises).then: {
        @promises.map: *.result
    }
}

sub process-task($task) {
    die "Processing error" if $task eq "bad";
    return "Processed: $task";
}

# Handling async errors
my @tasks = <task1 task2 bad task4>;
my @results = parallel-processing(@tasks);

say "Results:";
for @results.kv -> $i, $result {
    say "  Task {$i + 1}: " ~ ($result // "FAILED");
}

# Promise error handling
my $promise = async-operation("test");
try {
    my $result = await $promise;
    say "Async result: $result";
    CATCH {
        default { say "Async operation failed: {.message}" }
    }
}
```

Asynchronous operations require careful error handling within promises  
and across promise boundaries. Use try/catch within async blocks to  
handle errors locally. The `await` operator can throw exceptions from  
failed promises to the calling context.  

## Testing error conditions

Writing tests to verify error handling behavior.  

```raku
use Test;

sub divide($a, $b) {
    die "Division by zero" if $b == 0;
    return $a / $b;
}

class CustomError is Exception {
    has $.code;
    method message() { "Error code: {$.code}" }
}

sub throw-custom($code) {
    die CustomError.new(code => $code);
}

# Test successful operations
is divide(10, 2), 5, "Normal division works";

# Test exception throwing
throws-like { divide(10, 0) }, Exception, 
    "Division by zero throws exception";

throws-like { divide(10, 0) }, Exception, 
    message => /zero/, 
    "Exception message contains 'zero'";

# Test custom exceptions
throws-like { throw-custom(404) }, CustomError,
    "Custom exception type is thrown";

throws-like { throw-custom(500) }, CustomError,
    message => /500/,
    "Custom exception contains correct code";

# Test error recovery
{
    my $result;
    lives-ok { 
        $result = try { divide(10, 0) } // "default"
    }, "Error recovery doesn't throw";
    
    is $result, "default", "Default value used on error";
}

# Test exception details
{
    my $exception;
    try {
        throw-custom(123);
        CATCH {
            default { $exception = $_ }
        }
    }
    
    isa-ok $exception, CustomError, "Caught correct exception type";
    is $exception.code, 123, "Exception has correct code";
}

done-testing;
```

Testing error conditions ensures exception handling works correctly.  
Use `throws-like` to verify exceptions are thrown with correct types  
and messages. Test both error paths and recovery mechanisms. Verify  
exception attributes and error propagation behavior.  

## Logging and debugging

Implementing comprehensive error logging and debugging support.  

```raku
enum LogLevel <DEBUG INFO WARN ERROR FATAL>;

class Logger {
    has $.level = INFO;
    has $.output = $*OUT;
    
    method log($level, $message, :$exception) {
        return if $level < $.level;
        
        my $timestamp = DateTime.now.Str;
        my $level-name = $level.key;
        
        $.output.say: "[$timestamp] $level-name: $message";
        
        if $exception {
            $.output.say: "  Exception: {$exception.^name}";
            $.output.say: "  Message: {$exception.message}";
            $.output.say: "  Backtrace:";
            for $exception.backtrace.list -> $frame {
                $.output.say: "    {$frame}";
            }
        }
    }
    
    method debug($msg) { self.log(DEBUG, $msg) }
    method info($msg) { self.log(INFO, $msg) }
    method warn($msg) { self.log(WARN, $msg) }
    method error($msg, :$exception) { 
        self.log(ERROR, $msg, :$exception) 
    }
    method fatal($msg, :$exception) { 
        self.log(FATAL, $msg, :$exception) 
    }
}

my $logger = Logger.new(level => DEBUG);

sub risky-operation($data) {
    $logger.debug("Starting operation with data: $data");
    
    try {
        die "Simulated error" if $data eq "fail";
        $logger.info("Operation completed successfully");
        return "Success: $data";
        
        CATCH {
            default {
                $logger.error("Operation failed", exception => $_);
                die;  # Re-throw after logging
            }
        }
    }
}

# Usage with logging
try {
    my $result = risky-operation("fail");
    say $result;
    CATCH {
        default {
            $logger.fatal("Unhandled exception in main", exception => $_);
        }
    }
}
```

Comprehensive logging captures error context, stack traces, and timing  
information. Different log levels allow filtering by severity. Structured  
logging with exception details aids debugging. Log before re-throwing  
to maintain error propagation while capturing context.  

## Error recovery strategies

Implementing various strategies for recovering from errors.  

```raku
class CircuitBreaker {
    has $.threshold = 5;
    has $.timeout = 60;
    has $.failures = 0;
    has $.last-failure-time;
    has $.state = 'closed';  # closed, open, half-open
    
    method call(&operation) {
        given $.state {
            when 'open' {
                if DateTime.now - $.last-failure-time > $.timeout {
                    $.state = 'half-open';
                } else {
                    die "Circuit breaker is open";
                }
            }
        }
        
        try {
            my $result = &operation();
            
            if $.state eq 'half-open' {
                $.state = 'closed';
                $.failures = 0;
            }
            
            return $result;
            
            CATCH {
                default {
                    $.failures++;
                    $.last-failure-time = DateTime.now;
                    
                    if $.failures >= $.threshold {
                        $.state = 'open';
                    }
                    
                    die;
                }
            }
        }
    }
}

sub retry-with-backoff(&operation, :$max-attempts = 3, :$base-delay = 1) {
    for 1..$max-attempts -> $attempt {
        try {
            return &operation();
            CATCH {
                default {
                    if $attempt == $max-attempts {
                        die "Failed after $max-attempts attempts: {.message}";
                    }
                    
                    my $delay = $base-delay * (2 ** ($attempt - 1));
                    say "Attempt $attempt failed, retrying in {$delay}s...";
                    sleep $delay;
                }
            }
        }
    }
}

sub fallback-chain(*@operations) {
    for @operations -> &op {
        try {
            return &op();
            CATCH {
                default {
                    say "Operation failed: {.message}, trying next...";
                }
            }
        }
    }
    
    die "All fallback operations failed";
}

# Usage examples
my $breaker = CircuitBreaker.new(threshold => 2);

# Circuit breaker example
for 1..5 -> $i {
    try {
        my $result = $breaker.call({
            die "Simulated failure" if $i <= 3;
            return "Success on attempt $i";
        });
        say $result;
        CATCH {
            default { say "Call $i failed: {.message}" }
        }
    }
}

# Retry with exponential backoff
my $result = retry-with-backoff({
    die "Temporary failure" if rand > 0.3;
    return "Operation succeeded";
});

say $result;

# Fallback chain
my $final-result = fallback-chain(
    { die "Primary failed" },
    { die "Secondary failed" },
    { "Tertiary succeeded" }
);

say $final-result;
```

Recovery strategies include circuit breakers for failing services,  
exponential backoff for transient errors, and fallback chains for  
redundancy. Each strategy handles different failure patterns and  
provides automatic recovery without manual intervention.  

## Error handling best practices

Guidelines and patterns for effective error handling in Raku.  

```raku
# 1. Use specific exception types
class InvalidInputError is Exception is export {
    has $.input;
    has $.constraint;
    method message() {
        "Invalid input '{$.input}': must satisfy {$.constraint}";
    }
}

# 2. Validate early and fail fast
sub process-user-input($input) {
    die InvalidInputError.new(
        input => $input,
        constraint => 'non-empty string'
    ) if !$input.defined || $input eq '';
    
    die InvalidInputError.new(
        input => $input,
        constraint => 'maximum 100 characters'
    ) if $input.chars > 100;
    
    return $input.uc;
}

# 3. Provide meaningful error messages
sub connect-to-database($host, $port, $database) {
    try {
        # Connection logic here
        die "Connection timeout" if rand > 0.5;
        return "Connected to $database at $host:$port";
        
        CATCH {
            default {
                die "Failed to connect to database '$database' " ~
                    "at $host:$port: {.message}. " ~
                    "Check network connectivity and credentials.";
            }
        }
    }
}

# 4. Use consistent error handling patterns
class APIClient {
    method get($endpoint) {
        self!handle-request('GET', $endpoint);
    }
    
    method post($endpoint, $data) {
        self!handle-request('POST', $endpoint, $data);
    }
    
    method !handle-request($method, $endpoint, $data?) {
        try {
            # HTTP request logic
            die "HTTP 404" if rand > 0.7;
            return "Response from $endpoint";
            
            CATCH {
                when /404/ {
                    die "Endpoint '$endpoint' not found";
                }
                when /5\d\d/ {
                    die "Server error for '$endpoint': {.message}";
                }
                default {
                    die "Request failed for '$endpoint': {.message}";
                }
            }
        }
    }
}

# 5. Document error conditions
#| Process financial transaction with validation
#| Throws: InvalidInputError for invalid amounts
#| Throws: InsufficientFundsError for insufficient balance
#| Throws: NetworkError for communication failures
sub process-payment(
    Numeric $amount where * > 0,  #= Payment amount
    Str $account                   #= Account identifier
    --> Str                        #= Transaction ID
) {
    # Payment processing logic
    return "TXN-12345";
}

# 6. Test error conditions thoroughly
use Test;

# Test normal operation
is process-user-input("hello"), "HELLO", "Normal input processing";

# Test error conditions
throws-like 
    { process-user-input("") }, 
    InvalidInputError, 
    "Empty input throws InvalidInputError";

throws-like 
    { process-user-input("x" x 101) }, 
    InvalidInputError, 
    message => /maximum/,
    "Long input throws constraint error";

# 7. Use defensive programming
sub safe-array-access(@array, $index) {
    return Nil if $index < 0 || $index >= @array.elems;
    return @array[$index];
}

# 8. Implement graceful degradation
sub get-user-preferences($user-id) {
    try {
        return fetch-from-database($user-id);
        CATCH {
            when X::IO {
                say "Database unavailable, using defaults";
                return %{ theme => 'light', lang => 'en' };
            }
        }
    }
}

sub fetch-from-database($user-id) {
    die X::IO.new(message => "Database connection failed");
}

say "Best practices demonstration completed";
done-testing;
```

Best practices include: using specific exception types with context,  
failing fast with early validation, providing actionable error messages,  
implementing consistent error patterns across modules, documenting  
error conditions, testing all error paths, defensive programming for  
robustness, and graceful degradation when possible.  

## Exception chaining

Linking related exceptions to preserve error context.  

```raku
class ChainedError is Exception {
    has $.original-exception;
    has $.context;
    
    method message() {
        my $msg = "Error in {$.context}";
        if $.original-exception {
            $msg ~= ": caused by {$.original-exception.message}";
        }
        return $msg;
    }
}

sub outer-operation($data) {
    try {
        inner-operation($data);
        CATCH {
            default {
                die ChainedError.new(
                    context => 'outer-operation',
                    original-exception => $_
                );
            }
        }
    }
}

sub inner-operation($data) {
    try {
        core-operation($data);
        CATCH {
            default {
                die ChainedError.new(
                    context => 'inner-operation',
                    original-exception => $_
                );
            }
        }
    }
}

sub core-operation($data) {
    die "Core failure: invalid data" if $data eq "bad";
    return "Success";
}

try {
    outer-operation("bad");
    CATCH {
        when ChainedError {
            say "Chained error: {.message}";
            my $current = $_;
            while $current ~~ ChainedError && $current.original-exception {
                $current = $current.original-exception;
                say "  Caused by: {$current.message}";
            }
        }
    }
}
```

Exception chaining preserves the complete error context through multiple  
layers of function calls. Each layer can add its own context while  
maintaining the original root cause. This provides detailed debugging  
information for complex error scenarios.  

## Resource cleanup

Ensuring proper resource cleanup in the presence of exceptions.  

```raku
class FileHandle {
    has $.filename;
    has $.handle;
    has $.is-open = False;
    
    method open() {
        try {
            $.handle = $.filename.IO.open(:w);
            $.is-open = True;
            say "Opened file: {$.filename}";
            CATCH {
                default {
                    die "Failed to open {$.filename}: {.message}";
                }
            }
        }
    }
    
    method close() {
        if $.is-open {
            $.handle.close;
            $.is-open = False;
            say "Closed file: {$.filename}";
        }
    }
    
    method DESTROY() {
        self.close() if $.is-open;
    }
}

sub process-with-cleanup($filename, $content) {
    my $file = FileHandle.new(filename => $filename);
    
    try {
        $file.open();
        $file.handle.say($content);
        
        # Simulate potential error
        die "Processing error" if $content eq "fail";
        
        return "File processed successfully";
        
        CATCH {
            default {
                say "Error during processing: {.message}";
                die;
            }
        }
    }
    
    LEAVE {
        $file.close();  # Always executed
    }
}

# LEAVE ensures cleanup happens regardless of success or failure
try {
    process-with-cleanup("/tmp/test1.txt", "success");
    process-with-cleanup("/tmp/test2.txt", "fail");
    CATCH {
        default { say "Caught final error: {.message}" }
    }
}
```

Proper resource cleanup prevents resource leaks when exceptions occur.  
The `LEAVE` phaser ensures cleanup code runs regardless of how a block  
exits. Implementing cleanup in destructors provides additional safety  
for critical resources like files and network connections.  

## Exception context

Capturing and preserving exception context information.  

```raku
class ContextualException is Exception {
    has %.context;
    has $.base-message;
    
    method message() {
        my $msg = $.base-message;
        if %.context {
            $msg ~= " (Context: ";
            $msg ~= %.context.kv.map({"$^k=$^v"}).join(", ");
            $msg ~= ")";
        }
        return $msg;
    }
    
    method add-context($key, $value) {
        %.context{$key} = $value;
        return self;
    }
}

sub database-operation($table, $id, $operation) {
    try {
        # Simulate database operation
        die "Connection failed" if rand > 0.8;
        die "Table not found" if $table eq "invalid";
        die "Record not found" if $id == -1;
        
        return "Operation $operation completed on $table:$id";
        
        CATCH {
            default {
                die ContextualException.new(
                    base-message => .message
                ).add-context('table', $table)
                 .add-context('id', $id)
                 .add-context('operation', $operation)
                 .add-context('timestamp', DateTime.now.Str);
            }
        }
    }
}

# Usage with context preservation
for (
    ['users', 123, 'select'],
    ['invalid', 456, 'update'],
    ['orders', -1, 'delete']
) -> [$table, $id, $op] {
    try {
        my $result = database-operation($table, $id, $op);
        say $result;
        CATCH {
            when ContextualException {
                say "Database error: {.message}";
                for .context.kv -> $k, $v {
                    say "  $k: $v";
                }
            }
        }
    }
}
```

Contextual exceptions capture environmental information at the point  
of failure. This includes operation parameters, system state, and  
timing information. Rich context enables faster debugging and better  
error reporting for complex systems.  

## Exception filtering

Implementing selective exception handling based on conditions.  

```raku
enum ErrorSeverity <Low Medium High Critical>;

class SeverityException is Exception {
    has $.severity;
    has $.component;
    has $.base-message;
    
    method message() {
        "[{$.severity}] {$.component}: {$.base-message}";
    }
}

sub filtered-operation($component, $severity, $message) {
    die SeverityException.new(
        severity => $severity,
        component => $component,
        base-message => $message
    );
}

sub handle-with-filter(&operation, :@ignore-severities = []) {
    try {
        return &operation();
        CATCH {
            when SeverityException {
                if .severity ~~ any(@ignore-severities) {
                    say "Ignoring {.severity} error: {.base-message}";
                    return Nil;
                } else {
                    say "Handling {.severity} error: {.message}";
                    die;
                }
            }
        }
    }
}

# Filter exceptions based on severity
my @operations = (
    { filtered-operation('auth', Low, 'Cache miss') },
    { filtered-operation('db', Medium, 'Slow query') },
    { filtered-operation('network', High, 'Timeout') },
    { filtered-operation('core', Critical, 'System failure') }
);

for @operations -> &op {
    try {
        handle-with-filter(&op, ignore-severities => [Low, Medium]);
        CATCH {
            when SeverityException {
                say "Escalated error: {.message}";
            }
        }
    }
}
```

Exception filtering allows selective handling based on error attributes  
like severity, component, or type. This enables different treatment  
for different classes of errors: logging minor errors while escalating  
critical ones to monitoring systems.  

## Timeout handling

Managing time-based exceptions and operation timeouts.  

```raku
sub with-timeout(&operation, $timeout-seconds) {
    my $promise = start {
        try {
            return &operation();
            CATCH {
                default { die $_ }
            }
        }
    };
    
    my $timeout = Promise.in($timeout-seconds).then({
        die "Operation timed out after {$timeout-seconds} seconds";
    });
    
    try {
        return await Promise.anyof($promise, $timeout).then({
            if $promise.status == Kept {
                return $promise.result;
            } else {
                die "Operation timed out";
            }
        });
        CATCH {
            default { die $_ }
        }
    }
}

sub slow-operation($duration) {
    say "Starting operation (will take {$duration}s)...";
    sleep $duration;
    return "Operation completed after {$duration}s";
}

# Test timeout scenarios
my @scenarios = (
    { slow-operation(1) },  # Fast
    { slow-operation(3) },  # Medium
    { slow-operation(6) }   # Slow
);

for @scenarios.kv -> $i, &scenario {
    try {
        my $result = with-timeout(&scenario, 4);
        say "Scenario {$i + 1}: $result";
        CATCH {
            default { say "Scenario {$i + 1}: {.message}" }
        }
    }
}
```

Timeout handling prevents operations from hanging indefinitely. Using  
promises with timeouts allows graceful cancellation of long-running  
operations. This is essential for responsive applications and  
preventing resource exhaustion.  

## Exception aggregation

Collecting and reporting multiple exceptions together.  

```raku
class AggregateException is Exception {
    has @.exceptions;
    
    method message() {
        my $count = @.exceptions.elems;
        my $msg = "Multiple errors occurred ($count total):";
        for @.exceptions.kv -> $i, $ex {
            $msg ~= "\n  {$i + 1}. {$ex.message}";
        }
        return $msg;
    }
}

sub process-batch(@items) {
    my @errors;
    my @results;
    
    for @items.kv -> $i, $item {
        try {
            @results.push: process-item($item);
            CATCH {
                default {
                    @errors.push: Exception.new(
                        message => "Item $i failed: {.message}"
                    );
                }
            }
        }
    }
    
    if @errors {
        die AggregateException.new(exceptions => @errors);
    }
    
    return @results;
}

sub process-item($item) {
    die "Invalid item" if $item eq "bad";
    die "Processing error" if $item eq "error";
    return "Processed: $item";
}

# Batch processing with error aggregation
my @batch = <good bad okay error valid>;

try {
    my @results = process-batch(@batch);
    say "All items processed successfully";
    say "Results: {@results.join(', ')}";
    CATCH {
        when AggregateException {
            say .message;
            say "Processed {(@batch.elems - .exceptions.elems)} " ~
                "items successfully";
        }
    }
}
```

Exception aggregation collects multiple errors from batch operations  
and presents them together. This is useful for validation, bulk  
processing, and parallel operations where you want to report all  
failures rather than stopping at the first error.  

## State rollback

Implementing transaction-like rollback on exceptions.  

```raku
class StateManager {
    has %.current-state;
    has @.state-history;
    
    method checkpoint() {
        @.state-history.push: %.current-state.clone;
    }
    
    method set-value($key, $value) {
        %.current-state{$key} = $value;
    }
    
    method get-value($key) {
        return %.current-state{$key};
    }
    
    method rollback() {
        if @.state-history {
            %.current-state = @.state-history.pop;
            say "State rolled back";
        } else {
            say "No checkpoint to rollback to";
        }
    }
    
    method commit() {
        @.state-history = [];
        say "State committed";
    }
}

sub transaction(&operation, $state-manager) {
    $state-manager.checkpoint();
    
    try {
        my $result = &operation($state-manager);
        $state-manager.commit();
        return $result;
        
        CATCH {
            default {
                $state-manager.rollback();
                die "Transaction failed: {.message}";
            }
        }
    }
}

# Usage example
my $manager = StateManager.new(current-state => %{balance => 1000});

say "Initial balance: {$manager.get-value('balance')}";

try {
    transaction({
        my $sm = $^state-manager;
        $sm.set-value('balance', $sm.get-value('balance') - 500);
        say "After withdrawal: {$sm.get-value('balance')}";
        
        # Simulate validation failure
        die "Insufficient funds validation failed";
        
    }, $manager);
    CATCH {
        default { say "Transaction error: {.message}" }
    }
}

say "Final balance: {$manager.get-value('balance')}";
```

State rollback ensures system consistency when operations fail  
partway through. Checkpoint and rollback mechanisms maintain data  
integrity by reverting to known good states. This is crucial for  
financial systems, databases, and stateful applications.  

## Graceful shutdown

Handling shutdown and cleanup during exception scenarios.  

```raku
class ServiceManager {
    has @.services;
    has $.shutdown-initiated = False;
    
    method add-service($service) {
        @.services.push: $service;
    }
    
    method start-all() {
        for @.services -> $service {
            try {
                $service.start();
                CATCH {
                    default {
                        say "Failed to start {$service.name}: {.message}";
                        self.shutdown-all();
                        die "Service startup failed";
                    }
                }
            }
        }
    }
    
    method shutdown-all() {
        return if $.shutdown-initiated;
        $.shutdown-initiated = True;
        
        say "Initiating graceful shutdown...";
        
        for @.services.reverse -> $service {
            try {
                $service.stop();
                CATCH {
                    default {
                        say "Error stopping {$service.name}: {.message}";
                    }
                }
            }
        }
        
        say "Shutdown complete";
    }
}

class Service {
    has $.name;
    has $.should-fail-start = False;
    has $.started = False;
    
    method start() {
        die "Startup failure" if $.should-fail-start;
        $.started = True;
        say "Started service: {$.name}";
    }
    
    method stop() {
        if $.started {
            say "Stopping service: {$.name}";
            $.started = False;
        }
    }
}

# Graceful shutdown scenario
my $manager = ServiceManager.new;

$manager.add-service(Service.new(name => 'Database'));
$manager.add-service(Service.new(name => 'Cache'));
$manager.add-service(Service.new(name => 'WebServer', should-fail-start => True));
$manager.add-service(Service.new(name => 'Monitor'));

try {
    $manager.start-all();
    CATCH {
        default { say "System startup failed: {.message}" }
    }
}

# Ensure cleanup happens even if startup fails
LEAVE {
    $manager.shutdown-all();
}
```

Graceful shutdown ensures proper cleanup when exceptions occur during  
system initialization or operation. Services are stopped in reverse  
order of startup, and errors during shutdown don't prevent other  
services from stopping. This maintains system stability during failures.