# List

Lists are ordered, immutable sequences of values in Raku. Unlike arrays,  
lists cannot be modified after creation, making them ideal for representing  
fixed collections of data. They are created with the `$` sigil or simply  
as parenthesized comma-separated values. This chapter explores list  
creation, element access, common operations, and advanced features such  
as lazy evaluation and reduction operators.  

Lists are immutable data structures.  


## Creating lists

Creating lists using different syntaxes.  

```raku
my $nums = (1, 2, 3, 4, 5);
say $nums;
say $nums.^name;

my $words = <sky blue rock forest>;
say $words;
say $words.^name;

my $empty = ();
say $empty.^name;
```

Parenthesized comma-separated values create a `List`. The angle-bracket  
`<...>` word-quoting syntax creates a list of strings without needing  
quotes around each word. An empty pair of parentheses creates an empty  
list.  

## List vs array

Difference between lists and arrays in Raku.  

```raku
# list stored in a scalar variable
my $list = (1, 2, 3, 4, 5);
say $list.^name;
say $list.is-lazy;

# array stored in an array variable
my @arr = (1, 2, 3, 4, 5);
say @arr.^name;

# list assigned to an array variable is flattened into an array
my @from-list = (1, 2, 3, 4, 5);
say @from-list.^name;
```

A `List` stored in a `$` scalar variable remains a `List`. When assigned  
to a `@` array variable the list is flattened into a mutable `Array`.  
The `.^name` method reveals the actual type at runtime.  

## Immutability

Demonstrating that lists cannot be modified after creation.  

```raku
my $vals = (10, 20, 30, 40, 50);
say $vals;

# attempt to assign to an element raises an error
try {
    $vals[0] = 99;
    CATCH { default { say "Error: " ~ .message } }
}

# creating a new list instead of modifying
my $updated = (99, |$vals[1..*]);
say $updated;
```

Assigning to an index of a `List` throws a runtime error because lists  
are immutable. To get a modified version, create a new list using the  
slip operator `|` to include existing elements alongside the new value.  

## Accessing elements

Accessing individual list elements by index.  

```raku
my $fruits = <apple banana cherry date elderberry>;

say $fruits[0];
say $fruits[2];
say $fruits[*-1];
say $fruits[*-2];

say $fruits.head;
say $fruits.tail;
```

Square-bracket indexing accesses elements by zero-based position.  
The `*-1` notation accesses from the end, so `*-1` is the last element  
and `*-2` is the second-to-last. The `.head` and `.tail` methods are  
convenient aliases for the first and last element.  

## List methods

Common list methods for inspecting and traversing lists.  

```raku
my $vals = (3, 1, 4, 1, 5, 9, 2, 6);

say $vals.elems;
say $vals.head;
say $vals.tail;
say $vals.head(3);
say $vals.tail(3);
say $vals.keys;
say $vals.values;
```

The `.elems` method returns the count of elements. `.head(n)` and  
`.tail(n)` return the first or last `n` elements as a new list.  
`.keys` returns the indices and `.values` returns the elements,  
mirroring the same methods available on arrays and hashes.  

## Multiple assignment

Unpacking list elements into individual variables.  

```raku
my ($first, $second, $third) = <one two three four five>;
say $first;
say $second;
say $third;

my ($head, *@rest) = 10, 20, 30, 40, 50;
say $head;
say @rest;

my ($x, $y) = 1, 2;
($x, $y) = $y, $x;
say "$x $y";
```

List assignment unpacks elements left-to-right into the listed variables.  
Extra elements on the right are discarded unless a slurpy `*@rest` is  
used to capture them. Assigning `($x, $y) = $y, $x` evaluates the right  
side first, swapping the two variables without a temporary variable.  

## List ranges and sequences

Building lists from ranges and arithmetic sequences.  

```raku
my $nums = (1..10).List;
say $nums;

my $evens = (2, 4 ... 20).List;
say $evens;

my $desc = (10, 9 ... 1).List;
say $desc;

my $chars = ('a'..'f').List;
say $chars;
```

The range operator `..` generates an inclusive sequence of values. The  
sequence operator `...` infers the step from the first two elements,  
producing arithmetic progressions. Calling `.List` materializes the  
lazy range into a concrete `List`.  

## List slicing

Extracting sub-lists using ranges and index lists.  

```raku
my $letters = ('a'..'j').List;

say $letters[1..4];
say $letters[0, 2, 4, 6];
say $letters[2..*];
say $letters[*-3..*-1];
say $letters[1, 3 ... 9];
```

A range inside square brackets selects a contiguous slice. A comma-  
separated list of indices selects specific positions. `2..*` means  
from index 2 to the end, and `*-3..*-1` selects the last three  
elements. The sequence operator in subscripts picks every other index.  

## Filtering with grep

Selecting elements that satisfy a condition.  

```raku
my $numbers = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
my $words   = <sky blue rock forest book nice cool nap>;

say $numbers.grep(* > 5);
say $numbers.grep(* %% 2);
say $words.grep(*.chars == 4);
say $words.grep(*.starts-with('b'));
```

The `.grep` method returns a new list of elements for which the given  
condition is true. The `*` whatever-star is shorthand for an anonymous  
function. `%% 2` tests divisibility, `.chars` gives string length, and  
`.starts-with` checks the initial characters.  

## Transformation with map

Applying a function to every element of a list.  

```raku
my $numbers = (1, 2, 3, 4, 5);
my $words   = <hello world raku>;

say $numbers.map(* * 3);
say $numbers.map(* ** 2);
say $words.map(*.uc);
say $words.map(*.chars);
say $numbers.map({ "item-$_" });
```

The `.map` method returns a new list with each element replaced by the  
result of applying the given function. `* * 3` triples each value,  
`* ** 2` squares it, `*.uc` uppercases strings, `*.chars` measures  
length, and a bare block `{ }` allows arbitrary expressions using `$_`.  

## Zip operator

Combining two or more lists element-wise.  

```raku
my $names = <Alice Bob Charlie>;
my $ages  = (25, 30, 35);
my $cities = <NYC LA Chicago>;

say ($names Z $ages);
say ($names Z=> $ages);
say ($names Z $ages Z $cities);
say [Z] $names, $ages, $cities;
```

The `Z` infix operator interleaves elements from two lists, producing a  
list of pairs. `Z=>` creates `Pair` objects. Three or more lists can be  
zipped in one expression. The `[Z]` reduction form applies the zip  
operator across a list of lists.  

## Reduction operators

Reducing a list to a single value with a binary operator.  

```raku
my $nums = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

say [+] $nums;
say [*] $nums;
say [max] $nums;
say [min] $nums;
say [\+] $nums;
```

The `[op]` meta-operator reduces a list by repeatedly applying `op`  
between adjacent elements. `[+]` sums, `[*]` multiplies, `[max]` and  
`[min]` find extremes. The `[\op]` triangular form returns a list of  
running totals, one per prefix of the input.  

## Flat and nested lists

Working with nested lists and flattening them.  

```raku
my $nested = (1, (2, 3), (4, (5, 6)));
say $nested;
say $nested.elems;

my $flat = $nested.flat;
say $flat;
say $flat.elems;

my $joined = (|(1, 2), |(3, 4), 5);
say $joined;
```

A list can contain other lists, creating a nested structure. The  
`.flat` method recursively flattens all levels into a single-level  
list. The slip operator `|` inside a list literal inserts the elements  
of its argument directly into the surrounding list.  

## Lazy lists

Creating lists that compute elements on demand.  

```raku
my $naturals = (1..*).lazy;
say $naturals.^name;
say $naturals.head(10);
say $naturals.grep(* %% 7).head(5);

my $squares = (1..*).lazy.map(* ** 2);
say $squares.head(8);
```

Lazy lists defer computation until elements are actually needed. The  
`.lazy` method converts any list or range into a lazy sequence. This  
allows working with conceptually infinite sequences. `.head(n)` forces  
evaluation of only the first `n` elements, making the operation safe  
and efficient.  

## List coercion

Converting between lists and other types.  

```raku
my $list = (1, 2, 3, 4, 5);

my @arr  = $list.Array;
say @arr.^name;

my $seq  = $list.Seq;
say $seq.^name;

my $set  = $list.Set;
say $set.^name;

say $list.join(', ');
say $list.raku;
```

Lists can be coerced into other collection types. `.Array` produces a  
mutable `Array`, `.Seq` a lazy `Seq`, and `.Set` an unordered  
collection of unique values. `.join` converts the list to a string  
with a given separator, and `.raku` produces a Raku code representation  
that can reconstruct the list.  
