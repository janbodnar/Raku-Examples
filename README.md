# Raku-Examples
Raku code examples

## Conditions

`if/elsif/else` keywords  
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
