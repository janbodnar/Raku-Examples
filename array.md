# Arrays 

Arrays are mutable data structures.  They are created with the `@` sigil.  


## Array vs list  

```raku
# array
my @vals = (1, 2, 3, 4, 5);
say @vals.^name;

# list
my $vals = (1, 2, 3, 4, 5);
say $vals.^name;
```

## elems

`elems` returns the size of the list  

```raku
my @vals = 1, 2, 3, 4, 5;
say @vals.elems;
say @vals;
```

## Define arrays

define arrays with sequence & range operators  

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

## First/last element

get first/last elements

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

## keys & values  

```raku
my @words = <sky blue rock forest book nice cool nap>;

say @words.keys;
say @words.values;
```

## typed arrays  

```raku
my Str @words = 'sky', 'blue', 'rock', 'forest', 'book', 'nice', 'cool', 'nap';
say @words.WHAT;

my @words2 of Str = 'sky', 'blue', 'rock', 'forest', 'book', 'nice', 'cool', 'nap';
say @words2.WHAT;

my @words3 is Array[Str] = 'sky', 'blue', 'rock', 'forest', 'book', 'nice', 'cool', 'nap';
say @words3.WHAT;
```

## Traversing arrays

for loop  

```raku
my @vals = (1, 2, 3, 4, 5);

for @vals -> $val {
  say $val;
}
```

traversing with loop  

```raku
my @vals = (1, 2, 3, 4, 5);
my $n = @vals.elems;

loop (my $i=0; $i < $n; $i++) {
  say @vals[$i];
}
```

## Indexing & head & tail  

```raku
my @vals = (1, 2, 3, 4, 5);
say @vals[0];
say @vals[2];
say @vals.head;
say @vals.tail;
```

## The push & prepend

typed arrays; push & prepend methods  

```raku
my Int @vals = (1, 2, 3, 4, 5);
@vals.push(6);
@vals.push(7);
@vals.push(8);
@vals.prepend(0);
@vals.prepend(-1);

say @vals;
```
