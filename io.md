# Input/Output

## Read cli input

```raku

print 'Enter your name: ';
my $line = get;

say "Hello $line!";
```

## Read file

```raku
my $fname = 'words.txt';

# old school
my $fh = open $fname, :r;
my $contents = $fh.slurp;
say $contents;

$fh.close;

say '-------------------------';

# via attributes
# file handles automatically closed
$contents = $fname.IO.slurp;
say $contents;

say '-------------------------';

# procedural
$contents = slurp $fname;
say $contents;
```

## Line by line

```raku
my $fname = 'words.txt';

for $fname.IO.lines -> $line {
    say $line;
}

my @lines = $fname.IO.lines;
say @lines;
say "@lines[]";

@lines.say;
```



