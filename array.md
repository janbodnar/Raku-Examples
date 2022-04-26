# Array 

Arrays are mutable data structures.  They are created with the `@` sigil.  


array vs list  

```raku
# array
my @vals = (1, 2, 3, 4, 5);
say @vals.^name;

# list
my $vals = (1, 2, 3, 4, 5);
say $vals.^name;
```

`elems` returns the size of the list  

```raku
my @vals = 1, 2, 3, 4, 5;
say @vals.elems;
say @vals;
```

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

for loop  

```raku
my @vals = (1, 2, 3, 4, 5);

for @vals -> $val {
  say $val;
}
```

travering with loop  

```raku
my @vals = (1, 2, 3, 4, 5);
my $n = @vals.elems;

loop (my $i=0; $i < $n; $i++) {
  say @vals[$i];
}
```


Indexing & head & tail  

```raku
my @vals = (1, 2, 3, 4, 5);
say @vals[0];
say @vals[2];
say @vals.head;
say @vals.tail;
```

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
