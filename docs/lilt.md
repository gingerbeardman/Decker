<style>
/* general prose */
body{margin:1em 5em 5em 3em;}
h1,h2,figcaption{font-family:Helvetica Neue,Arial,sans-serif;}
h1{color:#21183c;font-weight:700;}
h2{color:#21183c;font-weight:300;margin-top:1.5em;}
pre,code{background-color:Gainsboro;tab-size:2;font-size:large;}
pre{margin:0 .5em;padding:.5em;border:1px solid #aaa;border-radius:5px;}
li{margin:1em 0;}
table{margin:0 .5em;border-collapse:collapse;border:1px solid #aaa;}
td{padding:5px;}th{padding:10px;border-bottom:1px solid #aaa;background-color:Gainsboro;}
td,th:not(:first-child){border-left:1px solid #aaa;}
figure{display:block;text-align:center;}
</style>

Lilt
====
_Lil Terminal_ is a command-line interface for the Lil programming language, allowing easy experimentation outside of Decker. Lilt also includes bindings for basic CLI and filesystem IO, making it potentially usable for shell scripting and other applications.

Installation
------------
The source code for Lilt is available [On GitHub](https://github.com/JohnEarnest/Decker).

Lilt depends upon the C standard library and some POSIX extensions. It should work on OSX or most Linux distros. To build and install Lilt, run the included `make` script. By default, a binary is installed in `/usr/local/bin`.
```
make lilt && make install
```

Invoking Lilt
-------------
```
$ lilt [FILE.lil...] [-e EXPR...]
	if present, execute a FILE and exit
	-e : evaluate EXPR and exit
	-h : display this information
```

Executing a `FILE` or `EXPR` argument will not automatically produce any output. Use `show[]` or `print[]` to produce results on _stdout_:
```
$ lilt -e "show[100+range 5]"
(100,101,102,103,104)

$ lilt -e "print[sys.version]"
0.6
```

If a `FILE` or `EXPR` argument has not been provided, Lilt will run in interactive "REPL" mode, which can be exited with Ctrl+C or the `exit[]` function. You may find it more comfortable to run the `lilt` REPL under the `rlwrap` utility (available in a \*nix package manager near you), which provides command history and line editing features.
```
$ rlwrap lilt
  2+3
5
  exit[]
$
```

For convenience, after each line is executed at the REPL, the result will be stored in a variable named `_`:
```
  1,2,3
(1,2,3)
  10+_
(11,12,13)
  _/5
(2.2,2.4,2.6)
```

You can write reasonably portable shell scripts by starting a file with a "shebang" something like:
```
#!/usr/bin/env lilt
```

If an environment variable named `LIL_HOME` is set, Lilt will search that directory path at startup, executing any `.lil` files. These could in turn easily load datasets or other useful definitions every time you open a REPL. Startup scripts are always loaded prior to executing `FILE` or `EXPR` arguments.

Global Variables
----------------
| Name            | Description                                                                                           |
| :-------------- | :---------------------------------------------------------------------------------------------------- |
| `args`          | List of CLI arguments to Lilt as strings.                                                             |
| `env`           | Dictionary of POSIX Environment variables.                                                            |
| `sys`           | An Interface which exposes information about the Lilt runtime.                                        |
| `rtext`         | An Interface with routines for working with rich text.                                                |
| `pi`            | The ratio of a circle's circumference to its diameter. Roughly 3.141.                                 |
| `e`             | Euler's number; the base of the natural logarithm. Roughly 2.718.                                     |
| `colors`        | A dictionary of named pattern indices.                                                                |

Built-in Functions
------------------
| Name             | Description                                                                                                                 | Purpose |
| :--------------- | :-------------------------------------------------------------------------------------------------------------------------- | :------ |
| `input[x]`       | Read a line from _stdin_ as a string, optionally displaying `x` as if with `print[]`, without the newline.                  | Console |
| `show[x...]`     | Print a human-comprehensible representation of the value `x` to _stdout_ followed by a newline, and return `x`.             | Console |
| `print[x...]`    | Print a string `x` to _stdout_ followed by a newline. If more args are provided, `format` all but the first using `x`.      | Console |
| `error[x...]`    | Print a string `x` to _stderr_ followed by a newline. If more args are provided, `format` all but the first using `x`.      | Console |
| `dir[x]`         | List the content of a directory as a table.(1)                                                                              | Files   |
| `path[x y]`      | Canonical path `x` (joined with `y`, if given) via [realpath()](https://www.man7.org/linux/man-pages/man3/realpath.3.html). | Files   |
| `read[x hint]`   | Read a file `x` using `hint` as necessary to control its interpretation.(2)                                                 | Files   |
| `write[x y]`     | Write a value `y` to a file `x`. Returns `1` on success.(3)                                                                 | Files   |
| `exit[x]`        | Stop execution with exit code `x`.                                                                                          | System  |
| `shell[x]`       | Execute string `x` as a shell command and block for its completion.(4)                                                      | System  |
| `eval[x y]`      | Parse and execute a string `x` as a Lil program, using any variable bindings in dictionary `y`.(5)                          | System  |
| `random[x y]`    | Choose `y` random elements from `x`.                                                                                        | System  |
| `readcsv[x y d]` | Turn a [RFC-4180](https://datatracker.ietf.org/doc/html/rfc4180) CSV string `x` into a Lil table with column spec `y`.(6)   | Data    |
| `writecsv[x y d]`| Turn a Lil table `x` into a CSV string with column spec `y`.(6)                                                             | Data    |
| `readxml[x]`     | Turn a useful subset of XML/HTML into a Lil structure.(6)                                                                   | Data    |
| `writexml[x]`    | Turn a Lil structure `x` into an indented XML string.(6)                                                                    | Data    |
| `readdeck[x]`    | Produce a _deck_ interface from a file at path `x`. If no path is given, produce a new _deck_ from scratch.                 | Decker  |
| `writedeck[x y]` | Serialize a _deck_ interface `y` to a file at path `x`. Returns `1` on success.(7)                                          | Decker  |
| `array[x y]`     | Create a new _array_ with size `x` and cast string `y`, or decode an encoded array string `x`.                              | Decker  |
| `image[x]`       | Create a new _image_ interface with size `x` (`(width,height)`) or decode an encoded image string.                          | Decker  |
| `sound[x]`       | Create a new _sound_ interface with size `x` (sample count) or decode an encoded sound string.                              | Decker  |

1) `dir[]` of a file results in an empty table. Directory tables contain:
- `dir`:  if an item is a directory `1`, and otherwise `0`.
- `name`: the filename of the item.
- `type`: the extension including a dot (like `.txt`), if any, always converted to lowercase.

2) `read[]` recognizes several types of file by extension and will interpret each appropriately:

- if the `hint` argument is the string `"array"`, the file will be read as an _array interface_ with a default `cast` of `u8`.
- `.gif` files are read as _image interfaces_.
- `.wav` files are read as _sound interfaces_.
- anything else is treated as a text file and read as a string.

If a GIF file is unreadable or missing, it will be loaded as a 0x0 image. Only the first frame of a GIF will be loaded. If the image contains transparent pixels, they will be read as pattern 0. By default, other pixels will be adapted to Decker's 16-color palette (patterns 32-47). If the `hint` argument is `"gray"`, they will instead be converted to 256 grays based on a perceptual weighting of their RGB channels. Note that a 256 gray image is not suitable for direct display on e.g. a canvas, but can be re-paletted or posterized in a variety of ways via `image.map[]` or dithered with `image.transform["dither"]`.

The [WAV file format](https://en.wikipedia.org/wiki/WAV) is much more complex than one might imagine. For this reason, and in order to avoid drawing in large dependencies, `read[]` in Lilt accepts only a very specific subset of valid WAV files corresponding to the output of `write[]`: monophonic, 8khz, with 8-bit unsigned PCM samples and no optional headers. Any other format (or an altogether invalid audio file) will be read as a `sound` with a `size` of 0. For reference, you can convert nearly any audio file into a compatible format using [ffmpeg](https://ffmpeg.org) like so:
```
ffmpeg -i input.mp3 -bitexact -map_metadata -1 -ac 1 -ar 8000 -acodec pcm_u8 output.wav
```

3) `write[]` recognizes several types of Lil value and will serialize each appropriately:
- _array interfaces_ are written as binary files.
- _sound interfaces_ are written as a .WAV audio file.
- _image interfaces_ are written as GIF89a images.
- a list of _image interfaces_ is written as an animated GIF89a image, with each image in the list written as one frame.
- anything else is converted to a string and written as a text file.

4) `shell[]` returns a dictionary containing:
- `exit`: the exit code of the process, as a number. If the process halted abnormally (i.e. due to a signal), this will be -1.
- `out`: _stdout_ of the process, as a string.

5) `eval[]` returns a dictionary containing:
- `error`: a string giving any error message produced during parsing, or the empty string.
- `value`: the value of the last expression in the program. On a parse error, `value` will be the number `0`.
- `vars`: a dictionary containing any variable bindings made while executing the program. (This also includes bindings from argument `y`.)

6) See the [Decker Manual](decker.html) for details of `readcsv[]`, `writecsv[]`, `readxml[]`, and `writexml[]`.

7) If the path given to `writedeck[]` ends in a `.html` suffix, the deck will be written as a "standalone" deck with a bundled HTML+JS runtime. Otherwise, the deck will be written as a "bare" deck, which is smaller.

Working With Decks
------------------
The `readdeck[]` and `writedeck[]` functions allow Lilt to operate on [Decker](decker.html) documents. Lilt can load, create, and manipulate multiple decks simultaneously, providing options for automated testing, data import/export, accessibility, and interacting with other technology from outside the Decker ecosystem.

In addition to the fields and methods available to Decker for the deck interface and each of its sub-interfaces, Lilt has access to _event injector_ members which behave as if the corresponding event was produced by a user interacting with the deck, running the appropriate scripts to completion and producing any appropriate side-effects on the deck. These injectors are as follows:

| Name                  | Description                                                                                                |
| :-------------------- | :--------------------------------------------------------------------------------------------------------- |
| `button.click[]`      | Simulate clicking a button.                                                                                |
| `grid.click[row]`     | Simulate selecting a row in a grid.                                                                        |
| `grid.order[col]`     | Simulate clicking a column header in a grid.                                                               |
| `grid.change[table]`  | Simulate editing the data in a grid. (Does not actually change `grid.value` unless the script does!)       |
| `canvas.click[pos]`   | Simulate depressing the pointer on a canvas.                                                               |
| `canvas.drag[pos]`    | Simulate dragging the pointer on a canvas.                                                                 |
| `canvas.release[pos]` | Simulate releasing a held pointer on a canvas.                                                             |
| `field.link[text]`    | Simulate clicking a link in a rich-text field.                                                             |
| `field.run[text]`     | Simulate pressing shift+return with `text` selected in the field.                                          |
| `field.change[text]`  | Simulate editing a field. (Does not actually change `field.text` or `field.value` unless the script does!) |
| `slider.change[text]` | Simulate manipulating a slider. (Does not actually change `slider.value` unless the script does!)          |
| `card.navigate[dir]`  | Simulate the user pressing navigation/cursor keys.                                                         |

Furthermore, Lilt can "copy" and "paste" entire cards from a deck or lists of widgets within a card:

| Name                  | Description                                                                                                |
| :-------------------- | :--------------------------------------------------------------------------------------------------------- |
| `card.copy[list]`     | Save a list of widgets on `card` as a string.                                                              |
| `card.paste[text]`    | Append the widgets within a string to `card`, returning a list of the new widgets.                         |
| `deck.copy[card]`     | Save a card and its contents as a string.                                                                  |
| `deck.paste[text]`    | Append a card and its contents from a string to `deck`, returning the new `card`.                          |

All of these operations work with a serialized form of the corresponding deck components as a string. The format of these strings is subject to change in the future and should be treated as opaque, but a valid string will always begin with the prefix `%%CRD0` (a copied card) or `%%WGT0` (a list of copied widgets). Strings can be freely copied and pasted between decks, and even pasted multiple times. As with `deck.add[]` and `card.add[]`, the `name` fields of pasted objects will be changed if they would collide with existing parts of the deck.


Changelog
---------
- v1.0 : initial release.
