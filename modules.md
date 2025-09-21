# Modules

Modules in Raku provide a way to organize code, create reusable components,  
and manage namespaces. They enable encapsulation, code sharing, and help  
structure larger applications. Raku modules can export functions, classes,  
operators, and other symbols, making them available to code that uses the  
module. This chapter covers everything from basic module creation to  
advanced concepts like roles, versioning, and third-party module usage.  

## Basic module creation

Creating a simple module with exported functions.  

```raku
# file: lib/MyUtils.rakumod
unit module MyUtils;

sub greet($name) is export {
    "Hello, $name!";
}

sub calculate-area($radius) is export {
    π * $radius²;
}

# Private function - not exported
sub internal-helper() {
    "This is internal";
}
```

The `unit module` declaration creates a module. Functions marked with  
`is export` become available to code that uses the module. Private  
functions without `is export` remain internal to the module.  

## Using modules

Importing and using functions from a module.  

```raku
use lib 'lib';
use MyUtils;

say greet('Alice');
say calculate-area(5);

# Alternative import syntax
use MyUtils :greet;
say greet('Bob');
```

The `use` statement imports a module. By default, all exported symbols  
are imported. You can selectively import specific functions using the  
`:function-name` syntax after the module name.  

## Module namespaces

Working with module namespaces and qualified names.  

```raku
use MyUtils;

# Fully qualified name
say MyUtils::greet('Charlie');

# Import into specific namespace
use MyUtils :ALL;
say calculate-area(3);

# Avoid name conflicts
use MyUtils :greet(&my-greet);
say my-greet('Dave');
```

Module functions can be called using fully qualified names with the  
`::` namespace operator. The `:ALL` tag imports all exportable symbols.  
You can rename imported functions to avoid naming conflicts.  

## Export control

Controlling what gets exported from a module.  

```raku
# file: lib/Calculator.rakumod
unit module Calculator;

# Export by default
sub add($a, $b) is export {
    $a + $b;
}

# Export with specific tag
sub advanced-math($x) is export(:advanced) {
    $x ** 3 + log($x);
}

# Multiple export tags
sub debug-info() is export(:debug, :util) {
    "Debug mode active";
}

# Export constant
constant PI is export = 3.14159;
```

Export tags allow grouping related functions. Use `is export(:tag-name)`  
to create custom export groups. Constants can also be exported using  
`is export`. This provides fine-grained control over module interfaces.  

## Import with tags

Selectively importing functions using export tags.  

```raku
use Calculator;                    # imports default exports
use Calculator :advanced;          # imports advanced functions
use Calculator :debug, :util;      # imports multiple tags
use Calculator :ALL;               # imports everything

say add(5, 3);
say advanced-math(2.5);
say debug-info();
say PI;
```

Import tags let you selectively choose which parts of a module to use.  
This keeps your namespace clean and makes dependencies explicit. The  
`:ALL` tag imports everything the module can export.  

## Classes in modules

Defining and exporting classes from modules.  

```raku
# file: lib/Shapes.rakumod
unit module Shapes;

class Circle is export {
    has $.radius;
    
    method area() {
        π * $.radius²;
    }
    
    method circumference() {
        2 * π * $.radius;
    }
}

class Rectangle is export {
    has $.width;
    has $.height;
    
    method area() {
        $.width * $.height;
    }
}
```

Classes are exported using `is export` just like functions. This allows  
modules to provide object-oriented interfaces. Classes can have  
attributes and methods that form cohesive data structures.  

## Using module classes

Creating instances of classes from imported modules.  

```raku
use Shapes;

my $circle = Circle.new(radius => 5);
say "Circle area: {$circle.area()}";
say "Circumference: {$circle.circumference()}";

my $rect = Rectangle.new(width => 4, height => 6);
say "Rectangle area: {$rect.area()}";

# Using fully qualified names
my $big-circle = Shapes::Circle.new(radius => 10);
say "Big circle area: {$big-circle.area()}";
```

Imported classes can be instantiated using `.new()` constructor. Access  
methods and attributes as usual. Fully qualified class names can be  
used to avoid naming conflicts with local classes.  

## Roles in modules

Defining and using roles from modules.  

```raku
# file: lib/Traits.rakumod
unit module Traits;

role Drawable is export {
    method draw() {
        "Drawing {self.^name}";
    }
}

role Colorable is export {
    has $.color = 'black';
    
    method set-color($new-color) {
        $!color = $new-color;
    }
}

class ColoredShape is export does Drawable does Colorable {
    has $.name;
}
```

Roles define reusable behavior that can be composed into classes. Export  
roles using `is export` to make them available to other modules. Classes  
can consume multiple roles using the `does` keyword.  

## Nested modules

Creating hierarchical module structures.  

```raku
# file: lib/Math/Basic.rakumod
unit module Math::Basic;

sub add($a, $b) is export {
    $a + $b;
}

# file: lib/Math/Advanced.rakumod  
unit module Math::Advanced;

sub factorial($n) is export {
    $n <= 1 ?? 1 !! $n * factorial($n - 1);
}

# Usage
use Math::Basic;
use Math::Advanced;

say add(3, 4);
say factorial(5);
```

Modules can be organized hierarchically using `::` separators in names.  
This creates logical grouping of related functionality. Directory  
structure should match the module hierarchy for automatic discovery.  

## Module versioning

Adding version information to modules.  

```raku
# file: lib/MyAPI.rakumod
unit module MyAPI:ver<1.2.3>:auth<github:username>;

sub api-version() is export {
    $?MODULE.^ver;
}

sub api-author() is export {
    $?MODULE.^auth;
}

# Using specific version
use MyAPI:ver<1.2.3>;
say api-version();
say api-author();
```

Modules can specify version and authority information. The `:ver<>` and  
`:auth<>` adverbs provide metadata. Users can request specific versions  
when importing modules. `$?MODULE` provides module introspection.  

## Built-in modules

Using Raku's built-in modules and libraries.  

```raku
use JSON::Fast;
use DateTime;
use Test;

# JSON processing
my %data = name => 'Alice', age => 30;
my $json = to-json(%data);
say $json;
my %parsed = from-json($json);
say %parsed<name>;

# Date/time handling
my $now = DateTime.now;
say "Current time: $now";

# Testing
plan 2;
ok 1 == 1, "Basic equality";
is 2 + 2, 4, "Addition works";
```

Raku comes with many built-in modules for common tasks. `JSON::Fast`  
handles JSON data, `DateTime` works with dates and times, and `Test`  
provides testing infrastructure. These modules follow the same import  
patterns as custom modules.  

## Pragma modules

Using compiler pragmas and directives.  

```raku
use v6.d;          # Specify language version
use strict;        # Enable strict mode  
use fatal;         # Make warnings fatal
use lib 'lib';     # Add library path

# Experimental features
use experimental :pack;
use experimental :cached;

sub slow-calculation($n) is cached {
    sleep 1;
    $n * 42;
}

say slow-calculation(5);  # Takes 1 second
say slow-calculation(5);  # Instant (cached)
```

Pragma modules control compiler behavior and enable experimental  
features. `use v6.d` sets the language version. `use lib` adds paths  
for module discovery. Experimental pragmas enable preview features.  

## Module introspection

Examining module properties at runtime.  

```raku
use MyUtils;

# Get module information
say $?MODULE.^name;           # Current module name
say MyUtils.^name;            # Module type name
say MyUtils.^methods;         # Available methods

# Check if function exists
if MyUtils.^can('greet') {
    say "Module has greet function";
}

# List exported symbols
say MyUtils.WHO.keys;         # Symbol table keys
```

Module introspection lets you examine module properties at runtime.  
The `.^` meta-operator accesses meta-methods. `.WHO` provides access  
to the module's symbol table for dynamic symbol lookup.  

## Module aliases

Creating shorter names for modules.  

```raku
use JSON::Fast:from<Perl5> :short;
use Math::Advanced :short(MA);

# Instead of JSON::Fast::to-json
say to-json({key => 'value'});

# Instead of Math::Advanced::factorial
say MA::factorial(5);

# Module renaming
use JSON::Fast :DEFAULT;
use JSON::Fast :from<Perl5> :as('JSON5');

my $data1 = to-json({a => 1});      # JSON::Fast
my $data2 = JSON5::to-json({b => 2}); # Perl5 version
```

Module aliases create shorter or alternative names for imported modules.  
The `:short` adverb creates abbreviated names. `:as('name')` provides  
custom renaming. This helps manage long module names and versions.  

## Error handling in modules

Implementing proper error handling within modules.  

```raku
# file: lib/SafeCalculator.rakumod
unit module SafeCalculator;

class CalculationError is Exception is export {
    has $.message;
    method message() { $.message }
}

sub safe-divide($a, $b) is export {
    die CalculationError.new(message => "Division by zero")
        if $b == 0;
    
    $a / $b;
}

sub safe-sqrt($n) is export {
    die CalculationError.new(message => "Negative square root")
        if $n < 0;
        
    sqrt($n);
}

# Usage with error handling
use SafeCalculator;

try {
    say safe-divide(10, 0);
    CATCH {
        when CalculationError { say "Error: {.message}" }
    }
}
```

Modules should define custom exception classes for specific error  
conditions. Export exception classes so users can catch them  
specifically. Use `die` with custom exceptions for clear error  
reporting. Provide meaningful error messages for debugging.  

## Module documentation

Adding documentation to modules using Pod6.  

```raku
# file: lib/DocumentedModule.rakumod
unit module DocumentedModule;

=begin pod

=head1 DocumentedModule

This module provides utility functions for common tasks.

=head2 Functions

=item greet($name) - Returns a greeting message
=item add($a, $b) - Adds two numbers

=end pod

#| Greets a person by name
#| Returns a greeting string  
sub greet($name) is export {
    "Hello, $name!";
}

#| Adds two numbers together
#| Returns the sum as a number
sub add($a, $b) is export {
    $a + $b;
}
```

Pod6 documentation can be embedded directly in modules. Use `=begin pod`  
and `=end pod` for block documentation. `#|` comments provide function  
documentation. This creates self-documenting modules with inline help.  

## Testing modules

Creating and running tests for modules.  

```raku
# file: t/myutils.t
use Test;
use lib 'lib';
use MyUtils;

plan 4;

# Test exported functions
ok greet('Test'), 'greet function works';
is greet('Alice'), 'Hello, Alice!', 'greet returns correct message';

# Test calculations
is calculate-area(1), π, 'area calculation for unit circle';
ok calculate-area(5) > 75, 'area calculation reasonable for radius 5';

done-testing;
```

Module tests should be placed in a `t/` directory. Use the `Test` module  
for test infrastructure. Import your module and test all exported  
functions. Use `plan` to specify expected test count and `done-testing`  
to finalize the test run.  

## Third-party modules

Installing and using modules from the ecosystem.  

```raku
# Install with zef (Raku package manager)
# $ zef install JSON::Fast
# $ zef install DateTime::Parse  
# $ zef install HTTP::UserAgent

use JSON::Fast;
use HTTP::UserAgent;

# Fetch and parse JSON from web API
my $ua = HTTP::UserAgent.new;
my $response = $ua.get('https://api.github.com/users/raku');

if $response.is-success {
    my %data = from-json($response.content);
    say "User: {%data<login>}";
    say "Repos: {%data<public_repos>}";
} else {
    say "Failed to fetch data: {$response.status-line}";
}
```

Third-party modules are installed using `zef`, Raku's package manager.  
Use them just like built-in modules with `use` statements. The Raku  
ecosystem provides modules for web development, data processing,  
networking, and many other domains.  

## Module best practices

Guidelines for creating well-structured modules.  

```raku
# file: lib/BestPractices.rakumod
use v6.d;
unit module BestPractices:ver<1.0.0>:auth<github:example>;

=begin pod
=head1 BestPractices

Example module demonstrating best practices.
=end pod

# Group related exports
sub public-function() is export(:main) { "Public" }
sub utility-function() is export(:util) { "Utility" }

# Use proper typing
sub typed-function(Int $number, Str $text --> Str) is export {
    "$text: $number";
}

# Provide sane defaults
sub configurable-function($data, :$format = 'json') is export {
    given $format {
        when 'json' { to-json($data) }
        when 'text' { ~$data }
        default { die "Unknown format: $format" }
    }
}

# Constants for configuration
constant DEFAULT-TIMEOUT is export = 30;
```

Best practices include: versioning modules, providing documentation,  
using export tags for organization, adding type annotations, providing  
sensible defaults, and exporting useful constants. Follow consistent  
naming conventions and include comprehensive tests.  

## Dynamic module loading

Loading modules at runtime based on conditions.  

```raku
# Dynamic module selection
my $format = %*ENV<OUTPUT_FORMAT> // 'json';

my $formatter = do given $format {
    when 'json' { 
        require JSON::Fast;
        JSON::Fast
    }
    when 'yaml' { 
        require YAMLish;
        YAMLish
    }
    default { die "Unknown format: $format" }
};

# Use loaded module
my %data = name => 'Alice', scores => [95, 87, 92];
say $formatter.to-yaml(%data) if $format eq 'yaml';
say to-json(%data) if $format eq 'json';

# Conditional module loading
try {
    require Term::ANSIColor;
    say Term::ANSIColor::colored('Success!', 'green');
    CATCH { say 'Color output not available' }
}
```

The `require` statement loads modules at runtime rather than compile  
time. This enables conditional loading based on environment variables,  
configuration, or feature availability. Use `try`/`CATCH` blocks to  
handle missing optional dependencies gracefully.  