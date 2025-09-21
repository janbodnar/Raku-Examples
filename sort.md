# Sorting

## Sort list 

```raku
my $vals = <8 0 -1 2 1 -3 3 5 7 4>;
my $words = <sky no boot certain universe fifty>;

say $vals.sort;
say $vals.sort: +*;
say $vals;
say $vals.sort: -*;

say '-----------------------------';

$vals.=reverse;
say $vals;


say '-----------------------------';

say $words.sort;
say $words.sort: { $^a leg $^b };
say $words.sort: { $^b leg $^a };
say $words.sort: { .chars };

say '-----------------------------';

# Sort numbers in descending order
say $vals.sort: {$^b <=> $^a};

# Sort numbers by their absolute value
say $vals.sort: *.abs;
```

## In-place sorting

```raku
my @numbers = 4, 2, 7, 1, 9;
@numbers.=sort;
say @numbers; # [1, 2, 4, 7, 9]

my @words = <dog cat bird>;
@words.=sort;
say @words; # [bird, cat, dog]
```

## Sorting with custom logic

```raku
my @words = <apple banana cherry date>;

# Sort by number of vowels
sub count-vowels(Str $s) { $s.chars - $s.transliterate('aeiouAEIOU', '').chars }
say @words.sort(&count-vowels);

# Sort by the last character of a string
say @words.sort({.substr(*-1)});
```

## Case-insensitive sorting

```raku
my @words = <Apple, banana, Cherry, aPPle>;
say @words.sort({.lc}); # (Apple, aPPle, banana, Cherry)
```

## Stable sorting with `sort-by`

The `sort-by` method is stable, meaning that elements that compare as equal will remain in their original relative order.

```raku
my @words = <a apple b banana c cherry>;

# Sort by length (stable)
say @words.sort-by({.chars}); # (a b c apple banana cherry)
```

## Sorting Pairs

```raku
my @pairs = (c => 3, a => 1, b => 2);

# Sort pairs by value
say @pairs.sort(*.value); # (a => 1, b => 2, c => 3)

# Sort pairs by key
say @pairs.sort(*.key); # (a => 1, b => 2, c => 3)
```

## Sorting with `cmp`

The `cmp` operator compares strings ASCIIbetically.

```raku
my @words = <C b A c a B>;
say @words.sort({$^a cmp $^b}); # (A B C a b c)
```

## Sorting mixed data types

When sorting a list with mixed data types, Raku will try to coerce the elements to a common type.

```raku
my @list = <a 10 b 2 c 1>;
say @list.sort; # (1 10 2 a b c)
```

## Sorting with Unicode characters

Raku handles Unicode characters correctly out of the box.

```raku
my @words = <é B a Ä c b>;
say @words.sort; # (a Ä b B c é)
```

## Sorting lazy lists

Sorting a lazy list will force its evaluation.

```raku
my $lazy-list = (1..10).lazy;
my @sorted = $lazy-list.sort;
say @sorted;
```

## Miscellaneous sorting examples

```raku
# Using a fat arrow block
my @numbers = 5, 2, 8, 1;
say @numbers.sort(-> $a, $b { $b <=> $a }); # (8 5 2 1)

# Sorting a list of lists
my @list-of-lists = [[1, 2], [3, 1], [2, 3]];
say @list-of-lists.sort(*[1]); # [[3, 1], [1, 2], [2, 3]]

# Sort by number of 'e's in a string
my @words = <cheese beer coffee tea>;
sub count-e(Str $s) { $s.comb('e').elems }
say @words.sort(&count-e);

# Sort by length then alphabetically for strings of same length
my @words2 = <beta alpha gamma delta>;
say @words2.sort({ .chars, .Str });

# Using .& with a subroutine name
sub by-length($s) { $s.chars }
say @words2.sort(&by-length);
```

## Sort multiple criteria

```raku
my @words = <snake Sun be some unit word Earth to sky falcon>;
put @words.sort:{.chars, .lc};
put sort -> $x { $x.chars, $x.lc }, @words;
```

## Sort objects

```raku

class User {
   has Str $.name;
   has Str $.occupation;

   method Str {
        "{$.name} the {$.occupation}"
   }
}

my $u1 = User.new(name => "John Doe", occupation => 'gardener');
my $u2 = User.new(name => "Roger Roe", occupation => 'driver');

my @team = $u1, $u2;

my @ordByName = @team.sort(*.name)».name;
my @ordByOccup = @team.sort(*.occupation);

say @ordByName.join(', ');
say @ordByOccup.join(', ');
```

## Sort list of hashes

```raku
my @people = (
    { name => 'Kevin', age => 20, }, { name => 'Amanda', age => 19, },
    { name => 'Robert', age => 25, }, { name => 'Patrick', age => 11, },
    { name => 'Jana', age => 45, }, { name => 'Roman', age => 31, }
);

@people.sort({
    %^a<age> <=> %^b<age>
}).say;

@people.sort( -> %a, %b {
    %b<age> <=> %a<age>
}).say;
```
