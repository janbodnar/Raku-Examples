# Functional Programming

Functional programming in Raku emphasizes immutability, pure functions, and  
declarative data transformations. This chapter covers 30 examples  
demonstrating functional programming techniques, from basic higher-order  
functions to advanced concepts like function composition, lazy evaluation,  
and reactive programming patterns. Each example builds progressive  
understanding of how to write elegant, maintainable code using functional  
paradigms in Raku.

## Basic map transformation

Transforming collection elements with pure functions.  

```raku
my @numbers = 1, 2, 3, 4, 5;

say @numbers.map(* * 2);        # (2 4 6 8 10)
say @numbers.map(* ** 2);       # (1 4 9 16 25)
say @numbers.map(&sqrt);        # (1 1.414213 1.732050 2 2.236067)
```

The `map` method applies a function to each element, creating a new  
collection without modifying the original. Use whatever stars (`*`) for  
simple transformations or pass function references for complex operations.  

## Filter with grep

Selecting elements based on predicates.  

```raku
my @numbers = 1..10;
my @words = <apple banana cherry date elderberry>;

say @numbers.grep(* %% 2);           # (2 4 6 8 10)
say @numbers.grep(* > 5);            # (6 7 8 9 10)
say @words.grep(*.chars > 5);        # (banana cherry elderberry)
say @words.grep(/^[aeiou]/);         # ()
```

The `grep` method filters collections using predicates. Combine with  
regular expressions, method calls, or mathematical conditions to select  
elements that meet specific criteria.  

## Reduce operations

Aggregating collections into single values.  

```raku
my @numbers = 1, 2, 3, 4, 5;

say @numbers.reduce(* + *);          # 15
say @numbers.reduce(* * *);          # 120
say @numbers.reduce(&max);           # 5
say @numbers.reduce(-> $a, $b { $a ~ '-' ~ $b }); # 1-2-3-4-5
```

The `reduce` method applies a binary operation cumulatively to combine  
all elements into a single result. Use for summation, multiplication,  
finding extremes, or building complex aggregate values.  

## Function composition

Combining simple functions to create complex transformations.  

```raku
sub compose(&f, &g) {
    sub (*@args) { f(g(|@args)) }
}

my &add-one = * + 1;
my &double = * * 2;
my &add-then-double = compose(&double, &add-one);

say add-then-double(5);              # 12 (5+1)*2
say (1..5).map(&add-then-double);    # (4 6 8 10 12)
```

Function composition creates new functions by combining existing ones.  
The `compose` function takes two functions and returns a new function  
that applies them in sequence, enabling modular function building.  

## Partial application

Creating specialized functions by fixing some arguments.  

```raku
sub partial(&func, *@fixed-args) {
    sub (*@remaining-args) {
        func(|@fixed-args, |@remaining-args);
    }
}

sub multiply($a, $b) { $a * $b }
my &double = partial(&multiply, 2);
my &triple = partial(&multiply, 3);

say double(5);                       # 10
say (1..5).map(&triple);             # (3 6 9 12 15)
```

Partial application creates specialized functions by pre-filling some  
arguments. This technique enables creating families of related functions  
from a single general-purpose function.  

## Currying

Converting multi-argument functions into chains of single-argument functions.  

```raku
sub curry(&func) {
    sub ($a) {
        sub ($b) {
            sub ($c) {
                func($a, $b, $c)
            }
        }
    }
}

sub add($a, $b, $c) { $a + $b + $c }
my &curried-add = curry(&add);

my &add5 = curried-add(5);
my &add5and3 = add5(3);
say add5and3(2);                     # 10
```

Currying transforms functions to take one argument at a time, returning  
functions that accept the remaining arguments. This enables elegant  
function specialization and composition patterns.  

## Higher-order function creation

Building functions that operate on other functions.  

```raku
sub apply-twice(&func) {
    sub ($value) { func(func($value)) }
}

sub negate(&predicate) {
    sub (*@args) { !predicate(|@args) }
}

my &square = * ** 2;
my &double-square = apply-twice(&square);
my &is-odd = * % 2;
my &is-even = negate(&is-odd);

say double-square(3);                # 81 (3²)²
say (1..5).grep(&is-even);           # (2 4)
```

Higher-order functions accept functions as parameters and return new  
functions. They enable powerful abstractions for function transformation,  
composition, and behavioral modification.  

## Lazy evaluation

Deferring computation until values are actually needed.  

```raku
my $lazy-range = lazy 1..*;
my $lazy-squares = $lazy-range.map(* ** 2);
my $lazy-evens = $lazy-squares.grep(* %% 2);

say $lazy-evens[0..4];               # (4 16 36 64 100)

# Infinite Fibonacci sequence
my @fib = lazy 0, 1, * + * ... *;
say @fib[0..9];                      # (0 1 1 2 3 5 8 13 21 34)
```

Lazy evaluation creates sequences that compute values on-demand rather  
than immediately. This enables working with infinite sequences and  
improves performance for operations on large datasets.  

## Memoization

Caching function results to avoid redundant calculations.  

```raku
sub memoize(&func) {
    my %cache;
    sub (*@args) {
        my $key = @args.join(',');
        %cache{$key} //= func(|@args);
    }
}

sub fibonacci($n) {
    return $n if $n <= 1;
    fibonacci($n - 1) + fibonacci($n - 2);
}

my &memo-fib = memoize(&fibonacci);
say memo-fib(40);                    # Much faster than regular fibonacci
```

Memoization caches expensive function results to improve performance.  
The cached version maintains mathematical correctness while dramatically  
reducing computation time for recursive functions.  

## Immutable data transformations

Creating new data structures instead of modifying existing ones.  

```raku
sub add-item(@list, $item) {
    |@list, $item
}

sub remove-item(@list, $index) {
    @list[0..$index-1], @list[$index+1..*]
}

sub update-item(@list, $index, $new-value) {
    @list[0..$index-1], $new-value, @list[$index+1..*]
}

my @original = 1, 2, 3, 4, 5;
my @with-item = add-item(@original, 6);
my @without-third = remove-item(@original, 2);
say @original;                       # (1 2 3 4 5) - unchanged
say @with-item;                      # (1 2 3 4 5 6)
say @without-third;                  # (1 2 4 5)
```

Immutable transformations create new data structures rather than modifying  
existing ones. This approach prevents side effects and makes code more  
predictable and easier to reason about.  

## Function pipelining

Chaining operations using the feed operator.  

```raku
my @words = <the quick brown fox jumps over lazy dog>;

my $result = @words
    ==> grep(*.chars > 3)
    ==> map(*.uc)
    ==> sort()
    ==> join(' ');

say $result;                         # BROWN JUMPS LAZY OVER QUICK

# Alternative pipeline syntax
my @numbers = 1..10;
my $sum = @numbers
    ==> grep(* %% 2)
    ==> map(* ** 2)
    ==> sum();

say $sum;                            # 220
```

Pipeline operations use the feed operator (`==>`) to pass data through  
a sequence of transformations. This creates readable processing chains  
that clearly show the flow of data transformation.  

## Closures and lexical scope

Functions that capture variables from their surrounding scope.  

```raku
sub make-counter($start = 0) {
    my $count = $start;
    sub {
        ++$count;
    }
}

sub make-multiplier($factor) {
    sub ($value) {
        $value * $factor;
    }
}

my &counter1 = make-counter();
my &counter2 = make-counter(10);
my &triple = make-multiplier(3);

say counter1();                      # 1
say counter1();                      # 2
say counter2();                      # 11
say triple(5);                       # 15
```

Closures are functions that retain access to variables from their  
creation scope. They enable creating stateful functions and function  
factories that generate specialized behavior.  

## Recursive functional patterns

Elegant solutions using function recursion.  

```raku
sub factorial($n) {
    $n <= 1 ?? 1 !! $n * factorial($n - 1);
}

sub quick-sort(@list) {
    return @list if @list <= 1;
    
    my $pivot = @list[0];
    my @less = @list[1..*].grep(* <= $pivot);
    my @greater = @list[1..*].grep(* > $pivot);
    
    |quick-sort(@less), $pivot, |quick-sort(@greater);
}

sub tree-map(&func, $tree) {
    given $tree {
        when List { $tree.map({ tree-map(&func, $_) }) }
        default   { func($tree) }
    }
}

say factorial(6);                    # 720
say quick-sort(<3 1 4 1 5 9 2 6>);   # (1 1 2 3 4 5 6 9)
```

Recursive patterns express solutions naturally for problems with  
self-similar structure. Functional recursion emphasizes breaking  
problems into smaller cases and combining results.  

## Function filtering and selection

Advanced techniques for working with collections of functions.  

```raku
my @predicates = 
    * > 5,
    * %% 2,
    *.is-prime,
    * < 20;

sub satisfies-all($value, @predicates) {
    @predicates.all($value)
}

sub satisfies-any($value, @predicates) {
    @predicates.any($value)
}

my @numbers = 1..30;
my @all-satisfied = @numbers.grep({ satisfies-all($_, @predicates) });
my @any-satisfied = @numbers.grep({ satisfies-any($_, @predicates) });

say @all-satisfied;                  # Numbers > 5, even, prime, < 20
say @any-satisfied.elems;            # Count meeting any criteria
```

Function collections enable building complex filtering logic by  
combining multiple predicates. Use junction operations to test  
values against multiple criteria simultaneously.  

## Functional data structures

Implementing stack and list operations functionally.  

```raku
role FunctionalStack {
    method push($item) { ... }
    method pop() { ... }
    method peek() { ... }
    method is-empty() { ... }
}

class ListStack does FunctionalStack {
    has @.items;
    
    method push($item) {
        self.new(items => |@.items, $item);
    }
    
    method pop() {
        return Nil, self if @.items == 0;
        my $top = @.items[*-1];
        my $new-stack = self.new(items => @.items[0..*-2]);
        return $top, $new-stack;
    }
    
    method peek() { @.items[*-1] }
    method is-empty() { @.items == 0 }
}

my $stack = ListStack.new();
my $stack1 = $stack.push(1).push(2).push(3);
my ($top, $stack2) = $stack1.pop();
say $top;                            # 3
say $stack2.peek();                  # 2
```

Functional data structures maintain immutability by returning new  
instances rather than modifying existing ones. This approach eliminates  
side effects and enables safe concurrent access.  

## Monadic patterns

Implementing Optional/Maybe pattern for safe value handling.  

```raku
role Maybe[::T] {
    method map(&func) { ... }
    method flat-map(&func) { ... }
    method filter(&predicate) { ... }
    method or-else($default) { ... }
}

class Some[::T] does Maybe[T] {
    has $.value;
    
    method map(&func) { Some.new(value => func($.value)) }
    method flat-map(&func) { func($.value) }
    method filter(&predicate) { 
        predicate($.value) ?? self !! None.new 
    }
    method or-else($default) { $.value }
}

class None does Maybe {
    method map(&func) { self }
    method flat-map(&func) { self }
    method filter(&predicate) { self }
    method or-else($default) { $default }
}

sub safe-divide($a, $b) {
    $b == 0 ?? None.new !! Some.new(value => $a / $b);
}

my $result = safe-divide(10, 2)
    .map(* * 2)
    .filter(* > 5)
    .or-else(0);

say $result;                         # 10
```

Monadic patterns provide safe computation chains that handle success  
and failure cases elegantly. The Maybe monad eliminates null pointer  
exceptions through explicit optional value handling.  

## Async functional programming

Applying functional patterns to asynchronous operations.  

```raku
sub async-map(@items, &func) {
    @items.map({ start { func($_) } });
}

sub async-filter(@items, &predicate) {
    my @promises = @items.map({ start { $_ if predicate($_) } });
    await(@promises).grep(*.defined);
}

sub fetch-data($id) {
    start {
        # Simulate async operation
        sleep 0.1;
        "Data for $id";
    }
}

my @ids = 1..5;
my @data-promises = async-map(@ids, &fetch-data);
my @results = await @data-promises;

say @results;                        # Fetched data for all IDs

# Async pipeline
my @filtered = await async-filter(1..10, { $_ %% 2 });
say @filtered;                       # (2 4 6 8 10)
```

Asynchronous functional programming combines promises with functional  
transformations. This enables efficient parallel processing while  
maintaining functional programming principles.  

## Stream processing

Working with infinite or large data streams functionally.  

```raku
sub fibonacci-stream() {
    gather {
        my ($a, $b) = 0, 1;
        loop {
            take $a;
            ($a, $b) = $b, $a + $b;
        }
    }
}

sub prime-stream() {
    gather {
        for 2..* -> $n {
            take $n if $n.is-prime;
        }
    }
}

sub take-while(@stream, &predicate) {
    gather {
        for @stream -> $item {
            last unless predicate($item);
            take $item;
        }
    }
}

my @first-10-fibs = fibonacci-stream().head(10);
my @small-primes = take-while(prime-stream(), * < 100);

say @first-10-fibs;                  # (0 1 1 2 3 5 8 13 21 34)
say @small-primes.elems;             # Count of primes < 100
```

Stream processing handles potentially infinite data sequences using  
functional transformations. The `gather/take` construct creates  
efficient lazy generators for streaming operations.  

## Function decorators

Enhancing functions with additional behavior.  

```raku
sub timing-decorator(&func) {
    sub (*@args) {
        my $start = now;
        my $result = func(|@args);
        my $duration = now - $start;
        say "Function took {$duration} seconds";
        $result;
    }
}

sub logging-decorator(&func) {
    sub (*@args) {
        say "Calling function with args: {@args}";
        my $result = func(|@args);
        say "Function returned: $result";
        $result;
    }
}

sub retry-decorator(&func, $max-attempts = 3) {
    sub (*@args) {
        for 1..$max-attempts -> $attempt {
            try {
                return func(|@args);
                CATCH {
                    say "Attempt $attempt failed: {.message}";
                    die $_ if $attempt == $max-attempts;
                }
            }
        }
    }
}

sub slow-operation($n) {
    sleep 0.1;
    $n * 2;
}

my &timed-operation = timing-decorator(&slow-operation);
say timed-operation(5);              # Shows timing information
```

Function decorators wrap existing functions to add cross-cutting  
concerns like logging, timing, or error handling without modifying  
the original function implementation.  

## Functional reactive programming

Creating reactive data streams using functional principles.  

```raku
class Observable {
    has @.observers;
    
    method subscribe(&observer) {
        @.observers.push(&observer);
    }
    
    method emit($value) {
        .($value) for @.observers;
    }
    
    method map(&func) {
        my $mapped = Observable.new;
        self.subscribe(-> $value {
            $mapped.emit(func($value));
        });
        $mapped;
    }
    
    method filter(&predicate) {
        my $filtered = Observable.new;
        self.subscribe(-> $value {
            $filtered.emit($value) if predicate($value);
        });
        $filtered;
    }
}

my $source = Observable.new;
my $doubled = $source.map(* * 2);
my $evens = $doubled.filter(* %% 2);

$evens.subscribe(-> $value {
    say "Received: $value";
});

$source.emit(1);                     # No output (1*2=2, but not even)
$source.emit(2);                     # "Received: 4"
$source.emit(3);                     # "Received: 6"
```

Functional reactive programming combines observables with functional  
transformations to create responsive data processing pipelines for  
event-driven applications.  

## Catamorphisms

Generalized folding operations for data structures.  

```raku
enum Tree { Leaf | Branch }

role TreeNode {
    method fold(&leaf-func, &branch-func) { ... }
}

class LeafNode does TreeNode {
    has $.value;
    method fold(&leaf-func, &branch-func) {
        leaf-func($.value);
    }
}

class BranchNode does TreeNode {
    has $.left;
    has $.right;
    method fold(&leaf-func, &branch-func) {
        my $left-result = $.left.fold(&leaf-func, &branch-func);
        my $right-result = $.right.fold(&leaf-func, &branch-func);
        branch-func($left-result, $right-result);
    }
}

my $tree = BranchNode.new(
    left => LeafNode.new(value => 1),
    right => BranchNode.new(
        left => LeafNode.new(value => 2),
        right => LeafNode.new(value => 3)
    )
);

my $sum = $tree.fold(
    { $_ },                          # leaf function
    { $^a + $^b }                    # branch function
);

my $max = $tree.fold(
    { $_ },
    { max($^a, $^b) }
);

say $sum;                            # 6
say $max;                            # 3
```

Catamorphisms provide a general pattern for folding tree-like data  
structures into single values. The fold operation abstracts common  
tree traversal patterns into reusable functions.  

## Function combinators

Building complex functions from simple combinators.  

```raku
sub identity($x) { $x }

sub const($value) {
    sub (*@args) { $value }
}

sub flip(&func) {
    sub ($b, $a, *@rest) { func($a, $b, |@rest) }
}

sub on(&binary-op, &unary-func) {
    sub ($a, $b) {
        binary-op(unary-func($a), unary-func($b));
    }
}

sub fork(&func1, &func2, &combiner) {
    sub ($value) {
        combiner(func1($value), func2($value));
    }
}

# Usage examples
my &subtract = { $^a - $^b };
my &flipped-subtract = flip(&subtract);
my &length-sum = on(&[+], &chars);
my &stat-summary = fork(&sum, &elems, { "$^a total, $^b items" });

say flipped-subtract(10, 3);         # -7 (3 - 10)
say length-sum("hello", "world");    # 10 (5 + 5)
say stat-summary((1, 2, 3, 4, 5));   # "15 total, 5 items"
```

Function combinators are higher-order functions that combine other  
functions in systematic ways. They provide building blocks for  
constructing complex function behavior from simple parts.  

## Purely functional algorithms

Implementing classic algorithms using only functional techniques.  

```raku
sub merge-sort(@list) {
    return @list if @list <= 1;
    
    my $mid = @list.elems div 2;
    my @left = merge-sort(@list[0..$mid-1]);
    my @right = merge-sort(@list[$mid..*]);
    
    merge(@left, @right);
}

sub merge(@left, @right) {
    return @right unless @left;
    return @left unless @right;
    
    if @left[0] <= @right[0] {
        @left[0], |merge(@left[1..*], @right);
    } else {
        @right[0], |merge(@left, @right[1..*]);
    }
}

sub binary-search(@sorted-list, $target) {
    sub search($low, $high) {
        return Nil if $low > $high;
        
        my $mid = ($low + $high) div 2;
        my $mid-value = @sorted-list[$mid];
        
        given $mid-value cmp $target {
            when Same { $mid }
            when More { search($low, $mid - 1) }
            when Less { search($mid + 1, $high) }
        }
    }
    
    search(0, @sorted-list.end);
}

my @unsorted = <5 2 8 1 9 3>;
my @sorted = merge-sort(@unsorted);
my $index = binary-search(@sorted, 8);

say @sorted;                         # (1 2 3 5 8 9)
say $index;                          # Index of 8 in sorted array
```

Pure functional algorithms avoid mutable state and side effects.  
They use recursion and immutable data structures to solve problems  
elegantly while maintaining referential transparency.  

## Functional error handling

Using functional patterns for robust error handling.  

```raku
role Result[::T, ::E] {
    method map(&func) { ... }
    method flat-map(&func) { ... }
    method map-error(&func) { ... }
    method or-else($default) { ... }
}

class Success[::T, ::E] does Result[T, E] {
    has $.value;
    
    method map(&func) { 
        try {
            Success.new(value => func($.value));
            CATCH {
                Failure.new(error => $_);
            }
        }
    }
    
    method flat-map(&func) { func($.value) }
    method map-error(&func) { self }
    method or-else($default) { $.value }
}

class Failure[::T, ::E] does Result[T, E] {
    has $.error;
    
    method map(&func) { self }
    method flat-map(&func) { self }
    method map-error(&func) { 
        Failure.new(error => func($.error)) 
    }
    method or-else($default) { $default }
}

sub safe-parse-int($str) {
    try {
        Success.new(value => +$str);
        CATCH {
            Failure.new(error => "Invalid number: $str");
        }
    }
}

sub safe-sqrt($n) {
    $n >= 0 
        ?? Success.new(value => sqrt($n))
        !! Failure.new(error => "Negative number: $n");
}

my $result = safe-parse-int("16")
    .flat-map(&safe-sqrt)
    .map(* * 2)
    .or-else(0);

say $result;                         # 8
```

Functional error handling uses Result types to make error conditions  
explicit in function signatures. This eliminates hidden exceptions  
and creates predictable error propagation patterns.  

## Function memoization with TTL

Advanced memoization with time-based cache expiration.  

```raku
sub memoize-with-ttl(&func, $ttl-seconds = 60) {
    my %cache;
    my %timestamps;
    
    sub (*@args) {
        my $key = @args.join(',');
        my $now = now;
        
        # Check if cached value exists and is still valid
        if %cache{$key}:exists {
            if $now - %timestamps{$key} <= $ttl-seconds {
                return %cache{$key};
            } else {
                # Cache expired, remove entry
                %cache{$key}:delete;
                %timestamps{$key}:delete;
            }
        }
        
        # Compute and cache new value
        my $result = func(|@args);
        %cache{$key} = $result;
        %timestamps{$key} = $now;
        $result;
    }
}

sub expensive-operation($n) {
    say "Computing expensive operation for $n";
    sleep 0.1;  # Simulate expensive computation
    $n ** 3;
}

my &cached-operation = memoize-with-ttl(&expensive-operation, 2);

say cached-operation(5);             # Computes and caches
say cached-operation(5);             # Returns cached value
sleep 3;
say cached-operation(5);             # Recomputes after expiration
```

Time-based memoization combines the performance benefits of caching  
with automatic cache invalidation. This prevents stale data while  
maintaining the speed benefits of memoization.  

## Transducers

Composable, efficient data transformation pipelines.  

```raku
sub mapping(&func) {
    sub (&reducer) {
        sub ($accumulator, $input) {
            reducer($accumulator, func($input));
        }
    }
}

sub filtering(&predicate) {
    sub (&reducer) {
        sub ($accumulator, $input) {
            predicate($input) 
                ?? reducer($accumulator, $input)
                !! $accumulator;
        }
    }
}

sub taking($n) {
    my $count = 0;
    sub (&reducer) {
        sub ($accumulator, $input) {
            return $accumulator if ++$count > $n;
            reducer($accumulator, $input);
        }
    }
}

sub transduce(&transducer, &reducer, $init, @collection) {
    my &transformed-reducer = transducer(&reducer);
    @collection.reduce(&transformed-reducer, $init);
}

my &pipeline = { filtering(* %% 2)(mapping(* ** 2)($_)) };
my @numbers = 1..10;

my $sum = transduce(
    &pipeline,
    &[+],
    0,
    @numbers
);

say $sum;                            # Sum of squares of even numbers
```

Transducers provide efficient, composable transformations that work  
with any reducible collection. They avoid creating intermediate  
collections, improving memory efficiency for large datasets.  

## Persistent data structures

Implementing efficient immutable data structures.  

```raku
class PersistentList {
    has $.head;
    has $.tail;
    has $.size = 0;
    
    method new($head?, $tail?) {
        my $size = $tail.defined ?? $tail.size + 1 !! ($head.defined ?? 1 !! 0);
        self.bless(:$head, :$tail, :$size);
    }
    
    method cons($item) {
        PersistentList.new($item, self);
    }
    
    method first() { $.head }
    method rest() { $.tail // PersistentList.new() }
    method is-empty() { $.size == 0 }
    
    method map(&func) {
        return PersistentList.new() if self.is-empty;
        self.rest.map(&func).cons(func(self.first));
    }
    
    method filter(&predicate) {
        return PersistentList.new() if self.is-empty;
        my $filtered-rest = self.rest.filter(&predicate);
        predicate(self.first) 
            ?? $filtered-rest.cons(self.first)
            !! $filtered-rest;
    }
    
    method to-array() {
        return () if self.is-empty;
        self.first, |self.rest.to-array;
    }
}

my $list = PersistentList.new()
    .cons(1)
    .cons(2)
    .cons(3)
    .cons(4)
    .cons(5);

my $doubled = $list.map(* * 2);
my $evens = $list.filter(* %% 2);

say $list.to-array;                  # (5 4 3 2 1)
say $doubled.to-array;               # (10 8 6 4 2)
say $evens.to-array;                 # (4 2)
```

Persistent data structures provide efficient immutable operations  
through structural sharing. They enable safe concurrent access and  
undo functionality while maintaining good performance characteristics.  

## Functional dependency injection

Using higher-order functions for dependency injection.  

```raku
sub create-user-service(&database, &logger, &validator) {
    sub create-user($user-data) {
        try {
            logger("Validating user data");
            return Failure.new(error => "Invalid data") 
                unless validator($user-data);
            
            logger("Saving user to database");
            my $user-id = database('insert', $user-data);
            
            logger("User created with ID: $user-id");
            Success.new(value => $user-id);
            
            CATCH {
                logger("Error creating user: {.message}");
                Failure.new(error => .message);
            }
        }
    }
}

# Mock implementations
sub mock-database($operation, $data) {
    given $operation {
        when 'insert' { 42 }  # Mock user ID
        default { die "Unknown operation: $operation" }
    }
}

sub console-logger($message) {
    say "[LOG] $message";
}

sub simple-validator($user-data) {
    $user-data<name>.defined && $user-data<email>.defined;
}

my &user-service = create-user-service(
    &mock-database,
    &console-logger,
    &simple-validator
);

my $result = user-service({
    name => "Alice",
    email => "alice@example.com"
});

say $result.or-else("Failed");       # 42
```

Functional dependency injection uses higher-order functions to provide  
dependencies rather than traditional IoC containers. This approach  
maintains functional purity while enabling testable, modular designs.  

## Performance optimization patterns

Techniques for optimizing functional code performance.  

```raku
# Tail recursion optimization
sub tail-recursive-factorial($n, $acc = 1) {
    $n <= 1 ?? $acc !! tail-recursive-factorial($n - 1, $n * $acc);
}

# Lazy evaluation for memory efficiency
sub lazy-prime-sieve(@numbers) {
    lazy gather {
        while @numbers {
            my $prime = @numbers.shift;
            take $prime;
            @numbers = @numbers.grep(* % $prime != 0);
        }
    }
}

# Short-circuiting for early termination
sub any-match(@items, &predicate) {
    for @items -> $item {
        return True if predicate($item);
    }
    False;
}

sub all-match(@items, &predicate) {
    for @items -> $item {
        return False unless predicate($item);
    }
    True;
}

# Batch processing for better cache locality
sub batch-process(@items, &processor, $batch-size = 100) {
    @items.batch($batch-size).map({ .map(&processor) }).flat;
}

# Example usage
my @large-numbers = 1..10000;
my @primes = lazy-prime-sieve(@large-numbers);

say @primes.head(10);                # First 10 primes
say tail-recursive-factorial(1000);  # Large factorial
say any-match(@large-numbers, * > 5000);  # True
say all-match(1..100, * > 0);        # True
```

Performance optimization in functional programming focuses on tail  
recursion, lazy evaluation, and efficient data access patterns.  
These techniques maintain functional purity while achieving good  
performance characteristics.  

## Functional type classes

Implementing generic functional abstractions with type classes.  

```raku
role Functor[::F] {
    method map(::F $self, &func) { ... }
}

role Applicative[::F] does Functor[F] {
    method pure(::F:U $type, $value) { ... }
    method apply(::F $self, ::F $func-container) { ... }
}

role Monad[::F] does Applicative[F] {
    method bind(::F $self, &func) { ... }
}

class Optional does Monad[Optional] {
    has $.value;
    has $.has-value;
    
    method new($value?) {
        self.bless(value => $value, has-value => $value.defined);
    }
    
    method map(&func) {
        return self unless $.has-value;
        Optional.new(func($.value));
    }
    
    method bind(&func) {
        return self unless $.has-value;
        func($.value);
    }
    
    method pure($type, $value) {
        Optional.new($value);
    }
    
    method apply($func-container) {
        return self unless $.has-value;
        return $func-container unless $func-container.has-value;
        Optional.new($func-container.value.($.value));
    }
}

my $opt1 = Optional.new(5);
my $opt2 = Optional.new();
my $result = $opt1.map(* * 2).bind(-> $x { Optional.new($x + 1) });

say $result.value if $result.has-value;  # 11
say $opt2.map(* * 2).has-value;          # False
```

Type classes provide generic abstractions for functional patterns.  
Functors enable mapping, Applicatives allow function application in  
context, and Monads provide sequential computation chaining. These  
patterns create reusable, composable functional abstractions.  
