# Command Line Programs

Raku provides excellent support for creating command-line programs with  
built-in features for argument parsing, input/output operations, and system  
interaction. This chapter covers 30 practical examples demonstrating how to  
build CLI applications from simple scripts to sophisticated utilities.  

## Hello world

Basic command-line program that outputs a greeting.  

```raku
#!/usr/bin/env raku

say "Hello, World!";
```

This is the simplest Raku CLI program. The shebang line makes the script  
executable on Unix-like systems. The `say` function outputs text followed  
by a newline to standard output.  

## Command line arguments

Accessing arguments passed to the script via @*ARGS.  

```raku
#!/usr/bin/env raku

say "Script name: $*PROGRAM-NAME";
say "Arguments: {@*ARGS.elems}";

for @*ARGS -> $arg {
    say "Argument: $arg";
}
```

The `@*ARGS` dynamic variable contains command-line arguments as an array.  
`$*PROGRAM-NAME` contains the script name. This example demonstrates basic  
argument iteration and shows how many arguments were provided.  

## Simple calculator

Command-line calculator accepting two numbers and an operation.  

```raku
#!/usr/bin/env raku

sub MAIN($num1, $op, $num2) {
    my $result = do given $op {
        when '+' { $num1 + $num2 }
        when '-' { $num1 - $num2 }
        when '*' { $num1 * $num2 }
        when '/' { $num2 == 0 ?? "Error: Division by zero" !! $num1 / $num2 }
        default  { "Error: Unknown operation $op" }
    };
    
    say "$num1 $op $num2 = $result";
}
```

The `MAIN` subroutine provides automatic command-line parsing. Raku  
automatically validates argument count and provides usage help. The  
`given/when` construct provides pattern matching for operations.  

## File existence checker

Check if files or directories exist from command line.  

```raku
#!/usr/bin/env raku

sub MAIN(*@files) {
    for @files -> $file {
        if $file.IO.e {
            my $type = $file.IO.d ?? "directory" !! "file";
            say "$file: $type exists";
        } else {
            say "$file: does not exist";
        }
    }
}
```

The `*@files` parameter accepts variable number of arguments. The `.IO.e`  
method checks existence, while `.IO.d` checks if it's a directory.  
This demonstrates file system interaction and slurpy parameters.  

## Line counter

Count lines in text files from the command line.  

```raku
#!/usr/bin/env raku

sub MAIN(*@files) {
    my $total-lines = 0;
    
    for @files -> $file {
        if $file.IO.f {
            my $lines = $file.IO.lines.elems;
            say "$file: $lines lines";
            $total-lines += $lines;
        } else {
            say "$file: not a readable file";
        }
    }
    
    say "Total lines: $total-lines" if @files > 1;
}
```

This utility reads files and counts lines using the `.lines` method.  
It demonstrates file validation with `.IO.f`, accumulating totals,  
and conditional output based on the number of input files.  

## Word frequency counter

Count word occurrences in text files.  

```raku
#!/usr/bin/env raku

sub MAIN($filename) {
    unless $filename.IO.f {
        say "Error: '$filename' is not a readable file";
        exit 1;
    }
    
    my %word-count;
    for $filename.IO.lines -> $line {
        for $line.words -> $word {
            %word-count{$word.lc}++;
        }
    }
    
    for %word-count.sort(*.value).reverse -> $pair {
        say "{$pair.value}: {$pair.key}";
    }
}
```

This program builds a hash of word frequencies, processes text line by  
line, normalizes to lowercase, and sorts results by frequency. It shows  
hash manipulation, sorting, and proper error handling with exit codes.  

## Environment variable viewer

Display environment variables with optional filtering.  

```raku
#!/usr/bin/env raku

sub MAIN($pattern?) {
    my @vars = %*ENV.sort(*.key);
    
    for @vars -> $pair {
        my $name = $pair.key;
        my $value = $pair.value;
        
        if $pattern {
            next unless $name ~~ /$pattern/;
        }
        
        say "$name = $value";
    }
}
```

The `%*ENV` hash contains environment variables. Optional parameter  
`$pattern?` allows filtering with regex matching. This demonstrates  
optional parameters, hash iteration, and pattern matching.  

## Directory listing

Enhanced directory listing with file information.  

```raku
#!/usr/bin/env raku

sub MAIN($dir = '.') {
    unless $dir.IO.d {
        say "Error: '$dir' is not a directory";
        exit 1;
    }
    
    for $dir.IO.dir.sort -> $item {
        my $type = $item.d ?? 'DIR' !! 'FILE';
        my $size = $item.f ?? $item.s !! 0;
        my $name = $item.basename;
        
        printf "%-4s %8d %s\n", $type, $size, $name;
    }
}
```

This example shows directory traversal using `.dir`, file property  
checking with `.d` and `.f`, and formatted output with `printf`.  
Default parameter value provides current directory fallback.  

## Text file grep

Search for patterns in text files with line numbers.  

```raku
#!/usr/bin/env raku

sub MAIN($pattern, *@files) {
    for @files -> $file {
        unless $file.IO.f {
            say "Skipping '$file': not a readable file";
            next;
        }
        
        my $line-num = 0;
        for $file.IO.lines -> $line {
            $line-num++;
            if $line ~~ /$pattern/ {
                say "$file:$line-num: $line";
            }
        }
    }
}
```

A simple grep implementation showing regex pattern matching, line  
numbering, and multi-file processing. The smart match operator `~~`  
applies the regex pattern to each line.  

## URL fetcher

Download and display content from URLs.  

```raku
#!/usr/bin/env raku

use HTTP::UserAgent;

sub MAIN(*@urls) {
    my $ua = HTTP::UserAgent.new;
    
    for @urls -> $url {
        try {
            my $response = $ua.get($url);
            if $response.is-success {
                say "=== $url ===";
                say $response.content;
            } else {
                say "Error fetching $url: {$response.status-line}";
            }
            CATCH {
                default { say "Failed to fetch $url: {.message}" }
            }
        }
    }
}
```

This example requires the HTTP::UserAgent module and demonstrates external  
library usage, HTTP requests, error handling with try/CATCH, and  
response status checking.  

## JSON processor

Read and pretty-print JSON files.  

```raku
#!/usr/bin/env raku

use JSON::Fast;

sub MAIN($filename) {
    unless $filename.IO.f {
        say "Error: '$filename' does not exist";
        exit 1;
    }
    
    try {
        my $json = from-json($filename.IO.slurp);
        say to-json($json, :pretty);
        CATCH {
            default { say "Error parsing JSON: {.message}" }
        }
    }
}
```

JSON processing utility using the JSON::Fast module. Shows file reading,  
JSON parsing and formatting, and error handling for malformed JSON.  
The `:pretty` adverb formats output with proper indentation.  

## File copy utility

Copy files with progress indication.  

```raku
#!/usr/bin/env raku

sub MAIN($source, $destination) {
    unless $source.IO.f {
        say "Error: Source file '$source' does not exist";
        exit 1;
    }
    
    if $destination.IO.e {
        print "Destination exists. Overwrite? (y/N): ";
        my $answer = get;
        exit 0 unless $answer.lc eq 'y';
    }
    
    my $content = $source.IO.slurp;
    $destination.IO.spurt($content);
    
    say "Copied $source to $destination";
    say "Size: {$destination.IO.s} bytes";
}
```

File copying utility with user confirmation for overwrites. Demonstrates  
interactive input with `get`, file size checking, and the `spurt` method  
for writing file content.  

## Log analyzer

Analyze log files for error patterns and statistics.  

```raku
#!/usr/bin/env raku

sub MAIN($logfile) {
    unless $logfile.IO.f {
        say "Error: Log file '$logfile' not found";
        exit 1;
    }
    
    my %levels;
    my $total-lines = 0;
    my @errors;
    
    for $logfile.IO.lines -> $line {
        $total-lines++;
        
        if $line ~~ /(\w+)\s* ':' .* $/ {
            my $level = $0.Str.uc;
            %levels{$level}++;
            
            if $level eq 'ERROR' {
                @errors.push($line);
            }
        }
    }
    
    say "=== Log Analysis ===";
    say "Total lines: $total-lines";
    say "Log levels:";
    for %levels.sort(*.key) -> $pair {
        say "  {$pair.key}: {$pair.value}";
    }
    
    if @errors {
        say "\nError messages:";
        for @errors -> $error {
            say "  $error";
        }
    }
}
```

Log analysis tool that categorizes log entries by level and extracts  
error messages. Shows regex capture with `$0`, string manipulation,  
and comprehensive reporting.  

## Password generator

Generate secure passwords with customizable options.  

```raku
#!/usr/bin/env raku

sub MAIN(:$length = 12, :$numbers = True, :$symbols = False) {
    my $chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
    $chars ~= '0123456789' if $numbers;
    $chars ~= '!@#$%^&*()_+-=[]{}|;:,.<>?' if $symbols;
    
    my $password = '';
    for 1..$length {
        $password ~= $chars.comb.pick;
    }
    
    say $password;
}
```

Password generator with named parameters for customization. The `:$length`  
syntax creates optional named parameters with defaults. The `.comb` method  
splits strings into characters, and `.pick` randomly selects one.  

## Base64 encoder/decoder

Encode and decode Base64 content from files or stdin.  

```raku
#!/usr/bin/env raku

use Base64;

multi sub MAIN('encode', $file?) {
    my $content = $file ?? $file.IO.slurp(:bin) !! $*IN.slurp(:bin);
    say encode-base64($content);
}

multi sub MAIN('decode', $file?) {
    my $content = $file ?? $file.IO.slurp !! $*IN.slurp;
    my $decoded = decode-base64($content);
    $*OUT.write($decoded);
}

multi sub MAIN() {
    say "Usage: script.raku [encode|decode] [file]";
}
```

Multi-dispatch MAIN demonstrates different command signatures. Binary  
mode reading with `:bin`, standard input/output handling, and the  
Base64 module for encoding operations.  

## CSV processor

Process CSV files with column operations.  

```raku
#!/usr/bin/env raku

sub MAIN($csvfile, :$column = 1, :$operation = 'sum') {
    unless $csvfile.IO.f {
        say "Error: CSV file '$csvfile' not found";
        exit 1;
    }
    
    my @values;
    for $csvfile.IO.lines -> $line {
        my @fields = $line.split(',');
        if @fields[$column - 1]:exists && @fields[$column - 1] ~~ /^ \d+ \.? \d* $/ {
            @values.push(+@fields[$column - 1]);
        }
    }
    
    my $result = do given $operation {
        when 'sum'  { [+] @values }
        when 'avg'  { ([+] @values) / @values.elems }
        when 'min'  { @values.min }
        when 'max'  { @values.max }
        when 'count' { @values.elems }
        default     { die "Unknown operation: $operation" }
    };
    
    say "$operation of column $column: $result";
}
```

CSV processing tool with column-based operations. Shows field splitting,  
numeric validation with regex, reduction operations with `[+]`, and  
statistical calculations.  

## File organizer

Organize files into directories by extension.  

```raku
#!/usr/bin/env raku

sub MAIN($source-dir = '.') {
    unless $source-dir.IO.d {
        say "Error: '$source-dir' is not a directory";
        exit 1;
    }
    
    my %extensions;
    
    for $source-dir.IO.dir -> $item {
        next unless $item.f;
        
        my $ext = $item.extension || 'no-extension';
        %extensions{$ext}.push($item);
    }
    
    for %extensions.kv -> $ext, @files {
        my $target-dir = "$source-dir/$ext-files";
        $target-dir.IO.mkdir unless $target-dir.IO.d;
        
        say "Moving {+@files} $ext files to $target-dir/";
        for @files -> $file {
            my $target = "$target-dir/{$file.basename}";
            $file.move($target);
        }
    }
}
```

File organization utility that groups files by extension. Demonstrates  
directory creation with `.mkdir`, file extension extraction, hash of  
arrays, and file moving operations.  

## System information

Display system information and statistics.  

```raku
#!/usr/bin/env raku

sub MAIN() {
    say "=== System Information ===";
    say "Operating System: {$*DISTRO.name} {$*DISTRO.version}";
    say "Kernel: {$*KERNEL.name} {$*KERNEL.version}";
    say "Architecture: {$*KERNEL.arch}";
    say "Raku Version: {$*RAKU.version}";
    say "VM: {$*VM.name} {$*VM.version}";
    
    say "\n=== Environment ===";
    say "Home Directory: {%*ENV<HOME> // 'Not set'}";
    say "Current Directory: {$*CWD}";
    say "User: {%*ENV<USER> // %*ENV<USERNAME> // 'Unknown'}";
    
    say "\n=== Process Information ===";
    say "Process ID: {$*PID}";
    say "Program Name: {$*PROGRAM-NAME}";
    
    my $uptime-cmd = run('uptime', :out);
    if $uptime-cmd.exitcode == 0 {
        say "System Uptime: {$uptime-cmd.out.slurp.trim}";
    }
}
```

System information display using Raku's built-in system variables.  
Shows how to access OS details, environment variables, process  
information, and external command execution with `run`.  

## Config file manager

Manage configuration files in key=value format.  

```raku
#!/usr/bin/env raku

sub MAIN($action, $config-file = 'config.conf', *@args) {
    given $action {
        when 'get' {
            my $key = @args[0] // die "Usage: get <key>";
            say get-config($config-file, $key);
        }
        when 'set' {
            my $key = @args[0] // die "Usage: set <key> <value>";
            my $value = @args[1] // die "Usage: set <key> <value>";
            set-config($config-file, $key, $value);
        }
        when 'list' {
            list-config($config-file);
        }
        default {
            say "Usage: config.raku [get|set|list] [config-file] [args...]";
        }
    }
}

sub get-config($file, $key) {
    return "Config file not found" unless $file.IO.f;
    
    for $file.IO.lines -> $line {
        next if $line ~~ /^ \s* '#'/ || $line.trim eq '';
        if $line ~~ /^ \s* $key \s* '=' \s* (.+) $/ {
            return $0.Str.trim;
        }
    }
    return "Key '$key' not found";
}

sub set-config($file, $key, $value) {
    my @lines = $file.IO.f ?? $file.IO.lines !! ();
    my $found = False;
    
    for @lines <-> $line {
        if $line ~~ /^ \s* $key \s* '=' / {
            $line = "$key = $value";
            $found = True;
            last;
        }
    }
    
    @lines.push("$key = $value") unless $found;
    $file.IO.spurt(@lines.join("\n") ~ "\n");
    say "Set $key = $value";
}

sub list-config($file) {
    return say "Config file not found" unless $file.IO.f;
    
    for $file.IO.lines -> $line {
        next if $line ~~ /^ \s* '#'/ || $line.trim eq '';
        if $line ~~ /^ \s* (\w+) \s* '=' \s* (.+) $/ {
            say "$0 = $1";
        }
    }
}
```

Configuration file manager with get/set/list operations. Demonstrates  
file parsing, regex matching, comment handling, and file modification.  
Shows how to build more complex CLI applications with sub-commands.  

## Network port scanner

Simple port scanner for checking open ports.  

```raku
#!/usr/bin/env raku

sub MAIN($host, $start-port = 1, $end-port = 1000) {
    say "Scanning $host for open ports ($start-port-$end-port)...";
    
    my @open-ports;
    
    for $start-port..$end-port -> $port {
        my $sock = try IO::Socket::INET.new(
            host => $host,
            port => $port,
            timeout => 1
        );
        
        if $sock {
            @open-ports.push($port);
            $sock.close;
            print ".";
        } else {
            print "x";
        }
        
        print "\n" if $port % 50 == 0;
    }
    
    say "\n\nOpen ports:";
    if @open-ports {
        for @open-ports -> $port {
            say "Port $port: Open";
        }
    } else {
        say "No open ports found";
    }
}
```

Network port scanner using socket connections. Shows network programming  
with IO::Socket::INET, timeout handling, and progress indication.  
Demonstrates try blocks for connection error handling.  

## Backup utility

Create and manage file backups with timestamps.  

```raku
#!/usr/bin/env raku

sub MAIN($action, *@args) {
    given $action {
        when 'create' {
            my $source = @args[0] // die "Usage: backup.raku create <source> [destination]";
            my $dest = @args[1] // './backups';
            create-backup($source, $dest);
        }
        when 'list' {
            my $backup-dir = @args[0] // './backups';
            list-backups($backup-dir);
        }
        when 'restore' {
            my $backup = @args[0] // die "Usage: backup.raku restore <backup-file>";
            my $dest = @args[1] // '.';
            restore-backup($backup, $dest);
        }
        default {
            say "Usage: backup.raku [create|list|restore] [args...]";
        }
    }
}

sub create-backup($source, $backup-dir) {
    unless $source.IO.e {
        say "Error: Source '$source' does not exist";
        return;
    }
    
    $backup-dir.IO.mkdir unless $backup-dir.IO.d;
    
    my $timestamp = DateTime.now.formatter('%Y%m%d_%H%M%S');
    my $basename = $source.IO.basename;
    my $backup-name = "$backup-dir/{$basename}_$timestamp.bak";
    
    if $source.IO.d {
        run('tar', 'czf', $backup-name, $source);
    } else {
        $source.IO.copy($backup-name);
    }
    
    say "Backup created: $backup-name";
}

sub list-backups($backup-dir) {
    unless $backup-dir.IO.d {
        say "Backup directory '$backup-dir' does not exist";
        return;
    }
    
    my @backups = $backup-dir.IO.dir.grep(*.extension eq 'bak').sort;
    
    if @backups {
        say "Available backups:";
        for @backups -> $backup {
            my $size = $backup.s;
            my $modified = $backup.modified;
            say "  {$backup.basename} ({$size} bytes, $modified)";
        }
    } else {
        say "No backups found in $backup-dir";
    }
}

sub restore-backup($backup-file, $dest) {
    unless $backup-file.IO.f {
        say "Error: Backup file '$backup-file' not found";
        return;
    }
    
    if $backup-file ~~ /\.tar\.gz$/ || $backup-file ~~ /\.bak$/ {
        run('tar', 'xzf', $backup-file, '-C', $dest);
    } else {
        my $basename = $backup-file.IO.basename.subst(/_\d+_\d+\.bak$/, '');
        $backup-file.IO.copy("$dest/$basename");
    }
    
    say "Backup restored to $dest";
}
```

Comprehensive backup utility with create, list, and restore operations.  
Shows timestamp formatting, external command execution with `run`,  
directory operations, and file pattern matching.  

## Download manager

Download files with progress tracking and resume capability.  

```raku
#!/usr/bin/env raku

use HTTP::UserAgent;

sub MAIN($url, :$output, :$resume = False) {
    my $filename = $output // $url.split('/')[*-1] // 'download';
    my $ua = HTTP::UserAgent.new;
    
    my $start-pos = 0;
    if $resume && $filename.IO.f {
        $start-pos = $filename.IO.s;
        say "Resuming download from byte $start-pos";
    }
    
    my %headers;
    %headers<Range> = "bytes=$start-pos-" if $start-pos > 0;
    
    try {
        my $response = $ua.get($url, :%headers);
        
        unless $response.is-success {
            say "Error: {$response.status-line}";
            exit 1;
        }
        
        my $content-length = $response.header('content-length') // 0;
        my $total-size = $content-length + $start-pos;
        
        say "Downloading: $url";
        say "File: $filename";
        say "Size: $total-size bytes";
        
        my $fh = $filename.IO.open(:a);
        my $downloaded = $start-pos;
        
        for $response.content.encode('latin1').comb(1024) -> $chunk {
            $fh.write($chunk);
            $downloaded += $chunk.bytes;
            
            my $progress = $total-size > 0 ?? ($downloaded / $total-size * 100).Int !! 0;
            print "\rProgress: $progress% ($downloaded/$total-size bytes)";
        }
        
        $fh.close;
        say "\nDownload completed: $filename";
        
        CATCH {
            default { say "Download failed: {.message}" }
        }
    }
}
```

Download manager with progress tracking and resume capability. Shows  
HTTP range requests, binary data handling, file append mode, and  
real-time progress display with carriage return.  

## Text encoder/decoder

Encode and decode text in various formats.  

```raku
#!/usr/bin/env raku

sub MAIN($operation, $text?, :$format = 'base64') {
    my $input = $text // $*IN.slurp.chomp;
    
    given $operation {
        when 'encode' {
            say encode-text($input, $format);
        }
        when 'decode' {
            say decode-text($input, $format);
        }
        default {
            say "Usage: encoder.raku [encode|decode] [text] --format=[base64|hex|url|rot13]";
        }
    }
}

sub encode-text($text, $format) {
    given $format {
        when 'base64' { 
            use Base64; 
            encode-base64($text.encode('utf8'));
        }
        when 'hex' { 
            $text.encode('utf8').list.map(*.fmt('%02x')).join;
        }
        when 'url' { 
            $text.subst(/(<-[A..Za..z0..9\-_.~]>)/, {sprintf('%%%02X', $0.ord)}, :g);
        }
        when 'rot13' { 
            $text.trans('A..Za..z' => 'N..ZA..Mn..za..m');
        }
        default { 
            die "Unknown format: $format";
        }
    }
}

sub decode-text($text, $format) {
    given $format {
        when 'base64' { 
            use Base64; 
            decode-base64($text).decode('utf8');
        }
        when 'hex' { 
            $text.comb(2).map({chr(:16($_))}).join;
        }
        when 'url' { 
            $text.subst(/\%(<[0..9A..Fa..f]>**2)/, {chr(:16($0))}, :g);
        }
        when 'rot13' { 
            $text.trans('A..Za..z' => 'N..ZA..Mn..za..m');
        }
        default { 
            die "Unknown format: $format";
        }
    }
}
```

Multi-format text encoder/decoder supporting Base64, hexadecimal, URL  
encoding, and ROT13. Demonstrates string transformation methods,  
character encoding, and format-specific processing.  

## Process monitor

Monitor running processes and display information.  

```raku
#!/usr/bin/env raku

sub MAIN(:$interval = 5, :$filter) {
    loop {
        clear-screen();
        show-processes($filter);
        sleep $interval;
    }
}

sub clear-screen() {
    print "\e[2J\e[H";  # ANSI escape codes to clear screen
}

sub show-processes($filter?) {
    my $ps-output = run('ps', 'aux', :out).out.slurp;
    my @lines = $ps-output.lines;
    
    say "=== Process Monitor (Updated: {DateTime.now}) ===";
    say @lines[0];  # Header line
    say "=" x 80;
    
    my @processes = @lines[1..*];
    
    if $filter {
        @processes = @processes.grep(*.contains($filter));
    }
    
    # Sort by CPU usage (descending)
    @processes = @processes.sort: {
        my @fields = $_.split(/\s+/);
        -(@fields[2] // 0);  # CPU% column
    };
    
    for @processes[0..min(19, @processes.end)] -> $line {
        say $line;
    }
    
    say "\nShowing top 20 processes";
    say "Filter: {$filter // 'none'}";
    say "Press Ctrl+C to exit";
}
```

Process monitoring utility with real-time updates. Shows external command  
execution, output parsing, screen clearing with ANSI codes, and  
continuous monitoring loops.  

## Markdown to HTML converter

Convert Markdown files to HTML with basic formatting.  

```raku
#!/usr/bin/env raku

sub MAIN($markdown-file, $html-file?) {
    unless $markdown-file.IO.f {
        say "Error: Markdown file '$markdown-file' not found";
        exit 1;
    }
    
    my $output-file = $html-file // $markdown-file.subst(/\.md$/, '.html');
    my $content = $markdown-file.IO.slurp;
    my $html = markdown-to-html($content);
    
    $output-file.IO.spurt($html);
    say "Converted $markdown-file to $output-file";
}

sub markdown-to-html($markdown) {
    my $html = qq:to/HTML/;
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>Converted Document</title>
        <style>
            body { font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
            code { background-color: #f4f4f4; padding: 2px 4px; border-radius: 3px; }
            pre { background-color: #f4f4f4; padding: 10px; border-radius: 5px; overflow-x: auto; }
        </style>
    </head>
    <body>
    HTML
    
    my @lines = $markdown.lines;
    my $in-code-block = False;
    
    for @lines -> $line {
        if $line ~~ /^ '```' / {
            if $in-code-block {
                $html ~= "</pre>\n";
                $in-code-block = False;
            } else {
                $html ~= "<pre><code>";
                $in-code-block = True;
            }
        } elsif $in-code-block {
            $html ~= escape-html($line) ~ "\n";
        } elsif $line ~~ /^ '#' \s+ (.+) / {
            $html ~= "<h1>{escape-html($0.Str)}</h1>\n";
        } elsif $line ~~ /^ '##' \s+ (.+) / {
            $html ~= "<h2>{escape-html($0.Str)}</h2>\n";
        } elsif $line ~~ /^ '###' \s+ (.+) / {
            $html ~= "<h3>{escape-html($0.Str)}</h3>\n";
        } elsif $line.trim eq '' {
            $html ~= "<br>\n";
        } else {
            my $processed = process-inline-markdown($line);
            $html ~= "<p>$processed</p>\n";
        }
    }
    
    $html ~= "</code></pre>\n" if $in-code-block;
    $html ~= "</body>\n</html>\n";
    
    return $html;
}

sub process-inline-markdown($text) {
    my $result = escape-html($text);
    $result = $result.subst(/'**' (.+?) '**'/, '<strong>$0</strong>', :g);
    $result = $result.subst(/'*' (.+?) '*'/, '<em>$0</em>', :g);
    $result = $result.subst(/'`' (.+?) '`'/, '<code>$0</code>', :g);
    return $result;
}

sub escape-html($text) {
    $text.subst('&', '&amp;', :g)
         .subst('<', '&lt;', :g)
         .subst('>', '&gt;', :g);
}
```

Markdown to HTML converter with basic formatting support. Demonstrates  
state tracking, regex processing, HTML escaping, and file format  
conversion. Shows how to build text processing utilities.  

## File synchronizer

Synchronize files between directories with comparison.  

```raku
#!/usr/bin/env raku

sub MAIN($source-dir, $target-dir, :$dry-run = False) {
    unless $source-dir.IO.d {
        say "Error: Source directory '$source-dir' does not exist";
        exit 1;
    }
    
    $target-dir.IO.mkdir unless $target-dir.IO.d;
    
    say "Synchronizing $source-dir -> $target-dir";
    say "(Dry run mode)" if $dry-run;
    
    my @actions = compare-directories($source-dir, $target-dir);
    
    for @actions -> $action {
        given $action<type> {
            when 'copy' {
                say "COPY: {$action<source>} -> {$action<target>}";
                unless $dry-run {
                    $action<target>.IO.parent.mkdir;
                    $action<source>.IO.copy($action<target>);
                }
            }
            when 'update' {
                say "UPDATE: {$action<target>}";
                unless $dry-run {
                    $action<source>.IO.copy($action<target>);
                }
            }
            when 'delete' {
                say "DELETE: {$action<target>}";
                unless $dry-run {
                    $action<target>.IO.unlink;
                }
            }
        }
    }
    
    say "Synchronization " ~ ($dry-run ?? "preview" !! "complete");
}

sub compare-directories($source, $target) {
    my @actions;
    my %target-files;
    
    # Build target file inventory
    if $target.IO.d {
        for find-files($target) -> $file {
            my $rel-path = $file.relative($target);
            %target-files{$rel-path} = $file;
        }
    }
    
    # Compare source files
    for find-files($source) -> $source-file {
        my $rel-path = $source-file.relative($source);
        my $target-file = "$target/$rel-path";
        
        if !$target-file.IO.e {
            @actions.push: {
                type => 'copy',
                source => $source-file,
                target => $target-file
            };
        } elsif $source-file.modified > $target-file.IO.modified {
            @actions.push: {
                type => 'update',
                source => $source-file,
                target => $target-file
            };
        }
        
        %target-files{$rel-path}:delete;
    }
    
    # Handle orphaned target files
    for %target-files.values -> $target-file {
        @actions.push: {
            type => 'delete',
            target => $target-file
        };
    }
    
    return @actions;
}

sub find-files($dir) {
    gather {
        for $dir.IO.dir -> $item {
            if $item.d {
                take $_ for find-files($item);
            } elsif $item.f {
                take $item;
            }
        }
    }
}
```

File synchronization utility with dry-run mode and detailed comparison.  
Shows recursive directory traversal with `gather/take`, file timestamp  
comparison, and complex file operations with action planning.  

## Database query tool

Simple database query tool for SQLite databases.  

```raku
#!/usr/bin/env raku

use DBIish;

sub MAIN($database, $query?) {
    unless $database.IO.f {
        say "Error: Database file '$database' not found";
        exit 1;
    }
    
    my $dbh = DBIish.connect('SQLite', database => $database);
    
    if $query {
        execute-query($dbh, $query);
    } else {
        interactive-mode($dbh);
    }
    
    $dbh.dispose;
}

sub execute-query($dbh, $query) {
    try {
        my $sth = $dbh.prepare($query);
        my @results = $sth.allrows();
        
        if @results {
            # Print column headers
            my @columns = $sth.column-names;
            say join(' | ', @columns);
            say '-' x (join(' | ', @columns).chars);
            
            # Print data rows
            for @results -> @row {
                say join(' | ', @row.map(*.Str));
            }
            
            say "\n{+@results} row(s) returned";
        } else {
            say "Query executed successfully";
        }
        
        CATCH {
            default { say "SQL Error: {.message}" }
        }
    }
}

sub interactive-mode($dbh) {
    say "SQLite Interactive Query Tool";
    say "Type 'quit' or 'exit' to quit, '.tables' to list tables";
    say "-" x 50;
    
    loop {
        print "sql> ";
        my $input = get.trim;
        
        last if $input eq 'quit' | 'exit';
        
        if $input eq '.tables' {
            execute-query($dbh, "SELECT name FROM sqlite_master WHERE type='table'");
        } elsif $input.chars > 0 {
            execute-query($dbh, $input);
        }
    }
    
    say "Goodbye!";
}
```

Database query tool with both command-line and interactive modes.  
Demonstrates database connectivity with DBIish, result formatting,  
interactive input loops, and comprehensive error handling.  

## Performance profiler

Simple profiler for analyzing script performance.  

```raku
#!/usr/bin/env raku

sub MAIN($script, *@args) {
    unless $script.IO.f {
        say "Error: Script '$script' not found";
        exit 1;
    }
    
    say "Profiling: $script {@args.join(' ')}";
    say "=" x 50;
    
    my $start-time = now;
    my $start-mem = get-memory-usage();
    
    my $proc = run($*EXECUTABLE, $script, |@args, :out, :err);
    
    my $end-time = now;
    my $end-mem = get-memory-usage();
    
    my $execution-time = $end-time - $start-time;
    my $memory-delta = $end-mem - $start-mem;
    
    say "\n" ~ "=" x 50;
    say "Performance Report:";
    say "Execution time: {$execution-time.fmt('%.3f')} seconds";
    say "Memory change: {$memory-delta} KB";
    say "Exit code: {$proc.exitcode}";
    
    if $proc.exitcode != 0 {
        say "\nStderr output:";
        say $proc.err.slurp;
    }
}

sub get-memory-usage() {
    try {
        my $ps-output = run('ps', '-o', 'rss=', '-p', $*PID, :out).out.slurp;
        return +$ps-output.trim;
        CATCH {
            return 0;
        }
    }
}
```

Performance profiling tool that measures execution time and memory usage.  
Shows process execution with `run`, time measurement with `now`,  
system process information gathering, and formatted output.  

## Hash generator

Generate and verify file hashes for integrity checking.  

```raku
#!/usr/bin/env raku

use Digest::SHA;

sub MAIN($action, *@args) {
    given $action {
        when 'generate' {
            my $file = @args[0] // die "Usage: hash.raku generate <file> [algorithm]";
            my $algorithm = @args[1] // 'sha256';
            generate-hash($file, $algorithm);
        }
        when 'verify' {
            my $file = @args[0] // die "Usage: hash.raku verify <file> <hash> [algorithm]";
            my $expected = @args[1] // die "Usage: hash.raku verify <file> <hash> [algorithm]";
            my $algorithm = @args[2] // 'sha256';
            verify-hash($file, $expected, $algorithm);
        }
        when 'batch' {
            my $dir = @args[0] // '.';
            my $algorithm = @args[1] // 'sha256';
            batch-hash($dir, $algorithm);
        }
        default {
            say "Usage: hash.raku [generate|verify|batch] [args...]";
        }
    }
}

sub generate-hash($file, $algorithm) {
    unless $file.IO.f {
        say "Error: File '$file' not found";
        return;
    }
    
    my $content = $file.IO.slurp(:bin);
    my $hash = do given $algorithm.lc {
        when 'md5'    { Digest::MD5.new.add($content).hex }
        when 'sha1'   { Digest::SHA1.new.add($content).hex }
        when 'sha256' { Digest::SHA256.new.add($content).hex }
        when 'sha512' { Digest::SHA512.new.add($content).hex }
        default       { die "Unsupported algorithm: $algorithm" }
    };
    
    say "$algorithm($file) = $hash";
}

sub verify-hash($file, $expected, $algorithm) {
    unless $file.IO.f {
        say "Error: File '$file' not found";
        return;
    }
    
    my $content = $file.IO.slurp(:bin);
    my $actual = do given $algorithm.lc {
        when 'md5'    { Digest::MD5.new.add($content).hex }
        when 'sha1'   { Digest::SHA1.new.add($content).hex }
        when 'sha256' { Digest::SHA256.new.add($content).hex }
        when 'sha512' { Digest::SHA512.new.add($content).hex }
        default       { die "Unsupported algorithm: $algorithm" }
    };
    
    if $actual.lc eq $expected.lc {
        say "✓ Hash verification PASSED for $file";
    } else {
        say "✗ Hash verification FAILED for $file";
        say "Expected: $expected";
        say "Actual:   $actual";
    }
}

sub batch-hash($dir, $algorithm) {
    unless $dir.IO.d {
        say "Error: Directory '$dir' not found";
        return;
    }
    
    my $output-file = "checksums-$algorithm.txt";
    my $fh = $output-file.IO.open(:w);
    
    for $dir.IO.dir.grep(*.f).sort -> $file {
        my $content = $file.IO.slurp(:bin);
        my $hash = do given $algorithm.lc {
            when 'md5'    { Digest::MD5.new.add($content).hex }
            when 'sha1'   { Digest::SHA1.new.add($content).hex }
            when 'sha256' { Digest::SHA256.new.add($content).hex }
            when 'sha512' { Digest::SHA512.new.add($content).hex }
            default       { die "Unsupported algorithm: $algorithm" }
        };
        
        my $line = "$hash  {$file.basename}";
        $fh.say($line);
        say $line;
    }
    
    $fh.close;
    say "\nChecksums saved to $output-file";
}
```

Hash generation and verification utility supporting multiple algorithms.  
Shows cryptographic hash functions, file integrity checking, batch  
processing, and checksum file generation. Useful for security and  
data integrity verification tasks.  

## Task scheduler

Simple cron-like task scheduler for running commands.  

```raku
#!/usr/bin/env raku

sub MAIN($action, *@args) {
    my $schedule-file = 'schedule.conf';
    
    given $action {
        when 'add' {
            my $cron = @args[0] // die "Usage: scheduler.raku add '<cron-pattern>' '<command>'";
            my $command = @args[1] // die "Usage: scheduler.raku add '<cron-pattern>' '<command>'";
            add-task($schedule-file, $cron, $command);
        }
        when 'list' {
            list-tasks($schedule-file);
        }
        when 'run' {
            run-scheduler($schedule-file);
        }
        when 'remove' {
            my $id = @args[0] // die "Usage: scheduler.raku remove <task-id>";
            remove-task($schedule-file, +$id);
        }
        default {
            say "Usage: scheduler.raku [add|list|run|remove] [args...]";
        }
    }
}

sub add-task($file, $cron, $command) {
    my @tasks = $file.IO.f ?? $file.IO.lines !! ();
    my $id = @tasks.elems + 1;
    
    @tasks.push("$id|$cron|$command|{DateTime.now}");
    $file.IO.spurt(@tasks.join("\n") ~ "\n");
    
    say "Task $id added: '$command' scheduled for '$cron'";
}

sub list-tasks($file) {
    unless $file.IO.f {
        say "No scheduled tasks found";
        return;
    }
    
    say "ID  Schedule        Command";
    say "-" x 50;
    
    for $file.IO.lines -> $line {
        my ($id, $cron, $command, $added) = $line.split('|');
        printf "%-3s %-15s %s\n", $id, $cron, $command;
    }
}

sub remove-task($file, $task-id) {
    unless $file.IO.f {
        say "No scheduled tasks found";
        return;
    }
    
    my @tasks = $file.IO.lines.grep: {
        my $id = $_.split('|')[0];
        +$id != $task-id;
    };
    
    $file.IO.spurt(@tasks.join("\n") ~ "\n");
    say "Task $task-id removed";
}

sub run-scheduler($file) {
    unless $file.IO.f {
        say "No scheduled tasks found";
        return;
    }
    
    say "Task scheduler started. Press Ctrl+C to stop.";
    
    loop {
        my $now = DateTime.now;
        
        for $file.IO.lines -> $line {
            my ($id, $cron, $command, $added) = $line.split('|');
            
            if should-run($cron, $now) {
                say "[{DateTime.now}] Running task $id: $command";
                
                my $proc = run($command, :out, :err);
                if $proc.exitcode == 0 {
                    say "[{DateTime.now}] Task $id completed successfully";
                } else {
                    say "[{DateTime.now}] Task $id failed with exit code {$proc.exitcode}";
                }
            }
        }
        
        sleep 60;  # Check every minute
    }
}

sub should-run($cron-pattern, $datetime) {
    # Simple cron pattern matching (minute hour day month weekday)
    my @pattern = $cron-pattern.split(/\s+/);
    my @current = (
        $datetime.minute,
        $datetime.hour,
        $datetime.day,
        $datetime.month,
        $datetime.day-of-week
    );
    
    for @pattern.kv -> $i, $field {
        next if $field eq '*';
        
        if $field ~~ /^ \d+ $/ {
            return False unless +$field == @current[$i];
        } elsif $field ~~ /^ (\d+) '-' (\d+) $/ {
            my $start = +$0;
            my $end = +$1;
            return False unless $start <= @current[$i] <= $end;
        } elsif $field ~~ /^ '*/' (\d+) $/ {
            my $step = +$0;
            return False unless @current[$i] % $step == 0;
        }
    }
    
    return True;
}
```

Simple task scheduler with cron-like patterns for automated job execution.  
Shows time-based scheduling, pattern matching, file-based configuration,  
and background process management. Demonstrates building system utilities  
for task automation and job scheduling.  

This comprehensive collection of 30 CLI examples demonstrates Raku's  
capabilities for building command-line applications, from simple utilities  
to complex system tools. Each example builds upon previous concepts  
while introducing new features and techniques for practical programming.