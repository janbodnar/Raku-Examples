# Grammars

Grammars in Raku are powerful pattern matching and parsing tools that extend  
regular expressions. They allow you to define complex language structures  
using hierarchical rules, making them ideal for parsing configuration files,  
data formats, and even entire programming languages. Grammars combine the  
flexibility of regular expressions with the structure of formal language  
grammars, providing a clean and maintainable way to handle complex text  
processing tasks.  

## Basic token definition

Creating simple tokens for basic pattern matching.  

```raku
grammar BasicTokens {
    token word { \w+ }
    token number { \d+ }
    token space { \s+ }
}

my $text = 'hello 123';

if $text ~~ /<BasicTokens::word>/ {
    say "Found word: {$/}";
}

if $text ~~ /<BasicTokens::number>/ {
    say "Found number: {$/}";
}
```

Tokens are the building blocks of grammars. They define reusable patterns  
that can be combined into more complex structures. The `::` operator  
accesses tokens from a grammar in regex contexts.  

## Simple rule matching

Using rules that automatically handle whitespace.  

```raku
grammar MathExpression {
    rule TOP { <number> <operator> <number> }
    token number { \d+ ['.' \d+]? }
    token operator { '+' | '-' | '*' | '/' }
}

my $expr = '42 + 15';
my $complex = '3.14 * 2.71';

say $expr ~~ MathExpression;      # True
say $complex ~~ MathExpression;   # True

if MathExpression.parse($expr) -> $match {
    say "First number: {$match<number>[0]}";
    say "Operator: {$match<operator>}";
    say "Second number: {$match<number>[1]}";
}
```

Rules automatically handle whitespace between elements, making them perfect  
for parsing human-readable formats. The `TOP` rule serves as the entry  
point for grammar matching.  

## Named capture groups

Using named captures for structured data extraction.  

```raku
grammar PersonInfo {
    token TOP { <name> ',' \s* <age> ',' \s* <occupation> }
    token name { \w+ \s+ \w+ }
    token age { \d+ }
    token occupation { \w+ }
}

my $person = 'John Doe, 35, Engineer';

if PersonInfo.parse($person) -> $match {
    say "Name: {$match<name>}";
    say "Age: {$match<age>}";
    say "Occupation: {$match<occupation>}";
}
```

Named captures in grammars create structured parse trees. Each named element  
becomes accessible through the match object, providing clean data extraction  
from complex text formats.  

## Multiple alternatives

Defining tokens with multiple possible patterns.  

```raku
grammar LogLevel {
    token TOP { <level> }
    token level { <debug> | <info> | <warning> | <error> }
    token debug { 'DEBUG' | 'DBG' | 'D' }
    token info { 'INFO' | 'INF' | 'I' }
    token warning { 'WARNING' | 'WARN' | 'W' }
    token error { 'ERROR' | 'ERR' | 'E' }
}

my @levels = 'DEBUG', 'WARN', 'E', 'INFO', 'INVALID';

for @levels -> $level {
    if LogLevel.parse($level) -> $match {
        say "$level is valid: {$match<level>}";
    } else {
        say "$level is invalid";
    }
}
```

Alternative patterns within tokens allow flexible matching. This approach  
enables grammars to handle multiple formats or synonyms while maintaining  
clear structure in the parse tree.  

## Quantified patterns

Using quantifiers for repeating elements.  

```raku
grammar WordList {
    rule TOP { <word>+ % ',' }
    token word { \w+ }
}

grammar NumberSequence {
    token TOP { <digit>+ }
    token digit { \d }
}

my $words = 'apple, banana, cherry, date';
my $numbers = '123456789';

say WordList.parse($words);        # Matches words separated by commas
say NumberSequence.parse($numbers); # Matches sequence of digits

if WordList.parse($words) -> $match {
    say "Words found: {$match<word>.join(', ')}";
}
```

Quantifiers like `+` (one or more) and `*` (zero or more) handle repeating  
patterns. The `%` operator specifies separators between repeated elements,  
perfect for parsing lists and sequences.  

## Nested structures

Creating grammars that handle hierarchical data.  

```raku
grammar BracketExpression {
    rule TOP { <expression> }
    rule expression { <term> | <bracketed> }
    rule term { \w+ }
    rule bracketed { '(' <expression>+ % ',' ')' }
}

my $simple = 'hello';
my $nested = '(foo, bar, (baz, qux))';

say BracketExpression.parse($simple);  # Matches simple term
say BracketExpression.parse($nested);   # Matches nested structure

if BracketExpression.parse($nested) -> $match {
    say "Parsed: {$match}";
}
```

Recursive grammar rules enable parsing of nested structures like parentheses,  
brackets, or tree-like data. The grammar can handle arbitrary levels of  
nesting through self-referential rules.  

## Actions for data processing

Adding action methods to transform parsed data.  

```raku
grammar Calculator {
    rule TOP { <expr> }
    rule expr { <term> [ <[+-]> <term> ]* }
    rule term { <factor> [ <[*/]> <factor> ]* }
    rule factor { <number> | '(' <expr> ')' }
    token number { \d+ ['.' \d+]? }
}

class CalculatorActions {
    method TOP($/) { make $<expr>.made }
    method expr($/) {
        my $result = $<term>[0].made;
        for $<term>[1..*] Z $<[+-]> -> ($term, $op) {
            $result = $op eq '+' ?? $result + $term.made !! $result - $term.made;
        }
        make $result;
    }
    method term($/) {
        my $result = $<factor>[0].made;
        for $<factor>[1..*] Z $<[*/]> -> ($factor, $op) {
            $result = $op eq '*' ?? $result * $factor.made !! $result / $factor.made;
        }
        make $result;
    }
    method factor($/) { make $<number> ?? +$<number> !! $<expr>.made }
    method number($/) { make +$/ }
}

my $expression = '2 + 3 * 4';
my $result = Calculator.parse($expression, :actions(CalculatorActions.new));
say "Result: {$result.made}";  # Result: 14
```

Action classes transform parse trees into meaningful data structures. Each  
method corresponds to a grammar rule and can perform calculations or build  
complex objects from the parsed input.  

## CSV parser

Building a grammar to parse comma-separated values.  

```raku
grammar CSV {
    rule TOP { <line>+ % \n }
    rule line { <field>+ % ',' }
    token field { <quoted> | <unquoted> }
    token quoted { '"' <( <-["]>* )> '"' }
    token unquoted { <-[,\n"]>+ }
}

my $csv-data = q:to/END/;
name,age,city
"John Doe",30,"New York"
Jane Smith,25,Boston
END

if CSV.parse($csv-data) -> $match {
    for $match<line> -> $line {
        say "Fields: {$line<field>.join(' | ')}";
    }
}
```

This CSV parser handles both quoted and unquoted fields, demonstrating how  
grammars can parse structured data formats. The `<( )>` construct captures  
only the content inside quotes.  

## JSON parser

Creating a simplified JSON grammar.  

```raku
grammar SimpleJSON {
    rule TOP { <value> }
    rule value { <object> | <array> | <string> | <number> | <boolean> | <null> }
    rule object { '{' <pair>* % ',' '}' }
    rule pair { <string> ':' <value> }
    rule array { '[' <value>* % ',' ']' }
    token string { '"' <-["]>* '"' }
    token number { '-'? \d+ ['.' \d+]? }
    token boolean { 'true' | 'false' }
    token null { 'null' }
}

my $json = '{"name": "Alice", "age": 30, "active": true}';

if SimpleJSON.parse($json) -> $match {
    say "Valid JSON structure parsed";
    say "Object pairs: {$match<value><object><pair>.elems}";
}
```

This JSON grammar demonstrates recursive data structures and multiple value  
types. It can parse objects, arrays, and primitive values while maintaining  
the hierarchical structure of JSON data.  

## Configuration file parser

Parsing configuration files with sections and key-value pairs.  

```raku
grammar ConfigFile {
    rule TOP { <section>* }
    rule section { '[' <section-name> ']' <entry>* }
    rule entry { <key> '=' <value> }
    token section-name { \w+ }
    token key { \w+ }
    token value { \N+ }
}

my $config = q:to/END/;
[database]
host=localhost
port=5432

[web]
port=8080
debug=true
END

if ConfigFile.parse($config) -> $match {
    for $match<section> -> $section {
        say "Section: [{$section<section-name>}]";
        for $section<entry> -> $entry {
            say "  {$entry<key>} = {$entry<value>}";
        }
    }
}
```

Configuration file parsing showcases how grammars handle structured text  
with sections and key-value pairs. The `\N+` pattern matches any non-newline  
characters for flexible value content.  

## Email address validation

Creating a grammar for email address format validation.  

```raku
grammar EmailAddress {
    token TOP { <local-part> '@' <domain> }
    token local-part { <word> ['.' <word>]* }
    token domain { <subdomain> ['.' <subdomain>]* }
    token subdomain { \w+ }
    token word { \w+ }
}

my @emails = (
    'user@example.com',
    'john.doe@company.org',
    'invalid@',
    '@invalid.com',
    'valid.email@sub.domain.com'
);

for @emails -> $email {
    if EmailAddress.parse($email) {
        say "$email is valid";
    } else {
        say "$email is invalid";
    }
}
```

Email validation demonstrates practical grammar usage for input validation.  
The grammar enforces the basic structure while allowing flexible subdomain  
and local-part formats.  

## URL parser

Parsing URLs into their component parts.  

```raku
grammar URL {
    token TOP { <scheme> '://' <authority> <path>? <query>? <fragment>? }
    token scheme { \w+ }
    token authority { <host> [':' <port>]? }
    token host { [\w+]+ % '.' }
    token port { \d+ }
    token path { '/' <path-segment>* % '/' }
    token path-segment { <-[/?#]>+ }
    token query { '?' <query-param>+ % '&' }
    token query-param { <key> '=' <value> }
    token key { \w+ }
    token value { <-[&#]>+ }
    token fragment { '#' \w+ }
}

my $url = 'https://example.com:8080/path/to/resource?param1=value1&param2=value2#section';

if URL.parse($url) -> $match {
    say "Scheme: {$match<scheme>}";
    say "Host: {$match<authority><host>}";
    say "Port: {$match<authority><port> // 'default'}";
    say "Path: {$match<path> // '/'}";
    say "Query: {$match<query> // 'none'}";
}
```

URL parsing breaks down complex URLs into manageable components. Optional  
parts like port, query, and fragment are handled gracefully with the `?`  
quantifier.  

## Arithmetic expression evaluator

A complete arithmetic parser with proper precedence.  

```raku
grammar ArithmeticGrammar {
    rule TOP { <expr> }
    rule expr { <term> [ <[+-]> <term> ]* }
    rule term { <factor> [ <[*/]> <factor> ]* }
    rule factor { <number> | '(' <expr> ')' }
    token number { \d+ ['.' \d+]? }
}

class ArithmeticEvaluator {
    method TOP($/) { make $<expr>.made }
    
    method expr($/) {
        my $result = $<term>[0].made;
        for $<term>[1..*] Z $<[+-]> -> ($term, $op) {
            given $op {
                when '+' { $result += $term.made }
                when '-' { $result -= $term.made }
            }
        }
        make $result;
    }
    
    method term($/) {
        my $result = $<factor>[0].made;
        for $<factor>[1..*] Z $<[*/]> -> ($factor, $op) {
            given $op {
                when '*' { $result *= $factor.made }
                when '/' { $result /= $factor.made }
            }
        }
        make $result;
    }
    
    method factor($/) {
        make $<number> ?? +$<number> !! $<expr>.made;
    }
    
    method number($/) { make +$/ }
}

my @expressions = '2 + 3 * 4', '(2 + 3) * 4', '10 / 2 - 1';

for @expressions -> $expr {
    my $result = ArithmeticGrammar.parse($expr, :actions(ArithmeticEvaluator.new));
    say "$expr = {$result.made}";
}
```

This arithmetic evaluator demonstrates proper operator precedence through  
grammar structure. The action class performs actual calculations while  
respecting mathematical precedence rules.  

## Log file parser

Parsing structured log files with timestamps and levels.  

```raku
grammar LogEntry {
    token TOP { <timestamp> \s+ <level> \s+ <component> \s+ <message> }
    token timestamp { <date> \s+ <time> }
    token date { \d**4 '-' \d**2 '-' \d**2 }
    token time { \d**2 ':' \d**2 ':' \d**2 ['.' \d+]? }
    token level { 'DEBUG' | 'INFO' | 'WARN' | 'ERROR' | 'FATAL' }
    token component { '[' \w+ ']' }
    token message { .+ }
}

my $log-line = '2023-12-25 14:30:15.123 ERROR [Database] Connection timeout occurred';

if LogEntry.parse($log-line) -> $match {
    say "Date: {$match<timestamp><date>}";
    say "Time: {$match<timestamp><time>}";
    say "Level: {$match<level>}";
    say "Component: {$match<component>}";
    say "Message: {$match<message>}";
}
```

Log parsing demonstrates real-world grammar applications. The hierarchical  
structure makes it easy to extract specific fields from complex log formats  
while maintaining readability.  

## INI file parser

Parsing Windows-style INI configuration files.  

```raku
grammar INIFile {
    rule TOP { <content>* }
    rule content { <section> | <entry> | <comment> }
    rule section { '[' <section-name> ']' }
    rule entry { <key> '=' <value> }
    rule comment { ';' \N* }
    token section-name { <-[\]]>+ }
    token key { \w+ }
    token value { \N* }
}

my $ini-content = q:to/END/;
; Configuration file
[General]
name=MyApp
version=1.0

[Database]
host=localhost
port=3306
; This is a comment
user=admin
END

if INIFile.parse($ini-content) -> $match {
    for $match<content> -> $item {
        given $item {
            when .<section> { say "Section: [{.<section><section-name>}]" }
            when .<entry> { say "  {.<entry><key>} = {.<entry><value>}" }
            when .<comment> { say "Comment: {.<comment>}" }
        }
    }
}
```

INI file parsing handles mixed content types within a single structure.  
Comments, sections, and entries are parsed distinctly but processed together  
in the content stream.  

## SQL query parser

Basic SQL SELECT statement parsing.  

```raku
grammar SimpleSQL {
    rule TOP { 'SELECT' <select-list> 'FROM' <table> <where-clause>? }
    rule select-list { <column>+ % ',' | '*' }
    rule where-clause { 'WHERE' <condition> }
    rule condition { <column> <operator> <value> }
    token column { \w+ }
    token table { \w+ }
    token operator { '=' | '!=' | '<' | '>' | '<=' | '>=' }
    token value { \d+ | "'" <-[']>* "'" }
}

my $sql = "SELECT name, age FROM users WHERE age > 18";

if SimpleSQL.parse($sql) -> $match {
    say "Columns: {$match<select-list><column>.join(', ')}";
    say "Table: {$match<table>}";
    if $match<where-clause> {
        my $cond = $match<where-clause><condition>;
        say "Condition: {$cond<column>} {$cond<operator>} {$cond<value>}";
    }
}
```

SQL parsing demonstrates domain-specific language parsing. Even simple SQL  
parsing requires handling keywords, identifiers, and various value types  
within a structured query format.  

## Markdown header parser

Parsing Markdown headers and extracting structure.  

```raku
grammar MarkdownHeaders {
    rule TOP { <header>+ }
    token header { <level> \s+ <title> \n? }
    token level { '#'+ }
    token title { \N+ }
}

class HeaderExtractor {
    method TOP($/) { make $<header>».made }
    method header($/) {
        make {
            level => $<level>.chars,
            title => ~$<title>
        }
    }
}

my $markdown = q:to/END/;
# Main Title
## Section One
### Subsection A
### Subsection B
## Section Two
END

my $result = MarkdownHeaders.parse($markdown, :actions(HeaderExtractor.new));

for $result.made -> $header {
    say "Level {$header<level>}: {$header<title>}";
}
```

Markdown parsing shows how grammars can extract document structure. The  
action class builds a data structure representing the document hierarchy  
for further processing.  

## Advanced patterns with lookahead

Using lookahead assertions for complex matching.  

```raku
grammar AdvancedPatterns {
    token TOP { <word-with-suffix> }
    token word-with-suffix { <word> <?before 'ing' | 'ed' | 'er'> <suffix> }
    token word { \w+ }
    token suffix { 'ing' | 'ed' | 'er' }
}

grammar EmailValidator {
    token TOP { <valid-email> }
    token valid-email { <local> '@' <domain> <!before '@'> }
    token local { \w+ ['.' \w+]* }
    token domain { \w+ ['.' \w+]+ }
}

my @words = 'running', 'walked', 'faster', 'book';
my @emails = 'user@domain.com', 'invalid@@domain.com';

for @words -> $word {
    say AdvancedPatterns.parse($word) ?? "$word matches pattern" !! "$word doesn't match";
}

for @emails -> $email {
    say EmailValidator.parse($email) ?? "$email is valid" !! "$email is invalid";
}
```

Lookahead assertions (`<?before>` and `<!before>`) enable complex pattern  
matching without consuming input. Negative lookahead prevents matching  
invalid patterns like double @ symbols in emails.  