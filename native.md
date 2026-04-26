# The native interface



## Curl example

fetch title of example.com

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

## GTK 3 example

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


