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

Single quotes create literal strings with no interpolation, while double  
quotes allow variable interpolation and escape sequences. The `chars`  
method returns the number of characters in the string, which is useful  
for checking string length including empty strings.  

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

The comma operator outputs arguments separated by spaces, while the tilde  
(~) operator creates a new concatenated string. String interpolation with  
double quotes is often the most readable option. The sprintf function  
provides formatted string creation with placeholders.  

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

Variable interpolation occurs within double quotes by prefixing variables  
with $. Parentheses allow expression evaluation, while curly braces enable  
complex expressions and method calls. Literal dollar signs require  
escaping with backslashes.  

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

The `chars` method counts graphemes (visible characters), `codes` counts  
Unicode code points, and `bytes` counts the encoded byte length. For  
Unicode strings like "café", these values may differ due to multi-byte  
character encoding and combining characters.  

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

The `uc` method converts to uppercase, `lc` to lowercase, `tc` capitalizes  
the first character, `tclc` capitalizes first and lowercases the rest, and  
`wordcase` capitalizes the first letter of each word. These methods handle  
Unicode properly for international text.  

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

The `trim` method removes whitespace from both ends, `trim-leading` removes  
from the start, and `trim-trailing` from the end. You can specify custom  
characters to trim by passing them as arguments. This is essential for  
cleaning user input and parsing data.  

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

The repetition operator `x` creates a new string by repeating the left  
operand the number of times specified by the right operand. This is useful  
for creating separators, padding, patterns, and simple text formatting  
without loops.  

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

String comparison uses `eq` for equality, `ne` for inequality, `lt` for  
less-than, and `gt` for greater-than based on lexicographic ordering. The  
`cmp` operator returns -1, 0, or 1 for less-than, equal, or greater-than  
respectively, useful for custom sorting.  

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

String indexing uses square brackets with zero-based positions. The `*-1`  
notation accesses the last character. Ranges like `0..4` extract  
substrings, while `6..*` gets from position 6 to the end. Multiple indices  
return individual characters at those positions.  

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

The `index` method finds the first occurrence position, `rindex` finds the  
last occurrence, and `contains` returns a boolean. The `starts-with` and  
`ends-with` methods check string boundaries. All methods return Nil or  
False when the substring is not found.  

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

The `subst` method replaces substrings, accepting both literal strings and  
regular expressions. By default, it replaces only the first match. The  
`:global` adverb replaces all occurrences. The original string remains  
unchanged; a new modified string is returned.  

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

The `split` method breaks strings into arrays using delimiters. It accepts  
both literal strings and regular expressions as separators. The `words`  
method is a shortcut for splitting on whitespace, automatically handling  
multiple spaces and other whitespace characters.  

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

The `join` method combines array elements into a single string using a  
specified separator. It can be called as a method on arrays or as a  
function with the separator as the first argument. This is the inverse  
operation of `split`.  

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

The `sprintf` function creates formatted strings using C-style format  
specifiers. `%s` for strings, `%d` for integers, `%f` for floating-point  
numbers. Precision can be specified (%.2f) and zero-padding for integers  
(%03d). Literal percent signs require doubling (%%).  

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

Single quotes preserve literal characters, while double quotes interpret  
escape sequences. Common escapes include `\"` for quotes, `\t` for tabs,  
`\n` for newlines, and `\\` for backslashes. Proper escaping is crucial  
for handling user input and generating output.  

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

Regular expressions use the `~~` smart match operator with patterns between  
forward slashes. Parentheses create capture groups accessible via `$0`,  
`$1`, etc. The `**3` quantifier matches exactly 3 repetitions. Patterns  
return match objects containing matched text and captures.  

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

Complex regex patterns can extract multiple parts of structured text  
simultaneously. Each capture group extracts a specific component, making  
it easy to parse log files, configuration data, and other formatted text.  
The `.+` pattern matches the remainder of the line.  

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

Validation functions use anchored regex patterns (`^` and `$`) to ensure  
complete string matches. The functions return match objects that evaluate  
to True/False in boolean context. This approach provides robust input  
validation for forms and data processing.  

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

String encoding converts text to byte sequences using specific character  
sets. The `encode` method returns a Blob, while `decode` converts back to  
strings. Different encodings (utf8, latin1, ascii) handle character sets  
differently, affecting storage and transmission.  

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

Unicode normalization ensures consistent representation of characters that  
can be encoded multiple ways. NFC (composed) and NFD (decomposed) forms  
handle combining characters differently. Normalization is essential for  
reliable string comparison and searching in international applications.  

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

Template strings allow dynamic content generation by replacing placeholders  
with actual values. This approach separates presentation from data, making  
code more maintainable. More sophisticated templating systems can handle  
conditionals, loops, and formatting options.  

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

Simple hash functions convert strings to numeric values for quick  
comparison and lookup. The `ord` method returns character codes, and  
`comb` splits strings into individual characters. Real applications  
should use cryptographic hash functions for security.  

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

Run-length encoding compresses repeated characters by replacing sequences  
with the character followed by its count. The regex `(.)$0+` matches a  
character followed by one or more of the same character. This simple  
compression works well for data with many repeated characters.  

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

The Levenshtein distance calculates the minimum number of single-character  
edits needed to transform one string into another. This dynamic programming  
algorithm is useful for spell checking, DNA analysis, and approximate  
string matching in search applications.  

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