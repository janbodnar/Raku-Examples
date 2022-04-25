# Raku-Examples
Raku code examples


## Mutators 

```raku
# a mutator is a procedure that modifies
# the state of an object

my @vals = <1 3 2 7 8 4 5 6 0 -2 -1>;

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
