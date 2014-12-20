# GRESHUNKEL

This is the C port of the GRESHUNKEL templating langue, used in way too many of
my projects. It's a basic templating language in C that probably performs really
badly in way too many situations, but I don't like any dependencies, so.

# Requirements

* A POSIX compliant system
* A C compiler
* At least 32k of RAM

# Building

Do the standard `./configure && make && make install` dance. Babbys first
autotools.

# Tests

Theres one test, and it incorporates everything. I think. It's built when you
run make:

    ./greshunkel_test

# Usage

The main use case for this is HTML documents, but you can use it for anything.
It just puts strings inside of other strings.

## Contexts
To start, you need to initalize a context:

    greshunkel_ctext *ctext = gshkl_init_context();

This needs to be `free()`'d again later:

    gshkl_free_context(ctext);

## Variables

To stick things in the context, you can use any of the `gshkl_add_*` functions,
of which there are a handful, supporting different datatypes:

    gshkl_add_string(greshunkel_ctext *ctext, const char name[WISDOM_OF_WORDS], const char *value);
    gshkl_add_int(greshunkel_ctext *ctext, const char name[WISDOM_OF_WORDS], const int value);

    greshunkel_var *gshkl_add_array(greshunkel_ctext *ctext, const char name[WISDOM_OF_WORDS]);

Arrays are a little different, but `add_string` and `add_int` are pretty
straight forward. They simple add integers/strings to the context. `float`s are
currently not supported. Cast your int to a float first. I don't care, go cry to
[Grandma Jinja](http://jinja.pocoo.org/docs/dev/).

## Arrays/Lists

Adding an array to a context (with `gshkl_add_array`) gives you back a
`greshunkel_var` pointer, which you can add other things to. I think you can
probably add arrays to this, but I don't remember. I haven't tried it. Not supported.
For example:

    greshunkel_var *loop_test = gshkl_add_array(ctext, "LOOP_TEST");

    gshkl_add_string_to_loop(loop_test, "a");
    gshkl_add_string_to_loop(loop_test, "b");
    gshkl_add_string_to_loop(loop_test, "c");

Hopefully this is self-explanatory. If not, [Grandma Jinja](http://jinja.pocoo.org/docs/dev/).

## Filters

Filters are just functions that can be called on certain chunks of text. They
take text and they return different text. Or the same text. It's up to you!

The `gshkl_add_filter` takes an optional fourth argument, a cleanup callback.
This is incase you want to free anything allocated inside of the filter function
itself.

An example filter:

```C
char *return_hello(const char *arg) {
    return "HELLO!";
}

int main(int argc, char *argv[]) {
    ...
    gshkl_add_filter(ctext, "return_z", &return_z, NULL);
    ...
}
```

## Rendering

Rendering the context is done with the `gshkl_render` function. It gives you
back a heap-allocated pointer, which must be `free()`'d later. The string is
NULL-terminated.

```C
    char *rendered = gshkl_render(ctext, document, strlen(document), &new_size);
    printf("%s\n", rendered);
    free(rendered);
```

# Template Syntax

The template syntax is something I've come to call the 'Xbox Live' syntax, as it
involves repeating patterns of variously capitalized/non-capitalized `X`s.

## Interpolation

Interpolation is handled with the `@` symbol:

    <li>xXx @TEST xXx</li>

This will render whatever is in the `TEST` variable inside of the context.

## Loops

Loops are handled with the `xXx LOOP (loop_variable) (array_to_loop_over) xXx`
syntax. Observe:

    xXx LOOP i LOOP_TEST xXx
        <li>xXx @i xXx</li>
    xXx BBL xXx

This will loop over the array named `LOOP_TEST`, assigning each value to `i` and
printing it out. Pay close attention to the capitalization of the `X`s, it will
be important later.

## Filters

Filters are called by using the inverse capitalization (hahaha), and can be
combined with loops/interpolation:

    <li>XxX return_z xXx @i xXx XxX</li>\n"

This will call the filter `return_z` on the interpolated value of `i`.
Interpolation happens before filtering. Caveat!

# Caveats

- Names of variables can only be 32 `char`s long
- Strings can only be 256 `char`s long
- Does a shitton of allocations
- There are no hashmaps/dictionaries, so it linearly searches through your
  context until it finds the variable you're trying to interpolate
- Crashes on failure (`assert` baby!)

