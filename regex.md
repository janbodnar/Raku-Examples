# Regular Expressions

Regular expressions are powerful pattern matching tools used for text  
processing, validation, and extraction. Raku's regex engine is feature-rich  
and provides both PCRE-compatible patterns and advanced Raku-specific  
features including grammars, named captures, and Unicode support.  

## Basic pattern matching

Simple pattern matching using the smart match operator.  

```raku
my $text = 'Hello World';

say $text ~~ /Hello/;      # 「Hello」
say $text ~~ /hello/;      # Nil (case sensitive)
say $text ~~ /World/;      # 「World」
say $text ~~ /Goodbye/;    # Nil
```

The smart match operator `~~` returns a match object when successful or  
Nil when the pattern doesn't match. The match object contains the matched  
text and can be used in boolean context.  

## Case insensitive matching

Using the :i modifier for case insensitive pattern matching.  

```raku
my $text = 'Hello World';

say $text ~~ /:i hello/;        # 「Hello」
say $text ~~ /:i WORLD/;        # 「World」
say $text ~~ m:i/goodbye/;      # Nil

# Alternative syntax
say $text ~~ m:ignorecase/hello/;    # 「Hello」
```

The `:i` or `:ignorecase` modifier makes the regex match regardless of case.  
This is useful for user input validation and flexible text processing.  

## Character classes

Matching specific sets of characters.  

```raku
my $text = 'abc123XYZ';

say $text ~~ /\d/;         # 「1」 (first digit)
say $text ~~ /\w/;         # 「a」 (first word character)
say $text ~~ /\s/;         # Nil (no whitespace)

say $text ~~ /[a-z]/;      # 「a」 (lowercase letter)
say $text ~~ /[A-Z]/;      # 「X」 (uppercase letter)
say $text ~~ /[0-9]/;      # 「1」 (digit)
```

Character classes match single characters from defined sets. Common classes  
include `\d` for digits, `\w` for word characters, and `\s` for whitespace.  
Custom ranges can be defined with square brackets.  

## Quantifiers

Specifying how many times a pattern should match.  

```raku
my $text = 'aaabbbccc123';

say $text ~~ /a+/;         # 「aaa」 (one or more)
say $text ~~ /a*/;         # 「aaa」 (zero or more)
say $text ~~ /a?/;         # 「a」 (zero or one)
say $text ~~ /\d**3/;      # 「123」 (exactly 3)
say $text ~~ /\d**2..4/;   # 「123」 (2 to 4)
```

Quantifiers control repetition: `+` (one or more), `*` (zero or more),  
`?` (optional), `**n` (exactly n), and `**n..m` (between n and m).  

## Anchors and boundaries

Matching at specific positions in the text.  

```raku
my $text = 'hello world hello';

say $text ~~ /^hello/;     # 「hello」 (start of string)
say $text ~~ /hello$/;     # 「hello」 (end of string)
say $text ~~ /\bhello\b/;  # 「hello」 (word boundary)

my $multiline = "line1\nhello\nline3";
say $multiline ~~ m:g/^hello/;    # No match (not at line start)
say $multiline ~~ m:m/^hello/;    # 「hello」 (with :m modifier)
```

Anchors match positions rather than characters. `^` matches start of string,  
`$` matches end, and `\b` matches word boundaries. The `:m` modifier enables  
multiline mode where `^` and `$` match line boundaries.  

## Capture groups

Extracting parts of matched text using parentheses.  

```raku
my $email = 'user@example.com';

if $email ~~ /(\w+) '@' (\w+) '.' (\w+)/ {
    say "Username: $0";     # user
    say "Domain: $1";       # example
    say "TLD: $2";          # com
}

# Multiple captures
my $date = '2023-12-25';
if $date ~~ /(\d**4) '-' (\d**2) '-' (\d**2)/ {
    say "Year: $0, Month: $1, Day: $2";
}
```

Parentheses create capture groups that extract matched substrings. Captures  
are accessible via `$0`, `$1`, etc., representing the order of opening  
parentheses in the pattern.  

## Named captures

Using named groups for more readable extractions.  

```raku
my $log = '2023-12-25 14:30:15 ERROR Connection failed';

if $log ~~ /:s $<date>=(\d**4 '-' \d**2 '-' \d**2) 
              $<time>=(\d**2 ':' \d**2 ':' \d**2)
              $<level>=(\w+)
              $<msg>=(.+)/ {
    say "Date: $<date>";
    say "Time: $<time>";
    say "Level: $<level>";
    say "Message: $<msg>";
}
```

Named captures use `$<name>=()` syntax for clearer code. The `:s` modifier  
treats whitespace in the pattern as `\s+`, making complex patterns more  
readable while matching flexible whitespace.  

## Alternation

Matching one of several alternatives using the pipe operator.  

```raku
my $response = 'yes';

say $response ~~ /yes | no | maybe/;     # 「yes」
say $response ~~ /^ [yes | no] $/;       # 「yes」

# With captures
my $animal = 'cat';
if $animal ~~ /(cat | dog | bird)/ {
    say "Pet type: $0";
}

my $color = 'red';
say $color ~~ /red | blue | green/;      # 「red」
```

The pipe `|` operator creates alternatives. Square brackets group  
alternatives without creating captures. This is useful for validating  
against known sets of values.  

## Global matching

Finding all matches in a string using the :g modifier.  

```raku
my $text = 'The cat and the dog and the bird';

my @matches = $text ~~ m:g/the/;
say @matches;               # [「the」 「the」 「the」]

my @words = $text ~~ m:g/\w+/;
say @words;                 # All words

# With captures
my $data = 'name=John age=30 city=NYC';
my @pairs = $data ~~ m:g/(\w+) '=' (\w+)/;
for @pairs -> $match {
    say "$match[0] = $match[1]";
}
```

The `:g` modifier finds all matches rather than just the first. Results  
are returned as a list of match objects, each containing any captures.  

## Word boundaries and lookarounds

Advanced position matching and assertions.  

```raku
my $text = 'The cat catches catfish';

say $text ~~ /\bcat\b/;        # 「cat」 (whole word)
say $text ~~ /cat/;            # 「cat」 (first occurrence)

# Positive lookahead
say $text ~~ /cat <?before fish>/;    # 「cat」 (followed by fish)

# Negative lookahead  
say $text ~~ /cat <?!before fish>/;   # 「cat」 (not followed by fish)

# Lookbehind
say $text ~~ /<?after The \s> \w+/;   # 「cat」 (after "The ")
```

Word boundaries `\b` match positions between word and non-word characters.  
Lookarounds assert what comes before or after without consuming characters.  

## Unicode and character properties

Working with Unicode characters and properties.  

```raku
my $text = 'café naïve résumé';

say $text ~~ /\w+/;                    # 「café」
say $text ~~ m:g/<:Letter>+/;          # All letter sequences
say $text ~~ m:g/<:Alphabetic>+/;      # Alphabetic characters

# Specific scripts
my $mixed = 'Hello こんにちは 你好';
say $mixed ~~ m:g/<:Script<Latin>>+/;      # 「Hello」
say $mixed ~~ m:g/<:Script<Hiragana>>+/;   # 「こんにちは」
say $mixed ~~ m:g/<:Script<Han>>+/;        # 「你好」
```

Raku's regex engine has full Unicode support. Character properties like  
`:Letter`, `:Alphabetic`, and script-specific classes enable sophisticated  
text processing for international content.  

## Regex modifiers

Controlling regex behavior with various flags.  

```raku
my $text = "Line 1\nLine 2\nLine 3";

# Single line mode - . matches newlines
say $text ~~ m:s/.+/;              # Matches entire string

# Multiline mode - ^ and $ match line boundaries
say $text ~~ m:m/^Line/;           # 「Line」

# Global and case insensitive
say $text ~~ m:g:i/line/;          # All instances

# Sigspace - treat spaces as \s+
say 'hello   world' ~~ m:s/hello world/;    # 「hello   world」
```

Modifiers change how patterns are interpreted: `:s` (single line), `:m`  
(multiline), `:i` (case insensitive), `:g` (global), and `:sigspace`  
(flexible whitespace matching).  

## String substitution

Replacing matched patterns with new content.  

```raku
my $text = 'Hello World';

say $text.subst(/World/, 'Universe');     # Hello Universe
say $text.subst(/\w+/, 'Hi', :g);         # Hi Hi

# With captures
my $date = '12/25/2023';
say $date.subst(/(\d+) '/' (\d+) '/' (\d+)/, '$2-$0-$1');

# Using a closure
my $numbers = '1 2 3 4 5';
say $numbers.subst(/\d/, -> $match { $match * 2 }, :g);
```

The `subst` method replaces matches with new text. Captures can be referenced  
in replacement strings with `$0`, `$1`, etc. Closures enable computed  
replacements based on matched content.  

## Email validation

Comprehensive email address pattern matching.  

```raku
sub is-valid-email($email) {
    $email ~~ /^ <[a-zA-Z0-9._%+-]>+ '@' <[a-zA-Z0-9.-]>+ '.' <[a-zA-Z]>**2..4 $/
}

sub extract-email-parts($email) {
    if $email ~~ /^ $<user>=(<[a-zA-Z0-9._%+-]>+) '@' $<domain>=(<[a-zA-Z0-9.-]>+) '.' $<tld>=(<[a-zA-Z]>**2..4) $/ {
        return %( user => ~$<user>, domain => ~$<domain>, tld => ~$<tld> );
    }
    return Nil;
}

say is-valid-email('user@example.com');    # True
say extract-email-parts('john.doe@company.org');
```

Email validation requires careful character class definitions and structure  
validation. Named captures make it easy to extract email components for  
further processing.  

## Phone number patterns

Matching various phone number formats.  

```raku
my @phones = (
    '555-123-4567',
    '(555) 123-4567', 
    '555.123.4567',
    '+1-555-123-4567'
);

for @phones -> $phone {
    given $phone {
        when /^ \d**3 '-' \d**3 '-' \d**4 $/ { 
            say "$phone: Simple dash format" 
        }
        when /^ '(' \d**3 ')' \s \d**3 '-' \d**4 $/ { 
            say "$phone: Parentheses format" 
        }
        when /^ \d**3 '.' \d**3 '.' \d**4 $/ { 
            say "$phone: Dot format" 
        }
        when /^ '+1-' \d**3 '-' \d**3 '-' \d**4 $/ { 
            say "$phone: International format" 
        }
    }
}
```

Phone number validation often requires matching multiple formats. Using  
`given-when` with different patterns provides clean format recognition.  

## URL parsing

Extracting components from URLs using regex.  

```raku
sub parse-url($url) {
    if $url ~~ m{^ $<scheme>=(\w+) '://' 
                   [ $<user>=(\w+) ':' $<pass>=(\w+) '@' ]?
                   $<host>=(<[a-zA-Z0-9.-]>+)
                   [ ':' $<port>=(\d+) ]?
                   [ '/' $<path>=(.*) ]?
                   [ '?' $<query>=(.*) ]? $} {
        return %(
            scheme => ~($<scheme> // ''),
            user   => ~($<user> // ''),
            pass   => ~($<pass> // ''),
            host   => ~($<host> // ''),
            port   => ~($<port> // ''),
            path   => ~($<path> // ''),
            query  => ~($<query> // '')
        );
    }
    return Nil;
}

my $url = 'https://user:pass@example.com:8080/path/to/page?param=value';
say parse-url($url);
```

URL parsing demonstrates complex regex with optional groups and named  
captures. The `//` operator provides default values for missing components.  

## Log file parsing

Extracting structured data from log entries.  

```raku
my @logs = (
    '2023-12-25 14:30:15 [INFO] User login successful',
    '2023-12-25 14:35:22 [ERROR] Database connection failed',
    '2023-12-25 14:40:10 [WARN] High memory usage detected'
);

for @logs -> $log {
    if $log ~~ /:s $<timestamp>=(\d**4 '-' \d**2 '-' \d**2 \s \d**2 ':' \d**2 ':' \d**2)
                     '[' $<level>=(\w+) ']'
                     $<message>=(.+)/ {
        say "Time: $<timestamp>";
        say "Level: $<level>";
        say "Message: $<message>";
        say "---";
    }
}
```

Log parsing benefits from the `:s` modifier which treats whitespace in the  
pattern as flexible `\s+` matching. This handles varying amounts of  
whitespace in log formats.  

## Data validation

Using regex for input validation and sanitization.  

```raku
sub validate-username($username) {
    # 3-16 characters, letters, numbers, underscore, hyphen
    $username ~~ /^ <[a-zA-Z0-9_-]>**3..16 $/
}

sub validate-password($password) {
    # At least 8 chars, must contain letter, number, special char
    $password.chars >= 8 &&
    $password ~~ /<[a-zA-Z]>/ &&
    $password ~~ /<[0-9]>/ &&
    $password ~~ /<[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]>/
}

sub sanitize-filename($filename) {
    $filename.subst(/<[<>:"/\\|?*]>/, '_', :g)
}

say validate-username('user123');     # True
say validate-password('Abc123!@');    # True
say sanitize-filename('file<name>.txt');  # file_name_.txt
```

Validation combines multiple regex checks for comprehensive input filtering.  
Character classes define allowed characters, while quantifiers enforce  
length requirements.  

## CSV parsing

Simple CSV data extraction using regex.  

```raku
my $csv-line = 'John,25,"Software Engineer","New York, NY"';

# Simple comma-separated (no quoted fields)
my @simple = 'apple,banana,cherry'.split(',');
say @simple;

# Handle quoted fields with embedded commas
my @fields = $csv-line ~~ m:g/
    [
        '"' ( <-["]>* ) '"'        # Quoted field
        |
        ( <-[",]>* )               # Unquoted field
    ]
    [ ',' | $ ]                    # Comma or end
/;

say @fields.map(*.Str.trim);
```

CSV parsing requires handling quoted fields that may contain commas.  
The alternation pattern matches either quoted or unquoted field content.  

## HTML tag extraction

Extracting tags and content from HTML using regex.  

```raku
my $html = '<p>Hello <strong>world</strong>!</p>';

# Extract all tags
my @tags = $html ~~ m:g/'<' (\w+) [.*?] '>'/;
say "Tags: @tags[]";

# Extract content between specific tags
if $html ~~ /'<p>' (.*?) '</p>'/ {
    say "Paragraph content: $0";
}

# Remove all HTML tags
my $clean = $html.subst(/'<' .*? '>'/, '', :g);
say "Clean text: $clean";

# Extract tag attributes
my $img = '<img src="photo.jpg" alt="A photo" width="300">';
my @attrs = $img ~~ m:g/(\w+) '="' (<-["]>*) '"'/;
for @attrs -> $attr {
    say "$attr[0] = $attr[1]";
}
```

HTML processing with regex requires careful handling of tag structures  
and attributes. Non-greedy quantifiers `*?` prevent over-matching.  

## IP address validation

Validating IPv4 and IPv6 addresses.  

```raku
sub is-ipv4($ip) {
    my $octet = /^ (25[0-5] | 2[0-4]\d | 1\d\d | [1-9]?\d) $/;
    my @parts = $ip.split('.');
    @parts.elems == 4 && @parts.all ~~ $octet;
}

sub is-ipv6($ip) {
    # Simplified IPv6 validation
    $ip ~~ /^ <[0-9a-fA-F:]>+ $/ &&
    $ip.split(':').elems <= 8 &&
    $ip !~~ /::.*::/  # Only one :: allowed
}

say is-ipv4('192.168.1.1');     # True  
say is-ipv4('256.1.1.1');       # False
say is-ipv6('2001:db8::1');     # True
```

IP validation combines range checking for octets with structural validation.  
IPv6 validation is complex and this shows a simplified approach.  

## Credit card validation

Basic credit card number pattern matching.  

```raku
sub get-card-type($number) {
    # Remove spaces and hyphens
    $number = $number.subst(/<[- ]>/, '', :g);
    
    given $number {
        when /^4\d**15$/ { 'Visa' }
        when /^5[1-5]\d**14$/ { 'MasterCard' }
        when /^3[47]\d**13$/ { 'American Express' }
        when /^6011\d**12$/ { 'Discover' }
        default { 'Unknown' }
    }
}

sub luhn-check($number) {
    # Simplified Luhn algorithm check structure
    my @digits = $number.comb.grep(/\d/).reverse;
    my $sum = 0;
    for @digits.kv -> $i, $digit {
        my $val = +$digit;
        $val *= 2 if $i % 2;
        $val = $val div 10 + $val % 10 if $val > 9;
        $sum += $val;
    }
    $sum % 10 == 0;
}

say get-card-type('4532-1234-5678-9012');    # Visa
```

Credit card validation combines pattern recognition with checksum validation.  
Different card types have distinct number patterns and lengths.  

## Date format parsing

Extracting and validating various date formats.  

```raku
sub parse-date($date-str) {
    given $date-str {
        # MM/DD/YYYY
        when /^ (\d**1..2) '/' (\d**1..2) '/' (\d**4) $/ {
            %(month => +$0, day => +$1, year => +$2, format => 'US');
        }
        # DD/MM/YYYY  
        when /^ (\d**1..2) '/' (\d**1..2) '/' (\d**4) $/ {
            %(day => +$0, month => +$1, year => +$2, format => 'EU');
        }
        # YYYY-MM-DD
        when /^ (\d**4) '-' (\d**1..2) '-' (\d**1..2) $/ {
            %(year => +$0, month => +$1, day => +$2, format => 'ISO');
        }
        # Month DD, YYYY
        when /:s (\w+) (\d+) ',' (\d**4)/ {
            %(month => ~$0, day => +$1, year => +$2, format => 'Long');
        }
        default { Nil }
    }
}

say parse-date('12/25/2023');
say parse-date('2023-12-25');  
say parse-date('December 25, 2023');
```

Date parsing handles multiple formats through pattern matching. Different  
regions use different conventions, requiring flexible pattern recognition.  

## SQL injection detection

Identifying potentially dangerous SQL patterns.  

```raku
sub has-sql-injection($input) {
    my @dangerous-patterns = (
        /(?:union|select|insert|update|delete|drop|create|alter)/,
        /[';"]/,                    # Quote chars
        /--/,                       # SQL comments
        /\/\*/,                     # Block comments
        /\bor\b.*?=.*?=/,          # OR with equals
        /\band\b.*?=.*?=/,         # AND with equals
    );
    
    my $lower-input = $input.lc;
    for @dangerous-patterns -> $pattern {
        return True if $lower-input ~~ $pattern;
    }
    return False;
}

sub sanitize-sql-input($input) {
    $input.subst(/'/, "''", :g).subst(/;/, '', :g);
}

say has-sql-injection("'; DROP TABLE users; --");    # True
say has-sql-injection("normal input");               # False
```

SQL injection detection looks for common attack patterns. Real security  
should use parameterized queries rather than regex filtering alone.  

## Configuration file parsing

Extracting key-value pairs from config files.  

```raku
my $config = q:to/EOF/;
# Database configuration
host = localhost
port = 5432
username = admin
password = secret123

# Application settings  
debug = true
timeout = 30
EOF

sub parse-config($content) {
    my %config;
    for $content.lines -> $line {
        # Skip comments and empty lines
        next if $line ~~ /^\s*$/ || $line ~~ /^\s*'#'/;
        
        # Parse key = value pairs
        if $line ~~ /:s^ $<key>=(\w+) '=' $<value>=(\S+) $/ {
            %config{~$<key>} = ~$<value>;
        }
    }
    return %config;
}

my %settings = parse-config($config);
say %settings;
```

Configuration parsing skips comments and empty lines while extracting  
key-value pairs. The `:s` modifier handles flexible whitespace around  
the equals sign.  

## Text markup processing

Converting simple markup to HTML using regex substitution.  

```raku
sub process-markup($text) {
    # Bold text: **text** -> <strong>text</strong>
    $text = $text.subst(/'**' (.*?) '**'/, '<strong>$0</strong>', :g);
    
    # Italic text: *text* -> <em>text</em>  
    $text = $text.subst(/'*' (.*?) '*'/, '<em>$0</em>', :g);
    
    # Links: [text](url) -> <a href="url">text</a>
    $text = $text.subst(/'[' (.*?) ']' '(' (.*?) ')'/, '<a href="$1">$0</a>', :g);
    
    # Code: `code` -> <code>code</code>
    $text = $text.subst(/'`' (.*?) '`'/, '<code>$0</code>', :g);
    
    return $text;
}

my $markup = 'This is **bold** and *italic* with [a link](http://example.com) and `code`.';
say process-markup($markup);
```

Markup processing transforms simple text patterns into HTML. Non-greedy  
quantifiers `*?` ensure proper nesting of markup elements.  

## Password strength analysis

Analyzing password complexity using multiple regex checks.  

```raku
sub analyze-password($password) {
    my %analysis = (
        length => $password.chars,
        has-lower => ?($password ~~ /<[a-z]>/),
        has-upper => ?($password ~~ /<[A-Z]>/),
        has-digit => ?($password ~~ /<[0-9]>/),
        has-special => ?($password ~~ /<[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]>/),
        has-space => ?($password ~~ /\s/),
        repeated-chars => +($password ~~ m:g/(.)$0+/).elems,
        sequential => ?($password ~~ /(abc|bcd|cde|def|123|234|345|456)/),
    );
    
    my $score = 0;
    $score += 2 if %analysis<length> >= 8;
    $score += 1 if %analysis<length> >= 12;
    $score += 1 if %analysis<has-lower>;
    $score += 1 if %analysis<has-upper>;
    $score += 1 if %analysis<has-digit>;
    $score += 2 if %analysis<has-special>;
    $score -= 1 if %analysis<has-space>;
    $score -= 1 if %analysis<repeated-chars> > 0;
    $score -= 2 if %analysis<sequential>;
    
    %analysis<score> = $score;
    return %analysis;
}

say analyze-password('MyP@ssw0rd123');
```

Password analysis combines multiple regex checks to evaluate strength.  
Different patterns contribute to or detract from the overall security score.  

## Social Security Number validation

Validating SSN format and checking for invalid patterns.  

```raku
sub validate-ssn($ssn) {
    # Remove hyphens for processing
    my $clean-ssn = $ssn.subst(/-/, '', :g);
    
    # Must be 9 digits
    return False unless $clean-ssn ~~ /^\d**9$/;
    
    # Invalid patterns
    my @invalid = (
        /^000/,          # Area number 000
        /^666/,          # Area number 666
        /^9/,            # Area number starting with 9
        /000$/,          # Serial number 000
        /00\d$/,         # Group number 00
    );
    
    for @invalid -> $pattern {
        return False if $clean-ssn ~~ $pattern;
    }
    
    # Check for all same digits
    return False if $clean-ssn ~~ /^(\d)$0**8$/;
    
    return True;
}

say validate-ssn('123-45-6789');    # True
say validate-ssn('000-12-3456');    # False
say validate-ssn('111-11-1111');    # False
```

SSN validation requires format checking plus business rule validation.  
Certain number patterns are reserved or invalid even if properly formatted.  

## File path parsing

Extracting components from file paths across different systems.  

```raku
sub parse-path($path) {
    my %result;
    
    given $path {
        # Windows path: C:\folder\file.txt
        when /^ $<drive>=(<[A-Z]>) ':' $<sep>=('\\') $<dirs>=(.*) $<sep> $<file>=(<-[\\]>+) $/ {
            %result = (
                type => 'Windows',
                drive => ~$<drive>,
                dirs => ~$<dirs> ?? ~$<dirs>.split('\\') !! (),
                filename => ~$<file>
            );
        }
        # Unix path: /home/user/file.txt
        when /^ $<sep>=('/') $<dirs>=(.*) $<sep> $<file>=(<-[/]>+) $/ {
            %result = (
                type => 'Unix', 
                dirs => ~$<dirs> ?? ~$<dirs>.split('/') !! (),
                filename => ~$<file>
            );
        }
        # Relative path: folder/file.txt
        when /^ $<dirs>=(.*) '/' $<file>=(<-[/]>+) $/ {
            %result = (
                type => 'Relative',
                dirs => ~$<dirs> ?? ~$<dirs>.split('/') !! (),
                filename => ~$<file>
            );
        }
        default {
            %result = (type => 'Unknown', filename => $path);
        }
    }
    
    # Extract file extension
    if %result<filename> ~~ /^ $<name>=(.*) '.' $<ext>=(\w+) $/ {
        %result<basename> = ~$<name>;
        %result<extension> = ~$<ext>;
    }
    
    return %result;
}

say parse-path('C:\\Users\\john\\document.pdf');
say parse-path('/home/user/script.raku');
```

Path parsing handles different operating system conventions while extracting  
useful components like directory structure, filename, and extension.  

## Advanced text processing

Complex text transformations using regex and grammars.  

```raku
# Convert camelCase to snake_case
sub camel-to-snake($text) {
    $text.subst(/(<[a-z]>)(<[A-Z]>)/, '$0_$1', :g).lc;
}

# Extract quoted strings while handling escapes
sub extract-quotes($text) {
    my @quotes = $text ~~ m:g/
        '"'                    # Opening quote
        (                      # Capture group
            [                  # Non-capturing group
                \\"            # Escaped quote
                |              # OR
                <-["]>         # Any non-quote character
            ]*                 # Zero or more
        )
        '"'                    # Closing quote
    /;
    return @quotes.map(*.Str);
}

# Word frequency counter with regex filtering
sub word-frequency($text) {
    my @words = $text.lc ~~ m:g/\w+/;
    my %freq;
    for @words -> $word {
        %freq{$word.Str}++;
    }
    return %freq.sort(*.value).reverse;
}

say camel-to-snake('getUserName');
say extract-quotes('He said "Hello \"world\"" to me');
say word-frequency('The quick brown fox jumps over the lazy dog');
```

Advanced text processing combines multiple regex techniques for complex  
transformations. These examples show practical applications for data  
processing and text analysis tasks.  

## Multiline text processing

Working with multiline strings and paragraph extraction.  

```raku
my $document = q:to/EOF/;
Title: Introduction to Raku
Author: John Smith
Date: 2023-12-25

This is the first paragraph of the document.
It contains multiple lines of text.

This is the second paragraph.
It also spans multiple lines.

References:
1. Raku documentation
2. Programming examples
EOF

# Extract metadata
if $document ~~ /:s^ 'Title:' $<title>=(.+) $/ {
    say "Title: $<title>";
}

# Extract paragraphs
my @paragraphs = $document ~~ m:g/
    ^^                          # Start of line
    (<-[\n]>+)                  # First line of paragraph
    [                           # Optional additional lines
        \n (<-[\n]>+)           # Non-empty lines
    ]*
    \n?                         # Optional trailing newline
    (?= \n | $)                 # Followed by blank line or end
/;

for @paragraphs -> $para {
    say "Paragraph: " ~ $para.Str.subst(/\n/, ' ', :g);
}
```

Multiline processing requires careful handling of line boundaries and  
paragraph separation. The `^^` anchor matches line starts, while lookaheads  
help identify paragraph boundaries.  

## Pattern replacement with callbacks

Dynamic text transformation using callback functions.  

```raku
my $template = 'Hello ${name}, today is ${date} and temperature is ${temp}°C';

sub process-template($text, %vars) {
    $text.subst(/
        '${' 
        $<var>=(\w+) 
        '}'
    /, -> $match {
        my $var = ~$<var>;
        %vars{$var} // "UNDEFINED($var)";
    }, :g);
}

my %data = (
    name => 'Alice',
    date => '2023-12-25', 
    temp => '22'
);

say process-template($template, %data);

# Function-based replacement
my $math = '2 + 3 = ? and 5 * 4 = ?';
say $math.subst(
    /(\d+) \s* (<[+\-*/]>) \s* (\d+) \s* '=' \s* '?'/,
    -> $match {
        my ($a, $op, $b) = $match[0, 1, 2];
        my $result = do given $op {
            when '+' { +$a + +$b }
            when '-' { +$a - +$b }
            when '*' { +$a * +$b }
            when '/' { +$a / +$b }
        };
        "$a $op $b = $result";
    }, :g
);
```

Callback-based replacements enable dynamic content generation. The callback  
receives the match object and can perform complex computations to generate  
replacement text.  

## Regular expression compilation

Pre-compiling regex patterns for better performance.  

```raku
# Compile patterns for reuse
my $email-pattern = regex {
    ^ 
    $<user>=(<[a-zA-Z0-9._%+-]>+)
    '@'
    $<domain>=(<[a-zA-Z0-9.-]>+)
    '.'
    $<tld>=(<[a-zA-Z]>**2..4)
    $
};

my $phone-pattern = regex {
    ^
    [ '(' \d**3 ')' | \d**3 ]
    <[- ]>?
    \d**3
    <[- ]>?
    \d**4
    $
};

# Use compiled patterns
my @test-emails = ('user@domain.com', 'invalid.email', 'test@example.org');
my @test-phones = ('555-123-4567', '(555) 123-4567', 'not-a-phone');

for @test-emails -> $email {
    if $email ~~ $email-pattern {
        say "Valid email: $email (user: $<user>, domain: $<domain>)";
    } else {
        say "Invalid email: $email";
    }
}

for @test-phones -> $phone {
    say $phone ~~ $phone-pattern ?? "Valid phone: $phone" !! "Invalid phone: $phone";
}
```

Compiled regex patterns improve performance when the same pattern is used  
repeatedly. The `regex` declarator creates reusable pattern objects with  
named captures.  

## Text tokenization

Breaking text into meaningful tokens using regex patterns.  

```raku
sub tokenize($text) {
    my @tokens = $text ~~ m:g/
        [
            $<string>=( '"' <-["]>* '"' )           # String literals
            |
            $<number>=( \d+ ['.' \d+]? )            # Numbers
            |
            $<identifier>=( <[a-zA-Z_]> \w* )       # Identifiers
            |
            $<operator>=( <[+\-*/=<>!&|]>+ )        # Operators
            |
            $<punctuation>=( <[(){}[\],;]> )        # Punctuation
            |
            $<whitespace>=( \s+ )                   # Whitespace
        ]
    /;
    
    my @result;
    for @tokens -> $token {
        for $token.hash.kv -> $type, $value {
            next unless $value;
            @result.push: %(type => $type, value => ~$value);
        }
    }
    
    return @result.grep(*.{ type ne 'whitespace' });
}

my $code = 'x = "hello" + 42';
my @tokens = tokenize($code);

for @tokens -> $token {
    say "$token<type>: '$token<value>'";
}

# Language-specific tokenizer
sub tokenize-sql($sql) {
    my @keywords = <SELECT FROM WHERE INSERT UPDATE DELETE CREATE TABLE>;
    my $keyword-pattern = @keywords.join('|');
    
    my @tokens = $sql ~~ m:g:i/
        [
            $<keyword>=( <{$keyword-pattern}> ) \b
            |
            $<identifier>=( <[a-zA-Z_]> \w* )
            |
            $<string>=( "'" <-[']>* "'" )
            |
            $<number>=( \d+ ['.' \d+]? )
            |
            $<operator>=( <[=<>!]>+ | LIKE | IN )
            |
            $<punctuation>=( <[(),.;]> )
        ]
    /;
    
    return @tokens.map({ .hash.first(*.value).kv }).map(-> ($type, $value) { 
        %(type => $type, value => ~$value) 
    });
}

say tokenize-sql('SELECT name FROM users WHERE age > 25');
```

Tokenization breaks input into meaningful units for parsing. Different  
token types can be handled separately, enabling sophisticated text analysis  
and language processing.  

## Grammar-like patterns

Using Raku's grammar features within regex for complex parsing.  

```raku
# Simple arithmetic expression parser
my regex number { \d+ ['.' \d+]? }
my regex operator { <[+\-*/]> }
my regex factor { <number> | '(' <expression> ')' }
my regex term { <factor> [ <operator> <factor> ]* }
my regex expression { <term> [ <[+\-]> <term> ]* }

sub evaluate-expression($expr-str) {
    if $expr-str ~~ /<expression>/ {
        say "Valid expression: $expr-str";
        # Simplified evaluation would go here
        return True;
    } else {
        say "Invalid expression: $expr-str";
        return False;
    }
}

# JSON-like structure parsing
my regex string-literal { '"' <-["]>* '"' }
my regex number-literal { '-'? \d+ ['.' \d+]? }
my regex boolean-literal { 'true' | 'false' }
my regex null-literal { 'null' }

my regex value {
    <string-literal> | <number-literal> | 
    <boolean-literal> | <null-literal> |
    <array> | <object>
}

my regex array { '[' \s* [ <value> [ \s* ',' \s* <value> ]* ]? \s* ']' }
my regex pair { <string-literal> \s* ':' \s* <value> }
my regex object { '{' \s* [ <pair> [ \s* ',' \s* <pair> ]* ]? \s* '}' }

my @test-expressions = ('2 + 3', '(4 * 5) + 2', '2 + ', 'invalid');
for @test-expressions -> $expr {
    evaluate-expression($expr);
}

# Test JSON parsing
my @json-tests = (
    '{"name": "John", "age": 30}',
    '[1, 2, "three"]',
    '{"valid": true, "count": null}',
    '{"incomplete": }'
);

for @json-tests -> $json {
    say $json ~~ /<object>|<array>/ ?? "Valid JSON: $json" !! "Invalid JSON: $json";
}
```

Grammar-like patterns enable recursive parsing of complex structures.  
Forward declarations and recursive rules handle nested expressions and  
data structures like JSON or mathematical expressions.  

## Context-sensitive matching

Using state and context for advanced pattern matching.  

```raku
# State-based parsing for nested comments
sub parse-nested-comments($text) {
    my $depth = 0;
    my $pos = 0;
    my @comments;
    my $start;
    
    while $pos < $text.chars {
        if $text.substr($pos, 2) eq '/*' {
            $start = $pos if $depth == 0;
            $depth++;
            $pos += 2;
        } elsif $text.substr($pos, 2) eq '*/' && $depth > 0 {
            $depth--;
            if $depth == 0 {
                @comments.push: $text.substr($start, $pos - $start + 2);
            }
            $pos += 2;
        } else {
            $pos++;
        }
    }
    
    return @comments;
}

# Regex with conditional matching
sub parse-quoted-strings($text) {
    my @strings = $text ~~ m:g/
        (                           # Capture group
            ["']                    # Opening quote
            (                       # String content
                [
                    \\"  |  \\'     # Escaped quotes
                    |
                    <-["']>         # Non-quote characters
                ]*
            )
            ["']                    # Closing quote
        )
        <?{ $0.substr(0,1) eq $0.substr(*-1,1) }>  # Assertion: matching quotes
    /;
    
    return @strings.map(*.Str);
}

# Balanced bracket matching
sub find-balanced-brackets($text) {
    my regex balanced-brackets {
        '['
        [
            <balanced-brackets>     # Recursive call
            |
            <-[\[\]]>              # Non-bracket characters
        ]*
        ']'
    }
    
    my @matches = $text ~~ m:g/<balanced-brackets>/;
    return @matches.map(*.Str);
}

my $code-with-comments = q:to/EOF/;
This is /* a comment /* with nesting */ inside */ normal text.
/* Another /* deeply /* nested */ comment */ block */
EOF

my $mixed-quotes = q{'single' "double" 'don\'t mix' "quotes"};

my $nested-brackets = 'array[index[sub]] and [1, [2, [3, 4]], 5]';

say "Nested comments:";
.say for parse-nested-comments($code-with-comments);

say "\nQuoted strings:";
.say for parse-quoted-strings($mixed-quotes);

say "\nBalanced brackets:";
.say for find-balanced-brackets($nested-brackets);
```

Context-sensitive matching handles complex parsing scenarios where simple  
patterns aren't sufficient. State machines, assertions, and recursive  
patterns enable parsing of nested structures and context-dependent syntax.  