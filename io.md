# Input/Output


```raku

my $fname = 'words.txt';

# old school
my $fh = open $fname, :r;
my $contents = $fh.slurp;
say $contents;

$fh.close;

say '-------------------------';

# via attributes
$contents = $fname.IO.slurp;
say $contents;

say '-------------------------';

# procedural
$contents = slurp $fname;
say $contents;
```
