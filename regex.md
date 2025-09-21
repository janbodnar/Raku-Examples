# Regular Expressions

Regular expressions (regex) provide powerful pattern matching and text  
processing capabilities in Raku. They allow you to search, extract,  
validate, and manipulate text based on patterns. Raku's regex engine  
is particularly expressive and includes advanced features like named  
captures, grammars, and seamless integration with the language syntax.  

## Basic pattern matching

Simple pattern matching using the smart match operator.  

```raku
my $text = 'Contact: john@example.com or call 555-1234';

say $text ~~ /\w+ '@' \w+ '.' \w+/;
say $text ~~ /\d**3 '-' \d**4/;

if $text ~~ /(.*) '@' (.*)/ {
    say "Found email with user part: $0";
    say "Domain part: $1";
}
```

The smart match operator `~~` tests if text matches a pattern. Patterns  
are written between forward slashes. Parentheses create capture groups  
that store matched text in numbered variables `$0`, `$1`, etc.  

## Character classes

Using predefined and custom character classes for matching.  

```raku
my $text = 'Hello123World';

say $text ~~ /\w+/;      # Word characters
say $text ~~ /\d+/;      # Digits
say $text ~~ /\s+/;      # Whitespace

# Custom character classes
say $text ~~ /<[aeiou]>+/;           # Vowels
say $text ~~ /<[0..9]>+/;            # Digits (custom range)
say $text ~~ /<[a..z A..Z]>+/;       # Letters with space

my $input = 'abc123XYZ';
say $input ~~ /<-[0..9]>+/;          # Non-digits
```

Character classes match specific types of characters. Use `\w` for word  
characters, `\d` for digits, `\s` for whitespace. Custom classes use  
`<[...]>` syntax. Negate with `<-[...]>` to match everything except.  

## Quantifiers

Controlling how many times patterns should match.  

```raku
my $text = 'aaa bb cccc d';

say $text ~~ /a+/;       # One or more 'a'
say $text ~~ /a*/;       # Zero or more 'a' 
say $text ~~ /a?/;       # Zero or one 'a'
say $text ~~ /a**3/;     # Exactly 3 'a'
say $text ~~ /a**2..4/;  # 2 to 4 'a'

# Real-world examples
my $phone = '555-123-4567';
say $phone ~~ /\d**3 '-' \d**3 '-' \d**4/;

my $code = 'ABC123';
say $code ~~ /\w**3 \d**3/;
```

Quantifiers specify repetition: `+` (one or more), `*` (zero or more),  
`?` (optional), `**n` (exactly n), `**n..m` (between n and m times).  
Use `**` for exact counts and ranges.  

## Anchors and boundaries

Matching at specific positions in text.  

```raku
my $text = 'The quick brown fox';

say $text ~~ /^The/;         # Start of string
say $text ~~ /fox$/;         # End of string
say $text ~~ /<<quick>>/;    # Word boundaries

my $email = 'user@domain.com';
say $email ~~ /^ \w+ '@' \w+ '.' \w+ $/;  # Complete match

my $words = 'cat caterpillar';
say $words ~~ /<<cat>>/;     # Matches 'cat' but not 'caterpillar'
```

Anchors ensure patterns match at specific positions. `^` matches start  
of string, `$` matches end. `<<` and `>>` are word boundaries that  
prevent partial word matches.  

## Capture groups

Extracting matched portions for processing.  

```raku
my $log = '2023-12-25 14:30:15 ERROR Database connection failed';

if $log ~~ /(\d**4 '-' \d**2 '-' \d**2) ' ' (\d**2 ':' \d**2 ':' \d**2) ' ' (\w+) ' ' (.+)/ {
    say "Date: $0";
    say "Time: $1";
    say "Level: $2";
    say "Message: $3";
}

# Named captures
if $log ~~ /$<date>=(\d**4 '-' \d**2 '-' \d**2) ' ' $<time>=(\d**2 ':' \d**2 ':' \d**2)/ {
    say "Date: $<date>";
    say "Time: $<time>";
}
```

Parentheses create numbered capture groups `$0`, `$1`, etc. Named  
captures use `$<name>=()` syntax and can be accessed as `$<name>`.  
This makes extracted data easier to work with.  

## Pattern validation

Using regex to validate input formats.  

```raku
sub is-email($email) {
    $email ~~ /^ \w+ '@' \w+ '.' \w+ $/
}

sub is-phone($phone) {
    $phone ~~ /^ \d**3 '-' \d**3 '-' \d**4 $/
}

sub is-ip-address($ip) {
    $ip ~~ /^ (\d**1..3) '.' (\d**1..3) '.' (\d**1..3) '.' (\d**1..3) $/
    and all($0, $1, $2, $3) <= 255
}

say is-email('user@domain.com');      # True
say is-email('invalid-email');        # False
say is-phone('555-123-4567');         # True
say is-phone('not-a-phone');          # False
say is-ip-address('192.168.1.1');     # True
```

Validation functions return boolean results. Combine pattern matching  
with additional logic checks for complex validation rules like IP  
address ranges.  

## Text extraction

Finding and extracting specific patterns from text.  

```raku
sub extract-numbers($text) {
    $text.comb(/\d+/).map(+*);
}

sub extract-emails($text) {
    $text.comb(/\w+ '@' \w+ ['.' \w+]+/);
}

sub extract-urls($text) {
    $text.comb(/'http' 's'? '://' \S+/);
}

my $sample = "Contact John (age: 25) at john@example.com or visit https://example.com";
say extract-numbers($sample);     # (25)
say extract-emails($sample);      # (john@example.com)
say extract-urls($sample);        # (https://example.com)
```

Use `.comb()` with regex patterns to extract all matches from text.  
The `map(+*)` converts string numbers to numeric values. Square  
brackets group alternatives without creating captures.  

## Pattern replacement

Replacing matched patterns with new content.  

```raku
my $text = 'Hello World';

say $text.subst(/World/, 'Universe');           # Hello Universe
say $text.subst(/o/, 'O');                      # HellO World
say $text.subst(/o/, 'O', :global);             # HellO WOrld

# Using captures in replacement
my $names = 'John Doe, Jane Smith';
say $names.subst(/(\w+) ' ' (\w+)/, '$1, $0', :global);  # Doe, John, Smith, Jane

# Complex replacements
my $phone = '(555) 123-4567';
say $phone.subst(/'(' (\d**3) ')' ' ' (\d**3) '-' (\d**4)/, '$0-$1-$2');
```

Use `.subst()` to replace matched patterns. Include `:global` to replace  
all matches. Reference captures with `$0`, `$1` in replacement text.  
This enables sophisticated text transformations.  

## Advanced patterns

Complex regex patterns for sophisticated matching.  

```raku
# Balanced parentheses (simple version)
my $expr = '(a + (b * c))';
if $expr ~~ /^ '(' <balanced> ')' $/ {
    say "Balanced expression";
}

# IPv4 address with proper validation
sub valid-ipv4($ip) {
    $ip ~~ /^ $<oct1>=(\d**1..3) '.' 
              $<oct2>=(\d**1..3) '.' 
              $<oct3>=(\d**1..3) '.' 
              $<oct4>=(\d**1..3) $/
    and all($<oct1>, $<oct2>, $<oct3>, $<oct4>) <= 255
}

# Credit card format (basic)
sub format-credit-card($number) {
    $number.subst(/(\d**4)/, '$0-', :global).subst(/-$/, '');
}

my $card = '1234567890123456';
say format-credit-card($card);  # 1234-5678-9012-3456
```

Advanced patterns can include named captures, complex validation, and  
recursive structures. Combine regex with additional logic for complete  
validation solutions.  