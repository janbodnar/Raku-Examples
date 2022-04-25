# Raku-Examples
Raku code examples


# Print n-times 

```raku

my $word = 'falcon';

say "$word\n" x 5;

say '--------------------';

my $n = 0;
repeat {
    say 'falcon';
    $n++;
} while $n <= 5;
```

## Conditions

`if/elsif/else` & `unless` keywords  
parantheses not needed  

```raku
my $age = 45;

if $age >= 18 {
  say 'adult';
}

say 'adult' if $age >= 18;

unless $age < 18 {
  say 'adult';
}
```


```raku
my @vals = <0 -1 -2 3 11 1 2 -4>;

for @vals -> $val {

  if $val > 0 {
    say "pos";
  } elsif $val < 0 {
    say "neg";
  } else {
    say "zero";
  }
}
```

## Types

```raku

my @vals = (1, 2, 3, 4);
my $word = "falcon";
my $n = 12;

say @vals.WHAT;
say $word.WHAT;
say $n.WHAT;

say $n.^name;

say '-------------------------';

my @data = 'storm', 1, 1.3, True, 1/2;

for @data -> $e {

  say "integer" if $e.isa(Int);
  say "string" if $e.isa(Str);
  say "real" if $e.isa(Real);
  say "boolean" if $e.isa(Bool);
  say "rational" if $e.isa(Rat);

  say '-------';
}
```


## Mutators 

```raku
# a mutator is a procedure that modifies
# the state of an object

my @vals = <1 3 2 7 8 4 5 6 0 -2 -1>;

# mutator
@vals.push(9);
say @vals;

# function
say @vals.sort;
say @vals;

say '------------------------------';

# mutator
@vals.=sort;
say @vals;
```
