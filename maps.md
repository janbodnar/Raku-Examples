# Maps

Maps are immutable key-value stores in Raku. Unlike hashes (`%`), a `Map`  
is frozen after construction — you cannot add, remove, or change its entries.  
This makes `Map` ideal for configuration tables, dispatch tables, and  
read-only reference data. Maps share most of the hash API, so anything that  
works on a hash works on a `Map` unless it tries to mutate the container.  


## Creating a Map

Creating a `Map` from a flat list of alternating key–value pairs.  

```raku
my %h = Map.new(1, 'sky', 2, 'cloud',
    3, 'cup', 4, 'war', 5, 'water');

say %h{1};
say %h{2};
say %h.elems;
say %h.keys;
say %h.values;
say %h.gist;
```

`Map.new` accepts a flat list where every odd element is a key and every  
even element is its value. The result is stored in a `%` variable but  
remains immutable; any attempt to assign to a key will throw an error.  

## Map from pairs

Building a `Map` from explicit `Pair` objects.  

```raku
my $m = Map.new(
    'one' => 1,
    'two' => 2,
    'three' => 3,
);

say $m<one>;
say $m<two three>;
say $m.WHAT;
```

Fat-arrow (`=>`) syntax creates `Pair` objects that `Map.new` accepts directly.  
Accessing multiple keys at once with angle-bracket notation returns a `List`  
of the corresponding values.  

## `is Map` trait

Declaring a hash variable as an immutable `Map` using the `is Map` trait.  

```raku
my %h is Map = 'a', 10, 'b', 20, 'c', 30;

say %h<a>;
say %h<b>;
say %h.WHAT;
```

The `is Map` trait tells Raku to use the `Map` type as the backing store.  
The variable still uses the `%` sigil, so the familiar `<key>` and  
`{key}` subscripts both work.  

## Map from a hash

Converting an existing mutable hash into an immutable `Map`.  

```raku
my %source = alpha => 1, beta => 2, gamma => 3;
my $frozen = %source.Map;

say $frozen<alpha>;
say $frozen.WHAT;
```

Calling `.Map` on any hash returns a new `Map` snapshot of the current  
state. Later changes to `%source` will not affect `$frozen`.  

## Accessing values

Multiple ways to look up values inside a `Map`.  

```raku
my %m is Map = name => 'Alice', age => 30, city => 'Prague';

say %m<name>;
say %m{'age'};
say %m.AT-KEY('city');
say %m<name age city>;
```

Angle brackets (`<key>`) work for identifier-like keys; curly braces  
(`{'key'}`) accept any expression. `.AT-KEY` is the low-level accessor.  
Supplying several keys at once returns a list of values.  

## `.kv`, `.pairs`, `.keys`, `.values`

Retrieving all keys, values, or pairs from a `Map`.  

```raku
my %m is Map = a => 1, b => 2, c => 3;

say %m.keys;
say %m.values;
say %m.kv;
say %m.pairs;
```

`.keys` and `.values` return flat lists; `.kv` interleaves them as  
`(k1 v1 k2 v2 ...)`, and `.pairs` returns a list of `Pair` objects  
suitable for further manipulation.  

## Iteration patterns

Iterating over all entries in a `Map`.  

```raku
my %m is Map = name => 'Bob', lang => 'Raku', year => 2024;

for %m.kv -> $k, $v {
    say "$k => $v";
}

for %m.pairs -> $p {
    say "$p.key(): $p.value()";
}
```

The `for` loop with `.kv` binds two variables per iteration. Using  
`.pairs` gives access to the `Pair` object and its `.key` / `.value`  
methods, which can be cleaner in complex loops.  

## Sorting

Sorting a `Map` by key or by value.  

```raku
my %h is Map = 1, 'sky', 2, 'cloud',
    3, 'cup', 4, 'war', 5, 'water', 6, 'atom';

say %h.sort;
say %h.sort({ $^b cmp $^a });

say %h.sort(*.value);
say %h.sort(*.value).reverse;
```

`.sort` on a `Map` returns a sorted list of `Pair` objects. Pass a  
comparator block or a key-extractor (e.g. `*.value`) to sort by an  
arbitrary criterion; chain `.reverse` for descending order.  

## Filtering with grep

Selecting entries whose value satisfies a condition.  

```raku
my %h is Map = 1, 'sky', 2, 'cloud',
    3, 'cup', 4, 'war', 5, 'water', 6, 'atom';

say %h.grep({ .value.starts-with('w') });
say %h.grep({ .value.chars > 3 });
```

`.grep` on a `Map` iterates over `Pair` objects; the block receives each  
pair as `$_`. Use `.key` and `.value` (or the shorthand `.`) to filter by  
any property.  

## Existence check

Testing whether a key exists without retrieving its value.  

```raku
my %m is Map = a => 1, b => 2, c => 3;

say %m<a>:exists;
say %m<z>:exists;

if %m<b>:exists {
    say "b is present";
}
```

The `:exists` adverb returns a `Bool` without triggering autovivification.  
It is the canonical way to distinguish "key missing" from "key with value  
`Nil`" or `0`.  

## Defined-or default

Providing a fallback when a key is absent.  

```raku
my %m is Map = host => 'localhost', port => 5432;

say %m<host> // 'unknown';
say %m<user> // 'root';
say %m<port> // 3306;
```

The `//` operator returns the right-hand side when the left evaluates to  
`Nil` or an undefined value. Because missing keys return `Nil`, this  
idiom is the standard way to supply defaults.  

## Merging maps

Combining two maps into a new, larger map.  

```raku
my %a is Map = x => 1, y => 2;
my %b is Map = y => 99, z => 3;

my %merged = Map.new(|%a, |%b);
say %merged;

my %merged-a-wins = Map.new(|%b, |%a);
say %merged-a-wins;
```

Because `Map` is immutable, merging creates a new `Map`. The slip  
operator `|` flattens each map into the argument list; keys appearing in  
both are resolved by the last one to appear, so argument order decides  
which map wins for overlapping keys.  

## Updating by creating a new Map

Deriving a new `Map` with one entry changed.  

```raku
my %cfg is Map = debug => False, level => 1, name => 'app';

my %updated = Map.new(|%cfg, level => 5);
say %updated;
```

The canonical way to "update" an immutable `Map` is to spread the  
original with `|` and override specific keys. Any key listed after the  
spread takes precedence.  

## Typed maps

Constraining the value type of a `Map`.  

```raku
my %scores is Map of Int = alice => 91, bob => 85, carol => 78;

say %scores<alice>;
say %scores.WHAT;
```

The `of Int` constraint causes a `X::TypeCheck` exception if any value  
does not coerce to `Int`. Combined with `is Map`, this gives a  
type-safe, immutable look-up table.  

## Maps with integer keys

Using non-string keys such as integers.  

```raku
my %m = Map.new(1, 'one', 2, 'two', 3, 'three');

say %m{1};
say %m{2};
say %m.keys.sort;
```

Raku maps store keys as their string representations internally, but  
`Map.new` with integer keys lets you subscript with integer expressions;  
the runtime coerces `1` to `"1"` transparently.  

## Maps with Callable values

Storing code references as values to create flexible tables.  

```raku
my %ops is Map =
    add  => -> $a, $b { $a + $b },
    sub  => -> $a, $b { $a - $b },
    mul  => -> $a, $b { $a * $b };

say %ops<add>.(10, 5);
say %ops<sub>.(10, 5);
say %ops<mul>.(10, 5);
```

Map values can be any object, including closures and `Sub` references.  
Call them with `.()` or by passing them to `&` calls. This pattern is  
the foundation for dispatch tables.  

## Dispatch table

Using a `Map` to route a command string to an action.  

```raku
my %cmd is Map =
    hi     => { say "Hello!" },
    bye    => { say "Goodbye!" },
    help   => { say "Commands: hi bye help" };

for <hi help bye> -> $c {
    (%cmd{$c} // { say "Unknown: $c" }).();
}
```

A dispatch table replaces a chain of `if`/`when` statements with a  
single key look-up. The `// { ... }` fallback handles unrecognised  
commands gracefully without an `else` branch.  

## Map as a cache

Memoising a computation with a mutable backing store exposed as a `Map`.  

```raku
my %cache;

sub fib-cached(Int $n --> Int) {
    return $n if $n <= 1;
    %cache{$n} //= fib-cached($n - 1) + fib-cached($n - 2);
}

say fib-cached(30);
say %cache.elems;
```

Even though `%cache` is a regular mutable hash here, the cached values  
are read-only after insertion — a common pattern where the cache itself  
is written once per key and behaves like a `Map` thereafter.  

## Maps and Bag

Converting a `Map` to a `Bag` to count occurrences.  

```raku
my @words = <raku is great raku rocks raku>;
my $bag = @words.Bag;

say $bag<raku>;
say $bag<great>;
say $bag.pairs.sort(-*.value);
```

A `Bag` is like a `Map` where values are always non-negative integers  
representing counts. Construct one from any sequence; `.pairs` returns  
`(elem => count)` pairs.  

## Maps and Set

Turning a `Map`'s key set into a `Set` for membership testing.  

```raku
my %allowed is Map = admin => 1, editor => 1, viewer => 1;
my $roles = %allowed.keys.Set;

say $roles<admin>;
say "guest" ∈ $roles;
say "editor" ∈ $roles;
```

`.keys.Set` extracts keys and wraps them in a `Set` for O(1) membership  
checks using `∈` or the ASCII alias `(elem)`.  

## Maps and Mix

Using a `Mix` for weighted or fractional frequencies.  

```raku
my $mix = Mix.new(<a b a c a b>);

say $mix<a>;
say $mix<b>;
say $mix.pairs.sort(-*.value);
```

A `Mix` extends `Bag` to allow non-integer weights. It shares the  
same look-up API as a `Map` and is useful for probability tables and  
weighted scoring.  

## Maps and Slip

Flattening a `Map` into a list with the `Slip` / `|` operator.  

```raku
my %base is Map = x => 1, y => 2;
my %extra = z => 3;

my %combined = |%base, |%extra;
say %combined;

my @flat = |%base;
say @flat;
```

Prefixing a `Map` with `|` turns it into a `Slip` of `Pair` objects.  
Inside a hash or `Map.new`, the pairs merge; inside an array or list  
context they appear as individual `Pair` elements.  

## Maps and signatures (`:%opts`)

Accepting a hash of named options through a signature.  

```raku
sub connect(:%opts) {
    my $host = %opts<host> // 'localhost';
    my $port = %opts<port> // 5432;
    say "Connecting to $host:$port";
}

connect(opts => { host => '10.0.0.1', port => 3306 });
connect(opts => {});
```

The `:%opts` parameter captures a hash argument by name. Inside the  
sub, `%opts` is an ordinary mutable hash so default handling with `//`  
works naturally.  

## Named-argument capture as a Map

Capturing all named arguments with `*%rest`.  

```raku
sub log-event($msg, *%meta) {
    say $msg;
    for %meta.pairs.sort(*.key) -> $p {
        say "  $p.key(): $p.value()";
    }
}

log-event("User login", user => 'alice', ip => '127.0.0.1', ts => 1234567);
```

`*%rest` in a signature collects every named argument that is not  
matched by an earlier parameter. The result behaves like a regular hash  
and can be iterated with `.pairs`.  

## Destructuring a Map

Pulling specific keys directly from a hash argument.  

```raku
sub greet(:$name, :$city = 'Unknown') {
    say "Hello, $name from $city!";
}

my %person = name => 'Jan', city => 'Bratislava', age => 40;
greet(|%person);
```

Slipping a hash into a call with `|` converts each pair into a named  
argument. The subroutine's named parameters act as a destructuring  
pattern, extracting only the keys it declares.  

## Constrained hash keys (`%h{Str}`)

Restricting which keys a hash accepts using a constraint.  

```raku
my %h{Str};
%h<alpha> = 1;
%h<beta>  = 2;

say %h.keys;
say %h.WHAT;
```

Placing a type inside braces after the variable name creates an  
object-keyed hash (also called an *ObjectHash*). `{Str}` restricts keys  
to strings — any object can be a key as long as it matches the type.  

## Object keys

Using objects as keys via their identity.  

```raku
class Point {
    has Int $.x;
    has Int $.y;
    method Str { "($!x,$!y)" }
}

my %grid{Point};
my $p = Point.new(x => 1, y => 2);
%grid{$p} = 'treasure';

say %grid{$p};
say %grid.keys;
```

An *ObjectHash* keyed on a class uses object identity (not stringified  
value) as the hash key. Two distinct `Point` instances with the same  
fields are different keys.  

## JSON serialisation

Converting a `Map` to a JSON string and back with `JSON::Fast`.  

```raku
# Requires: zef install JSON::Fast
# (examples are commented out so the file is readable without the module)
# use JSON::Fast;
# 
# my %cfg is Map = host => 'db', port => 5432, ssl => True;
# my $json = to-json %cfg.Hash;
# say $json;
# 
# my %restored = from-json($json);
# say %restored<host>;
```

`JSON::Fast` is the standard module for JSON in Raku. Because `Map`  
shares the hash API, `to-json` accepts it directly. The comment guards  
make this snippet self-documenting without requiring the module to be  
installed to read the file.  

## YAML serialisation

Serialising a `Map` to YAML with `YAMLish`.  

```raku
# Requires: zef install YAMLish
# use YAMLish;
#
# my %conf is Map = name => 'app', version => '1.0', debug => False;
# say save-yaml(%conf.Hash);
```

`YAMLish` is the most common pure-Raku YAML module. Convert a `Map`  
with `.Hash` before passing it because `save-yaml` expects a mutable  
hash.  

## Performance notes

Key facts about `Map` look-up performance.  

```raku
# Map uses the same O(1) average hash look-up as Hash.
# Integer or short-string keys are the fastest.
# Key stringification happens once at construction time, not per look-up.
# For hot paths with many look-ups, prefer Map over Hash because
# immutability removes concurrency guards and COW bookkeeping.

my %big is Map = (1..1000).map({ $_ => $_ * $_ });
my $t0 = now;
for ^100_000 { my $v = %big{500} }
say "100k look-ups in {(now - $t0).fmt('%.4f')}s";
```

`Map` look-up is O(1) amortised, identical to `Hash`. The immutability  
guarantee means Raku can skip certain write-barrier checks, giving a  
small but measurable throughput advantage in read-heavy workloads.  


## Common pitfalls

### Accidental flattening

Passing a map to a sub that expects a list may silently flatten it.  

```raku
my %m is Map = a => 1, b => 2;

# Unintended: %m flattens to a list of Pairs
sub show-list(*@items) { say @items.elems }
show-list(%m);        # 2 Pairs, not 1 Map

# Intended: pass as a single argument
sub show-map($m) { say $m.elems }
show-map(%m);         # 1 Map with 2 entries
```

When you pass `%m` to a `*@` slurpy, each `Pair` becomes one list  
element. Use `$m` as the parameter type to keep the map intact.  

### Key stringification

All built-in hash and map keys are stored as strings.  

```raku
my %m = Map.new(1, 'one', 2, 'two');

say %m{1};     # 'one'  — works because 1 coerces to "1"
say %m{"1"};   # 'one'  — same key
say ~1 eq "1"; # True   — tilde forces string form before comparison
```

Integer and other non-string keys are silently coerced to `Str` during  
storage. Use an `ObjectHash` (`%h{SomeType}`) if you need identity-  
based or typed keys.  

### Autovivification

Regular hashes autovivify nested structures; `Map` does not.  

```raku
my %h;
%h<a><b> = 1;   # OK: %h<a> is autovivified as a Hash
say %h;

my %m is Map = a => {};
# %m<a><b> = 1;  # Error: cannot modify an immutable Map
```

Subscripting a missing key on a *mutable* hash creates an empty inner  
hash automatically. On a `Map`, any attempt to assign raises  
`X::Assignment::RO`.  

### Binding vs assignment

`Map` variables bound with `:=` share the same object; `=` copies.  

```raku
my %a is Map = x => 1;
my %b := %a;    # binding: same Map object
my %c  = %a;    # assignment: new mutable Hash copy

say %b === %a;  # True  — identical object
say %c === %a;  # False — independent copy
say %c.WHAT;    # Hash, not Map
```

Use `:=` when you want an alias (useful for passing maps into subs  
without copying); use `=` to get an independent, mutable snapshot.  

### Copying vs referencing nested values

Values inside a `Map` are references, not deep copies.  

```raku
my @data = [1, 2, 3];
my %m is Map = list => @data;

%m<list>.push(4);   # mutates the original @data!
say @data;          # (1 2 3 4)
```

`Map` prevents *you* from rebinding its keys, but it does not deep-  
clone its values. If a value is a mutable container (an array, a hash),  
that container can still be modified through the reference.  


## Comparison with other languages

### Go maps

In Go, `map[K]V` is mutable and generic; it requires explicit  
initialisation with `make`. There is no built-in immutable map type —  
callers rely on conventions or custom wrappers to prevent mutation.  
Raku's `Map` enforces immutability at the type level.  

```raku
# Raku:
my %m is Map = a => 1, b => 2;

# Go equivalent (mutable):
# m := map[string]int{"a": 1, "b": 2}
```

### Python dicts

Python dicts are mutable by default. An immutable equivalent is  
achieved via `types.MappingProxyType` or a frozen `dataclass`. Raku's  
`Map` is a first-class immutable type, not an adapter.  

```raku
# Raku:
my %m is Map = x => 10;
say %m<x>;

# Python equivalent (mutable):
# d = {'x': 10}
# print(d['x'])
```

### JavaScript objects and Maps

JavaScript has two kinds of key-value stores: plain objects (string/  
symbol keys only, prototype-polluted) and `Map` (any key type, ordered  
by insertion). Raku's `Map` is closer to JS `Map` but is immutable,  
uses string keys by default, and has no insertion-order guarantee.  

```raku
# Raku:
my %m is Map = a => 1;
say %m<a>;

# JS equivalent (mutable):
# const m = new Map([['a', 1]]);
# console.log(m.get('a'));
```

### Ruby Hash

Ruby `Hash` is mutable; Ruby's `freeze` method makes any object  
immutable at runtime. Raku separates mutability into distinct types  
(`Hash` vs `Map`) rather than bolting it on after creation.  

```raku
# Raku:
my %m is Map = k => 'v';

# Ruby: h = {k: 'v'}.freeze
```


## Exercises

1. Create a `Map` that maps the digits 0–9 to their English names and  
   print each entry in the form `"3 => three"`.  

2. Given a `Map` of country codes to country names, write code that  
   prints every country whose name is longer than six characters, sorted  
   alphabetically.  

3. Write a sub `lookup($key, %map)` that returns the value for `$key`  
   if present, or the string `"not found"` otherwise, without using `//`.  

4. Merge two `Map` objects so that the second map's values win for any  
   shared keys. Return the result as a new `Map`.  

5. Implement a simple memoised factorial using a mutable `%cache` hash.  
   Print the cache after computing `factorial(10)`.  

6. Create a dispatch table `Map` for four arithmetic operations (`+`, `-`,  
   `*`, `/`). Read an expression like `"8 * 7"` from a string, split it,  
   and evaluate it using the table.  

7. Given a list of words, build a `Map` from word-length to a sorted  
   list of words of that length (group-by), then print each group.  

8. Use `Bag` to count word frequencies in a sentence, then print the  
   top three most-frequent words.  

9. Write a function that takes two `Map` objects and returns a `Map`  
   containing only the keys that appear in *both* maps, with values from  
   the first map.  

10. Implement a tiny router: store URL patterns as keys and handler  
    closures as values. Match `/users/42` by splitting on `/` and look up  
    the first path segment in the table, calling the handler with the  
    remaining segments.
