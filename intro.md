# Introduction to Raku

Raku is a powerful, expressive, and feature-rich programming language that  
builds upon decades of programming language research and design. Originally  
known as Perl 6, Raku represents a complete redesign and reimplementation  
that maintains the spirit of Perl while introducing modern language features,  
improved syntax, and enhanced capabilities. This chapter introduces you to  
Raku's background, core philosophy, installation process, and essential  
tools, providing the foundation you need to start your Raku journey.  

## Origins and influences

Raku's development began in 2000 as the successor to Perl 5, with the goal  
of creating a more powerful and expressive language while maintaining Perl's  
philosophy of "There's More Than One Way To Do It" (TMTOWTDI).  

```raku
# Raku embraces multiple paradigms
say "Hello, World!";                    # Procedural
"Hello, World!".say;                    # Object-oriented  
given "Hello, World!" { .say }          # Functional
```

The language draws inspiration from many sources: Perl's expressiveness,  
Python's readability, Haskell's type system, and Smalltalk's object model.  
Raku was designed by Larry Wall and the Raku community through an open  
RFC process, ensuring the language serves real-world programming needs.  

## Goals and features

Raku aims to be a language that scales from one-liners to large applications,  
supporting multiple programming paradigms while maintaining readability and  
expressiveness.  

```raku
# Gradual typing - start simple, add types as needed
my $name = "Alice";                     # No type annotation
my Str $title = "Developer";            # Explicit type
my Int:D $age = 30;                     # Non-nullable integer
```

Key features include:
- Gradual typing with rich type system
- Powerful pattern matching and destructuring
- Built-in concurrency and parallelism
- Unicode support throughout
- Extensible syntax and operators
- Comprehensive object-oriented programming
- Functional programming constructs

```raku
# Pattern matching with when/given
given 42 {
    when Int     { say "It's an integer" }
    when * > 100 { say "It's large" }
    when * < 0   { say "It's negative" }
    default      { say "It's something else" }
}
```

## Installation

There are several ways to install Raku on your system. The recommended  
approach varies by operating system and use case.  

### Official packages

Many Linux distributions provide Raku packages in their repositories:

```bash
# Ubuntu/Debian
sudo apt install rakudo

# Fedora
sudo dnf install rakudo

# Arch Linux  
sudo pacman -S rakudo
```

### Pre-built binaries

Download official releases from raku.org for Windows, macOS, and Linux.  
These include the Rakudo compiler and essential tools.

### Building from source

For the latest features or custom configurations:

```bash
git clone https://github.com/rakudo/rakudo.git
cd rakudo
perl Configure.pl --gen-moar --gen-nqp --backends=moar
make
make install
```

The source build provides maximum flexibility but requires more time and  
dependencies.

## rakubrew

rakubrew is a version manager for Raku, similar to rbenv for Ruby or nvm  
for Node.js. It allows you to install and switch between multiple Raku  
versions easily.  

### Installing rakubrew

```bash
# Install rakubrew
curl https://rakubrew.org/install-on-perl.sh | sh

# Add to your shell profile
echo 'eval "$(rakubrew init Bash)"' >> ~/.bashrc
source ~/.bashrc
```

### Using rakubrew

```bash
# List available versions
rakubrew available

# Install latest stable version
rakubrew download

# Install specific version
rakubrew download 2023.09

# Switch versions
rakubrew switch 2023.09

# List installed versions
rakubrew list
```

rakubrew manages not only the Raku compiler but also associated tools like  
zef, making it ideal for development environments where you need different  
Raku versions for different projects.

## zef

zef is Raku's primary package manager and module installer, providing access  
to the rich ecosystem of Raku modules available through the Raku ecosystem.  

### Basic zef usage

```bash
# Search for modules
zef search JSON

# Install a module
zef install JSON::Fast

# Install from specific source
zef install https://github.com/user/repo.git

# List installed modules
zef list --installed
```

### Using installed modules

```raku
# After installing with: zef install JSON::Fast
use JSON::Fast;

my %data = name => "Alice", age => 30, city => "Prague";
my $json = to-json(%data);
say $json;

my %parsed = from-json($json);
say "Name: {%parsed<name>}";
```

zef can install modules from multiple sources: the Raku ecosystem,  
Git repositories, local directories, and archive files. It automatically  
handles dependencies and provides detailed installation feedback.

## Basic examples

Let's explore some fundamental Raku concepts through simple examples that  
demonstrate the language's expressiveness and power.

### Hello World variations

```raku
# Simple output
say "Hello, World!";

# Method call syntax
"Hello, World!".say;

# Using interpolation
my $greeting = "Hello";
my $target = "World";
say "$greeting, $target!";
```

The `say` function automatically adds a newline, while method call syntax  
demonstrates Raku's object-oriented nature where everything is an object.

### Variables and sigils

```raku
# Scalar variables (single values)
my $name = "Alice";
my $age = 30;

# Array variables (ordered lists)
my @colors = "red", "green", "blue";
my @numbers = 1..10;

# Hash variables (key-value pairs)
my %person = name => "Bob", age => 25;
my %scores = Alice => 95, Bob => 87;

say $name;
say @colors[1];        # green
say %person<name>;     # Bob
```

Sigils (`$`, `@`, `%`) indicate the type of container, making code more  
readable and helping the compiler optimize performance.

### Control flow

```raku
# Conditional statements
my $score = 85;
if $score >= 90 {
    say "Excellent!";
} elsif $score >= 70 {
    say "Good job!";
} else {
    say "Keep trying!";
}

# Loops with ranges
for 1..5 -> $i {
    say "Iteration $i";
}

# Processing arrays
my @fruits = "apple", "banana", "orange";
for @fruits -> $fruit {
    say "I like $fruit";
}
```

Raku's control structures are similar to other languages but offer additional  
features like the arrow notation (`->`) for loop variables and more  
expressive conditional syntax.

### Subroutines

```raku
# Simple subroutine
sub greet($name) {
    return "Hello, $name!";
}

# With type annotations
sub add(Int $a, Int $b --> Int) {
    $a + $b;
}

# Multiple dispatch
multi sub factorial(0) { 1 }
multi sub factorial(Int $n where $n > 0) {
    $n * factorial($n - 1);
}

say greet("Alice");
say add(5, 3);
say factorial(5);
```

Raku subroutines support optional type annotations, multiple dispatch based  
on parameter types, and automatic return of the last expression value.

### Basic operators

```raku
# Arithmetic
say 10 + 5;     # 15
say 10 - 3;     # 7  
say 4 * 7;      # 28
say 15 / 3;     # 5

# String operations
say "Hello" ~ " " ~ "World";    # Concatenation
say "Raku" x 3;                 # Repetition: RakuRakuRaku

# Comparison
say 5 == 5;     # True
say 5 != 3;     # True
say "abc" eq "abc";  # True (string equality)

# Smart matching
say 42 ~~ Int;      # True
say "hello" ~~ /l/; # 「l」 (regex match)
```

The smart match operator (`~~`) is particularly powerful, enabling pattern  
matching, type checking, and many other comparison operations with a  
single, consistent syntax.

This introduction provides the foundation for exploring Raku's rich feature  
set. The following chapters dive deeper into specific aspects of the  
language, from basic data types to advanced concepts like concurrency  
and metaprogramming.