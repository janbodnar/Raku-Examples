# Arrays 

Arrays are one of the fundamental data structures in Raku, providing a flexible  
and efficient way to store, access, and manipulate ordered collections of data.  
Arrays in Raku are mutable, can contain elements of any type, and come with  
a rich set of built-in methods for traversal, transformation, searching, sorting,  
and more. This chapter explores the many features and idioms of Raku arrays, from  
basic definition and indexing to advanced operations like chunking, set algebra,  
and memory management. Whether you're new to Raku or looking to deepen your   
understanding, the following sections will provide practical examples and  
explanations for working with arrays effectively.

Arrays are mutable data structures.  They are created with the `@` sigil.  


## Array vs list  

Difference between arrays and lists in Raku.  

```raku
# array
my @vals = (1, 2, 3, 4, 5);
say @vals.^name;

# list
my $vals = (1, 2, 3, 4, 5);
say $vals.^name;
```

Arrays use the `@` sigil and are mutable containers, while lists use the  
`$` sigil and are immutable sequences. The `.^name` method reveals the  
actual type of the data structure.  

## elems

Getting the number of elements in an array.  

```raku
my @vals = 1, 2, 3, 4, 5;
say @vals.elems;
say @vals;
```

The `elems` method returns the count of elements in the array. This is  
equivalent to the length or size of the array in other languages.  

## Define arrays

Creating arrays using range and sequence operators.  

```raku
my @vals = 1..10;
say @vals;

my @vals2 = (1..10).reverse;
say @vals2;

my @vals3 = 1,3...20;
say @vals3;

my @vals4 = 'a'..'z';
say @vals4;
```

The range operator `..` creates sequential values, `.reverse` reverses  
order, `...` creates arithmetic sequences, and ranges work with  
characters too.  

## First/last element

Accessing the first and last elements of an array.  

```raku
my @vals = <sky blue rock forest book nice cool nap>;
say @vals.first;
say @vals.first :end;

say '----------------------';

say @vals.head;
say @vals.tail;

say '----------------------';

say @vals[0];
say @vals[*-1];
```

Multiple ways to get first/last elements: `.first` method with `:end`  
adverb, `.head`/`.tail` methods, or direct indexing with `[0]` and  
`[*-1]` for first and last positions.  

## keys & values  

Getting indices and values from an array.  

```raku
my @words = <sky blue rock forest book nice cool nap>;

say @words.keys;
say @words.values;
```

The `.keys` method returns the indices (0, 1, 2, ...) and `.values`  
returns the actual elements. This is useful for iteration with both  
index and value.  

## typed arrays  

Creating arrays constrained to specific data types.  

```raku
my Str @words = 'sky', 'blue', 'rock', 'forest', 'book', 'nice', 'cool', 'nap';
say @words.WHAT;

my @words2 of Str = 'sky', 'blue', 'rock', 'forest', 'book', 'nice', 'cool', 'nap';
say @words2.WHAT;

my @words3 is Array[Str] = 'sky', 'blue', 'rock', 'forest', 'book', 'nice', 'cool', 'nap';
say @words3.WHAT;
```

Three syntax variants for typed arrays: `Str @words`, `@words of Str`,  
and `@words is Array[Str]`. All restrict the array to contain only  
strings, providing compile-time type safety.  

## Traversing arrays

Different ways to iterate through array elements.  

```raku
my @vals = (1, 2, 3, 4, 5);

for @vals -> $val {
  say $val;
}
```

The `for` loop with arrow syntax `->` is the idiomatic way to iterate  
through arrays in Raku, binding each element to the `$val` variable.  

```raku
my @vals = (1, 2, 3, 4, 5);
my $n = @vals.elems;

loop (my $i=0; $i < $n; $i++) {
  say @vals[$i];
}
```

Traditional C-style loop using index-based access for cases where you  
need the index position during iteration.  

## Indexing & head & tail  

Accessing array elements by position and getting first/last elements.  

```raku
my @vals = (1, 2, 3, 4, 5);
say @vals[0];
say @vals[2];
say @vals.head;
say @vals.tail;
```

Square brackets `[]` access elements by index starting from 0. The  
`.head` method returns the first element, `.tail` returns the last  
element.  

## The push & prepend

Adding elements to the end and beginning of arrays.  

```raku
my Int @vals = (1, 2, 3, 4, 5);
@vals.push(6);
@vals.push(7);
@vals.push(8);
@vals.prepend(0);
@vals.prepend(-1);

say @vals;
```

The `.push` method adds elements to the end of the array, while  
`.prepend` adds elements to the beginning. Both modify the original  
array in place.  

## Array slicing

Extracting portions of arrays using range subscripts.  

```raku
my @fruits = <apple banana cherry date elderberry fig grape>;

say @fruits[1..3];
say @fruits[2..*];
say @fruits[*-3..*-1];
say @fruits[0,2,4];
say @fruits[1,3...7];
```

Array slicing uses ranges and lists within square brackets. `1..3`  
selects elements 1-3, `2..*` from index 2 to end, `*-3..*-1` last  
three elements, and `0,2,4` specific indices.  

## Array concatenation

Combining multiple arrays into single arrays.  

```raku
my @nums1 = 1, 2, 3;
my @nums2 = 4, 5, 6;
my @nums3 = 7, 8, 9;

my @combined = |@nums1, |@nums2, |@nums3;
say @combined;

my @flattened = flat @nums1, @nums2, @nums3;
say @flattened;

my @slip = (@nums1, @nums2).flat;
say @slip;
```

Use the slip operator `|` to unpack arrays for concatenation, or the  
`flat` function to flatten nested structures. Both create new arrays  
without modifying originals.  

## Array filtering with grep

Selecting elements that match specific criteria.  

```raku
my @numbers = 1, 2, 3, 4, 5, 6, 7, 8, 9, 10;
my @words = <sky blue rock forest book nice cool nap>;

say @numbers.grep(* > 5);
say @numbers.grep(* %% 2);
say @words.grep(/^.{4}$/);
say @words.grep(*.starts-with('b'));
```

The `.grep` method filters arrays using conditions. `* > 5` finds  
elements greater than 5, `* %% 2` finds even numbers, `/^.{4}$/`  
matches 4-character words, and `*.starts-with('b')` finds words  
starting with 'b'.  

## Array transformation with map

Converting each element using a transformation function.  

```raku
my @numbers = 1, 2, 3, 4, 5;
my @words = <hello world from raku>;

say @numbers.map(* * 2);
say @numbers.map(* ** 2);
say @words.map(*.uc);
say @words.map(*.chars);
say @numbers.map({ "Number: $_" });
```

The `.map` method transforms each element. `* * 2` doubles values,  
`* ** 2` squares them, `*.uc` converts to uppercase, `*.chars` gets  
string lengths, and `{ "Number: $_" }` creates formatted strings.  

## Array sorting and reversing

Organizing array elements in different orders.  

```raku
my @numbers = 5, 2, 8, 1, 9, 3;
my @words = <zebra apple banana cherry>;

say @numbers.sort;
say @numbers.sort(* R<=> *);
say @words.sort;
say @words.sort(*.chars);
say @numbers.reverse;
```

The `.sort` method orders elements ascending by default. `* R<=> *`  
sorts descending, `*.chars` sorts by string length. `.reverse`  
reverses the current order without sorting.  

## Array searching and finding

Locating elements and their positions in arrays.  

```raku
my @words = <sky blue rock forest book nice cool nap>;
my @numbers = 10, 20, 30, 40, 50;

say @words.first(*.starts-with('b'));
say @words.first-index(* eq 'rock');
say @numbers.grep-index(* > 25);
say @words.contains('forest');
say @words.any eq 'book';
```

Use `.first` to find first matching element, `.first-index` for its  
position, `.grep-index` for all matching positions, `.contains` for  
membership testing, and `.any` for existence checking.  

## Array reduction and aggregation

Computing single values from array contents.  

```raku
my @numbers = 1, 2, 3, 4, 5, 6, 7, 8, 9, 10;

say @numbers.sum;
say @numbers.max;
say @numbers.min;
say @numbers.minmax;
say @numbers.reduce(* + *);
say @numbers.reduce(* * *);
```

Built-in methods: `.sum` adds all elements, `.max`/`.min` find  
extremes, `.minmax` returns both. `.reduce` applies an operation  
cumulatively: `* + *` sums, `* * *` multiplies all elements.  

## Array unique and distinct operations

Removing duplicate elements from arrays.  

```raku
my @numbers = 1, 2, 2, 3, 3, 3, 4, 5, 5;
my @words = <apple banana apple cherry banana date>;

say @numbers.unique;
say @words.unique;
say @words.squish;
say @numbers.repeated;
say @words.Set;
```

The `.unique` method removes all duplicates, `.squish` removes only  
consecutive duplicates, `.repeated` returns only duplicate elements,  
and `.Set` creates a set with unique elements.  

## Array comparison and equality

Comparing arrays for equality and relationships.  

```raku
my @arr1 = 1, 2, 3, 4, 5;
my @arr2 = 1, 2, 3, 4, 5;
my @arr3 = 5, 4, 3, 2, 1;

say @arr1 eqv @arr2;
say @arr1 eqv @arr3;
say @arr1 (==) @arr2;
say @arr1.Set eqv @arr3.Set;
say @arr1 ~~ @arr2;
```

Use `eqv` for exact equality including order, `(==)` for numeric  
comparison, `.Set` conversion for order-independent comparison, and  
`~~` for smart matching.  

## Array chunking and grouping

Dividing arrays into smaller sub-arrays.  

```raku
my @numbers = 1, 2, 3, 4, 5, 6, 7, 8, 9, 10;
my @words = <one two three four five six seven eight>;

say @numbers.batch(3);
say @numbers.rotor(2);
say @numbers.rotor(3 => 1);
say @words.classify(*.chars);
say @words.categorize(*.substr(0,1));
```

The `.batch` method creates chunks of specified size, `.rotor` creates  
overlapping or non-overlapping windows, `.classify` groups by  
calculated keys, and `.categorize` groups by multiple criteria.  

## Array rotation and shifting

Moving elements within arrays cyclically.  

```raku
my @letters = <a b c d e f g>;

say @letters.rotate(2);
say @letters.rotate(-2);
my @shifted = @letters[2..*], @letters[0..1];
say @shifted;

@letters = @letters.rotate(1);
say @letters;
```

The `.rotate` method moves elements cyclically: positive numbers  
rotate left, negative rotate right. Manual shifting combines slices  
to achieve custom rotations.  

## Multi-dimensional arrays

Creating and working with nested array structures.  

```raku
my @matrix = [1, 2, 3], [4, 5, 6], [7, 8, 9];
my @table[3;3] = (1, 2, 3; 4, 5, 6; 7, 8, 9);

say @matrix[1][2];
say @table[1;2];
say @matrix.elems;
say @matrix[0].elems;

for @matrix -> @row {
    say @row.join(' ');
}
```

Arrays can contain other arrays creating matrices. Access with  
`[row][col]` for arrays of arrays, or `[row;col]` for shaped arrays.  
Iterate with nested loops or by rows.  

## Array zip and pairing

Combining multiple arrays element-wise.  

```raku
my @names = <Alice Bob Charlie>;
my @ages = 25, 30, 35;
my @cities = <NYC LA Chicago>;

say (@names Z @ages);
say (@names Z=> @ages);
say (@names Z @ages Z @cities);
say @names.pairs;
say [Z] @names, @ages, @cities;
```

The `Z` zip operator combines arrays element-wise. `Z=>` creates  
pairs, multiple arrays can be zipped together. `.pairs` creates  
index-value pairs, `[Z]` reduces multiple arrays.  

## Array set operations

Performing mathematical set operations on arrays.  

```raku
my @set1 = 1, 2, 3, 4, 5;
my @set2 = 4, 5, 6, 7, 8;

say (@set1 (&) @set2);
say (@set1 (|) @set2);
say (@set1 (-) @set2);
say (@set1 (^) @set2);
say @set1 ⊆ @set2;
```

Set operators: `(&)` intersection (common elements), `(|)` union (all  
elements), `(-)` difference (elements in first but not second), `(^)`  
symmetric difference, and `⊆` subset testing.  

## Array copying and cloning

Creating independent copies of arrays.  

```raku
my @original = 1, 2, 3, [4, 5], 6;

my @shallow = @original;
my @deep = @original.clone;
my @manual = |@original;

@original[0] = 99;
@original[3][0] = 999;

say @original;
say @shallow;
say @deep;
```

Assignment creates references, `.clone` creates shallow copies,  
manual copying with `|` also creates shallow copies. For deep copies  
of nested structures, use specialized copying methods.  

## Array serialization

Converting arrays to and from string representations.  

```raku
my @data = 1, 2, 3, 'hello', 4.5;

say @data.perl;
say @data.raku;
say @data.gist;
say @data.Str;
say @data.join(',');

my $serialized = @data.raku;
my @restored = EVAL $serialized;
say @restored;
```

The `.perl` and `.raku` methods create code that can recreate the  
array, `.gist` provides readable output, `.Str` basic string  
conversion, `.join` with custom separators.  

## Array statistics

Computing statistical measures from numeric arrays.  

```raku
my @scores = 85, 92, 78, 96, 88, 91, 87, 83, 94, 89;

say @scores.sum / @scores.elems;  # mean
say @scores.sort[@scores.elems div 2];  # median
say @scores.max - @scores.min;  # range

my $mean = @scores.sum / @scores.elems;
my $variance = @scores.map({($_ - $mean) ** 2}).sum / @scores.elems;
say $variance.sqrt;  # standard deviation
```

Calculate common statistics: mean (average), median (middle value),  
range (max - min), and standard deviation for measure of spread.  
Custom calculations using map and reduce operations.  

## Array benchmarking

Measuring performance of array operations.  

```raku
use experimental :cached;

my @large = 1..100000;

my $start = now;
my @doubled = @large.map(* * 2);
my $map-time = now - $start;

$start = now;
my @filtered = @large.grep(* %% 2);
my $grep-time = now - $start;

say "Map time: {$map-time}s";
say "Grep time: {$grep-time}s";
```

Use `now` to measure execution time of operations. Compare different  
approaches to find optimal solutions for large datasets. Consider  
lazy evaluation for memory efficiency.  

## Advanced array manipulation

Complex array operations and transformations.  

```raku
my @data = 1, 2, 3, 4, 5, 6, 7, 8, 9, 10;

say @data.combinations(3).elems;
say @data.permutations.elems;
say @data.pick(5);
say @data.roll(5);
say @data.grab(3);

my @nested = @data.rotor(2).map(*.Array);
say @nested.tree(*.sum, *.join('-'));
```

Advanced methods: `.combinations` generates combinations,  
`.permutations` all arrangements, `.pick` selects without  
replacement, `.roll` with replacement, `.grab` removes selected  
elements, and `.tree` for complex transformations.  

## Array memory management

Understanding memory usage and optimization for large arrays.  

```raku
my @small = 1..100;
my @large = 1..1000000;

say @small.elems;
say @large.elems;

# Lazy arrays for memory efficiency
my @lazy = (1..*).lazy.map(* ** 2);
say @lazy[99];  # Only computes up to index 99

# Clear array to free memory
@large = ();
say @large.elems;
```

Large arrays consume significant memory. Use lazy evaluation with  
`.lazy` for infinite sequences that compute elements on demand.  
Clear arrays with `()` assignment to free memory immediately.  
