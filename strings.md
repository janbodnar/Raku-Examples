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

String concatenation in loops can be inefficient for large texts. The  
join method is more efficient as it allocates memory once rather than  
repeatedly growing strings. For performance-critical applications, use  
join or build arrays first, then join them.  

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

Text parsing breaks structured data into usable components. The CSV parser  
handles comma-separated values with trimming. The key-value parser ignores  
comments and empty lines while extracting configuration data. Regex  
patterns help identify and extract the relevant parts.  

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

String generators create random or patterned strings programmatically.  
The password generator uses `pick` to randomly select characters from  
different character sets. The UUID generator creates hexadecimal  
identifiers with proper formatting for unique identification.  

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

Stream processing handles large texts by processing words one at a time  
rather than loading everything into memory. The word counter removes  
punctuation and normalizes case. The `sort(-*.value)` sorts by frequency  
in descending order for finding most common words.  

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

Advanced string manipulation combines regex and string methods for complex  
transformations. Case conversion functions handle programming conventions.  
The regex `(<[A..Z]>)` matches uppercase letters for camelCase  
conversion. These utilities are essential for code generators and formatters.  

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

Tokenization breaks text into meaningful components for analysis. The  
tokenizer separates words from punctuation while preserving both. The  
`comb` method with regex patterns extracts specific data types like  
numbers and emails. This is fundamental for lexical analysis and parsing.  

## String interpolation with complex expressions

Advanced interpolation techniques with nested expressions and formatting.  

```raku
my @items = <apple banana cherry>;
my %prices = apple => 1.50, banana => 0.75, cherry => 2.25;

say "Shopping list: {@items.map({\"$_ (${%prices{$_}})\"}).join(', ')}";

my $date = DateTime.now;
my $user = 'Alice';
my $count = 42;

say "Hello $user! Today is {$date.month-name} {$date.day}, " ~
    "and you have {$count > 1 ?? 'multiple items' !! 'one item'} waiting.";

my @matrix = [1, 2, 3], [4, 5, 6];
say "Matrix sum: {[+] @matrix.map({[+] $_})}";
```

Complex interpolation combines method calls, conditionals, and nested  
expressions within strings. The `map` method transforms arrays, while  
conditional expressions use the ternary operator. Brackets ensure  
proper evaluation order in complex expressions.  

## String mutation methods

String transformation using in-place modification techniques.  

```raku
my $text = 'Hello World';

# Using method chaining for transformations
my $processed = $text.uc.subst(' ', '_').subst(/L/, 'X', :global);
say $processed;

# Character-level transformations
sub rot13($str) {
    $str.trans('A..Za..z' => 'N..ZA..Mn..za..m');
}

sub caesar-cipher($str, $shift) {
    my $alpha = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
    my $shifted = $alpha.substr($shift) ~ $alpha.substr(0, $shift);
    $str.uc.trans($alpha => $shifted);
}

say rot13('Hello World');
say caesar-cipher('HELLO', 3);
```

String mutation applies multiple transformations in sequence. Method  
chaining enables fluent transformations. The `trans` method performs  
character translation, useful for ciphers and character mapping. These  
techniques create efficient string processing pipelines.  

## String character encoding conversions

Converting between different character encodings and normalization forms.  

```raku
my $text = "Héllo Wörld! 你好世界";

# Different encoding formats
my $utf8 = $text.encode('utf8');
my $utf16 = $text.encode('utf16');
my $latin1 = $text.encode('latin1', :replacement);

say "UTF-8 bytes: {$utf8.elems}";
say "UTF-16 bytes: {$utf16.elems}";
say "Latin-1 bytes: {$latin1.elems}";

# Base64 encoding for data transmission
my $base64 = $utf8.encode('base64');
say "Base64: $base64";

# Handling encoding errors gracefully
sub safe-encode($text, $encoding) {
    try {
        return $text.encode($encoding);
    }
    CATCH {
        return $text.encode($encoding, :replacement);
    }
}

say safe-encode("Unicode: ♠♣♥♦", 'ascii');
```

Character encoding conversion handles international text across different  
systems. The `:replacement` option substitutes invalid characters with  
safe alternatives. Base64 encoding enables safe transmission of binary  
data. Error handling ensures robust text processing.  

## String palindrome checking

Detecting palindromes with various normalization options.  

```raku
sub is-palindrome($str) {
    my $normalized = $str.lc.subst(/\W/, '', :global);
    $normalized eq $normalized.flip;
}

sub is-word-palindrome($str) {
    my @words = $str.lc.words;
    @words eq @words.reverse;
}

sub palindrome-distance($str) {
    my $normalized = $str.lc.subst(/\W/, '', :global);
    my $reversed = $normalized.flip;
    levenshtein($normalized, $reversed);
}

my @tests = 'racecar', 'A man a plan a canal Panama', 
           'race a car', 'hello world', 'Madam Im Adam';

for @tests -> $test {
    say "$test: " ~ (is-palindrome($test) ?? 'palindrome' !! 'not palindrome');
    say "  Distance from palindrome: {palindrome-distance($test)}";
}
```

Palindrome detection requires text normalization to handle punctuation  
and case differences. The `flip` method reverses strings efficiently.  
Word-level palindromes check sentence structure. Distance calculation  
shows how far text is from being palindromic.  

## String acronym generation

Creating acronyms and abbreviations from text input.  

```raku
sub create-acronym($text) {
    $text.words.map(*.substr(0, 1).uc).join;
}

sub smart-acronym($text) {
    my @important-words = $text.lc.words.grep({
        $_ !~~ /^ (a|an|the|of|in|on|at|by|for|to|and|or|but) $/
    });
    @important-words.map(*.substr(0, 1).uc).join;
}

sub abbreviate($text, $max-length = 10) {
    return $text if $text.chars <= $max-length;
    
    my @words = $text.words;
    my $result = @words[0];
    
    for @words[1..*] -> $word {
        my $candidate = $result ~ ' ' ~ $word;
        if $candidate.chars > $max-length {
            $result ~= ' ' ~ $word.substr(0, 1) ~ '.';
        } else {
            $result = $candidate;
        }
    }
    $result;
}

say create-acronym('Artificial Intelligence');
say smart-acronym('The United States of America');
say abbreviate('International Business Machines Corporation', 15);
```

Acronym generation extracts first letters from words to create  
abbreviations. Smart acronyms skip common words like articles and  
prepositions. The abbreviation function balances readability with  
length constraints, gradually shortening words as needed.  

## String masking and obfuscation

Hiding sensitive information while preserving string structure.  

```raku
sub mask-email($email) {
    if $email ~~ /^ (.+) '@' (.+) $/ {
        my $user = $0;
        my $domain = $1;
        my $masked-user = $user.substr(0, 2) ~ '*' x ($user.chars - 2);
        "$masked-user@$domain";
    } else {
        $email;
    }
}

sub mask-credit-card($number) {
    my $digits = $number.subst(/\D/, '', :global);
    return $number unless $digits.chars >= 12;
    
    $digits.substr(0, 4) ~ ' ' ~ 
    '*' x 4 ~ ' ' ~ 
    '*' x 4 ~ ' ' ~ 
    $digits.substr(*-4);
}

sub redact-words($text, @sensitive-words) {
    my $result = $text;
    for @sensitive-words -> $word {
        $result = $result.subst(
            rx:i/« $word »/, 
            '[REDACTED]', 
            :global
        );
    }
    $result;
}

say mask-email('john.doe@example.com');
say mask-credit-card('1234 5678 9012 3456');
say redact-words('The password is secret123', <password secret123>);
```

Data masking protects sensitive information while maintaining string  
structure for testing and logging. Email masking preserves domain  
information. Credit card masking shows only first and last digits.  
Word redaction uses word boundaries to avoid partial matches.  

## String word wrapping

Intelligent text wrapping with various formatting options.  

```raku
sub word-wrap($text, $width = 70) {
    my @words = $text.words;
    my @lines;
    my $current-line = '';
    
    for @words -> $word {
        if ($current-line ~ ' ' ~ $word).chars > $width {
            @lines.push($current-line) if $current-line;
            $current-line = $word;
        } else {
            $current-line = $current-line 
                ?? $current-line ~ ' ' ~ $word 
                !! $word;
        }
    }
    @lines.push($current-line) if $current-line;
    @lines.join("\n");
}

sub justify-text($text, $width = 70) {
    my @lines = word-wrap($text, $width).lines;
    my @justified;
    
    for @lines -> $line {
        my @words = $line.words;
        next unless @words > 1;
        
        my $spaces-needed = $width - [+] @words.map(*.chars);
        my $gaps = @words.elems - 1;
        my $space-per-gap = $spaces-needed div $gaps;
        my $extra-spaces = $spaces-needed % $gaps;
        
        my $justified = @words[0];
        for 1..^@words -> $i {
            $justified ~= ' ' x $space-per-gap;
            $justified ~= ' ' if $i <= $extra-spaces;
            $justified ~= @words[$i];
        }
        @justified.push($justified);
    }
    @justified.join("\n");
}

my $long-text = "This is a very long piece of text that needs to be wrapped " ~
                "at a specific width to ensure proper formatting and readability " ~
                "in various display contexts and output formats.";

say word-wrap($long-text, 30);
say "\n" ~ "=" x 30;
say justify-text($long-text, 30);
```

Word wrapping breaks long text into lines without splitting words.  
The algorithm tracks line length and starts new lines when needed.  
Justified text distributes extra spaces evenly between words for  
professional formatting. This is essential for document generation.  

## String similarity percentage

Advanced string comparison with similarity scoring.  

```raku
sub similarity-percentage($str1, $str2) {
    return 100 if $str1 eq $str2;
    return 0 if $str1.chars == 0 && $str2.chars == 0;
    
    my $max-length = max($str1.chars, $str2.chars);
    my $distance = levenshtein($str1, $str2);
    
    (($max-length - $distance) / $max-length * 100).round(2);
}

sub jaccard-similarity($str1, $str2) {
    my $set1 = set($str1.lc.comb);
    my $set2 = set($str2.lc.comb);
    
    my $intersection = $set1 ∩ $set2;
    my $union = $set1 ∪ $set2;
    
    return 0 if $union.elems == 0;
    ($intersection.elems / $union.elems * 100).round(2);
}

sub soundex($str) {
    my $s = $str.uc.subst(/\W/, '', :global);
    return '' unless $s;
    
    my $first = $s.substr(0, 1);
    $s = $s.subst(/[AEIOUHWY]/, '', :global);
    $s = $s.trans('BFPVCGJKQSXZDTLMNR' => '111122222222334556');
    $s = $s.subst(/(.)$0+/, {$0}, :global);
    
    ($first ~ $s ~ '000').substr(0, 4);
}

my @test-pairs = 
    ['kitten', 'sitting'],
    ['hello', 'hallo'],
    ['raku', 'perl'],
    ['smith', 'smyth'];

for @test-pairs -> [$str1, $str2] {
    say "$str1 vs $str2:";
    say "  Levenshtein similarity: {similarity-percentage($str1, $str2)}%";
    say "  Jaccard similarity: {jaccard-similarity($str1, $str2)}%";
    say "  Soundex: {soundex($str1)} vs {soundex($str2)}";
    say "";
}
```

String similarity algorithms measure text relatedness using different  
approaches. Levenshtein similarity uses edit distance. Jaccard similarity  
compares character sets. Soundex handles phonetic similarity for names.  
These metrics enable fuzzy matching and data deduplication.  