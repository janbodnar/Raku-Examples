# Unicode

Unicode support is built into Raku's core, providing comprehensive handling
of international text, characters, and scripts from around the world.

## Basic Unicode literals

Creating strings with Unicode characters using different notation methods.

```raku
my $greeting = "Hello, 世界!";
my $emoji = "🌍🌎🌏";
my $accents = "café naïve résumé";

say $greeting;
say $emoji;
say $accents;

# Using Unicode code points
my $heart = "\x[2665]";     # ♥
my $lambda = "\x[03BB]";    # λ
my $music = "\x[1F3B5]";    # 🎵

say "$heart $lambda $music";
```

Raku handles Unicode strings natively. You can include Unicode characters
directly in string literals or use hexadecimal escape sequences with
`\x[code]` notation. The language supports the full Unicode standard
including emojis, symbols, and international scripts.

## Unicode character properties

Examining Unicode character properties and classifications.

```raku
my $text = "Aα1₂🎵";

for $text.comb -> $char {
    say "$char:";
    say "  Code point: U+{$char.ord.base(16)}";
    say "  Is letter: {$char.uniprop('General_Category') ~~ /^L/ ?? 'Yes' !! 'No'}";
    say "  Is number: {$char.uniprop('General_Category') ~~ /^N/ ?? 'Yes' !! 'No'}";
    say "  Is symbol: {$char.uniprop('General_Category') ~~ /^S/ ?? 'Yes' !! 'No'}";
    say "  Category: {$char.uniprop('General_Category')}";
    say "  Script: {$char.uniprop('Script')}";
    say "---";
}
```

Unicode character properties provide detailed classification information.
The `uniprop` method accesses Unicode properties like General_Category
and Script. 

## Unicode normalization

Converting Unicode text to standard normalized forms.

```raku
my $composed = "café";              # Single é character
my $decomposed = "cafe\x[0301]";    # e + combining acute accent

say "Original strings equal: {$composed eq $decomposed}";
say "NFC equal: {$composed.NFC eq $decomposed.NFC}";
say "NFD equal: {$composed.NFD eq $decomposed.NFD}";

say "Composed length: {$composed.chars} graphemes";
say "Decomposed length: {$decomposed.chars} graphemes";
say "Decomposed code points: {$decomposed.codes} codes";


# Canonical forms
say "NFC: {$decomposed.NFC.Str.chars} chars";    # Composed
say "NFD: {$composed.NFD.Str.chars} chars";      # Decomposed
```

Unicode normalization ensures consistent text representation. NFC
(Canonical Composition) combines characters, while NFD (Canonical
Decomposition) separates them. Normalization is crucial for reliable
string comparison and text processing in international applications.
Note that while the number of graphemes (`chars`) might be the same, the number
of underlying code points (`codes`) can differ.

## Unicode case conversion

Converting case with full Unicode support for all scripts.

```raku
my $text = "Hello ΚΌΣΜΟΣ мир العالم";

say "Original: $text";
say "Uppercase: {$text.uc}";
say "Lowercase: {$text.lc}";
say "Title case: {$text.tc}";
say "Fold case: {$text.fc}";

# Language-specific casing
my $german = "Straße";
say "German uppercase: {$german.uc}";

my $turkish = "İstanbul";
say "Turkish lowercase: {$turkish.lc}";
```

Raku's case conversion methods handle Unicode properly across all
scripts. The `fc` method performs case folding for case-insensitive
comparisons. Language-specific rules are applied automatically,
such as German ß and Turkish dotted/dotless i handling.

## Unicode grapheme clusters

Working with grapheme clusters and combining characters.

```raku
my $flags = "🏴󠁧󠁢󠁳󠁣󠁴󠁿🇺🇸";  # Scotland flag + US flag
my $family = "👨‍👩‍👧‍👦";          # Family emoji
my $accented = "é̂̃̈";               # Multiple combining marks

say "Flags: {$flags.chars} graphemes, {$flags.codes} codes";
say "Family: {$family.chars} graphemes, {$family.codes} codes";
say "Accented: {$accented.chars} graphemes, {$accented.codes} codes";

# Split into graphemes
for $flags.comb -> $grapheme {
    say "Grapheme: $grapheme ({$grapheme.codes} codes)";
}
```

Grapheme clusters represent user-perceived characters that may consist
of multiple Unicode code points. The `chars` method counts graphemes
while `codes` counts individual code points. Complex emojis and
combining characters demonstrate the difference.

## Unicode encoding and decoding

Converting between Unicode and byte representations.

```raku
my $text = "Hello 世界 🌍";

# Encode to different formats
my $utf8 = $text.encode('utf8');
my $utf16 = $text.encode('utf16');
my $utf32 = $text.encode('utf32');

say "UTF-8 bytes: {$utf8.elems}";
say "UTF-16 bytes: {$utf16.elems}";
say "UTF-32 bytes: {$utf32.elems}";

# Decode back to strings
say "Decoded UTF-8: {$utf8.decode('utf8')}";
say "Decoded UTF-16: {$utf16.decode('utf16')}";

# Handle encoding errors
my $invalid = Buf.new(0xFF, 0xFE, 0xFF);
say "With replacement: {$invalid.decode('utf8', :replacement)}";
```

Unicode encoding converts strings to byte sequences for storage and
transmission. Different encodings (UTF-8, UTF-16, UTF-32) have varying
space efficiency. The `:replacement` option handles invalid sequences
gracefully during decoding operations.

## Unicode collation and sorting

Sorting Unicode text according to linguistic rules.

```raku
my @words = <café naïve résumé Åse Øle>;

# Default sort (code point order)
say "Code point sort: {@words.sort}";

# Unicode collation (linguistic sort)
# The Unicode::Collate module needs to be installed: zef install Unicode::Collate
use Unicode::Collate;
my $collator = Unicode::Collate.new;
say "Collated sort: {@words.sort($collator)}";

# Case-insensitive collation
my @mixed = <Apple apple BANANA banana>;
say "Case-insensitive: {@mixed.sort: *.fc}";
```

Unicode collation provides linguistically appropriate sorting that
considers cultural conventions. Simple code point sorting may not
match user expectations for international text. Case folding
enables case-insensitive comparisons and sorting.

## Unicode regular expressions

Pattern matching with Unicode character properties.

```raku
my $mixed = "Hello こんにちは 你好 🌍 123";

# Match by script
say "Latin: " ~ $mixed.match(/:g <:Script<Latin>>+/).map(*.Str).join(' ');
say "Hiragana: " ~ $mixed.match(/:g <:Script<Hiragana>>+/).map(*.Str).join(' ');
say "Han: " ~ $mixed.match(/:g <:Script<Han>>+/).map(*.Str).join(' ');

# Match by category
say "Letters: " ~ $mixed.match(/:g <:L>+/).map(*.Str).join(' ');
say "Numbers: " ~ $mixed.match(/:g <:N>+/).map(*.Str).join(' ');
say "Symbols: " ~ $mixed.match(/:g <:S>+/).map(*.Str).join(' ');

# Emoji matching
say "Emoji: " ~ $mixed.match(/:g <:Emoji>+/).map(*.Str).join(' ');
```

Unicode regular expressions can match characters by script, category,
or other properties. This enables sophisticated text processing for
international content, allowing patterns that work across multiple
languages and writing systems simultaneously.

## Unicode script detection

Identifying writing scripts in multilingual text.

```raku
my $multilingual = "Hello мир 世界 العالم हैलो";

for $multilingual.comb -> $char {
    next if $char ~~ /\s/;
    my $script = $char.uniprop('Script');
    say "$char: $script";
}

# Group by script using classify
my %by-script = $multilingual.comb.grep(* !~~ /\s/).classify(*.uniprop('Script'));

for %by-script.kv -> $script, @chars {
    say "$script: {@chars.join}";
}
```

Script detection helps identify the writing system of characters,
enabling language-specific processing. Grouping characters by script
allows for targeted operations like font selection or input method
switching in multilingual applications.

## Unicode bidirectional text

Handling text that flows in different directions.

```raku
my $hebrew = "שלום";
my $arabic = "مرحبا";
my $mixed = "Hello שלום world مرحبا!";

say "Hebrew: $hebrew";
say "Arabic: $arabic";
say "Mixed: $mixed";

# Check text direction
for $mixed.comb -> $char {
    next if $char ~~ /\s/;
    my $bidi = $char.uniprop('Bidi_Class');
    say "$char: $bidi" if $bidi ne 'L';
}

# Logical vs visual order
say "Logical order: {$mixed.comb.join(' | ')}";
```

Bidirectional text handling is crucial for languages like Hebrew and
Arabic that read right-to-left. The Bidi_Class property indicates
text direction. Raku maintains logical order while display systems
handle visual reordering.

## Unicode character names

Looking up character names and properties.

```raku
my $symbols = "♠♣♥♦★☆";

for $symbols.comb -> $char {
    my $name = $char.uniname;
    my $code = $char.ord.base(16).uc;
    say "U+$code: $name ($char)";
}

# Find characters by name pattern
# The Unicode::Names module needs to be installed: zef install Unicode::Names
use Unicode::Names;
say "Heart symbols:";
for unicode-names.grep(/HEART/) -> $name {
    my $char = $name.uniparse;
    say "  $name: $char";
}
```

Unicode character names provide human-readable descriptions. The
`uniname` method returns the official Unicode name. Character
lookup by name pattern helps discover related symbols and
enables programmatic character selection.

## Unicode numeric values

Extracting numeric values from Unicode characters.

```raku
my $numbers = "123 ¼ ½ ¾ ⅓ ⅔ ² ³ ₁ ₂";

for $numbers.comb -> $char {
    next if $char ~~ /\s/;

    my $numeric = $char.uniprop('Numeric_Value');
    my $type = $char.uniprop('Numeric_Type');

    say "$char: numeric=$numeric, type=$type" if $numeric.defined;
}

# Roman numerals
my $roman = "ⅠⅡⅢⅣⅤⅩⅬⅭⅮⅯ";
for $roman.comb -> $char {
    my $value = $char.uniprop('Numeric_Value');
    say "$char = $value";
}
```

Unicode characters can have numeric values beyond ASCII digits.
Fractions, superscripts, subscripts, and Roman numerals all carry
numeric properties. This enables mathematical processing of
diverse numeric representations.

## Unicode emoji handling

Working with emoji characters and sequences.

```raku
my $emojis = "😀👍🎉🌈👨‍👩‍👧‍👦🏴󠁧󠁢󠁳󠁣󠁴󠁿";

say "Emoji count: {$emojis.chars}";
say "Code points: {$emojis.codes}";

# Detect emoji
for $emojis.comb -> $emoji {
    say "$emoji: " ~ ($emoji ~~ /\p{Emoji}/ ?? "emoji" !! "not emoji");
}

# Skin tone modifiers
my $person = "👋";
my @skin-tones = "\x[1F3FB]", "\x[1F3FC]", "\x[1F3FD]",
                 "\x[1F3FE]", "\x[1F3FF]";

say "Base: $person";
for @skin-tones -> $tone {
    say "With tone: $person$tone";
}
```

Emoji handling requires understanding of Unicode sequences and
modifiers. Some emoji consist of multiple code points including
skin tone modifiers and zero-width joiners. Proper emoji processing
accounts for these complex character combinations.

## Unicode text segmentation

Segmenting text into words, sentences, and other boundaries.

```raku
my $text = "Hello world! How are you? Fine, thanks.";
my $japanese = "これは日本語の文章です。分かりますか？";

# Word boundaries
say "English words: {$text.words}";
say "Japanese chars: {$japanese.comb}";

# Simple sentence splitting
my @sentences = $text.match(/.*? <[.?!]>+ /, :g).map(*.Str);
say "Sentences: {@sentences}";

# Character boundary iteration
for $japanese.comb.kv -> $i, $char {
    say "Position $i: $char";
}
```

Text segmentation varies by language and script. English uses spaces
for word boundaries, while languages like Japanese and Chinese don't.
Unicode provides boundary algorithms, though implementation may
require specialized libraries for complex scripts.

## Unicode transliteration

Converting between different scripts and writing systems.

```raku
# Simple transliteration table
my %cyrillic-to-latin =
    'а' => 'a', 'б' => 'b', 'в' => 'v', 'г' => 'g',
    'д' => 'd', 'е' => 'e', 'ё' => 'yo', 'ж' => 'zh',
    'з' => 'z', 'и' => 'i', 'й' => 'y', 'к' => 'k',
    'л' => 'l', 'м' => 'm', 'н' => 'n', 'о' => 'o',
    'п' => 'p', 'р' => 'r', 'с' => 's', 'т' => 't',
    'у' => 'u', 'ф' => 'f', 'х' => 'h', 'ц' => 'ts',
    'ч' => 'ch', 'ш' => 'sh', 'щ' => 'sch', 'ъ' => '',
    'ы' => 'y', 'ь' => '', 'э' => 'e', 'ю' => 'yu',
    'я' => 'ya';

my $russian = "привет мир";
my $transliterated = $russian.trans(%cyrillic-to-latin);
say "Russian: $russian";
say "Transliterated: $transliterated";
```

Transliteration converts text between scripts while preserving
pronunciation. The `trans` method performs character-by-character
substitution using mapping tables. Complex transliteration may
require context-sensitive rules and phonetic considerations.

## Unicode input/output handling

Reading and writing Unicode text from various sources.

```raku
# Writing Unicode to file
my $unicode-text = "Hello 世界 🌍\nこんにちは\nমরহবা";

spurt "unicode-test.txt", $unicode-text, :enc<utf8>;

# Reading Unicode from file
my $read-text = slurp "unicode-test.txt", :enc<utf8>;
say "Read from file: $read-text";

# Handle different encodings
# Note: slurp with a wrong encoding like 'latin1' on a UTF-8 file may not
# throw an error, but produce garbled text (mojibake).
try {
    my $latin1-text = slurp "unicode-test.txt", :enc<latin1>;
    say "Read with latin1 (may be garbled): $latin1-text";
    CATCH {
        say "Encoding error: {.message}";
    }
}

# "Safe" encoding detection (naive approach)
# This function tries a list of encodings and returns the first one that
# succeeds. This is not a reliable way to detect encoding and may return
# garbled text if an incorrect encoding is chosen.
sub safe-read-file($filename) {
    for <utf8 utf16 latin1> -> $encoding {
        try {
            return slurp $filename, :enc($encoding);
            CATCH { next }
        }
    }
    die "Could not read file with any encoding";
}
```

Unicode I/O requires proper encoding specification. UTF-8 is the
most common encoding for Unicode text. Error handling is important
when dealing with files of unknown encoding. Encoding detection
helps ensure reliable text processing.

## Unicode error handling

Gracefully handling Unicode errors and invalid sequences.

```raku
# Invalid UTF-8 sequence
my $invalid-bytes = Buf.new(0xFF, 0xFE, 0xFF, 0x41, 0x42);

# Default behavior (throws exception)
try {
    my $decoded = $invalid-bytes.decode('utf8');
    CATCH {
        say "Decoding error: {.message}";
    }
}

# With replacement character
my $with-replacement = $invalid-bytes.decode('utf8', :replacement);
say "With replacement: $with-replacement";

# Custom replacement
my $custom-replacement = $invalid-bytes.decode('utf8',
                                               :replacement('?'));
say "Custom replacement: $custom-replacement";

# Validation function
sub is-valid-unicode($text) {
    try {
        $text.encode('utf8').decode('utf8');
        return True;
        CATCH { return False }
    }
}

say "Valid: {is-valid-unicode('Hello 世界')}";
```

Unicode error handling prevents crashes when encountering invalid
byte sequences. Replacement characters (�) substitute for invalid
data. Custom error handling strategies include logging, skipping,
or using domain-specific replacements.

## Unicode performance considerations

Optimizing Unicode text processing for better performance.

```raku
# The Bench module needs to be installed: zef install Bench
use Bench;

my $large-text = "Hello 世界! " x 10000;

# Compare different operations
my @results = Bench.new.timethese(1000, {
    'chars-count' => { $large-text.chars },
    'codes-count' => { $large-text.codes },
    'bytes-count' => { $large-text.encode('utf8').elems },
    'comb-split'  => { $large-text.comb },
    'regex-match' => { $large-text ~~ m:g/<:L>/ },
});

# Memory-efficient processing
sub process-large-unicode($text) {
    # Process in chunks to avoid memory issues
    my $chunk-size = 1000;
    my $start = 0;
    while $start < $text.chars {
        my $chunk = $text.substr($start, $chunk-size);
        # Process chunk
        $start += $chunk.chars;
    }
}
```

Unicode operations can be performance-sensitive with large texts.
Grapheme counting is slower than code point counting. Chunked
processing reduces memory usage. Consider caching results of
expensive Unicode property lookups.

## Unicode ranges and character sets

Working with Unicode ranges and creating character sets.

```raku
# Define Unicode ranges
my $ascii-range = 0x0000..0x007F;
my $latin1-range = 0x0000..0x00FF;
my $cyrillic-range = 0x0400..0x04FF;
my $cjk-range = 0x4E00..0x9FFF;

# Test character membership
my $text = "Hello Привет 你好";
for $text.comb -> $char {
    my $code = $char.ord;
    say "$char (U+{$code.base(16).uc}):";
    say "  ASCII" if $code ∈ $ascii-range;
    say "  Cyrillic" if $code ∈ $cyrillic-range;
    say "  CJK" if $code ∈ $cjk-range;
}

# Create custom character sets from a sample string
my $vowels = set <a e i o u A E I O U>;
my $text-with-vowels = "áéíóúýàèìòùâêîôûäëïöüÿ";
my $unicode-vowels-from-text = $text-with-vowels.comb.grep(*.uniname.contains('VOWEL')).Set;

say "Unicode vowels from sample: $unicode-vowels-from-text.keys.join(' ')";
```

Unicode ranges organize characters by script and purpose. Range
membership testing enables efficient character classification.
Custom character sets can be built using Unicode properties
for specialized text processing needs.

## Unicode comparison and equality

Comparing Unicode strings with different representations.

```raku
my $str1 = "café";                    # Precomposed
my $str2 = "cafe\x[0301]";           # Decomposed
my $str3 = "CAFÉ";                   # Different case

say "Direct comparison: {$str1 eq $str2}";
say "NFC comparison: {$str1.NFC eq $str2.NFC}";
say "NFD comparison: {$str1.NFD eq $str2.NFD}";

# Case-insensitive comparison
say "Case-insensitive: {$str1.fc eq $str3.fc}";

# Custom comparison function
sub unicode-equal($a, $b, :$case-sensitive = True, :$normalize = True) {
    my ($x, $y) = ($a, $b);

    $x = $x.NFC if $normalize;
    $y = $y.NFC if $normalize;

    unless $case-sensitive {
        $x = $x.fc;
        $y = $y.fc;
    }

    return $x eq $y;
}

say "Custom equal: {unicode-equal($str1, $str2)}";
say "Custom case-insensitive: {unicode-equal($str1, $str3, :!case-sensitive)}";
```

Unicode equality requires careful consideration of normalization
and case. Different representations of the same text may not be
equal without normalization. Custom comparison functions provide
consistent behavior across applications.

## Unicode formatting and display

Formatting Unicode text for display and alignment.

```raku
my @words = "Hello", "世界", "🌍", "Café";

# Simple formatting
for @words -> $word {
    say sprintf "%-10s (%d chars, %d codes)",
        $word, $word.chars, $word.codes;
}

# Width-aware formatting (East Asian characters are often double-width)
sub display-width($text) {
    my $width = 0;
    for $text.comb -> $char {
        my $eaw = $char.uniprop('East_Asian_Width');
        $width += $eaw eq any(<F W A>) ?? 2 !! 1;
    }
    return $width;
}

say "Display width calculations:";
for @words -> $word {
    say "$word: {display-width($word)} display units";
}

# Truncation with proper boundary handling
sub truncate-unicode($text, $max-width) {
    my $width = 0;
    my $result = "";

    for $text.comb -> $char {
        my $char-width = display-width($char);
        last if $width + $char-width > $max-width;
        $result ~= $char;
        $width += $char-width;
    }

    return $result;
}

say "Truncated: '{truncate-unicode("Hello 世界! 🌍", 8)}'";
```

Unicode formatting requires consideration of character display width.
East Asian characters typically occupy two character cells in
terminal displays. Proper truncation respects grapheme boundaries
to avoid splitting composite characters.

## Unicode security considerations

Protecting against Unicode-based security vulnerabilities.

```raku
# Homograph detection
my $legitimate = "example.com";
my $spoofed = "еxamplе.com";    # Contains Cyrillic е

say "Legitimate: $legitimate";
say "Spoofed: $spoofed";
say "Are equal: {$legitimate eq $spoofed}";

# Script mixing detection
sub detect-script-mixing($text) {
    my %scripts;
    for $text.comb -> $char {
        next if $char ~~ /\W/;  # Skip non-word characters
        my $script = $char.uniprop('Script');
        %scripts{$script}++;
    }
    return %scripts.keys > 1;
}

my $mixed = "example.сom";  # Latin + Cyrillic
say "Script mixing in '$mixed': {detect-script-mixing($mixed)}";

# Normalization attacks
my $original = "file.txt";
my $attack = "file\x[FDFA].txt";  # Contains Arabic ligature

say "Original: $original";
say "Attack: $attack";
say "NFD lengths: {$original.NFD.codes} vs {$attack.NFD.codes}";

# Safe identifier validation
sub is-safe-identifier($id) {
    # Only allow single script + common characters
    my %scripts;
    for $id.comb -> $char {
        my $script = $char.uniprop('Script');
        next if $script eq 'Common';  # Numbers, punctuation
        %scripts{$script}++;
    }
    return %scripts.keys <= 1;
}

say "Safe identifier 'user123': {is-safe-identifier('user123')}";
say "Unsafe identifier 'uѕer123': {is-safe-identifier('uѕer123')}";
```

Unicode security vulnerabilities include homograph attacks using
similar-looking characters from different scripts. Mixed-script
detection helps identify suspicious text. Normalization can
sometimes be exploited, so validation is important.

## Unicode version compatibility

Handling different Unicode versions and feature support.

```raku
# Check Unicode version
say "Unicode version: {$*UNICODE-VERSION}";

# Version-dependent character properties
my $new-emoji = "🫠";  # Melting face (Unicode 14.0)

try {
    my $name = $new-emoji.uniname;
    say "Emoji name: $name";
    CATCH {
        say "Character not recognized in this Unicode version";
    }
}

# Fallback for unsupported characters
sub safe-char-name($char) {
    try {
        return $char.uniname;
        CATCH {
            return "U+{$char.ord.base(16).uc} (unknown)";
        }
    }
}

my $test-chars = "😀🫠";
for $test-chars.comb -> $char {
    say "$char: {safe-char-name($char)}";
}

# Version-aware processing
sub process-text-safely($text) {
    return $text.comb.map(-> $char {
        # Handle potential version incompatibilities
        try {
            my $props = $char.uniprop('General_Category');
            return $char;
            CATCH {
                return '?';  # Replacement for unknown characters
            }
        }
    }).join;
}
```

Unicode standards evolve, adding new characters and properties.
Version compatibility checking prevents errors with newer Unicode
features. Graceful degradation ensures applications work across
different Unicode implementations and versions.

## Unicode data processing

Processing and analyzing large Unicode datasets efficiently.

```raku
# Sample multilingual dataset
my @texts = (
    "Hello world",
    "Bonjour le monde",
    "Hola mundo",
    "こんにちは世界",
    "Привет мир",
    "مرحبا بالعالم",
    "नमस्ते दुनिया"
);

# Language detection by script
sub detect-language-by-script($text) {
    my %script-counts;
    for $text.comb -> $char {
        next if $char ~~ /\s/;
        my $script = $char.uniprop('Script');
        %script-counts{$script}++;
    }

    return %script-counts.max(*.value).key;
}

# Statistical analysis
my %stats;
for @texts -> $text {
    my $script = detect-language-by-script($text);
    %stats{$script}<count>++;
    %stats{$script}<total-chars> += $text.chars;
    %stats{$script}<total-codes> += $text.codes;
}

say "Script statistics:";
for %stats.kv -> $script, %data {
    say "$script: {%data<count>} texts, avg {({%data<total-chars> / %data<count>}).round(2)} chars";
}

# Character frequency analysis
my %char-freq;
for @texts -> $text {
    for $text.comb -> $char {
        next if $char ~~ /\s/;
        %char-freq{$char}++;
    }
}

say "\nMost frequent characters:";
for %char-freq.sort(-*.value).head(10) -> (:$key, :$value) {
    say "$key: $value times ({$key.uniname // 'unknown'})";
}
```

Unicode data processing involves analyzing text patterns across
multiple scripts and languages. Statistical analysis reveals
usage patterns and helps optimize text processing algorithms
for specific linguistic contexts.

## Advanced Unicode manipulation

Complex Unicode text manipulation and transformation techniques.

```raku
# Text transformation pipeline
class UnicodeProcessor {
    has $.normalize = True;
    has $.case-fold = False;
    has $.remove-diacritics = False;

    method process($text) {
        my $result = $text;

        # Normalization
        $result = $result.NFC if $.normalize;

        # Case folding
        $result = $result.fc if $.case-fold;

        # Remove diacritics
        if $.remove-diacritics {
            $result = $result.NFD;
            $result = $result.subst(/\p{Mark}/, '', :g);
            $result = $result.NFC;
        }

        return $result;
    }
}

my $processor = UnicodeProcessor.new(:case-fold, :remove-diacritics);

my @test-texts = "Café", "naïve", "RÉSUMÉ", "François";
for @test-texts -> $text {
    say "$text → {$processor.process($text)}";
}

# Advanced text analysis
sub analyze-unicode-complexity($text) {
    my %analysis;

    # Script diversity
    my %scripts;
    for $text.comb -> $char {
        next if $char ~~ /\s/;
        %scripts{$char.uniprop('Script')}++;
    }
    %analysis<script-count> = %scripts.keys.elems;

    # Combining character ratio
    my $combining = $text.comb.grep(*.uniprop('General_Category').starts-with('M')).elems;
    %analysis<combining-ratio> = $text.codes ?? $combining / $text.codes !! 0;

    # Grapheme complexity
    %analysis<grapheme-ratio> = $text.codes ?? $text.chars / $text.codes !! 0;

    return %analysis;
}

my $complex-text = "🏴󠁧󠁢󠁳󠁣󠁴󠁿 Café naïve é̂̃̈ مرحبا";
my %complexity = analyze-unicode-complexity($complex-text);
say "\nComplexity analysis for '$complex-text':";
for %complexity.kv -> $metric, $value {
    say "  $metric: {$value.round(2)}";
}
```

Advanced Unicode manipulation combines multiple processing steps
for sophisticated text transformation. Complexity analysis helps
identify challenging texts that may require special handling
or performance optimization in text processing systems.
