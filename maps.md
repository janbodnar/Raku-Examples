# Maps

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
