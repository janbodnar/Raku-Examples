# Raku-Examples
Raku code examples

## String concatenation

```raku
my $w1 = 'old';
my $w2 = 'falcon';

say $w1, " ", $w2;

say $w1 ~ " " ~ $w2;

say "$w1 $w2";

say sprintf "%s %s", $w1, $w2;
```

## Assign vs bind

```raku

# assignment with =
my Int $val = 23;
say $val;

$val = 24;
say $val;

# binding with :=
# cannot be changed
my Str $word := 'falcon';
say $word;

# $word = 'eagle';
# say $word;
```

## Variable interpolation

```raku
my $name = 'John Doe';
my $age = 34;

say "$name is $age years old";

my @words = <sky cup touch war cloud noise>;

# not interpolated
say "@words";

# interpolated
say "@words[]";

my $x = 11;
my $y = 12;

say "\$x + \$y = $($x + $y)";
```

## Pairs

```raku
my $p = Pair.new('key', 'value');

say $p;
say $p.key;
say $p.value;

say $p.WHAT;

my $p2 = 1 => 'book';
say $p2.WHAT;
say $p2;

my $p3 = :one<'book'>;
say $p3.WHAT;
say $p3;

my $p4 = :name('John Doe');
say $p4.WHAT;
say $p4.fmt("%s is %s");
```

## Print n-times 

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

`WHAT` attribute  

```raku
my $x = 12;
my $w = 'cup';

say 'integer' if $x.WHAT === Int;
say 'string' if $w.WHAT === Str;
```

booleans  

```raku

say so True;
say so False;
say so "";
say so " ";
say so "\n";
say so ();
say so (1,);

say 3 > 6;

say '-------------------------';

say [1, 2, 3] eqv [1, 2, 3];
say [1, 2, 3] eqv [1, 2, [3]];
say Any eqv Any;
say 1 eqv 2;
say 1 eqv 1.0;
say (1...100) eqv (1...100);
```

`WHAT` & `name`

```raku

my @vals = (1, 2, 3, 4);
my $word = "falcon";
my $n = 12;

say @vals.WHAT;
say $word.WHAT;
say $n.WHAT;

say @vals.^name;
say $word.^name;
say $n.^name;
```

```raku
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

## Fold/reduce

Fold/reduce is a terminal operation that aggregates list values into a single value.  

```raku

my @numbers = <1 2 3 4 5 6>;

say [+] @numbers;
say [+] 0, |@numbers;

say reduce { $^a + $^b }, 0, |@numbers;
say @numbers.reduce: {$^a + $^b}
```

## Sequencer 

Sequncer/pipe operation passes processed data from one function to another function.  

```raku
my @array = (1, 2, 3, 4, 5);
@array ==> sum() ==> say();

<people cup of mop a earth sum talk bye buy car lot pot smell falcon>
    ==> grep /^...$/
    ==> sort()
    ==> map({ .tc })
    ==> say();
```
