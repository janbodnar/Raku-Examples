# Sorting

## Sort list 

```raku
my $vals = <8 0 -1 2 1 -3 3 5 7 4>;
my $words = <"sky" "no" "boot" "certain" "universe" "fifty">;

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
