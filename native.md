# The native interface



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
