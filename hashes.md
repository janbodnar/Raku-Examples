# Hashes

Hashes are associative arrays that store key-value pairs, providing fast  
lookup of values by their associated keys. In Raku, hashes use the `%` sigil  
and offer powerful features for data organization, transformation, and  
manipulation. They are mutable by default and can contain values of any type,  
with keys typically being strings. This chapter explores hash creation,  
access patterns, modification techniques, and advanced operations that make  
hashes essential for data processing and application development.  

## Hash creation

Basic hash creation using different syntaxes.  

```raku
# Using => (fat arrow) pairs
my %colors = (
    'red' => '#FF0000',
    'green' => '#00FF00',
    'blue' => '#0000FF'
);

# Using colon pairs
my %animals = :cat<meow>, :dog<woof>, :cow<moo>;

# Using angle brackets for string values
my %fruits = <apple red banana yellow orange orange>;

say %colors;
say %animals;
say %fruits;
```

The fat arrow `=>` syntax creates key-value pairs explicitly. Colon pairs  
`:key<value>` provide a concise syntax for string values. Angle bracket  
notation creates pairs from consecutive elements where odd positions become  
keys and even positions become values.  

## Hash literals

Different ways to create hash literals inline.  

```raku
# Empty hash
my %empty = ();

# Hash with mixed value types
my %mixed = (
    'name' => 'Alice',
    'age' => 30,
    'active' => True,
    'score' => 95.5
);

# Hash from list of pairs
my %grades = 'Alice' => 95, 'Bob' => 87, 'Carol' => 92;

say %empty.elems;
say %mixed;
say %grades;
```

Empty hashes are created with empty parentheses. Hash values can be of  
different types including strings, numbers, and booleans. Lists of pairs  
are automatically converted to hashes when assigned to a hash variable.  

## Hash access

Accessing values using keys and handling missing keys.  

```raku
my %person = (
    'name' => 'John Doe',
    'age' => 35,
    'city' => 'New York'
);

# Basic access
say %person<name>;
say %person{'age'};
say %person<city>;

# Multiple key access
say %person<name age>;

# Missing key returns Nil
say %person<salary>;
say %person<salary> // 'Not specified';
```

Hash access uses angle brackets `<key>` or curly braces `{'key'}`. Multiple  
keys can be accessed simultaneously returning a list of values. Missing keys  
return `Nil`, which can be handled with the defined-or operator `//` to  
provide default values.  

## Hash assignment

Modifying and adding new key-value pairs to hashes.  

```raku
my %config = (
    'debug' => False,
    'port' => 8080
);

# Modify existing key
%config<debug> = True;

# Add new keys
%config<host> = 'localhost';
%config{'timeout'} = 30;

# Multiple assignment
%config<user password> = 'admin', 'secret';

say %config;
```

Hash assignment uses the same syntax as access but with the assignment  
operator `=`. New keys are automatically created when assigned. Multiple  
key assignment allows setting several keys at once with corresponding  
values from a list.  

## Hash existence

Checking if keys exist in hashes before accessing them.  

```raku
my %inventory = (
    'apples' => 50,
    'bananas' => 0,
    'oranges' => 25
);

# Check key existence
say 'apples' ∈ %inventory;
say %inventory<apples>:exists;
say %inventory<grapes>:exists;

# Check for defined values
say %inventory<bananas>:defined;
say %inventory<apples>.defined;

say 'Has apples' if %inventory<apples>:exists;
```

The `∈` operator or `:exists` adverb check if a key exists in the hash.  
The `:defined` adverb checks if a value is defined (not Nil). Note that  
a key can exist but have an undefined or zero value, so existence and  
definedness are different concepts.  

## Hash deletion

Removing key-value pairs from hashes.  

```raku
my %data = (
    'temp' => 25,
    'humidity' => 60,
    'pressure' => 1013,
    'wind' => 5
);

say %data;

# Delete single key
%data<temp>:delete;

# Delete multiple keys
%data<humidity pressure>:delete;

say %data;
say %data.elems;
```

The `:delete` adverb removes keys and their associated values from the  
hash. Multiple keys can be deleted simultaneously by providing a list of  
keys. After deletion, the hash size decreases and the keys no longer exist  
in the hash.  

## Hash keys and values

Extracting all keys and values from hashes.  

```raku
my %scores = (
    'Alice' => 95,
    'Bob' => 87,
    'Carol' => 92,
    'David' => 78
);

# Get all keys
my @names = %scores.keys;
say @names;

# Get all values
my @points = %scores.values;
say @points;

# Get sorted keys
say %scores.keys.sort;

# Get values in key order
say %scores{%scores.keys.sort};
```

The `.keys` method returns all hash keys as a list, while `.values` returns  
all values. Keys are returned in arbitrary order unless explicitly sorted.  
To get values in a specific key order, use hash access with the sorted keys  
as indices.  

## Hash pairs

Working with key-value pairs as Pair objects.  

```raku
my %products = (
    'laptop' => 999,
    'mouse' => 25,
    'keyboard' => 75
);

# Get all pairs
for %products.pairs -> $pair {
    say "Product: {$pair.key}, Price: \${$pair.value}";
}

# Pairs in list context
my @product_pairs = %products.pairs;
say @product_pairs;

# Using kv method for alternating keys/values
for %products.kv -> $product, $price {
    say "$product costs \$$price";
}
```

The `.pairs` method returns Pair objects containing both key and value.  
Each pair has `.key` and `.value` methods for accessing components. The  
`.kv` method provides an alternative that returns alternating keys and  
values suitable for iteration with two-parameter blocks.  

## Hash iteration

Different ways to iterate through hash contents.  

```raku
my %cities = (
    'Tokyo' => 14000000,
    'Delhi' => 11000000,
    'Shanghai' => 9500000
);

# Iterate over pairs
for %cities -> $pair {
    say "{$pair.key}: {$pair.value} people";
}

# Iterate with kv
for %cities.kv -> $city, $population {
    say "$city has $population inhabitants";
}

# Iterate over keys
for %cities.keys -> $city {
    say "$city: {%cities{$city}}";
}
```

Hash iteration can be done over pairs (default), using `.kv` for separate  
key-value parameters, or iterating over keys and looking up values. The  
pair iteration is most common, while `.kv` is useful when you need separate  
variables for keys and values.  

## Hash size

Getting information about hash dimensions and contents.  

```raku
my %library = (
    'fiction' => 245,
    'science' => 156,
    'history' => 89,
    'art' => 67
);

# Number of key-value pairs
say %library.elems;
say %library.end;

# Check if empty
my %empty = ();
say %library.so;
say %empty.so;

# Bool context
say 'Library has books' if %library;
say 'Empty library' unless %empty;
```

The `.elems` method returns the number of key-value pairs in the hash. The  
`.end` method returns the highest valid index (elems - 1). Hashes evaluate  
to `True` in boolean context when they contain elements, and `False` when  
empty, making them useful in conditional statements.  

## Hash sorting

Sorting hashes by keys or values.  

```raku
my %grades = (
    'Alice' => 95,
    'Bob' => 87,
    'Carol' => 92,
    'David' => 78,
    'Eve' => 98
);

# Sort by keys
for %grades.keys.sort -> $name {
    say "$name: {%grades{$name}}";
}

# Sort by values
for %grades.sort(*.value) -> $pair {
    say "{$pair.key}: {$pair.value}";
}

# Sort by values descending
for %grades.sort(-*.value) -> $pair {
    say "{$pair.key}: {$pair.value}";
}
```

Sorting hash keys returns them in alphabetical order. Sorting the hash  
itself by `.value` returns pairs ordered by their values. The negative  
unary operator `-` before `*.value` sorts in descending order. Custom  
sort criteria can be applied using comparison functions.  

## Hash filtering

Selecting specific key-value pairs based on conditions.  

```raku
my %temperatures = (
    'Monday' => 22,
    'Tuesday' => 18,
    'Wednesday' => 25,
    'Thursday' => 19,
    'Friday' => 27,
    'Saturday' => 30,
    'Sunday' => 24
);

# Filter by value
my %hot_days = %temperatures.grep(*.value > 25);
say %hot_days;

# Filter by key
my %weekdays = %temperatures.grep(*.key ne 'Saturday' | 'Sunday');
say %weekdays;

# Filter with block
my %mild_days = %temperatures.grep({ 20 <= $_.value <= 25 });
say %mild_days;
```

The `.grep` method filters hash pairs based on conditions. `*.value > 25`  
filters pairs where values exceed 25. Key filtering uses string comparison  
operators and junctions. Block syntax allows complex conditions using both  
key and value properties in the filter expression.  

## Hash transformation

Converting hash values while preserving the structure.  

```raku
my %prices_usd = (
    'coffee' => 3.50,
    'sandwich' => 8.25,
    'salad' => 6.75,
    'soup' => 4.50
);

# Convert to different currency
my %prices_eur = %prices_usd.map({ $_.key => $_.value * 0.85 });
say %prices_eur;

# Transform values only
my %rounded_prices = %prices_usd.map({ $_.key => $_.value.Int });
say %rounded_prices;

# Transform keys and values
my %uppercase = %prices_usd.map({ $_.key.uc => "\${$_.value}" });
say %uppercase;
```

The `.map` method transforms hash contents by applying functions to pairs.  
Transformations can modify values while keeping keys, change both keys and  
values, or apply mathematical operations. The result maintains the hash  
structure with the transformed key-value relationships.  

## Hash merging

Combining multiple hashes into one.  

```raku
my %defaults = (
    'timeout' => 30,
    'retries' => 3,
    'debug' => False
);

my %user_config = (
    'timeout' => 60,
    'host' => 'localhost'
);

# Merge with user config overriding defaults
my %final_config = %defaults, %user_config;
say %final_config;

# Using the merge operator
my %merged = %defaults (,) %user_config;
say %merged;

# Multiple hash merge
my %extra = ('port' => 8080);
my %complete = %defaults, %user_config, %extra;
say %complete;
```

Hash merging combines key-value pairs from multiple hashes. When the same  
key exists in multiple hashes, the rightmost value takes precedence. The  
comma operator and merge operator `(,)` both combine hashes. Multiple  
hashes can be merged in a single operation by listing them with commas.  

## Hash slicing

Accessing multiple values using key lists.  

```raku
my %student = (
    'name' => 'Alice Johnson',
    'age' => 20,
    'major' => 'Computer Science',
    'gpa' => 3.85,
    'year' => 'Junior',
    'credits' => 78
);

# Basic slicing
my @info = %student<name age major>;
say @info;

# Slicing with variables
my @academic_keys = <major gpa year>;
my @academic_info = %student{@academic_keys};
say @academic_info;

# Slice assignment
%student<gpa credits> = 3.90, 82;
say %student<gpa credits>;
```

Hash slicing extracts multiple values using a list of keys, returning an  
array of corresponding values. Variables containing key lists can be used  
for dynamic slicing. Slice assignment allows updating multiple hash entries  
simultaneously by providing matching lists of keys and values.  

## Hash inversion

Swapping keys and values in hashes.  

```raku
my %country_codes = (
    'USA' => 'United States',
    'UK' => 'United Kingdom',
    'DE' => 'Germany',
    'FR' => 'France',
    'JP' => 'Japan'
);

# Invert the hash
my %code_lookup = %country_codes.invert;
say %code_lookup;

# Manual inversion with map
my %inverted = %country_codes.map({ $_.value => $_.key });
say %inverted;

# Handle duplicate values
my %roles = ('admin' => 'John', 'user' => 'Jane', 'guest' => 'Jane');
my %by_person = %roles.classify(*.value);
say %by_person;
```

The `.invert` method swaps keys and values, making values the new keys.  
Manual inversion using `.map` provides more control over the process.  
When multiple keys have the same value, `.classify` creates a hash where  
values become keys pointing to arrays of original keys.  

## Hash reduction

Aggregating hash values into single results.  

```raku
my %monthly_sales = (
    'January' => 15000,
    'February' => 12000,
    'March' => 18000,
    'April' => 16000,
    'May' => 20000
);

# Sum all values
my $total_sales = %monthly_sales.values.sum;
say "Total sales: \$$total_sales";

# Find maximum value
my $max_sales = %monthly_sales.values.max;
say "Best month sales: \$$max_sales";

# Reduce with custom operation
my $average = %monthly_sales.values.sum / %monthly_sales.elems;
say "Average monthly sales: \${$average.fmt('%.2f')}";

# Find month with maximum sales
my $best_month = %monthly_sales.max(*.value).key;
say "Best performing month: $best_month";
```

Hash reduction operations aggregate values into single results. Common  
reductions include `.sum`, `.max`, `.min` on the values. The `.max` method  
on the hash itself returns the pair with the maximum value, allowing  
access to both the key and value of the extreme element.  

## Hash categorization

Grouping and categorizing data using hashes.  

```raku
my @students = (
    { name => 'Alice', grade => 'A', subject => 'Math' },
    { name => 'Bob', grade => 'B', subject => 'Math' },
    { name => 'Carol', grade => 'A', subject => 'Science' },
    { name => 'David', grade => 'C', subject => 'Math' },
    { name => 'Eve', grade => 'A', subject => 'Science' }
);

# Group by grade
my %by_grade = @students.classify(*<grade>);
say %by_grade;

# Group by subject
my %by_subject = @students.classify(*<subject>);
say %by_subject;

# Count occurrences
my %grade_counts = @students.classify(*<grade>).map({ 
    $_.key => $_.value.elems 
});
say %grade_counts;
```

The `.classify` method groups array elements into hash buckets based on  
a classification function. Elements with the same classification result  
are grouped together in arrays. This creates a hash where keys are the  
classification results and values are arrays of matching elements.  

## Hash validation

Checking hash structure and content validity.  

```raku
my %user_data = (
    'username' => 'alice123',
    'email' => 'alice@example.com',
    'age' => 25,
    'active' => True
);

# Check required keys
my @required_keys = <username email age>;
my $has_required = @required_keys.all ∈ %user_data;
say "Has all required keys: $has_required";

# Validate data types
my $valid_age = %user_data<age> ~~ Int && %user_data<age> > 0;
my $valid_email = %user_data<email> ~~ /\w+ '@' \w+ '.' \w+/;
say "Valid age: $valid_age";
say "Valid email: $valid_email";

# Check for extra keys
my @allowed_keys = <username email age active>;
my @extra_keys = %user_data.keys (-) @allowed_keys;
say "Extra keys: @extra_keys[]";
```

Hash validation checks for required keys using the `.all` quantifier and  
set membership operator `∈`. Data type validation uses smart matching `~~`  
with type objects or patterns. Set difference `(-)` identifies unexpected  
keys by comparing actual keys against allowed keys.  

## Hash serialization

Converting hashes to and from string representations.  

```raku
my %config = (
    'database' => 'postgresql',
    'host' => 'localhost',
    'port' => 5432,
    'ssl' => True
);

# Convert to string representation
my $hash_str = %config.gist;
say $hash_str;

# Convert to Perl representation
my $perl_str = %config.perl;
say $perl_str;

# Create hash from string pairs
my $config_string = "debug=true;port=8080;host=server1";
my %parsed = $config_string.split(';').map({
    my ($key, $value) = .split('=');
    $key => $value
});
say %parsed;
```

The `.gist` method provides a human-readable string representation of the  
hash. The `.perl` method generates valid Raku code that recreates the hash.  
Parsing configuration strings involves splitting on delimiters and mapping  
the results to key-value pairs for hash construction.  

## Hash defaults

Providing default values for missing keys.  

```raku
my %settings = (
    'theme' => 'dark',
    'language' => 'en'
);

# Using defined-or operator
my $font_size = %settings<font_size> // 12;
my $auto_save = %settings<auto_save> // True;
say "Font size: $font_size";
say "Auto save: $auto_save";

# Hash with default values
my %defaults = (
    'theme' => 'light',
    'language' => 'en',
    'font_size' => 14,
    'auto_save' => True
);

# Merge with defaults
my %final_settings = %defaults, %settings;
say %final_settings;

# Using a subroutine for defaults
sub get_setting($key) {
    %settings{$key} // %defaults{$key} // 'unknown'
}

say get_setting('theme');
say get_setting('font_size');
say get_setting('invalid_key');
```

Default values prevent errors when accessing missing keys. The defined-or  
operator `//` provides fallback values for undefined hash entries. Merging  
with a defaults hash ensures all required keys have values. Subroutines  
can implement complex default value logic with multiple fallback levels.  

## Hash composition

Building complex data structures with nested hashes.  

```raku
my %company = (
    'name' => 'Tech Corp',
    'employees' => (
        'engineering' => (
            'alice' => { role => 'senior', salary => 90000 },
            'bob' => { role => 'junior', salary => 60000 }
        ),
        'marketing' => (
            'carol' => { role => 'manager', salary => 75000 },
            'david' => { role => 'analyst', salary => 55000 }
        )
    )
);

# Access nested data
say %company<name>;
say %company<employees><engineering><alice><salary>;

# Iterate over departments
for %company<employees>.kv -> $dept, %staff {
    say "Department: $dept";
    for %staff.kv -> $name, %info {
        say "  $name: {%info<role>} (\${%info<salary>})";
    }
}
```

Nested hashes create hierarchical data structures by using hashes as  
values. Deep access uses chained hash subscripts. Iterating over nested  
structures requires multiple loops, with each level providing its own  
key-value pairs for processing complex organizational data.  

## Hash caching

Using hashes for memoization and caching computed values.  

```raku
my %fibonacci_cache;

sub fibonacci($n) {
    return %fibonacci_cache{$n} if %fibonacci_cache{$n}:exists;
    
    my $result = $n <= 1 ?? $n !! fibonacci($n-1) + fibonacci($n-2);
    %fibonacci_cache{$n} = $result;
    return $result;
}

say fibonacci(10);
say fibonacci(20);
say %fibonacci_cache;

# Simple cache with expiration
my %cache_with_time = ();

sub cached_computation($input) {
    my $now = now;
    my $cached = %cache_with_time{$input};
    
    if $cached && ($now - $cached<timestamp>) < 60 {
        return $cached<value>;
    }
    
    my $result = $input ** 2;  # Expensive computation
    %cache_with_time{$input} = { value => $result, timestamp => $now };
    return $result;
}
```

Hash caching stores computed results to avoid recalculation. The cache  
check uses `:exists` to verify if a value was previously computed. Time-  
based caching includes timestamps to implement expiration policies.  
Memoization dramatically improves performance for recursive algorithms.  

## Hash statistics

Computing statistical information from hash data.  

```raku
my %test_scores = (
    'Alice' => 95, 'Bob' => 87, 'Carol' => 92, 'David' => 78,
    'Eve' => 98, 'Frank' => 85, 'Grace' => 91, 'Henry' => 83
);

# Basic statistics
my @scores = %test_scores.values;
my $mean = @scores.sum / @scores.elems;
my $min = @scores.min;
my $max = @scores.max;

say "Mean score: {$mean.fmt('%.2f')}";
say "Range: $min - $max";

# Count by grade ranges
my %grade_distribution;
for %test_scores.values -> $score {
    my $grade = $score >= 90 ?? 'A' !!
                 $score >= 80 ?? 'B' !!
                 $score >= 70 ?? 'C' !! 'F';
    %grade_distribution{$grade}++;
}

say %grade_distribution;

# Find outliers (scores > 1 std dev from mean)
my $variance = @scores.map({ ($_ - $mean) ** 2 }).sum / @scores.elems;
my $std_dev = $variance.sqrt;
my @outliers = %test_scores.grep(*.value > $mean + $std_dev);
say "High performers: {@outliers.map(*.key).join(', ')}";
```

Statistical analysis extracts insights from hash data using mathematical  
operations on values. Grade distribution uses ranges to categorize scores.  
Outlier detection applies statistical formulas to identify exceptional  
values. Hash methods facilitate data analysis workflows.  

## Hash patterns

Common design patterns and idioms with hashes.  

```raku
# Counter pattern
my %word_count;
my @words = <the quick brown fox jumps over the lazy dog>;
for @words -> $word {
    %word_count{$word}++;
}
say %word_count;

# Index pattern - group by first letter
my %names_by_letter;
my @names = <Alice Bob Carol David Eve Frank>;
for @names -> $name {
    my $first_letter = $name.substr(0, 1);
    %names_by_letter{$first_letter}.push($name);
}
say %names_by_letter;

# Factory pattern - functions in hash
my %operations = (
    'add' => { $^a + $^b },
    'multiply' => { $^a * $^b },
    'power' => { $^a ** $^b }
);

say %operations<add>(5, 3);
say %operations<multiply>(4, 7);
say %operations<power>(2, 8);

# Registry pattern
my %handlers;
%handlers<GET> = { "Handling GET request for $_" };
%handlers<POST> = { "Handling POST request for $_" };

say %handlers<GET>('/users');
say %handlers<POST>('/data');
```

The counter pattern uses auto-vivification to count occurrences. The index  
pattern groups items by a computed key using `.push` on auto-vivified  
arrays. The factory pattern stores functions in hashes for dynamic  
dispatch. The registry pattern maps identifiers to behavior implementations.  

## Hash utilities

Utility functions and advanced operations with hashes.  

```raku
# Deep copying hashes
my %original = (
    'level1' => (
        'level2' => (
            'value' => 42
        )
    )
);

my %shallow_copy = %original;
my %deep_copy = %original.deepmap(*.clone);

%original<level1><level2><value> = 99;
say %shallow_copy<level1><level2><value>;  # 99 (shared reference)
say %deep_copy<level1><level2><value>;     # 42 (independent copy)

# Hash comparison
my %hash1 = (a => 1, b => 2, c => 3);
my %hash2 = (a => 1, b => 2, c => 3);
my %hash3 = (a => 1, b => 2, c => 4);

say %hash1 eqv %hash2;  # True (equivalent)
say %hash1 eqv %hash3;  # False (different values)

# Hash union and intersection
my %set1 = (a => 1, b => 2, c => 3);
my %set2 = (b => 2, c => 3, d => 4);

my %union = %set1 (∪) %set2;
my %intersection = %set1 (∩) %set2;

say %union;
say %intersection;
```

Deep copying preserves nested structure independence using `.deepmap` and  
`.clone`. Hash comparison uses the `eqv` operator for structural equality.  
Set operations like union `(∪)` and intersection `(∩)` work on hashes,  
treating them as sets of key-value pairs for mathematical operations.  

## Hash interpolation

Using hashes in string interpolation and templates.  

```raku
my %user = (
    'name' => 'Alice Smith',
    'email' => 'alice@company.com',
    'department' => 'Engineering',
    'level' => 'Senior'
);

# Direct interpolation
say "Welcome {%user<name>} from {%user<department>}!";

# Template with placeholders
my $template = "Hello {name}, your email {email} is registered.";
my $message = $template;
for %user.kv -> $key, $value {
    $message = $message.subst("\{$key\}", $value, :global);
}
say $message;

# Using sprintf-style formatting
say sprintf "User: %s <%s>, Level: %s", 
    %user<name>, %user<email>, %user<level>;
```

Hash values can be directly interpolated into strings using curly braces  
and hash access syntax. Template processing involves substituting  
placeholders with hash values using pattern matching. The `sprintf`  
function provides formatted output using hash values as arguments.  

## Hash constraints

Enforcing type and value constraints on hash contents.  

```raku
# Hash with typed values
my Int %ages = (
    'Alice' => 25,
    'Bob' => 30,
    'Carol' => 28
);

# Only integers allowed as values
# %ages<David> = 'thirty';  # Would cause type error

say %ages;

# Hash with constrained keys
my %valid_grades{<A B C D F>} = (
    'A' => 95,
    'B' => 85,
    'C' => 75
);

say %valid_grades;
# %valid_grades<X> = 50;  # Would cause constraint error

# Custom validation with where clauses
subset PositiveInt of Int where * > 0;
my PositiveInt %inventory = (
    'apples' => 50,
    'bananas' => 25
);

say %inventory;
```

Type constraints ensure hash values conform to specific types by declaring  
the type before the `%` sigil. Key constraints limit allowed keys using  
angle bracket notation with a list of valid keys. Custom types with  
`where` clauses provide complex validation rules for hash contents.  

## Hash performance

Optimizing hash operations for better performance.  

```raku
# Pre-size hash for known capacity
my %large_hash = Hash.new;
# %large_hash.WHICH;  # Get identity for performance monitoring

# Batch operations are more efficient
my @keys = <a b c d e f g h i j>;
my @values = 1..10;
my %batch_hash = @keys Z=> @values;
say %batch_hash;

# Avoid repeated key lookups
my %cache = ('expensive_key' => 'computed_value');
my $key = 'expensive_key';
if %cache{$key}:exists {
    my $value = %cache{$key};  # Store locally
    say "Using cached: $value";
    say "Processing: $value";
    say "Final result: $value";
}

# Use exists check before access
my %data = ('valid' => 42);
if %data<valid>:exists {
    process-data(%data<valid>);
}

sub process-data($value) {
    say "Processing: $value";
}
```

Hash performance benefits from pre-sizing when capacity is known. The zip  
operator `Z=>` efficiently creates hashes from paired arrays. Storing  
frequently accessed values locally avoids repeated hash lookups. Existence  
checks prevent unnecessary value retrievals and potential errors.  

## Hash debugging

Techniques for debugging and inspecting hash contents.  

```raku
my %complex_hash = (
    'metadata' => (
        'version' => '1.0',
        'created' => '2024-01-01'
    ),
    'data' => (
        'users' => ['alice', 'bob'],
        'settings' => { debug => True, timeout => 30 }
    )
);

# Inspect hash structure
say "Hash type: {%complex_hash.^name}";
say "Hash size: {%complex_hash.elems}";
say "Hash keys: {%complex_hash.keys.sort.join(', ')}";

# Pretty print with gist
say %complex_hash.gist;

# Generate Raku code representation
say %complex_hash.perl;

# Debug specific paths
say "Nested value: {%complex_hash<data><settings><debug>}";

# Walk through all key-value pairs recursively
sub debug-hash(%hash, $indent = 0) {
    my $prefix = '  ' x $indent;
    for %hash.kv -> $key, $value {
        if $value ~~ Hash {
            say "$prefix$key => \{";
            debug-hash($value, $indent + 1);
            say "$prefix\}";
        } else {
            say "$prefix$key => $value";
        }
    }
}

debug-hash(%complex_hash);
```

Hash debugging starts with inspecting basic properties like type, size, and  
keys. The `.gist` and `.perl` methods provide different views of hash  
structure. Recursive traversal functions help explore complex nested  
structures. Custom debug routines can format hash contents for analysis.  
