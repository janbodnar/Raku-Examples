# Maps

Maps are immutable hashes.  

```raku
my %h = Map.new(1, 'sky', 2, 'cloud', 
    3, 'cup', 4, 'war', 5, 'water');

say %h{1};
say %h{2};
say %h{3, 4};

say %h.elems;
say %h.keys;
say %h.values;

say %h.gist;
```

## Sorting 

```raku
#!/usr/bin/rakudo

my %h is Map = 1, 'sky', 2, 'cloud', 
    3, 'cup', 4, 'war', 5, 'water', 6, 'atom';


say %h.sort;
say %h.sort({$^b cmp $^a});

say '-----------------------';

say %h.sort: *.values;
say %h.sort(*.values).reverse;
```

## Filter with grep

```raku
#!/usr/bin/rakudo

my %h is Map = 1, 'sky', 2, 'cloud', 
    3, 'cup', 4, 'war', 5, 'water', 6, 'atom';

say %h.grep({ $_.value.starts-with('w') });
```
