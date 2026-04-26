# Input/Output

Raku provides a rich and layered IO system that handles everything from  
reading a single line of console input to binary file manipulation,  
directory traversal, and network sockets. Understanding how these layers  
fit together is key to writing effective Raku programs.  

At the core of Raku's IO system are two important abstractions. The  
`IO::Path` class represents a filesystem path — a file or directory  
location — and provides methods for querying metadata, reading content,  
and navigating the filesystem. The `IO::Handle` class represents an open  
file handle and offers fine-grained control over reading, writing,  
seeking, and encoding. Most everyday tasks can be accomplished through  
convenience methods without touching handles directly.  

The `.IO` coercer is the standard entry point: calling `.IO` on a `Str`  
wraps it in an `IO::Path` object. From there you can call `.slurp` to  
read the entire file, `.lines` to get a lazy list of lines, `.spurt` to  
write content, or `.open` to obtain an `IO::Handle` for lower-level  
access. The procedural forms `slurp`, `spurt`, and `lines` also exist  
as top-level subroutines for convenience.  

File handles support multiple open modes through adverbs: `:r` (read,  
default), `:w` (write, truncating), `:a` (append), and `:x` (exclusive  
create, failing if the file already exists). Encoding is controlled with  
`:enc` — Raku defaults to `utf8` for text IO but supports `latin-1`,  
`ascii`, and others. Binary IO is enabled with the `:bin` adverb, which  
makes reads and writes work with `Buf` objects instead of strings.  

Standard streams `$*IN`, `$*OUT`, and `$*ERR` are globally available  
`IO::Handle`-like objects. The `say`, `print`, `put`, and `note`  
builtins write to `$*OUT` and `$*ERR` respectively. These streams can  
be redirected, and alternative handles can be passed to most output  
routines.  

Directory operations are also part of `IO::Path`. You can create, remove,  
and list directories, and the `.dir` method returns a lazy list of  
`IO::Path` children. Recursive traversal is implemented by calling  
`.dir` recursively on each subdirectory entry.  

Raku also integrates process IO through the `run` and `shell` builtins,  
which return `Proc` objects that let you capture standard output and  
standard error, pipe data, and check exit codes. For network IO, the  
`IO::Socket::INET` class provides a low-level TCP interface.  

This document covers all these areas with practical, self-contained  
examples that build from the simplest console interaction to binary  
files, process pipelines, and TCP sockets.  

## Read cli input

Reading a single line of text from the console with `get`.  

```raku
print 'Enter your name: ';
my $line = get;

say "Hello $line!";
```

The `print` function writes a prompt without a trailing newline so the  
cursor stays on the same line. `get` reads one line from standard input  
(`$*IN`) and returns it as a `Str` with the newline stripped. The  
returned value is then interpolated into the greeting.  

## Read file

Three equivalent ways to read an entire file into a string.  

```raku
my $fname = 'words.txt';

# old school
my $fh = open $fname, :r;
my $contents = $fh.slurp;
say $contents;

$fh.close;

say '-------------------------';

# via IO::Path — handle closed automatically
$contents = $fname.IO.slurp;
say $contents;

say '-------------------------';

# procedural subroutine form
$contents = slurp $fname;
say $contents;
```

The first approach opens a handle with `open`, calls `.slurp` to read  
all bytes as a string, then closes the handle manually. The second and  
third approaches use the `.IO` coercer and the top-level `slurp` sub  
respectively — both manage the handle lifetime automatically. All three  
produce identical output.  

## Line by line

Reading a file one line at a time and storing lines in an array.  

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

`IO::Path.lines` returns a lazy `Seq` of lines with newlines stripped.  
Iterating it directly in a `for` loop is memory-efficient for large  
files. Assigning to an array forces eager evaluation, collecting all  
lines into memory at once. The three `say` calls demonstrate different  
ways to inspect the resulting array.  

## Write to file

Creating a new file and writing a string to it.  

```raku
my $fname = 'output.txt';

$fname.IO.spurt("Hello, World!\n");

say $fname.IO.slurp;
```

`IO::Path.spurt` opens the file for writing (`:w`), writes the given  
string, and closes the handle automatically. If the file already exists  
it is truncated. Passing `:append` instead of the default keeps existing  
content. The `.slurp` call immediately after verifies the written data.  

## Append to file

Adding content to an existing file without overwriting it.  

```raku
my $fname = 'log.txt';

$fname.IO.spurt("First line\n");
$fname.IO.spurt("Second line\n", :append);
$fname.IO.spurt("Third line\n",  :append);

print $fname.IO.slurp;
```

Calling `spurt` with the `:append` adverb opens the file in append mode  
so each call adds to the end rather than replacing existing content.  
Without `:append` every `spurt` call would truncate the file to just  
the new string.  

## Write lines to file

Writing a list of lines to a file at once.  

```raku
my @words = <sky cloud wind rain snow>;

my $fname = 'words.txt';
$fname.IO.spurt(@words.join("\n") ~ "\n");

for $fname.IO.lines -> $line {
    say $line;
}
```

Arrays must be converted to a single string before passing to `spurt`.  
Using `.join("\n")` places a newline between each element, and the  
trailing `~ "\n"` ensures the final line is also terminated. The `for`  
loop reads the file back to confirm the content.  

## Print to file handle

Writing to a file using an explicit handle and `print`.  

```raku
my $fh = open 'notes.txt', :w;

$fh.print("Line one\n");
$fh.say("Line two");
$fh.printf("Value: %d\n", 42);

$fh.close;

say 'notes.txt'.IO.slurp;
```

An `IO::Handle` opened with `:w` supports the same output methods as  
standard output: `print` writes without a trailing newline, `say` appends  
one automatically, and `printf` formats the string using C-style format  
specifiers before writing. Calling `.close` flushes buffers and releases  
the OS file descriptor.  

## File existence check

Testing whether a file or path exists before using it.  

```raku
my $fname = 'words.txt';

if $fname.IO.e {
    say "$fname exists";
} else {
    say "$fname does not exist";
}

say $fname.IO.f ?? "it is a file"    !! "not a file";
say $fname.IO.d ?? "it is a directory" !! "not a directory";
```

The `.e` method returns `True` if the path exists (file, directory, or  
symlink). `.f` is true only for plain files, `.d` only for directories.  
These boolean predicates let you guard operations that would otherwise  
throw exceptions on missing paths.  

## File and directory tests

A comprehensive set of IO predicates for path inspection.  

```raku
my $path = '/etc/hosts'.IO;

say "exists:     ", $path.e;
say "is file:    ", $path.f;
say "is dir:     ", $path.d;
say "readable:   ", $path.r;
say "writable:   ", $path.w;
say "executable: ", $path.x;
say "is symlink: ", $path.l;
say "is absolute:", $path.is-absolute;
```

`IO::Path` exposes a full suite of test predicates mirroring Unix file  
tests. They work on any OS Raku supports and report platform-appropriate  
results. Checking permissions before opening a file prevents cryptic  
errors and makes defensive code more readable.  

## File size

Querying the size of a file in bytes.  

```raku
my $fname = 'words.txt';

my $size = $fname.IO.s;
say "Size: $size bytes";

if $size > 0 {
    say "File is non-empty";
}

say "Size in KB: { ($size / 1024).fmt('%.2f') }";
```

The `.s` method returns the file size in bytes as an `Int`. It queries  
the operating system and does not read the file content, making it very  
fast even for large files. The result can be formatted with `.fmt` for  
human-readable output.  

## File timestamps

Reading modification and access times from a file's metadata.  

```raku
my $path = 'words.txt'.IO;

my $modified = $path.modified;
my $accessed = $path.accessed;
my $changed  = $path.changed;

say "Modified: ", DateTime.new($modified);
say "Accessed: ", DateTime.new($accessed);
say "Changed:  ", DateTime.new($changed);
```

`.modified`, `.accessed`, and `.changed` return `Instant` objects  
representing the file's mtime, atime, and ctime respectively. Wrapping  
them in `DateTime.new` converts the raw timestamp to a human-readable  
date and time in UTC.  

## IO::Path object

Working with `IO::Path` objects and their path-component methods.  

```raku
my $path = IO::Path.new('/home/user/documents/report.txt');

say $path.absolute;        # full absolute path string
say $path.dirname;         # /home/user/documents
say $path.basename;        # report.txt
say $path.extension;       # txt
say $path.stem;            # report

say $path.parent;          # /home/user/documents  (IO::Path)
say $path.parent.parent;   # /home/user
```

`IO::Path` decomposes a filesystem path into its components. `.dirname`  
and `.basename` mirror the Unix commands of the same name. `.extension`  
returns the part after the last dot, and `.stem` is the basename without  
the extension. `.parent` returns a new `IO::Path` for the containing  
directory and can be chained.  

## Build paths

Constructing new paths by joining components safely.  

```raku
my $base = $*HOME;

my $config = $base.add('config');
my $file   = $config.add('settings.ini');

say $config;
say $file;

# Also works from string
my $tmp = '/tmp'.IO.add('myapp').add('cache');
say $tmp;
```

`IO::Path.add` appends a path segment using the correct separator for  
the current operating system. This is safer than string concatenation  
because it handles trailing slashes and OS differences automatically.  
`$*HOME` is a built-in dynamic variable holding the user's home  
directory as an `IO::Path`.  

## Copy a file

Copying a file to another location.  

```raku
my $src  = 'words.txt'.IO;
my $dest = 'words_backup.txt'.IO;

$src.copy($dest);

say "Copied: ", $dest.e;
say "Size matches: ", ($src.s == $dest.s);
```

`IO::Path.copy` duplicates the file at `$src` to the path given by  
`$dest`. If `$dest` already exists it is overwritten. The operation is  
not recursive; copying a directory requires iterating its contents.  
After copying, comparing `.s` (file size) is a quick sanity check.  

## Rename a file

Moving or renaming a file.  

```raku
my $old = 'draft.txt'.IO;
my $new = 'final.txt'.IO;

$old.spurt("This is the content.");

$old.rename($new);

say $new.slurp;
say "Old still exists: ", $old.e;
```

`IO::Path.rename` moves the file to the new path, which may be on the  
same filesystem or a different directory. On most systems this is an  
atomic operation when both paths are on the same filesystem. If the  
destination exists it is replaced.  

## Delete a file

Removing a file from the filesystem.  

```raku
my $fname = 'temp.txt'.IO;

$fname.spurt("temporary data");
say "Created: ", $fname.e;

$fname.unlink;
say "Deleted: ", !$fname.e;
```

`IO::Path.unlink` removes the file. It raises an exception if the file  
does not exist, so guard the call with `.e` if the file might be absent.  
Unlinking a file removes the directory entry; the data persists until  
all open handles to it are closed.  

## Create a directory

Making new directories, including nested ones.  

```raku
my $dir = 'myapp/data/cache'.IO;

$dir.mkdir;
say "Created: ", $dir.d;

# mkdir also accepts a mode (permissions)
'myapp/logs'.IO.mkdir(:mode(0o755));
```

`IO::Path.mkdir` creates the directory and all missing parent  
directories (like `mkdir -p`). The optional `:mode` adverb sets Unix  
permissions as an octal integer. The method returns the created path,  
making it chainable.  

## Remove a directory

Deleting empty directories.  

```raku
'myapp/data/cache'.IO.rmdir;
'myapp/data'.IO.rmdir;
'myapp/logs'.IO.rmdir;
'myapp'.IO.rmdir;

say "Gone: ", !'myapp'.IO.e;
```

`IO::Path.rmdir` removes an empty directory. Attempting to remove a  
non-empty directory raises an exception. To remove a directory tree you  
must first delete all files and subdirectories, typically by recursing  
through `.dir`.  

## List directory

Listing the contents of a directory.  

```raku
my @entries = '/etc'.IO.dir;

for @entries.head(8) -> $entry {
    my $type = $entry.d ?? 'dir' !! 'file';
    say "$type  {$entry.basename}";
}

say "Total entries: ", @entries.elems;
```

`IO::Path.dir` returns a lazy `Seq` of `IO::Path` objects for every  
entry in the directory (excluding `.` and `..`). Each element supports  
the same predicates as any other `IO::Path`. Calling `.head(8)` limits  
output to the first eight items without reading the rest.  

## Filter directory entries

Selecting specific files from a directory listing.  

```raku
my @raku-files = '/home/user/code'.IO.dir(test => / '.raku' $ /);

for @raku-files -> $f {
    say $f.basename, "  (", $f.s, " bytes)";
}
```

The `test` named argument accepts a regex, a `Callable`, or a `Str`  
that is matched against each entry's basename. Only entries whose  
basename matches are returned. This avoids manually filtering the  
result list and keeps the intent clear.  

## Recursive directory listing

Walking a directory tree to find all files.  

```raku
sub all-files(IO::Path $dir) {
    gather for $dir.dir -> $entry {
        if $entry.d {
            .take for all-files($entry);
        } else {
            take $entry;
        }
    }
}

for all-files('/etc'.IO).head(10) -> $f {
    say $f;
}
```

`gather/take` builds a lazy sequence from the recursive traversal.  
Calling the recursive `all-files` on each subdirectory descends the  
tree. Using `.head(10)` demonstrates that the sequence is lazy: only  
enough of the tree is traversed to yield ten results.  

## Read binary file

Reading raw bytes from a file into a `Buf`.  

```raku
my $fname = 'image.png';

if $fname.IO.e {
    my $bytes = $fname.IO.slurp(:bin);
    say "Bytes read: ", $bytes.bytes;
    say "First 4 bytes: ", $bytes[0..3]>>.fmt('%02X').join(' ');
}
```

Passing `:bin` to `.slurp` returns a `Buf[uint8]` instead of a `Str`.  
Individual bytes are unsigned 8-bit integers. The `>>.fmt('%02X')`  
hyperoperator formats each byte as a two-digit hexadecimal value,  
useful for inspecting file signatures (magic numbers).  

## Write binary file

Writing a `Buf` of raw bytes to a file.  

```raku
my $buf = Buf.new(0x48, 0x65, 0x6C, 0x6C, 0x6F, 0x0A);  # "Hello\n"

'hello.bin'.IO.spurt($buf, :bin);

my $back = 'hello.bin'.IO.slurp;
say $back;    # Hello
```

`spurt` with `:bin` writes a `Buf` directly without any encoding  
conversion. Reading the file back without `:bin` interprets the bytes  
as UTF-8 text, so the ASCII bytes in this example produce readable  
output. Mixing text and binary IO on the same file requires care.  

## Seek in a file

Moving the read position within an open file handle.  

```raku
my $fh = open 'words.txt', :r;

$fh.seek(0, SeekFromEnd);      # go to end
say "File size: ", $fh.tell;

$fh.seek(0, SeekFromBeginning); # back to start
say "First line: ", $fh.get;

$fh.seek(5, SeekFromBeginning); # skip 5 bytes
say "From byte 5: ", $fh.get;

$fh.close;
```

`IO::Handle.seek` takes a byte offset and a seek anchor constant:  
`SeekFromBeginning`, `SeekFromCurrent`, or `SeekFromEnd`. After  
seeking, `.tell` returns the current byte position. Seeking requires  
a handle opened in a mode that supports random access (`:r`, `:w`).  

## Tell file position

Checking the current byte offset within an open file handle.  

```raku
my $fh = open 'words.txt', :r;

say "Start: ", $fh.tell;     # 0

$fh.get;                      # read first line
say "After line 1: ", $fh.tell;

$fh.get;                      # read second line
say "After line 2: ", $fh.tell;

$fh.close;
```

`.tell` returns an `Int` byte offset measured from the beginning of  
the file. After each `.get` call the position advances by the number  
of bytes in the line including the newline character. This is useful  
for implementing parsers or resumable file readers.  

## Standard output

Writing to standard output through `$*OUT`.  

```raku
$*OUT.print("no newline");
$*OUT.say(" <- print, this is say");
$*OUT.printf("formatted %s %d\n", "value", 42);

# Equivalent top-level calls
print "via print\n";
say "via say";
printf "via printf %s\n", "ok";
```

`$*OUT` is the standard output handle. The top-level `print`, `say`,  
and `printf` functions are shortcuts that write to `$*OUT`. Using  
`$*OUT` directly is useful when you want to pass the output handle as  
an argument or redirect it programmatically.  

## Standard error

Writing diagnostic messages to standard error.  

```raku
$*ERR.say("Error: something went wrong");
note "shortcut for writing to STDERR";

my $value = -1;
note "Warning: negative value $value" if $value < 0;
```

`$*ERR` is the standard error handle. The `note` built-in is a  
shortcut that writes to `$*ERR` with a trailing newline, equivalent  
to `$*ERR.say(...)`. Standard error is separate from standard output  
so error messages can be filtered or redirected independently in  
the shell.  

## print, say, and put

The three main text output routines and their differences.  

```raku
print "no newline added";
print "\n";

say "say adds newline";

put "put also adds newline";

my @words = <one two three>;

say @words;     # (one two three)  — uses .gist
put @words;     # one two three    — uses .Str (space-separated)
print @words;   # onetwothree      — .Str, no separator
```

`print` outputs the stringified (`.Str`) value with no newline.  
`say` calls `.gist` which includes type decorations for containers,  
then appends a newline. `put` calls `.Str` (plain text) and appends  
a newline. The difference is visible when printing arrays: `say`  
shows the parenthesized list while `put` joins with spaces.  

## Formatted output with printf

Producing formatted text output with format specifiers.  

```raku
printf "%-15s %5d  %8.2f\n", "apples",  42,  3.14;
printf "%-15s %5d  %8.2f\n", "bananas",  7, 99.9;
printf "%-15s %5d  %8.2f\n", "cherries", 1000, 0.5;

my $report = sprintf "Total items: %d\n", 1049;
print $report;
```

`printf` writes formatted output to `$*OUT`. `sprintf` returns the  
formatted string for further processing. Format specifiers begin with  
`%`: `%s` for strings, `%d` for integers, `%f` for floats. Width and  
alignment modifiers control column spacing, making `printf` ideal for  
producing tabular reports.  

## Read words from file

Splitting file content into individual words.  

```raku
my $fname = 'words.txt';

my @words = $fname.IO.words;

say "Word count: ", @words.elems;
say "Unique words: ", @words.unique.elems;
say "Sorted: ", @words.sort.head(5);
```

`IO::Path.words` reads the file and splits on whitespace, returning  
a `Seq` of word strings with no punctuation processing. Assigning to  
an array forces eager evaluation. The `.unique` method deduplicates  
and `.sort` orders alphabetically, demonstrating that the result is  
a standard Raku list.  

## Read with prompt

Prompting the user for input with a message.  

```raku
my $name = prompt("Enter your name: ");
my $age  = prompt("Enter your age: ").Int;

say "Hello, $name! You are $age years old.";

if $age >= 18 {
    say "You are an adult.";
} else {
    say "You are a minor.";
}
```

`prompt` combines `print` and `get` into a single call: it outputs  
the given string, then reads and returns one line from `$*IN`. The  
return value is always a `Str`, so calling `.Int` converts it to a  
number for arithmetic. `prompt` is useful for simple interactive  
scripts.  

## Read multiple inputs

Collecting several values from the user in a loop.  

```raku
my @names;

loop {
    my $input = prompt("Enter a name (blank to stop): ");
    last if $input eq '';
    @names.push($input);
}

say "You entered {+@names} names:";
say "  $_" for @names;
```

`loop` without a condition runs indefinitely until `last` exits it.  
Each iteration prompts for a name; an empty line signals the end of  
input. The collected names are stored in `@names` and printed after  
the loop finishes. `+@names` coerces the array to a numeric count.  

## Read CSV file

Parsing a simple comma-separated values file.  

```raku
my $csv = "name,age,city\nAlice,30,London\nBob,25,Paris\n";
'data.csv'.IO.spurt($csv);

my @rows = 'data.csv'.IO.lines;
my @headers = @rows.shift.split(',');

say @headers;

for @rows -> $row {
    my %record = @headers Z=> $row.split(',');
    say %record;
}
```

The first line is treated as a header row and split into column names.  
Subsequent rows are paired with headers using the `Z=>` zip-with-pair  
operator, which produces a list of `Pair` objects used to initialize  
a hash directly. For production use the `Text::CSV` module handles  
quoting and escaping.  

## Write CSV file

Producing a comma-separated values file from data.  

```raku
my @headers = <name score grade>;
my @data = [
    ("Alice", 95, "A"),
    ("Bob",   82, "B"),
    ("Carol", 78, "C"),
];

my $fh = open 'results.csv', :w;
$fh.say(@headers.join(','));

for @data -> @row {
    $fh.say(@row.join(','));
}

$fh.close;
print 'results.csv'.IO.slurp;
```

Each row is joined with commas and written with `say`, which adds a  
newline automatically. Opening a new handle with `:w` truncates any  
existing file. For values that may contain commas or newlines, wrap  
them in double quotes or use the `Text::CSV` module.  

## File encoding

Reading and writing files with explicit character encoding.  

```raku
# Write with UTF-8 (default)
'utf8.txt'.IO.spurt("Héllo Wörld\n");

# Read with explicit encoding
my $text = open('utf8.txt', :r, :enc<utf8>).slurp-rest;
say $text;

# Write Latin-1
'latin1.txt'.IO.spurt("caf\xE9\n", :enc<latin-1>);
my $lat  = 'latin1.txt'.IO.slurp(:enc<latin-1>);
say $lat;
```

Raku defaults to UTF-8 for all text IO. The `:enc` adverb overrides  
the encoding on both `open` and `slurp`. Mismatched encodings produce  
garbled output or decoding errors. `slurp-rest` reads the remaining  
content of an already-open handle.  

## Lock a file

Using advisory file locks to prevent concurrent access.  

```raku
my $fh = open 'counter.txt', :w;

$fh.lock;                       # exclusive lock
$fh.say(42);
$fh.unlock;

$fh.close;

# Shared (read) lock
my $rfh = open 'counter.txt', :r;
$rfh.lock(:shared);
say $rfh.slurp-rest;
$rfh.unlock;
$rfh.close;
```

`IO::Handle.lock` obtains an exclusive advisory lock by default,  
preventing other processes from locking the file simultaneously. Pass  
`:shared` for a shared (read) lock that allows multiple concurrent  
readers. Locks are advisory: processes that do not use `lock` can  
still access the file, so all cooperating processes must use locking  
consistently.  

## Temporary files

Creating files in the system's temporary directory.  

```raku
my $tmp-dir  = $*TMPDIR;
say "Temp dir: $tmp-dir";

my $tmp-file = $tmp-dir.add("myapp-{$*PID}.tmp");
$tmp-file.spurt("temporary content");

say $tmp-file;
say $tmp-file.slurp;

$tmp-file.unlink;
say "Cleaned up: ", !$tmp-file.e;
```

`$*TMPDIR` is a dynamic variable that holds the system temporary  
directory as an `IO::Path` (e.g. `/tmp` on Unix). Including `$*PID`  
(the process ID) in the filename avoids collisions between concurrent  
processes. Always clean up temporary files in a `LEAVE` phaser or an  
exception handler for robustness.  

## Run external command

Executing an external program and capturing its output.  

```raku
my $proc = run 'ls', '-l', '/etc', :out, :err;

my $stdout = $proc.out.slurp(:close);
my $stderr = $proc.err.slurp(:close);

say "Exit code: ", $proc.exitcode;
say "Output lines: ", $stdout.lines.elems;
say $stdout.lines.head(5).join("\n");
```

`run` launches an external program and returns a `Proc` object. The  
`:out` and `:err` adverbs capture standard output and standard error  
as `IO::Handle` objects. Calling `.slurp(:close)` reads all output  
and closes the handle. `.exitcode` gives the process exit status.  

## Pipe to external command

Sending data to an external process via its standard input.  

```raku
my $proc = run 'sort', '-r', :in, :out;

$proc.in.say("banana");
$proc.in.say("apple");
$proc.in.say("cherry");
$proc.in.close;

say $proc.out.slurp(:close);
```

Passing `:in` gives you a writable `IO::Handle` connected to the  
process's standard input. Writing lines and closing the handle signals  
end-of-file to the process, allowing it to finish. Here `sort -r`  
reverse-sorts the three fruit names.  

## Shell command

Running a shell command string with `shell`.  

```raku
my $result = shell 'echo "Hello from shell" | tr a-z A-Z', :out;
say $result.out.slurp(:close);

# Check success
my $check = shell 'test -f /etc/hosts';
say "hosts file exists: ", $check.exitcode == 0;
```

`shell` passes its argument to the system shell (`/bin/sh -c` on Unix)  
enabling shell features like pipes, redirection, and globbing. The  
return value is the same `Proc` type as `run`. Use `run` with explicit  
arguments when possible to avoid shell injection vulnerabilities.  

## Redirect output to file

Sending a program's output directly to a file.  

```raku
my $log = open 'run.log', :w;

my $proc = run 'ls', '-la', '/tmp', :out($log);

$log.close;

say "Log written: ", 'run.log'.IO.s, " bytes";
say 'run.log'.IO.lines.head(3).join("\n");
```

Passing an open `IO::Handle` as the value of `:out` redirects the  
subprocess's stdout directly into the file without buffering in  
memory. This is efficient for large outputs. The same technique works  
with `:err` to redirect standard error.  

## IO::Socket TCP client

Connecting to a remote server over TCP and reading a response.  

```raku
use IO::Socket::INET;

my $sock = IO::Socket::INET.new(
    host => 'example.com',
    port => 80,
);

$sock.print("GET / HTTP/1.0\r\nHost: example.com\r\n\r\n");

my $response = $sock.recv(512);
say $response.lines.head(5).join("\n");

$sock.close;
```

`IO::Socket::INET` is Raku's built-in TCP socket class. `.new` opens  
a connection to the given host and port. `.print` sends raw bytes, and  
`.recv` reads up to the given number of bytes. Always call `.close`  
when done to release the OS socket descriptor.  

## IO::Socket TCP server

Accepting incoming connections on a TCP port.  

```raku
use IO::Socket::INET;

my $server = IO::Socket::INET.new(
    :localhost<127.0.0.1>,
    :localport(9999),
    :listen,
);

say "Listening on port 9999 ...";

my $client = $server.accept;
my $msg    = $client.recv(1024);
say "Received: $msg";
$client.print("Echo: $msg");
$client.close;

$server.close;
```

Passing `:listen` creates a listening server socket. `.accept` blocks  
until a client connects and returns a new `IO::Handle`-like socket for  
that client. Reading and writing the client socket communicates with  
the remote end. For multiple clients, wrap the accept loop in a  
`start` block or use the `Cro` framework.  

## Read from STDIN lazily

Processing piped input line by line from standard input.  

```raku
for $*IN.lines -> $line {
    my $upper = $line.uc;
    say $upper;
}
```

`$*IN.lines` returns a lazy sequence of lines read from standard input.  
This makes the program behave like a Unix filter: it reads from a pipe  
or terminal and writes transformed output. The program exits naturally  
when standard input reaches EOF. Run as `echo "hello" | raku upper.raku`  
to test.  

## Slurp STDIN

Reading all of standard input into a single string.  

```raku
my $all = $*IN.slurp;
my @lines = $all.lines;

say "Lines: ",     @lines.elems;
say "Words: ",     $all.words.elems;
say "Characters: ", $all.chars;
```

Calling `.slurp` on `$*IN` reads until EOF, which occurs when the  
user presses Ctrl-D (Unix) or Ctrl-Z (Windows), or when a pipe  
closes. The entire input is held in memory as a string, making  
subsequent operations like `.lines` and `.words` fast but unsuitable  
for very large inputs.  

## IO supply and react

Reacting to file-handle events asynchronously.  

```raku
my $proc = Proc::Async.new('ping', '-c', '4', 'localhost');

react {
    whenever $proc.stdout.lines -> $line {
        say "out: $line";
    }
    whenever $proc.stderr.lines -> $line {
        note "err: $line";
    }
    whenever $proc.start -> $status {
        say "exit: ", $status.exitcode;
        done;
    }
}
```

`Proc::Async` is the asynchronous counterpart to `run`. It exposes  
`.stdout` and `.stderr` as `Supply` objects that emit lines as they  
arrive. The `react`/`whenever` construct subscribes to these supplies  
and handles each event until `done` is called. This is ideal for  
long-running processes where you want interleaved output.  
