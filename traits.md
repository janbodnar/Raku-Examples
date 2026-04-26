# Traits in Raku  

Traits in Raku are compile-time modifiers that attach metadata, alter semantics,  
or inject behavior directly into declarations. Unlike passive annotations or  
attributes in many other languages, Raku traits are deeply integrated into the  
compiler grammar and apply uniformly across variables, attributes, parameters,  
subroutines, methods, roles, and classes. They are invoked using three distinct  
keywords: `is` modifies how a declaration is interpreted or constrained, `does`  
composes roles (Raku’s equivalent to PHP/Rust-style traits), and `will` attaches  
blocks or phasers that execute at declaration time or later.  

Because traits are resolved during compilation, they impose zero runtime  
overhead. The compiler executes trait handlers—special `trait_mod` routines that  
can inspect, validate, or transform declarations before the program runs. This  
compile-time execution enables both stronger safety (catching configuration or  
type errors early) and greater expressiveness (allowing declarative,  
domain-specific syntax without macros or runtime reflection).  

Raku ships with a comprehensive set of built-in traits that cover everyday  
needs: `is rw` for mutability, `is export` for symbol visibility, `is copy` and  
`is raw` for parameter semantics, `is DEPRECATED` for forward compatibility, `is native`  
for foreign function interfaces, and `is required` for attribute  
validation. Beyond these, the system is fully extensible. By writing your own  
`multi trait_mod:<is>` (or `<does>`, `<will>`) routines, you can enforce  
architectural rules, auto-generate boilerplate, validate domain constraints, or  
introduce new language-like constructs.  

*(Note on naming: Raku’s “traits” are conceptually distinct from the `trait`  
keyword in PHP or Rust, which refer to reusable method bundles or behavioral  
contracts. In Raku, that role is filled by `role`s; traits are strictly  
compile-time declaration modifiers.)*  

This tutorial walks through the most important built-in traits, demonstrates how  
they interact with Raku’s type and scope system, and provides step-by-step  
guidance for authoring custom traits. By the end, you’ll understand how to  
leverage traits to write cleaner, more declarative code and safely extend the  
language to match your domain’s needs.  

  

## Read-write attribute

Making a class attribute publicly writable with `is rw`.  

```raku
class Point {
    has $.x is rw;
    has $.y is rw;
}

my $p = Point.new(x => 1, y => 2);
say "$p.x() $p.y()";

$p.x = 10;
$p.y = 20;
say "$p.x() $p.y()";
```

By default a public attribute declared with `$.` is read-only from  
outside the class. Adding `is rw` generates a read-write accessor so  
the attribute can be assigned to directly after construction. Private  
attributes (`$!`) are always read-write inside the class regardless  
of this trait.  

## Read-only parameter

Preventing accidental mutation of subroutine parameters.  

```raku
sub double(Int $n is copy) {
    $n *= 2;
    $n;
}

sub show(Str $s) {
    say $s;
}

say double(5);
show("hello");
```

Subroutine parameters are read-only by default; attempting to assign  
to `$n` without `is copy` would be a compile-time error. The `is copy`  
trait gives the parameter its own writable copy of the argument so  
the original variable is never modified. Use `is rw` instead when you  
intentionally want the subroutine to modify the caller's variable.  

## Writable parameter

Allowing a subroutine to modify the caller's variable in place.  

```raku
sub increment(Int $n is rw) {
    $n++;
}

my $count = 41;
increment($count);
say $count;
```

The `is rw` trait on a parameter creates an alias to the argument  
rather than a copy. Any assignment inside the subroutine is reflected  
in the original variable. This is Raku's explicit, opt-in equivalent  
of pass-by-reference; the caller can see that the subroutine may  
mutate its argument.  

## Default value

Providing a fallback value with `is default`.  

```raku
my Int $retries is default(3);

say $retries;    # 3

$retries = 10;
say $retries;    # 10

$retries = Int;  # reset to type object
say $retries;    # 3 (default restored)
```

The `is default` trait sets the value a variable holds when it contains  
the type object (undefined) of its type. Assigning the type object  
itself — `Int` for an `Int` variable — resets it back to the default.  
This is useful for configuration variables that should revert to a  
known value when explicitly cleared.  

## Required attribute

Enforcing that a constructor argument is always supplied.  

```raku
class Config {
    has Str $.host is required;
    has Int $.port is required;
    has Str $.scheme = 'https';
}

my $cfg = Config.new(host => 'example.com', port => 443);
say "$cfg.scheme()://$cfg.host():$cfg.port()";
```

The `is required` trait causes object construction to throw an error  
at runtime if the attribute is not provided as a named argument to  
`.new`. Optional attributes can still use `=` to supply a default.  
Combining both traits on the same attribute is not allowed.  

## Lazy attribute

Deferring expensive initialisation until first access.  

```raku
class Report {
    has $.title;
    has $.body is built(False);
    has $.rendered is lazy {
        "<h1>$.title</h1><p>$.body</p>"
    };
}

my $r = Report.new(title => 'Hello', body => 'World');
say $r.rendered;
```

The `is lazy` trait delays evaluation of the initialiser block until  
the attribute is first read. The block runs exactly once; subsequent  
accesses return the cached result. This avoids paying the cost of  
initialisation when the attribute might never be used. The `is built`  
trait with `False` prevents the attribute from being set via `.new`.  

## Deprecated subroutine

Marking a subroutine as deprecated with `is DEPRECATED`.  

```raku
sub old-greet($name) is DEPRECATED('greet()') {
    say "Hello, $name!";
}

sub greet($name) {
    say "Hi, $name!";
}

old-greet('Alice');
greet('Bob');
```

The `is DEPRECATED` trait causes Raku to emit a deprecation warning  
when the annotated subroutine (or method) is called. An optional  
string argument names the preferred replacement so callers know what  
to switch to. Warnings are printed to STDERR and include the call site,  
making it easy to locate all uses that need to be updated.  

## Pure subroutine

Declaring that a function has no side effects with `is pure`.  

```raku
sub square(Numeric $n --> Numeric) is pure {
    $n ** 2;
}

say square(4);
say square(1.5);

my @squares = (1..5).map(&square);
say @squares;
```

The `is pure` trait declares that the subroutine always returns the  
same result for the same arguments and has no observable side effects.  
The compiler may use this information to constant-fold calls with  
literal arguments at compile time. Many built-in operators carry  
`is pure` internally, enabling the optimisations you rely on every day.  

## Cached subroutine

Memoising expensive computations with `is cached`.  

```raku
sub fib(Int $n --> Int) is cached {
    return $n if $n <= 1;
    fib($n - 1) + fib($n - 2);
}

say fib(10);
say fib(30);
```

The `is cached` trait automatically memoises the subroutine: the first  
call with a given argument list stores the result, and subsequent calls  
with the same arguments return the stored result without re-executing  
the body. This turns the naïve exponential Fibonacci into an efficient  
linear-time computation simply by adding one trait.  

## Exported subroutine

Making subroutines available to importers with `is export`.  

```raku
# Imagine this is in a module file MyMath.rakumod:
# unit module MyMath;
#
# sub add(Int $a, Int $b --> Int) is export {
#     $a + $b;
# }
#
# sub secret-helper { ... }   # not exported

# Using the module:
# use MyMath;
# say add(3, 4);

sub add(Int $a, Int $b --> Int) is export {
    $a + $b;
}

say add(2, 3);
```

The `is export` trait registers a subroutine in the module's export  
list. When another file writes `use MyMath`, only symbols marked  
`is export` (or collected in `@EXPORT`) are imported into the  
caller's namespace. You can also pass a tag, such as  
`is export(:math)`, so importers can select subsets with  
`use MyMath :math`.  

## Hidden from backtrace

Suppressing internal frames in error output.  

```raku
sub validate(Int $n) is hidden-from-backtrace {
    die "Must be positive" if $n <= 0;
}

sub process($n) {
    validate($n);
}

try {
    process(-1);
    CATCH {
        default {
            say .message;
        }
    }
}
```

The `is hidden-from-backtrace` trait removes the annotated frame from  
the exception backtrace printed when a program dies. This is ideal for  
thin wrapper or validation helpers where including the frame would only  
add noise. The underlying exception and its message are unaffected;  
only the display is filtered.  

## Trait on a class

Applying traits directly to a class declaration.  

```raku
class Singleton is repr('CStruct') { }

class Config is rw {
    has $.debug;
    has $.verbose;
}

my $cfg = Config.new(debug => True, verbose => False);
$cfg.debug = False;
say $cfg.debug;
```

Traits can be placed on the class keyword itself. `is rw` applied to  
a class makes all its public attributes read-write by default, removing  
the need to annotate each one individually. `is repr` controls the  
in-memory representation used by the VM, which matters when interfacing  
with native code via `NativeCall`.  

## Native representation

Declaring a C-compatible struct for foreign-function calls.  

```raku
use NativeCall;

class Point is repr('CStruct') {
    has int32 $.x;
    has int32 $.y;
}

my $pt = Point.new(x => 10, y => 20);
say "x=$pt.x(), y=$pt.y()";
```

The `is repr('CStruct')` trait instructs Raku to lay out the class in  
memory exactly as a C compiler would for a struct with the same fields.  
Native integer types like `int32` and `num64` must be used instead of  
Raku's high-level types. The resulting object can be passed directly to  
C functions via `NativeCall` without marshalling.  

## Role composition via does

Mixing in a role as a trait on a class.  

```raku
role Printable {
    method print-info() {
        say self.raku;
    }
}

role Serializable {
    method to-string() {
        self.raku;
    }
}

class Item does Printable does Serializable {
    has $.name;
    has $.value;
}

my $item = Item.new(name => 'sword', value => 150);
$item.print-info;
say $item.to-string;
```

`does` is itself a trait that composes a role into a class at compile  
time. Multiple roles can be composed in a single class declaration.  
Role composition is checked at compile time: if two composed roles  
both provide a method with the same name, the compiler raises an error  
unless the class provides its own overriding implementation.  

## Custom trait via trait_mod

Defining your own `is` trait using `trait_mod:<is>`.  

```raku
multi trait_mod:<is>(Attribute $attr, :$logged!) {
    $attr.set_build(-> $, | {
        note "Attribute $attr.name() initialised";
    });
}

class Server {
    has $.host is logged;
    has $.port is logged;
}

my $s = Server.new(host => 'localhost', port => 8080);
say $s.host;
```

A custom trait is a multi-dispatch subroutine named `trait_mod:<is>`  
(or `trait_mod:<does>`). The first parameter is the meta-object being  
decorated (`Attribute`, `Routine`, `Package`, etc.) and the named  
parameter becomes the trait name. Traits run at compile time, giving  
you access to the full meta-object protocol to inspect and modify the  
thing being decorated before the program starts executing.  

## Trait on a variable

Using `is` traits on lexical variables.  

```raku
my $counter is default(0);
my @items   is List;

$counter = 5;
say $counter;

$counter = Any;     # reset
say $counter;

my $pi is default(3.14159);
$pi = Num;          # reset to default
say $pi;
```

Traits are not limited to declarations inside classes; any `my`  
variable can carry a trait. `is default` on a scalar gives the  
undefined/reset state a meaningful value. The `is List` trait changes  
the variable's container type so it behaves as an immutable list rather  
than a mutable array.  

## Will trait

Running code at specific compile-time phases with `will`.  

```raku
my $cache will start { Hash.new };

sub heavy-computation(Int $n --> Int) {
    $cache{$n} //= do {
        note "Computing for $n…";
        $n ** 3;
    };
}

say heavy-computation(4);
say heavy-computation(4);
say heavy-computation(7);
```

The `will` trait family runs a block at a designated phase. `will start`  
runs the block once the first time the variable is accessed, creating  
the value lazily. Other phases include `will first` (before the first  
iteration of a loop variable) and `will last` (after the final  
iteration). These are idiomatic alternatives to manual initialisation  
guards.  

## Combining multiple traits

Stacking several traits on the same declaration.  

```raku
class Temperature {
    has Numeric $.celsius is rw is default(0);
    has Str     $.unit    is rw is default('C');

    method to-fahrenheit(--> Numeric) is pure is cached {
        $.celsius * 9/5 + 32;
    }
}

my $t = Temperature.new(celsius => 100);
say $t.to-fahrenheit;
say $t.to-fahrenheit;   # served from cache

$t.celsius = Numeric;   # reset to default
say $t.celsius;
```

Multiple traits can be chained on a single declaration — they are  
applied left to right at compile time. Here an attribute gets both  
`is rw` and `is default`, while the method combines `is pure` and  
`is cached`. Stacking traits is idiomatic Raku and incurs no runtime  
overhead beyond what each individual trait contributes.  

## Trait introspection

Inspecting traits attached to attributes and routines at runtime.  

```raku
class Product {
    has Str $.name  is rw;
    has Num $.price is rw is default(0.0);
}

my $meta = Product.^attributes;
for $meta -> $attr {
    say "Attribute : {$attr.name}";
    say "  type    : {$attr.type}";
    say "  rw      : {$attr.rw}";
    say "  required: {$attr.required}";
}
```

The meta-object protocol (`^attributes`, `^methods`, `^roles`) gives  
access to compile-time information about any class. Attribute  
meta-objects expose predicates like `.rw`, `.required`, and `.built`  
that correspond directly to the traits used in the source code. This  
makes it straightforward to build generic serialisers, validators, or  
documentation generators that react to the traits a class author  
declared.  

## Trait on a role attribute

Traits work inside role declarations too.  

```raku
role Timestamped {
    has Str $.created-at  is required;
    has Str $.modified-at is rw is default('');

    method touch() {
        $!modified-at = DateTime.now.Str;
    }
}

class Article does Timestamped {
    has Str $.title is required;
}

my $a = Article.new(
    title      => 'Raku Traits',
    created-at => DateTime.now.Str,
);

say $a.created-at;
$a.touch;
say $a.modified-at;
```

Roles can carry the full range of attribute traits just like classes.  
When the role is composed into a class the traits travel with the  
attributes, so the consuming class automatically honours `is required`,  
`is rw`, and `is default` without any extra code. This makes roles  
powerful reusable building blocks that enforce their own invariants.  
