# Array 

Arrays are mutable data structures.  They are created with the `@` sigil.  

`elems` returns the size of the list  

```raku
my @vals = 1, 2, 3, 4, 5;
say @vals.elems;
say @vals;
```

Indexing & head & tail  

```raku
my @vals = (1, 2, 3, 4, 5);
say @vals[0];
say @vals[2];
say @vals.head;
say @vals.tail;
```
