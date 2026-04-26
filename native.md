# The native interface

## Introduction

NativeCall is Raku's built-in foreign function interface (FFI) that lets  
you call functions in shared C libraries directly from Raku code. Activate  
it with `use NativeCall` and then declare bindings for the C functions  
you need using the `is native` trait.  

NativeCall bridges Raku's high-level, garbage-collected world with native  
C code. It handles type marshalling between Raku and C types, memory  
layout for structs, pointer passing, callbacks, and library loading.  

Key types and tools provided by NativeCall:  

- **`is native`** – binds a Raku sub declaration to a C function  
- **Native integer types** – `int8`, `int16`, `int32`, `int64`, `uint8`,  
  `uint16`, `uint32`, `uint64`  
- **Native float types** – `num32` (C `float`), `num64` (C `double`)  
- **`Str`** – automatically marshalled to/from a C `char *` (null-terminated)  
- **`Pointer`** – an opaque raw C pointer (`void *`)  
- **`CArray[T]`** – a C-compatible fixed array of native-type elements  
- **`CStruct`** – a Raku class laid out in memory exactly like a C struct  
- **`CUnion`** – a Raku class laid out like a C union  
- **`nativecast`** – reinterprets a raw `Pointer` as a typed value  
- **`is symbol`** – maps a Raku sub name to a different C symbol name  

When you declare a sub with `is native`, Raku looks up the symbol in the  
specified shared library at the first call and calls the C ABI directly.  
No C glue code or wrapper modules are needed.  

NativeCall supports virtually every C library — from standard libc  
utilities and math functions to SQLite, cURL, and GTK — all from pure  
Raku, with no additional C extensions required.  

## Printf function

The `printf` function from the C standard library is bound and called  
directly using NativeCall.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub printf(Str $format, Str $name, int32 $age) returns int32 is native { * }

my $name = "John Doe";
my $age  = 34;

printf("Hello, %s! You are %d years old.\n", $name, $age);
```

Passing no library name to `is native` resolves to the C standard library  
(`libc`) on the current platform. The return type `int32` mirrors C's `int`.  
Raku converts a `Str` parameter to a null-terminated `char *` automatically.  
Because NativeCall does not support true variadic C functions, a separate  
binding must be written for each argument-count variation of `printf`.  

## Curl example

The example fetches the HTML of `example.com` using libcurl and extracts  
the page title from the response.  

```raku
#!/usr/bin/env raku

use NativeCall;

constant CURLOPT_URL           = 10000 + 2;
constant CURLOPT_WRITEFUNCTION = 20000 + 11;

sub curl_easy_init() returns Pointer is native('curl', v4) { * }
sub curl_easy_setopt_str(Pointer, int32, Pointer) returns int32  is native('curl', v4) is symbol('curl_easy_setopt') { * }
sub curl_easy_setopt_long(Pointer, int32, int64) returns int32  is native('curl', v4) is symbol('curl_easy_setopt') { * }
sub curl_easy_setopt_cb(Pointer $curl, int32 $opt, &callback (Pointer, size_t, size_t, Pointer --> size_t))
                     returns int32 is native('curl', v4) is symbol('curl_easy_setopt') { * }
sub curl_easy_perform(Pointer) returns int32  is native('curl', v4) { * }
sub curl_easy_cleanup(Pointer) is native('curl', v4) { * }

my $content = '';

my &write-callback = sub (Pointer $ptr, size_t $size, size_t $nmemb, Pointer --> size_t) {
    my $bytes = $size * $nmemb;
    $content ~= nativecast(Str, $ptr).substr(0, $bytes);
    $bytes;
}

my $curl = curl_easy_init();

my $url = 'https://example.com';
my $url-buf = CArray[uint8].new($url.encode.list);
$url-buf[$url.chars] = 0;
curl_easy_setopt_str($curl, CURLOPT_URL, nativecast(Pointer, $url-buf));
curl_easy_setopt_cb($curl, CURLOPT_WRITEFUNCTION, &write-callback);

my $ret = curl_easy_perform($curl);
curl_easy_cleanup($curl);

if $ret == 0 {
    $content ~~ /:i '<title>' (.*?) '</title>' /;
    say "Title: {~$0}" if $0;
} else {
    say "Error: curl_easy_perform returned $ret";
}
```

Three separate NativeCall bindings are created for `curl_easy_setopt` using  
`is symbol` because each variant accepts a different third argument type.  
The write callback is a native-compatible Raku closure: NativeCall converts  
it to a C function pointer automatically. `nativecast(Str, $ptr)` reinterprets  
the raw `Pointer` delivered by libcurl as a Raku string so the data can be  
appended to `$content`.  

## GTK 3 example

The first variant creates a GTK window with a single button placed at a  
fixed position using `GtkFixed`.  

```raku
use NativeCall;

# --- Library Definitions ---
constant LIBGTK = ('gtk-3', v0);
constant LIBGOBJECT = ('gobject-2.0', v0);

constant GTK_WIN_POS_CENTER = 1;

# --- GTK Function Signatures ---
sub gtk_init(Pointer, Pointer) is native(LIBGTK) { * }
sub gtk_window_new(int32) returns Pointer is native(LIBGTK) { * }
sub gtk_window_set_title(Pointer, Str) is native(LIBGTK) { * }
sub gtk_window_set_default_size(Pointer, int32, int32) is native(LIBGTK) { * }
sub gtk_window_set_position(Pointer, int32) is native(LIBGTK) { * }


# Fixed container for manual positioning
sub gtk_fixed_new() returns Pointer is native(LIBGTK) { * }
sub gtk_fixed_put(Pointer, Pointer, int32, int32) is native(LIBGTK) { * }

sub gtk_button_new_with_label(Str) returns Pointer is native(LIBGTK) { * }
sub gtk_container_add(Pointer, Pointer) is native(LIBGTK) { * }
sub gtk_widget_show_all(Pointer) is native(LIBGTK) { * }
sub gtk_main() is native(LIBGTK) { * }
sub gtk_main_quit() is native(LIBGTK) { * }

# --- GObject Signal Connection ---
sub g_signal_connect_data(
    Pointer $instance,
    Str     $detailed_signal,
    &handler (Pointer, Pointer),
    Pointer $data,
    Pointer $destroy_data,
    int32   $connect_flags
) returns long is native(LIBGOBJECT) { * }

# --- Signal Handlers ---
sub on_quit(Pointer $widget, Pointer $data) {
    say "Quit button clicked!";
    gtk_main_quit();
}

sub on_destroy(Pointer $widget, Pointer $data) {
    say "Window closed. Quitting...";
    gtk_main_quit();
}

# --- Main Program ---
gtk_init(Pointer, Pointer);

my $window = gtk_window_new(0);
gtk_window_set_title($window, "Raku GTK Top-Left Button");
gtk_window_set_default_size($window, 550, 300);

# Create a Fixed container
my $fixed = gtk_fixed_new();
gtk_container_add($window, $fixed);

my $button = gtk_button_new_with_label("Quit");

# Put the button at (x=0, y=0) in the fixed container
gtk_fixed_put($fixed, $button, 15, 15);

# Connect signals
g_signal_connect_data($button, "clicked", &on_quit, Pointer, Pointer, 0);
g_signal_connect_data($window, "destroy", &on_destroy, Pointer, Pointer, 0);


gtk_widget_show_all($window);
gtk_window_set_position($window, GTK_WIN_POS_CENTER);
gtk_main();
```

Signal handlers are plain Raku subs passed by reference to  
`g_signal_connect_data`, which is the real C function behind the  
`g_signal_connect` macro. Constants like `GTK_WIN_POS_CENTER` are  
declared in Raku to mirror their C enum values.  

The second variant uses a `GtkBox` layout to place the button with  
standard GTK alignment rather than absolute coordinates.  

```raku
#!/usr/bin/env raku

# Simple GTK3 application using NativeCall
# Demonstrates: window, button, signal handling, and GTK main loop

use v6;
use NativeCall;

# Suppress broken GIO module warnings injected by VS Code's snap environment
%*ENV<GIO_MODULE_DIR> = '/usr/lib/x86_64-linux-gnu/gio/modules';
# --- GTK3/GObject NativeCall bindings ---

constant LIBGTK = 'libgtk-3.so';

sub gtk_init(Pointer $argc, Pointer $argv) returns int32 is native(LIBGTK) { * }
sub gtk_main() is native(LIBGTK) { * }
sub gtk_main_quit() is native(LIBGTK) { * }

sub gtk_window_new(int32) returns Pointer is native(LIBGTK) { * }
sub gtk_window_set_title(Pointer, Str) is native(LIBGTK) { * }
sub gtk_window_set_default_size(Pointer, int32, int32) is native(LIBGTK) { * }
sub gtk_window_set_position(Pointer, int32) is native(LIBGTK) { * }

sub gtk_button_new_with_label(Str) returns Pointer is native(LIBGTK) { * }

sub gtk_box_new(int32 $orientation, int32 $spacing) returns Pointer is native(LIBGTK) { * }
sub gtk_box_pack_start(Pointer $box, Pointer $child, int32 $expand, int32 $fill, int32 $padding) is native(LIBGTK) { * }
sub gtk_box_pack_end(Pointer $box, Pointer $child, int32 $expand, int32 $fill, int32 $padding) is native(LIBGTK) { * }

sub gtk_container_add(Pointer, Pointer) is native(LIBGTK) { * }

sub gtk_widget_show_all(Pointer) is native(LIBGTK) { * }

# g_signal_connect_data is the real function (g_signal_connect is a C macro)
# signature: instance, detailed_signal, c_handler, data, destroy_data, connect_flags
# NativeCall has no void return; use int32 — GTK ignores signal handler return values
sub g_signal_connect_data(Pointer, Str, &handler (Pointer, Pointer --> int32), Pointer, Pointer, int32)
    returns uint64
    is native('libgobject-2.0.so')
{ * }

# --- Callbacks ---

# GTK callback signature: void callback(GtkWidget*, gpointer)
# Must handle exceptions to avoid process termination
sub on-destroy(Pointer $widget, Pointer $data --> int32) {
    gtk_main_quit();
    CATCH { default { say "Error in destroy callback: $_" } }
    return 0;
}

sub on-click(Pointer $widget, Pointer $data --> int32) {
    gtk_main_quit();
    CATCH { default { say "Error in click callback: $_" } }
    return 0;
}

# --- Main ---

# Initialize GTK
gtk_init(Pointer.new, Pointer.new);

# Create main window
my $window = gtk_window_new(0);  # GTK_WINDOW_TOPLEVEL = 0
gtk_window_set_title($window, "Raku GTK Demo");
gtk_window_set_default_size($window, 300, 150);
gtk_window_set_position($window, 1);  # GTK_WIN_POS_CENTER = 1

# Create button
my $button = gtk_button_new_with_label("Quit");

# Layout: vertical box fills the window
# GTK_ORIENTATION_VERTICAL = 1, GTK_ORIENTATION_HORIZONTAL = 0
my $vbox = gtk_box_new(1, 0);
my $hbox = gtk_box_new(0, 0);

# Pack button to the right end of hbox (bottom-right)
gtk_box_pack_end($hbox, $button, 0, 0, 8);

# Pack hbox at the bottom of vbox (no expand), with spacer above (expand)
my $spacer = gtk_box_new(0, 0);  # empty widget as spacer
gtk_box_pack_start($vbox, $spacer, 1, 1, 0);
gtk_box_pack_end($vbox, $hbox, 0, 0, 8);

gtk_container_add($window, $vbox);

# Connect signals: pass Raku subs as function pointers
g_signal_connect_data($window, "destroy", &on-destroy, Pointer.new, Pointer.new, 0);
g_signal_connect_data($button, "clicked", &on-click, Pointer.new, Pointer.new, 0);

# Show all widgets
gtk_widget_show_all($window);

# Run GTK main loop
gtk_main();

say "Application exited.";
```

`GtkBox` containers allow packing widgets with expand/fill/padding rules.  
Horizontal and vertical boxes are created with `gtk_box_new`. The button  
is packed at the trailing end of the inner `hbox`, and the `hbox` is  
placed at the bottom of the outer `vbox` with a spacer above it.  

## Calling abs

The `abs` function from the C standard library returns the absolute  
value of an integer.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub abs(int32 $n) returns int32 is native { * }

say abs(-42);    # 42
say abs(7);      # 7
say abs(-100);   # 100
```

Binding to `abs` shows the simplest possible NativeCall pattern: one  
integer in, one integer out. `is native` with no argument loads the symbol  
from the default C runtime library. The Raku `int32` type maps directly  
to the C `int` type used in the original function signature.  

## Math functions

The C `libm` math library exposes trigonometric and power functions that  
operate on double-precision floating-point values.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub sin(num64 $x) returns num64 is native('m') { * }
sub cos(num64 $x) returns num64 is native('m') { * }
sub sqrt(num64 $x) returns num64 is native('m') { * }
sub pow(num64 $base, num64 $exp) returns num64 is native('m') { * }

say sin(0e0);          # 0
say cos(0e0);          # 1
say sqrt(2e0);         # 1.4142135623730951
say pow(2e0, 10e0);    # 1024
```

The library name `'m'` maps to `libm.so` on Linux and is resolved  
automatically by NativeCall. `num64` corresponds to C's `double`. Numeric  
literals must carry the `e` exponent suffix so Raku treats them as  
`num64` rather than `Int` when calling native-typed parameters.  

## strlen

`strlen` returns the number of bytes in a null-terminated C string,  
not counting the terminating zero byte.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub strlen(Str $s) returns size_t is native { * }

say strlen("hello");          # 5
say strlen("Raku");           # 4
say strlen("");               # 0
say strlen("日本語");          # 9  (UTF-8 bytes, not characters)
```

NativeCall marshals the Raku `Str` to a `char *` before the call.  
The return type `size_t` is a Raku built-in alias for the platform word  
size (`uint32` or `uint64`). Because C strings are byte arrays, `strlen`  
counts UTF-8 bytes; multi-byte characters produce counts greater than  
their character count.  

## strcmp

`strcmp` lexicographically compares two C strings and returns zero when  
they are equal, a negative value when the first is less, or a positive  
value when the first is greater.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub strcmp(Str $a, Str $b) returns int32 is native { * }

say strcmp("apple", "apple");    # 0
say strcmp("apple", "banana");   # negative
say strcmp("banana", "apple");   # positive

if strcmp("hello", "hello") == 0 {
    say "strings are equal";
}
```

`strcmp` is useful when you need exact C-compatible string ordering,  
for example when working with sorted C data structures. The return value  
is `int32` matching C's `int`. In pure Raku you would normally use  
the `eq` operator, but binding `strcmp` illustrates passing two `Str`  
arguments to the same native function.  

## sprintf

`sprintf` formats a string into a caller-supplied buffer. A `CArray`  
of `uint8` acts as the writable byte buffer passed to C.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub sprintf_int(Pointer $buf, Str $fmt, int32 $n)
    returns int32 is native is symbol('sprintf') { * }

my $buf = CArray[uint8].new(0 xx 64);
sprintf_int(nativecast(Pointer, $buf), "Value: %d\n", 42);

say nativecast(Str, nativecast(Pointer, $buf));
```

Because `sprintf` is variadic, a fixed-argument binding is written for  
each format variation. The output buffer is a `CArray[uint8]` pre-filled  
with 64 zero bytes, then cast to a raw `Pointer` for the call. After  
the call, `nativecast(Str, ...)` reads the buffer back as a Raku string.  

## getpid

`getpid` returns the process identifier of the calling process.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub getpid() returns int32 is native { * }
sub getppid() returns int32 is native { * }

say "PID:  { getpid()  }";
say "PPID: { getppid() }";
```

Process-info functions take no arguments, making their NativeCall  
bindings trivial. `getppid` returns the parent process ID. Both  
functions are part of POSIX and reside in the default C runtime.  

## getenv

`getenv` looks up an environment variable by name and returns a pointer  
to its value string, or a null `Pointer` if the variable is not set.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub getenv(Str $name) returns Str is native { * }

for <HOME USER SHELL PATH> -> $var {
    my $val = getenv($var);
    say "$var = { $val // '(not set)' }";
}
```

Declaring the return type as `Str` tells NativeCall to read the  
returned `char *` as a null-terminated string directly. If the C  
function returns a null pointer, NativeCall maps it to Raku's  
undefined value, so the `//` defined-or operator works naturally.  

## time function

The `time` function returns the number of seconds elapsed since the  
Unix epoch (1970-01-01 00:00:00 UTC).  

```raku
#!/usr/bin/env raku

use NativeCall;

sub time(Pointer $tloc) returns int64 is native { * }

my $now = time(Pointer);
say "Seconds since epoch: $now";

my $later = time(Pointer);
say "Elapsed: { $later - $now } second(s)";
```

Passing `Pointer` (the type object, not an instance) for the `tloc`  
argument is the conventional way to pass a null pointer in NativeCall.  
The return type is `int64` to accommodate 64-bit time values and avoid  
the year-2038 problem on 64-bit systems.  

## rand and srand

`srand` seeds the pseudo-random number generator and `rand` returns  
successive pseudo-random integers in the range `[0, RAND_MAX]`.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub srand(uint32 $seed) is native { * }
sub rand() returns int32 is native { * }

srand(42);
say rand() for ^5;
```

Seeding with a constant produces a reproducible sequence, useful in  
tests. In production code you would seed with the current time. Both  
functions live in the C standard library and need no library name  
argument to `is native`.  

## Memory allocation

`malloc` allocates a block of uninitialized memory and returns a raw  
pointer; `free` releases it back to the allocator.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub malloc(size_t $size) returns Pointer is native { * }
sub free(Pointer $ptr)   is native { * }
sub memset(Pointer $ptr, int32 $c, size_t $n) returns Pointer is native { * }

my $ptr = malloc(16);
memset($ptr, 0, 16);
say "Allocated 16 bytes at { $ptr.defined ?? 'a valid address' !! 'null' }";
free($ptr);
```

`malloc` returns a typed `Pointer` that you must release with `free`  
when done; NativeCall does not track manually allocated memory. `memset`  
fills the block with a byte value (here zero). Always check that the  
returned pointer is defined before using it.  

## File operations

Standard C file I/O functions (`fopen`, `fprintf`, `fclose`) can be  
called through NativeCall for low-level file access.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub fopen(Str $path, Str $mode)     returns Pointer is native { * }
sub fprintf_str(Pointer $fp, Str $fmt, Str $s)
    returns int32 is native is symbol('fprintf') { * }
sub fclose(Pointer $fp)             returns int32 is native { * }

my $fp = fopen('/tmp/native-test.txt', 'w');
if $fp {
    fprintf_str($fp, "Hello from %s\n", "Raku NativeCall");
    fclose($fp);
    say "File written.";
} else {
    say "Could not open file.";
}
```

`fopen` returns a `FILE *` as an opaque `Pointer`. Because `fprintf` is  
variadic, a dedicated binding for the string-argument case is created  
with `is symbol`. Failing to call `fclose` leaks the OS file descriptor.  

## CArray type

`CArray[T]` represents a C-compatible fixed-size array of a single  
native element type.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub qsort(Pointer $base, size_t $nmemb, size_t $size,
          &compar (Pointer, Pointer --> int32)) is native { * }

my $arr = CArray[int32].new(5, 3, 8, 1, 9, 2, 7, 4, 6);
my $n   = $arr.elems;

qsort(nativecast(Pointer, $arr), $n, nativesizeof(int32),
    -> Pointer $a, Pointer $b --> int32 {
        (nativecast(CArray[int32], $a)[0]
         <=> nativecast(CArray[int32], $b)[0]).Int
    });

say $arr[$_] for ^$n;
```

`CArray.new` accepts a list of initial values. `nativesizeof(T)` returns  
the byte size of a native type, needed here for the element size argument  
to `qsort`. The comparison callback casts the opaque `Pointer` arguments  
back to `CArray[int32]` to dereference the element values.  

## CStruct type

A Raku class declared `is repr('CStruct')` has the same memory layout  
as the equivalent C struct and can be passed to or returned from C  
functions.  

```raku
#!/usr/bin/env raku

use NativeCall;

class Point is repr('CStruct') {
    has num64 $.x;
    has num64 $.y;
}

sub hypot(num64 $x, num64 $y) returns num64 is native('m') { * }

my $p = Point.new(x => 3e0, y => 4e0);
say "Distance from origin: { hypot($p.x, $p.y) }";    # 5

my $q = Point.new(x => -1e0, y => -1e0);
say "Distance: { hypot($q.x, $q.y) }";
```

Fields declared with `has` in a `CStruct` are laid out in declaration  
order without padding adjustments by Raku (the C ABI padding rules  
apply). Here the struct is read-only and its fields are passed to `hypot`  
individually; a `CStruct` pointer can also be passed directly when a C  
function expects a `struct *` argument.  

## Pointer type

`Pointer` is NativeCall's opaque pointer type (`void *`). It can be  
typed as `Pointer[T]` to carry element-type information for  
`nativecast`.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub malloc(size_t $size)          returns Pointer is native { * }
sub free(Pointer $p)              is native { * }

# Write an int32 into the allocation
sub memcpy(Pointer $dst, Pointer $src, size_t $n)
    returns Pointer is native { * }

my $src = CArray[int32].new(123);
my $dst = malloc(nativesizeof(int32));

memcpy($dst, nativecast(Pointer, $src), nativesizeof(int32));

my $result = nativecast(CArray[int32], $dst);
say $result[0];    # 123

free($dst);
```

`Pointer` values cannot be dereferenced in Raku directly; use `nativecast`  
to reinterpret the raw pointer as a typed `CArray` or `CStruct`. Here  
`memcpy` copies one `int32`-worth of bytes from a `CArray` source buffer  
to a `malloc`-allocated destination.  

## Out parameters

Many C functions return results through pointer arguments. A `Pointer[T]`  
parameter passed by reference acts as an out parameter.  

```raku
#!/usr/bin/env raku

use NativeCall;

# div_t struct: quotient and remainder
class DivResult is repr('CStruct') {
    has int32 $.quot;
    has int32 $.rem;
}

sub div(int32 $numer, int32 $denom) returns DivResult is native { * }

my $r = div(17, 5);
say "17 ÷ 5 = { $r.quot } remainder { $r.rem }";   # 3 remainder 2

my $s = div(100, 7);
say "100 ÷ 7 = { $s.quot } remainder { $s.rem }";  # 14 remainder 2
```

The C function `div` returns a `div_t` struct by value. In NativeCall the  
corresponding return type is declared as a `CStruct` subclass. The field  
names and order match those of the C struct definition so the data is  
interpreted correctly.  

## is symbol trait

The `is symbol` trait maps a Raku sub name to a different underlying  
C symbol, letting you give native functions friendlier Raku names.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub c-abs(int32 $n)      returns int32 is native is symbol('abs')      { * }
# labs returns the absolute value of a long (int64) integer
sub c-labs(int64 $n)     returns int64 is native is symbol('labs')     { * }
sub c-strlen(Str $s)     returns size_t is native is symbol('strlen')  { * }

say c-abs(-99);                # 99
say c-labs(-9_999_999_999);    # 9999999999  (absolute value of -9_999_999_999)
say c-strlen("Raku");          # 4
```

`is symbol` is essential when a C symbol name clashes with a Raku  
built-in (e.g. `abs`), when multiple C functions share a name with  
different signatures, or simply when you prefer a more descriptive Raku  
name. The string argument is the exact symbol looked up in the shared  
library.  

## Library versioning

`is native` accepts an optional version number after the library name  
to request a specific shared-library version.  

```raku
#!/usr/bin/env raku

use NativeCall;

# Request libcurl version 4 explicitly
sub curl_version() returns Str is native('curl', v4) { * }

say curl_version();
```

On Linux, `is native('curl', v4)` resolves to `libcurl.so.4`. Specifying  
the version prevents accidental linking to an incompatible major release.  
Omit the version to let the dynamic linker use its default resolution  
(typically the highest installed version via the unversioned symlink).  

## Native sized types

NativeCall provides exact-width integer and float types that correspond  
directly to C types.  

```raku
#!/usr/bin/env raku

use NativeCall;

my int8   $i8  = 127;
my int16  $i16 = 32767;
my int32  $i32 = 2_147_483_647;
my int64  $i64 = 9_223_372_036_854_775_807;

my uint8  $u8  = 255;
my uint16 $u16 = 65535;
my uint32 $u32 = 4_294_967_295;
my uint64 $u64 = 18_446_744_073_709_551_615;

my num32  $f32 = 3.14e0;
my num64  $f64 = 3.141592653589793e0;

say "$i8  $i16  $i32  $i64";
say "$u8  $u16  $u32  $u64";
say "$f32  $f64";
```

Use the narrowest type that fits your data to ensure that the C ABI  
receives the correct byte width. Passing a Raku `Int` where a C function  
expects `int8` without a native type would risk truncation or sign errors.  
`num32` and `num64` correspond to C's `float` and `double`.  

## Callbacks

NativeCall converts a Raku block or sub with a matching signature to a  
C function pointer when passed to a parameter declared with `&`.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub atexit(&handler ()) returns int32 is native { * }

atexit(sub { say "Goodbye from atexit!" });

say "Program continues...";
# "Goodbye from atexit!" is printed when the process exits normally
```

The `&handler ()` parameter declaration tells NativeCall that the  
argument must be a function pointer with no parameters. Raku's GC keeps  
the callback alive for the lifetime of the process. For callbacks invoked  
by a long-lived C library (e.g. GUI toolkits), store a reference to the  
Raku sub in a module-level variable to prevent premature collection.  

## nativecast

`nativecast` reinterprets a raw `Pointer` as a typed NativeCall value,  
similar to a C cast.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub malloc(size_t $n) returns Pointer is native { * }
sub free(Pointer $p)  is native { * }

# Allocate space for 4 int32 values
my $ptr = malloc(4 * nativesizeof(int32));

# View it as a CArray[int32]
my $arr = nativecast(CArray[int32], $ptr);
$arr[$_] = ($_ + 1) * 10 for ^4;

say $arr[$_] for ^4;    # 10  20  30  40

free($ptr);
```

`nativecast(T, $pointer)` does not copy data; it creates a new Raku  
object that points to the same memory with type `T`. It works with  
`CStruct`, `CArray[T]`, `Str`, and `Pointer` targets. Reading beyond  
the allocated size is undefined behaviour exactly as in C.  

## errno

The C global variable `errno` records the error code set by the most  
recently failed system call. NativeCall exposes it through a helper  
function.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub fopen(Str $path, Str $mode) returns Pointer is native { * }
sub strerror(int32 $errnum)     returns Str     is native { * }

# Access errno via a wrapper (errno is a macro in C, not a simple global)
sub get_errno() returns int32
    is native is symbol('__errno_location') { * }

my $fp = fopen('/nonexistent/path/file.txt', 'r');
unless $fp {
    my $err = nativecast(Pointer[int32], get_errno())[0];
    say "Error $err: { strerror($err) }";
}
```

On Linux, `errno` is actually a thread-local accessed through  
`__errno_location()` which returns a pointer to the current value.  
`strerror` converts the numeric code to a human-readable message.  
On macOS the function is named `__error`; wrap the difference in  
an `if $*KERNEL.name eq 'linux'` guard for portable code.  

## system call

The C `system` function executes a shell command and returns its exit  
status.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub system(Str $command) returns int32 is native { * }

my $ret = system('echo "Hello from the shell"');
say "Exit status: $ret";

$ret = system('ls /tmp | head -5');
say "ls exit status: $ret";
```

`system` blocks until the command completes and returns the value that  
the shell's `waitpid` call produces (exit code shifted left by 8 bits  
on POSIX). This is the simplest way to run a shell command from a C  
binding. For more control, bind `popen`/`pclose` or use Raku's built-in  
`run` instead.  

## SQLite

SQLite is a self-contained C library for embedded relational databases.  
NativeCall can bind its API directly without any Raku wrapper module.  

```raku
#!/usr/bin/env raku

use NativeCall;

constant SQLITE_OK   = 0;
constant SQLITE_ROW  = 100;
constant SQLITE_DONE = 101;

sub sqlite3_open(Str, Pointer $db is rw) returns int32 is native('sqlite3') { * }
sub sqlite3_close(Pointer)              returns int32 is native('sqlite3') { * }
sub sqlite3_exec(Pointer, Str, Pointer, Pointer, Pointer)
    returns int32 is native('sqlite3') { * }
sub sqlite3_prepare_v2(Pointer, Str, int32, Pointer $stmt is rw, Pointer)
    returns int32 is native('sqlite3') { * }
sub sqlite3_step(Pointer)               returns int32 is native('sqlite3') { * }
sub sqlite3_column_text(Pointer, int32) returns Str   is native('sqlite3') { * }
sub sqlite3_finalize(Pointer)           returns int32 is native('sqlite3') { * }

my Pointer $db;
sqlite3_open(':memory:', $db);

sqlite3_exec($db,
    'CREATE TABLE t(id INTEGER PRIMARY KEY, name TEXT);
     INSERT INTO t(name) VALUES("Alice"),("Bob"),("Carol");',
    Pointer, Pointer, Pointer);

my Pointer $stmt;
sqlite3_prepare_v2($db, 'SELECT id, name FROM t ORDER BY id', -1, $stmt, Pointer);

while sqlite3_step($stmt) == SQLITE_ROW {
    say "{ sqlite3_column_text($stmt, 0) } | { sqlite3_column_text($stmt, 1) }";
}

sqlite3_finalize($stmt);
sqlite3_close($db);
```

The `Pointer $db is rw` and `Pointer $stmt is rw` parameters are  
passed by reference so that sqlite3 can fill them in. Constants mirror  
the integer values defined in `sqlite3.h`. `sqlite3_column_text` returns  
the column value as a `Str` by declaring `returns Str`.  

## Wrapping native calls in a class

Encapsulating NativeCall bindings inside a Raku class provides a  
clean, object-oriented interface and hides raw pointer management.  

```raku
#!/usr/bin/env raku

use NativeCall;

# Low-level bindings (kept private to the module)
my sub _open(Str, Pointer $db is rw)  returns int32 is native('sqlite3')
    is symbol('sqlite3_open')  { * }
my sub _close(Pointer)                returns int32 is native('sqlite3')
    is symbol('sqlite3_close') { * }
my sub _exec(Pointer, Str, Pointer, Pointer, Pointer)
    returns int32 is native('sqlite3') is symbol('sqlite3_exec') { * }

class SQLiteDB {
    has Pointer $!db;

    method open(Str $path) {
        my Pointer $h;
        my $rc = _open($path, $h);
        die "Cannot open: $rc" if $rc != 0;
        $!db = $h;
        self
    }

    method exec(Str $sql) {
        _exec($!db, $sql, Pointer, Pointer, Pointer);
    }

    method close() { _close($!db) }
}

my $db = SQLiteDB.new.open(':memory:');
$db.exec('CREATE TABLE t(x INTEGER); INSERT INTO t VALUES(1),(2),(3);');
say "Database ready.";
$db.close;
```

The raw NativeCall subs are declared with `my` to prevent them from  
leaking into the caller's namespace. The class `open` method initialises  
the opaque `$!db` handle and returns `self` for chaining. This pattern  
makes native libraries feel like first-class Raku objects.  

## CUnion type

A Raku class declared `is repr('CUnion')` overlaps all its fields at  
offset zero, exactly like a C union.  

```raku
#!/usr/bin/env raku

use NativeCall;

class FloatBits is repr('CUnion') {
    has num32 $.f;
    has uint32 $.bits;
}

my $u = FloatBits.new(f => 1e0.Num);
say $u.f.raku;         # 1e0
printf "0x%08X\n", $u.bits;    # 0x3F800000  (IEEE 754 representation of 1.0)

$u = FloatBits.new(bits => 0x40000000);
say $u.f;              # 2  (0x40000000 is 2.0 in IEEE 754)
```

`CUnion` uses `is repr('CUnion')` instead of `is repr('CStruct')`. Both  
fields share the same 4 bytes of memory: writing `.f` and reading `.bits`  
shows the raw IEEE 754 bit pattern of the float. This is the standard C  
technique for type-punning between float and integer representations.  

## nativesizeof

`nativesizeof` returns the byte size of a native type or a `CStruct`  
instance, mirroring C's `sizeof` operator.  

```raku
#!/usr/bin/env raku

use NativeCall;

class Point is repr('CStruct') {
    has num64 $.x;
    has num64 $.y;
}

say nativesizeof(int8);    # 1
say nativesizeof(int16);   # 2
say nativesizeof(int32);   # 4
say nativesizeof(int64);   # 8
say nativesizeof(num32);   # 4
say nativesizeof(num64);   # 8
say nativesizeof(Point);   # 16  (two num64 fields)
```

`nativesizeof` is essential when calling C functions that require element  
sizes (e.g. `qsort`, `bsearch`, `memcpy`). It accepts both type objects  
and values. For a `CStruct`, it accounts for any padding the C ABI  
inserts between fields.  

## Passing structs by pointer

When a C function expects a `struct *`, declare the parameter as the  
`CStruct` subclass type directly; NativeCall passes the address.  

```raku
#!/usr/bin/env raku

use NativeCall;

class Timeval is repr('CStruct') {
    has long $.tv_sec;
    has long $.tv_usec;
}

sub gettimeofday(Timeval $tv, Pointer $tz) returns int32 is native { * }

my $tv = Timeval.new;
gettimeofday($tv, Pointer);
say "Seconds:      { $tv.tv_sec  }";
say "Microseconds: { $tv.tv_usec }";
```

`Timeval.new` allocates a zeroed `CStruct`. Passing it to a function  
that takes a `struct timeval *` works because NativeCall passes the  
object's internal C-layout buffer by pointer automatically. After the  
call, the struct's fields hold the values written by C.  

## String buffers

When a C function writes into a caller-supplied string buffer, a  
`CArray[uint8]` serves as the mutable byte array.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub gethostname(CArray[uint8] $buf, size_t $len) returns int32 is native { * }

my $buf = CArray[uint8].new(0 xx 256);
gethostname($buf, 256);

# Convert the null-terminated byte array to a Raku Str
my $hostname = Buf.new($buf[^256].grep(* != 0)).decode('utf-8');
say "Hostname: $hostname";
```

`CArray[uint8]` is the NativeCall equivalent of a `char[]` buffer.  
After the call, the filled bytes are extracted up to (but not including)  
the null terminator, wrapped in a `Buf`, and decoded to a Raku `Str`.  
The buffer size is passed as a separate `size_t` argument following the  
common C convention.  

## Variadic-style bindings

C variadic functions like `printf` cannot be bound with a single  
NativeCall declaration. Instead, write one binding per argument  
pattern you need.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub printf_s(Str $fmt, Str $s)
    returns int32 is native is symbol('printf') { * }
sub printf_d(Str $fmt, int32 $n)
    returns int32 is native is symbol('printf') { * }
sub printf_sd(Str $fmt, Str $s, int32 $n)
    returns int32 is native is symbol('printf') { * }
sub printf_f(Str $fmt, num64 $f)
    returns int32 is native is symbol('printf') { * }

printf_s("Name:  %s\n", "Raku");
printf_d("Year:  %d\n", 2024);
printf_sd("User: %s, age %d\n", "Alice", 30);
printf_f("Pi:   %.5f\n", 3.141592653589793e0);
```

Each binding has the same C symbol `printf` via `is symbol` but a  
distinct Raku name and argument list. NativeCall matches the declared  
types to the C ABI calling convention for each variant. This pattern  
generalises to any variadic C API.  

## Lazy library loading

NativeCall loads the shared library and resolves the symbol on the  
first call. You can verify availability with `try` and `CATCH`.  

```raku
#!/usr/bin/env raku

use NativeCall;

sub sqlite3_libversion() returns Str is native('sqlite3') { * }

my $version = try sqlite3_libversion();

if $! {
    say "SQLite not available: $!";
} else {
    say "SQLite version: $version";
}
```

Library loading is deferred until the first call, so declaring native  
subs at the top level does not fail immediately if the library is absent.  
Wrapping the first call in `try` and inspecting `$!` is the recommended  
way to detect missing libraries at runtime and report a meaningful  
diagnostic.  
