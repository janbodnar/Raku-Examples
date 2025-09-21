# Strings

Strings are sequences of characters used to represent text data.  

## String literals

Basic string creation with single and double quotes.  

```raku
my $word = 'falcon';
my $sentence = "The quick brown fox";
my $empty = '';

say $word;
say $sentence;
say $empty.chars;
```

## String concatenation

Different ways to combine strings together.  

```raku
my $w1 = 'old';
my $w2 = 'falcon';

say $w1, " ", $w2;
say $w1 ~ " " ~ $w2;
say "$w1 $w2";
say sprintf "%s %s", $w1, $w2;
```

## String interpolation

Embedding variables and expressions within strings.  

```raku
my $name = 'John Doe';
my $age = 34;

say "$name is $age years old";

my $x = 11;
my $y = 12;

say "\$x + \$y = $($x + $y)";
say "Square of $x is {$x ** 2}";
```

## String length

Getting the number of characters in a string.  

```raku
my $text = 'Hello, World!';

say $text.chars;
say $text.codes;
say $text.bytes;

my $unicode = "café";
say "$unicode has {$unicode.chars} characters";
```

## String case conversion

Converting between uppercase and lowercase.  

```raku
my $text = 'Hello World';

say $text.uc;
say $text.lc;
say $text.tc;
say $text.tclc;
say $text.wordcase;
```

## String trimming

Removing whitespace from string edges.  

```raku
my $text = '   hello world   ';

say "'$text'";
say "'{$text.trim}'";
say "'{$text.trim-leading}'";
say "'{$text.trim-trailing}'";

my $custom = '...hello...';
say "'{$custom.trim('.')}'";
```

## String repetition

Creating repeated string patterns.  

```raku
my $word = 'falcon';

say $word x 3;
say "-" x 20;
say "abc" x 5;

my $pattern = "* ";
say $pattern x 10;
```

## String comparison

Comparing strings for equality and order.  

```raku
my $str1 = 'apple';
my $str2 = 'banana';
my $str3 = 'apple';

say $str1 eq $str3;
say $str1 ne $str2;
say $str1 lt $str2;
say $str1 gt $str2;
say $str1 cmp $str2;
```

## String indexing and slicing

Accessing individual characters and substrings.  

```raku
my $text = 'Hello World';

say $text[0];
say $text[6];
say $text[*-1];
say $text[0..4];
say $text[6..*];
say $text[1,3,5];
```

## String searching

Finding substrings within strings.  

```raku
my $text = 'The quick brown fox jumps over the lazy dog';

say $text.index('quick');
say $text.rindex('the');
say $text.contains('fox');
say $text.starts-with('The');
say $text.ends-with('dog');
```

## String replacement

Replacing parts of strings with new content.  

```raku
my $text = 'Hello World';

say $text.subst('World', 'Universe');
say $text.subst(/o/, 'O');
say $text.subst(/o/, 'O', :global);

my $numbers = '1-2-3-4-5';
say $numbers.subst('-', '_', :global);
```

## String splitting

Breaking strings into arrays based on delimiters.  

```raku
my $text = 'apple,banana,cherry,date';
my $sentence = 'The quick brown fox';

say $text.split(',');
say $sentence.split(' ');
say $sentence.words;

my $path = '/home/user/documents/file.txt';
say $path.split('/');
```

## String joining

Combining array elements into a single string.  

```raku
my @words = <apple banana cherry>;
my @numbers = 1, 2, 3, 4, 5;

say @words.join(' ');
say @words.join(', ');
say @numbers.join('-');

my $result = join ' | ', @words;
say $result;
```

## String formatting

Creating formatted strings with sprintf-style placeholders.  

```raku
my $name = 'Alice';
my $score = 95.67;
my $count = 42;

say sprintf "Name: %s", $name;
say sprintf "Score: %.2f", $score;
say sprintf "Count: %03d", $count;
say sprintf "%s scored %.1f%% (%d points)", $name, $score, $count;
```

## String escaping

Handling special characters in strings.  

```raku
my $quoted = 'He said "Hello"';
my $escaped = "He said \"Hello\"";
my $tab = "Column1\tColumn2\tColumn3";
my $newline = "Line 1\nLine 2\nLine 3";

say $quoted;
say $escaped;
say $tab;
say $newline;
```

## Regular expressions

Pattern matching and extraction using regex.  

```raku
my $text = 'Contact: john@example.com or call 555-1234';

say $text ~~ /\w+ '@' \w+ '.' \w+/;
say $text ~~ /\d**3 '-' \d**4/;

if $text ~~ /(\w+) '@' (\w+) '.' (\w+)/ {
    say "User: $0, Domain: $1.$2";
}
```

## String extraction with regex

Extracting specific patterns from strings.  

```raku
my $log = '2023-12-25 14:30:15 ERROR Database connection failed';

if $log ~~ /(\d**4 '-' \d**2 '-' \d**2) ' ' (\d**2 ':' \d**2 ':' \d**2) ' ' (\w+) ' ' (.+)/ {
    say "Date: $0";
    say "Time: $1";
    say "Level: $2";
    say "Message: $3";
}
```

## String validation

Checking if strings match specific patterns.  

```raku
sub is-email($email) {
    $email ~~ /^ \w+ '@' \w+ '.' \w+ $/
}

sub is-phone($phone) {
    $phone ~~ /^ \d**3 '-' \d**3 '-' \d**4 $/
}

say is-email('user@domain.com');
say is-email('invalid-email');
say is-phone('555-123-4567');
say is-phone('not-a-phone');
```

## String encoding

Working with different character encodings.  

```raku
my $text = "café";
my $encoded = $text.encode('utf8');
my $decoded = $encoded.decode('utf8');

say $text;
say $encoded;
say $decoded;

my $latin1 = $text.encode('latin1');
say $latin1;
```

## String normalization

Normalizing Unicode strings for comparison.  

```raku
my $str1 = "café";
my $str2 = "cafe\x[0301]";  # e with combining acute accent

say $str1 eq $str2;
say $str1.NFC eq $str2.NFC;
say $str1.NFD eq $str2.NFD;

say $str1.chars;
say $str2.chars;
```

## String templates

Creating template strings with placeholders.  

```raku
sub template($template, %values) {
    my $result = $template;
    for %values.kv -> $key, $value {
        $result = $result.subst("\{$key\}", $value, :global);
    }
    $result;
}

my $tmpl = "Hello {name}, your score is {score}!";
my %data = name => 'Alice', score => 95;

say template($tmpl, %data);
```

## String hashing

Computing hash values for strings.  

```raku
my $text = "Hello, World!";

say $text.chars;
say $text.comb.join('-');

my $short = "Hi";
my $long = "This is a much longer string";

# Simple hash using character codes
sub simple-hash($str) {
    my $hash = 0;
    for $str.comb -> $char {
        $hash += $char.ord;
    }
    $hash;
}

say simple-hash($short);
say simple-hash($long);
```

## String compression

Basic string compression and decompression.  

```raku
sub compress-runs($str) {
    $str.subst(/(.)$0+/, -> $match { $match[0] ~ $match.chars }, :global);
}

sub expand-runs($str) {
    $str.subst(/(\w)(\d+)/, -> $match { $match[0] x $match[1] }, :global);
}

my $text = "aaabbccccdddd";
my $compressed = compress-runs($text);
say $compressed;
say expand-runs($compressed);
```

## String metrics

Calculating various string distance metrics.  

```raku
sub levenshtein($s1, $s2) {
    return $s2.chars if $s1.chars == 0;
    return $s1.chars if $s2.chars == 0;
    
    my @matrix;
    for 0..$s1.chars -> $i {
        @matrix[$i][0] = $i;
    }
    for 0..$s2.chars -> $j {
        @matrix[0][$j] = $j;
    }
    
    for 1..$s1.chars -> $i {
        for 1..$s2.chars -> $j {
            my $cost = $s1.substr($i-1, 1) eq $s2.substr($j-1, 1) ?? 0 !! 1;
            @matrix[$i][$j] = min(
                @matrix[$i-1][$j] + 1,
                @matrix[$i][$j-1] + 1,
                @matrix[$i-1][$j-1] + $cost
            );
        }
    }
    
    @matrix[$s1.chars][$s2.chars];
}

say levenshtein('kitten', 'sitting');
say levenshtein('hello', 'world');
```

## String building

Efficient string construction for large texts.  

```raku
my $builder = '';

for 1..1000 -> $i {
    $builder ~= "Line $i\n";
}

say $builder.lines.elems;
say $builder.chars;

# More efficient with join
my @lines = (1..1000).map: "Line $_";
my $result = @lines.join("\n");
say $result.lines.elems;
```

## String parsing

Parsing structured text data.  

```raku
sub parse-csv($line) {
    $line.split(',').map(*.trim);
}

sub parse-key-value($text) {
    my %result;
    for $text.lines -> $line {
        next if $line.trim eq '';
        next if $line.starts-with('#');
        
        if $line ~~ /(\w+) '=' (.+)/ {
            %result{$0} = $1.trim;
        }
    }
    %result;
}

my $csv = "name, age, city";
say parse-csv($csv);

my $config = q:to/END/;
# Configuration file
name=John Doe
age=30
city=New York
END

say parse-key-value($config);
```

## String generators

Creating strings with specific patterns.  

```raku
sub generate-password($length = 12) {
    my @chars = 'a'..'z', 'A'..'Z', '0'..'9', <! @ # $ % ^ &>;
    @chars.pick($length).join;
}

sub generate-uuid() {
    my @hex = '0'..'9', 'a'..'f';
    my $uuid = @hex.pick(8).join ~ '-' ~
               @hex.pick(4).join ~ '-' ~
               @hex.pick(4).join ~ '-' ~
               @hex.pick(4).join ~ '-' ~
               @hex.pick(12).join;
    $uuid;
}

say generate-password();
say generate-password(16);
say generate-uuid();
```

## String stream processing

Processing large strings efficiently.  

```raku
sub count-words($text) {
    my %counts;
    for $text.lc.words -> $word {
        $word = $word.subst(/\W/, '', :global);
        next if $word eq '';
        %counts{$word}++;
    }
    %counts;
}

sub most-common-words($text, $n = 5) {
    my %counts = count-words($text);
    %counts.sort(-*.value).head($n);
}

my $sample = q:to/END/;
The quick brown fox jumps over the lazy dog.
The dog was really lazy, but the fox was quick.
Brown foxes are quick, and lazy dogs are slow.
END

say count-words($sample);
say most-common-words($sample, 3);
```

## Advanced string manipulation

Complex string transformations and operations.  

```raku
sub camel-to-snake($str) {
    $str.subst(/(<[A..Z]>)/, -> $match { '_' ~ $match.lc }, :global)
        .subst(/^_/, '');
}

sub snake-to-camel($str) {
    $str.split('_').map({ .tc }).join.subst(/^(.)/, -> $match { $match.lc });
}

sub title-case($str) {
    $str.split(/\s+/).map(*.tc).join(' ');
}

sub reverse-words($str) {
    $str.words.reverse.join(' ');
}

say camel-to-snake('CamelCaseString');
say snake-to-camel('snake_case_string');
say title-case('the quick brown fox');
say reverse-words('The quick brown fox');
```

## String tokenization

Breaking text into meaningful tokens and components.  

```raku
sub tokenize($text) {
    my @tokens;
    my $current = '';
    
    for $text.comb -> $char {
        if $char ~~ /\w/ {
            $current ~= $char;
        } else {
            @tokens.push($current) if $current ne '';
            @tokens.push($char) if $char !~~ /\s/;
            $current = '';
        }
    }
    @tokens.push($current) if $current ne '';
    @tokens;
}

sub extract-numbers($text) {
    $text.comb(/\d+/).map(+*);
}

sub extract-emails($text) {
    $text.comb(/\w+ '@' \w+ ['.' \w+]+/);
}

my $sample = "Contact John (age: 25) at john@example.com or call 555-1234.";
say tokenize($sample);
say extract-numbers($sample);
say extract-emails($sample);
```