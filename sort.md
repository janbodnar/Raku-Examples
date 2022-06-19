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

## Sort multiple criteria

```raku
my @words = <snake Sun be some unit word Earth to sky falcon>;
put @words.sort:{.chars, .lc};
put sort -> $x { $x.chars, $x.lc }, @words;
```

## Sort objects

```raku
#!/usr/bin/rakudo

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
