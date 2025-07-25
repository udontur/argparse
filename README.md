<p align="center">
  <img height="100" src="https://i.imgur.com/oDXeMUQ.png" alt="argparse"/>
</p>

<p align="center">
  <a href="https://github.com/p-ranav/argparse/blob/master/LICENSE">
    <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="license"/>
  </a>
  <img src="https://img.shields.io/badge/version-3.2-blue.svg?cacheSeconds=2592000" alt="version"/>
</p>

## Highlights

* Single header file
* Requires C++17
* MIT License

## Table of Contents

*    [Quick Start](#quick-start)
     *    [Positional Arguments](#positional-arguments)
     *    [Optional Arguments](#optional-arguments)
          *    [Requiring optional arguments](#requiring-optional-arguments)
          *    [Accessing optional arguments without default values](#accessing-optional-arguments-without-default-values)
          *    [Deciding if the value was given by the user](#deciding-if-the-value-was-given-by-the-user)
          *    [Joining values of repeated optional arguments](#joining-values-of-repeated-optional-arguments)
          *    [Repeating an argument to increase a value](#repeating-an-argument-to-increase-a-value)
          *    [Mutually Exclusive Group](#mutually-exclusive-group)
     *    [Storing values into variables](#store-into)
     *    [Negative Numbers](#negative-numbers)
     *    [Combining Positional and Optional Arguments](#combining-positional-and-optional-arguments)
     *    [Printing Help](#printing-help)
     *    [Adding a description and an epilog to help](#adding-a-description-and-an-epilog-to-help)
     *    [List of Arguments](#list-of-arguments)
     *    [Compound Arguments](#compound-arguments)
     *    [Converting to Numeric Types](#converting-to-numeric-types)
     *    [Default Arguments](#default-arguments)
     *    [Gathering Remaining Arguments](#gathering-remaining-arguments)
     *    [Parent Parsers](#parent-parsers)
     *    [Subcommands](#subcommands)
     *    [Getting Argument and Subparser Instances](#getting-argument-and-subparser-instances)
     *    [Parse Known Args](#parse-known-args)
     *    [Hidden argument and alias](#hidden-argument-alias)
     *    [ArgumentParser in bool Context](#argumentparser-in-bool-context)
     *    [Custom Prefix Characters](#custom-prefix-characters)
     *    [Custom Assignment Characters](#custom-assignment-characters)
*    [Further Examples](#further-examples)
     *    [Construct a JSON object from a filename argument](#construct-a-json-object-from-a-filename-argument)
     *    [Positional Arguments with Compound Toggle Arguments](#positional-arguments-with-compound-toggle-arguments)
     *    [Restricting the set of values for an argument](#restricting-the-set-of-values-for-an-argument)
     *    [Using `option=value` syntax](#using-optionvalue-syntax)
     *    [Advanced usage formatting](#advanced-usage-formatting)
*    [Developer Notes](#developer-notes)
     *    [Copying and Moving](#copying-and-moving)
*    [CMake Integration](#cmake-integration)
*    [Building, Installing, and Testing](#building-installing-and-testing)
*    [Supported Toolchains](#supported-toolchains)
*    [Contributing](#contributing)
*    [License](#license)

## Quick Start

Simply include argparse.hpp and you're good to go.

```cpp
#include <argparse/argparse.hpp>
```

To start parsing command-line arguments, create an ```ArgumentParser```.

```cpp
argparse::ArgumentParser program("program_name");
```

**NOTE:** There is an optional second argument to the `ArgumentParser` which is the program version. Example: `argparse::ArgumentParser program("libfoo", "1.9.0");`

**NOTE:** There are optional third and fourth arguments to the `ArgumentParser` which control default arguments. Example: `argparse::ArgumentParser program("libfoo", "1.9.0", default_arguments::help, false);` See [Default Arguments](#default-arguments), below.

To add a new argument, simply call ```.add_argument(...)```. You can provide a variadic list of argument names that you want to group together, e.g., ```-v``` and ```--verbose```

```cpp
program.add_argument("foo");
program.add_argument("-v", "--verbose"); // parameter packing
```

Argparse supports a variety of argument types including positional, optional, and compound arguments. Below you can see how to configure each of these types:

### Positional Arguments

Here's an example of a ***positional argument***:

```cpp
#include <argparse/argparse.hpp>

int main(int argc, char *argv[]) {
  argparse::ArgumentParser program("program_name");

  program.add_argument("square")
    .help("display the square of a given integer")
    .scan<'i', int>();

  try {
    program.parse_args(argc, argv);
  }
  catch (const std::exception& err) {
    std::cerr << err.what() << std::endl;
    std::cerr << program;
    return 1;
  }

  auto input = program.get<int>("square");
  std::cout << (input * input) << std::endl;

  return 0;
}
```

And running the code:

```console
foo@bar:/home/dev/$ ./main 15
225
```

Here's what's happening:

* The ```add_argument()``` method is used to specify which command-line options the program is willing to accept. In this case, I’ve named it square so that it’s in line with its function.
* Command-line arguments are strings. To square the argument and print the result, we need to convert this argument to a number. In order to do this, we use the ```.scan``` method to convert user input into an integer.
* We can get the value stored by the parser for a given argument using ```parser.get<T>(key)``` method.

### Optional Arguments

Now, let's look at ***optional arguments***. Optional arguments start with ```-``` or ```--```, e.g., ```--verbose``` or ```-a```. Optional arguments can be placed anywhere in the input sequence.


```cpp
argparse::ArgumentParser program("test");

program.add_argument("--verbose")
  .help("increase output verbosity")
  .default_value(false)
  .implicit_value(true);

try {
  program.parse_args(argc, argv);
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

if (program["--verbose"] == true) {
  std::cout << "Verbosity enabled" << std::endl;
}
```

```console
foo@bar:/home/dev/$ ./main --verbose
Verbosity enabled
```

Here's what's happening:
* The program is written so as to display something when --verbose is specified and display nothing when not.
* Since the argument is actually optional, no error is thrown when running the program without ```--verbose```. Note that by using ```.default_value(false)```, if the optional argument isn’t used, it's value is automatically set to false.
* By using ```.implicit_value(true)```, the user specifies that this option is more of a flag than something that requires a value. When the user provides the --verbose option, it's value is set to true.

#### Flag

When defining flag arguments, you can use the shorthand `flag()` which is the same as `default_value(false).implicit_value(true)`. 

```cpp
argparse::ArgumentParser program("test");

program.add_argument("--verbose")
  .help("increase output verbosity")
  .flag();

try {
  program.parse_args(argc, argv);
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

if (program["--verbose"] == true) {
  std::cout << "Verbosity enabled" << std::endl;
}
```

#### Requiring optional arguments

There are scenarios where you would like to make an optional argument ***required***. As discussed above, optional arguments either begin with `-` or `--`. You can make these types of arguments required like so:

```cpp
program.add_argument("-o", "--output")
  .required()
  .help("specify the output file.");
```

If the user does not provide a value for this parameter, an exception is thrown.

Alternatively, you could provide a default value like so:

```cpp
program.add_argument("-o", "--output")
  .default_value(std::string("-"))
  .required()
  .help("specify the output file.");
```

#### Accessing optional arguments without default values

If you require an optional argument to be present but have no good default value for it, you can combine testing and accessing the argument as following:

```cpp
if (auto fn = program.present("-o")) {
    do_something_with(*fn);
}
```

Similar to `get`, the `present` method also accepts a template argument.  But rather than returning `T`, `parser.present<T>(key)` returns `std::optional<T>`, so that when the user does not provide a value to this parameter, the return value compares equal to `std::nullopt`.

#### Deciding if the value was given by the user

If you want to know whether the user supplied a value for an argument that has a ```.default_value```, check whether the argument ```.is_used()```.

```cpp
program.add_argument("--color")
  .default_value(std::string{"orange"})   // might otherwise be type const char* leading to an error when trying program.get<std::string>
  .help("specify the cat's fur color");

try {
  program.parse_args(argc, argv);    // Example: ./main --color orange
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

auto color = program.get<std::string>("--color");  // "orange"
auto explicit_color = program.is_used("--color");  // true, user provided orange
```

#### Joining values of repeated optional arguments

You may want to allow an optional argument to be repeated and gather all values in one place.

```cpp
program.add_argument("--color")
  .default_value<std::vector<std::string>>({ "orange" })
  .append()
  .help("specify the cat's fur color");

try {
  program.parse_args(argc, argv);    // Example: ./main --color red --color green --color blue
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

auto colors = program.get<std::vector<std::string>>("--color");  // {"red", "green", "blue"}
```

Notice that ```.default_value``` is given an explicit template parameter to match the type you want to ```.get```.

#### Repeating an argument to increase a value

A common pattern is to repeat an argument to indicate a greater value.

```cpp
int verbosity = 0;
program.add_argument("-V", "--verbose")
  .action([&](const auto &) { ++verbosity; })
  .append()
  .default_value(false)
  .implicit_value(true)
  .nargs(0);

program.parse_args(argc, argv);    // Example: ./main -VVVV

std::cout << "verbose level: " << verbosity << std::endl;    // verbose level: 4
```

#### Mutually Exclusive Group

Create a mutually exclusive group using `program.add_mutually_exclusive_group(required = false)`. `argparse`` will make sure that only one of the arguments in the mutually exclusive group was present on the command line:

```cpp
auto &group = program.add_mutually_exclusive_group();
group.add_argument("--first");
group.add_argument("--second");
```

with the following usage will yield an error:

```console
foo@bar:/home/dev/$ ./main --first 1 --second 2
Argument '--second VAR' not allowed with '--first VAR'
```

The `add_mutually_exclusive_group()` function also accepts a `required` argument, to indicate that at least one of the mutually exclusive arguments is required:

```cpp
auto &group = program.add_mutually_exclusive_group(true);
group.add_argument("--first");
group.add_argument("--second");
```

with the following usage will yield an error:

```console
foo@bar:/home/dev/$ ./main
One of the arguments '--first VAR' or '--second VAR' is required
```

### Storing values into variables

It is possible to bind arguments to a variable storing their value, as an
alternative to explicitly calling ``program.get<T>(arg_name)`` or ``program[arg_name]``

This is currently implementeted for variables of type ``bool`` (this also
implicitly calls ``flag()``), ``int``, ``double``, ``std::string``,
``std::vector<std::string>`` and ``std::vector<int>``.
If the argument is not specified in the command
line, the default value (if set) is set into the variable.

```cpp
bool flagvar = false;
program.add_argument("--flagvar").store_into(flagvar);

int intvar = 0;
program.add_argument("--intvar").store_into(intvar);

double doublevar = 0;
program.add_argument("--doublevar").store_into(doublevar);

std::string strvar;
program.add_argument("--strvar").store_into(strvar);

std::vector<std::string> strvar_repeated;
program.add_argument("--strvar-repeated").append().store_into(strvar_repeated);

std::vector<std::string> strvar_multi_valued;
program.add_argument("--strvar-multi-valued").nargs(2).store_into(strvar_multi_valued);

std::vector<int> intvar_repeated;
program.add_argument("--intvar-repeated").append().store_into(intvar_repeated);

std::vector<int> intvar_multi_valued;
program.add_argument("--intvar-multi-valued").nargs(2).store_into(intvar_multi_valued);
```

### Negative Numbers

Optional arguments start with ```-```. Can ```argparse``` handle negative numbers? The answer is yes!

```cpp
argparse::ArgumentParser program;

program.add_argument("integer")
  .help("Input number")
  .scan<'i', int>();

program.add_argument("floats")
  .help("Vector of floats")
  .nargs(4)
  .scan<'g', float>();

try {
  program.parse_args(argc, argv);
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

// Some code to print arguments
```

```console
foo@bar:/home/dev/$ ./main -5 -1.1 -3.1415 -3.1e2 -4.51329E3
integer : -5
floats  : -1.1 -3.1415 -310 -4513.29
```

As you can see here, ```argparse``` supports negative integers, negative floats and scientific notation.

### Combining Positional and Optional Arguments

```cpp
argparse::ArgumentParser program("main");

program.add_argument("square")
  .help("display the square of a given number")
  .scan<'i', int>();

program.add_argument("--verbose")
  .default_value(false)
  .implicit_value(true);

try {
  program.parse_args(argc, argv);
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

int input = program.get<int>("square");

if (program["--verbose"] == true) {
  std::cout << "The square of " << input << " is " << (input * input) << std::endl;
}
else {
  std::cout << (input * input) << std::endl;
}
```

```console
foo@bar:/home/dev/$ ./main 4
16

foo@bar:/home/dev/$ ./main 4 --verbose
The square of 4 is 16

foo@bar:/home/dev/$ ./main --verbose 4
The square of 4 is 16
```

### Printing Help

`std::cout << program` prints a help message, including the program usage and information about the arguments registered with the `ArgumentParser`. For the previous example, here's the default help message:

```
foo@bar:/home/dev/$ ./main --help
Usage: main [-h] [--verbose] square

Positional arguments:
  square       	display the square of a given number

Optional arguments:
  -h, --help   	shows help message and exits
  -v, --version	prints version information and exits
  --verbose
```

You may also get the help message in string via `program.help().str()`.

#### Adding a description and an epilog to help

`ArgumentParser::add_description` will add text before the detailed argument
information. `ArgumentParser::add_epilog` will add text after all other help output.

```cpp
#include <argparse/argparse.hpp>

int main(int argc, char *argv[]) {
  argparse::ArgumentParser program("main");
  program.add_argument("thing").help("Thing to use.").metavar("THING");
  program.add_argument("--member").help("The alias for the member to pass to.").metavar("ALIAS");
  program.add_argument("--verbose").default_value(false).implicit_value(true);

  program.add_description("Forward a thing to the next member.");
  program.add_epilog("Possible things include betingalw, chiz, and res.");

  program.parse_args(argc, argv);

  std::cout << program << std::endl;
}
```

```console
Usage: main [-h] [--member ALIAS] [--verbose] THING

Forward a thing to the next member.

Positional arguments:
  THING         	Thing to use.

Optional arguments:
  -h, --help    	shows help message and exits
  -v, --version 	prints version information and exits
  --member ALIAS	The alias for the member to pass to.
  --verbose

Possible things include betingalw, chiz, and res.
```

### List of Arguments

ArgumentParser objects usually associate a single command-line argument with a single action to be taken. The ```.nargs``` associates a different number of command-line arguments with a single action. When using ```nargs(N)```, N arguments from the command line will be gathered together into a list.

```cpp
argparse::ArgumentParser program("main");

program.add_argument("--input_files")
  .help("The list of input files")
  .nargs(2);

try {
  program.parse_args(argc, argv);   // Example: ./main --input_files config.yml System.xml
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

auto files = program.get<std::vector<std::string>>("--input_files");  // {"config.yml", "System.xml"}
```

```ArgumentParser.get<T>()``` has specializations for ```std::vector``` and ```std::list```. So, the following variant, ```.get<std::list>```, will also work.

```cpp
auto files = program.get<std::list<std::string>>("--input_files");  // {"config.yml", "System.xml"}
```

Using ```.scan```, one can quickly build a list of desired value types from command line arguments. Here's an example:

```cpp
argparse::ArgumentParser program("main");

program.add_argument("--query_point")
  .help("3D query point")
  .nargs(3)
  .default_value(std::vector<double>{0.0, 0.0, 0.0})
  .scan<'g', double>();

try {
  program.parse_args(argc, argv); // Example: ./main --query_point 3.5 4.7 9.2
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

auto query_point = program.get<std::vector<double>>("--query_point");  // {3.5, 4.7, 9.2}
```

You can also make a variable length list of arguments with the ```.nargs```.
Below are some examples.

```cpp
program.add_argument("--input_files")
  .nargs(1, 3);  // This accepts 1 to 3 arguments.
```

Some useful patterns are defined like "?", "*", "+" of argparse in Python.

```cpp
program.add_argument("--input_files")
  .nargs(argparse::nargs_pattern::any);  // "*" in Python. This accepts any number of arguments including 0.
```
```cpp
program.add_argument("--input_files")
  .nargs(argparse::nargs_pattern::at_least_one);  // "+" in Python. This accepts one or more number of arguments.
```
```cpp
program.add_argument("--input_files")
  .nargs(argparse::nargs_pattern::optional);  // "?" in Python. This accepts an argument optionally.
```

### Compound Arguments

Compound arguments are optional arguments that are combined and provided as a single argument. Example: ```ps -aux```

```cpp
argparse::ArgumentParser program("test");

program.add_argument("-a")
  .default_value(false)
  .implicit_value(true);

program.add_argument("-b")
  .default_value(false)
  .implicit_value(true);

program.add_argument("-c")
  .nargs(2)
  .default_value(std::vector<float>{0.0f, 0.0f})
  .scan<'g', float>();

try {
  program.parse_args(argc, argv);                  // Example: ./main -abc 1.95 2.47
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

auto a = program.get<bool>("-a");                  // true
auto b = program.get<bool>("-b");                  // true
auto c = program.get<std::vector<float>>("-c");    // {1.95, 2.47}

/// Some code that prints parsed arguments
```

```console
foo@bar:/home/dev/$ ./main -ac 3.14 2.718
a = true
b = false
c = {3.14, 2.718}

foo@bar:/home/dev/$ ./main -cb
a = false
b = true
c = {0.0, 0.0}
```

Here's what's happening:
* We have three optional arguments ```-a```, ```-b``` and ```-c```.
* ```-a``` and ```-b``` are toggle arguments.
* ```-c``` requires 2 floating point numbers from the command-line.
* argparse can handle compound arguments, e.g., ```-abc``` or ```-bac``` or ```-cab```. This only works with short single-character argument names.
  - ```-a``` and ```-b``` become true.
  - argv is further parsed to identify the inputs mapped to ```-c```.
  - If argparse cannot find any arguments to map to c, then c defaults to {0.0, 0.0} as defined by ```.default_value```

### Converting to Numeric Types

For inputs, users can express a primitive type for the value.

The ```.scan<Shape, T>``` method attempts to convert the incoming `std::string` to `T` following the `Shape` conversion specifier. An `std::invalid_argument` or `std::range_error` exception is thrown for errors.

```cpp
program.add_argument("-x")
       .scan<'d', int>();

program.add_argument("scale")
       .scan<'g', double>();
```

`Shape` specifies what the input "looks like", and the type template argument specifies the return value of the predefined action. Acceptable types are floating point (i.e float, double, long double) and integral (i.e. signed char, short, int, long, long long).

The grammar follows `std::from_chars`, but does not exactly duplicate it. For example, hexadecimal numbers may begin with `0x` or `0X` and numbers with a leading zero may be handled as octal values.

| Shape      | interpretation                            |
| :--------: | ----------------------------------------- |
| 'a' or 'A' | hexadecimal floating point                |
| 'e' or 'E' | scientific notation (floating point)      |
| 'f' or 'F' | fixed notation (floating point)           |
| 'g' or 'G' | general form (either fixed or scientific) |
|            |                                           |
| 'd'        | decimal                                   |
| 'i'        | `std::from_chars` grammar with base == 10 |
| 'o'        | octal (unsigned)                          |
| 'u'        | decimal (unsigned)                        |
| 'x' or 'X' | hexadecimal (unsigned)                    |

### Default Arguments

`argparse` provides predefined arguments and actions for `-h`/`--help` and `-v`/`--version`. By default, these actions will **exit** the program after displaying a help or version message, respectively. This exit does not call destructors, skipping clean-up of taken resources.

These default arguments can be disabled during `ArgumentParser` creation so that you can handle these arguments in your own way. (Note that a program name and version must be included when choosing default arguments.)

```cpp
argparse::ArgumentParser program("test", "1.0", default_arguments::none);

program.add_argument("-h", "--help")
  .action([=](const std::string& s) {
    std::cout << help().str();
  })
  .default_value(false)
  .help("shows help message")
  .implicit_value(true)
  .nargs(0);
```

The above code snippet outputs a help message and continues to run. It does not support a `--version` argument.

The default is `default_arguments::all` for included arguments. No default arguments will be added with `default_arguments::none`. `default_arguments::help` and `default_arguments::version` will individually add `--help` and `--version`.

The default arguments can be used while disabling the default exit with these arguments. This forth argument to `ArgumentParser` (`exit_on_default_arguments`) is a bool flag with a default **true** value. The following call will retain `--help` and `--version`, but will not exit when those arguments are used.

```cpp
argparse::ArgumentParser program("test", "1.0", default_arguments::all, false)
```

### Gathering Remaining Arguments

`argparse` supports gathering "remaining" arguments at the end of the command, e.g., for use in a compiler:

```console
foo@bar:/home/dev/$ compiler file1 file2 file3
```

To enable this, simply create an argument and mark it as `remaining`. All remaining arguments passed to argparse are gathered here.

```cpp
argparse::ArgumentParser program("compiler");

program.add_argument("files")
  .remaining();

try {
  program.parse_args(argc, argv);
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

try {
  auto files = program.get<std::vector<std::string>>("files");
  std::cout << files.size() << " files provided" << std::endl;
  for (auto& file : files)
    std::cout << file << std::endl;
} catch (std::logic_error& e) {
  std::cout << "No files provided" << std::endl;
}
```

When no arguments are provided:

```console
foo@bar:/home/dev/$ ./compiler
No files provided
```

and when multiple arguments are provided:

```console
foo@bar:/home/dev/$ ./compiler foo.txt bar.txt baz.txt
3 files provided
foo.txt
bar.txt
baz.txt
```

The process of gathering remaining arguments plays nicely with optional arguments too:

```cpp
argparse::ArgumentParser program("compiler");

program.add_argument("-o")
  .default_value(std::string("a.out"));

program.add_argument("files")
  .remaining();

try {
  program.parse_args(argc, argv);
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

auto output_filename = program.get<std::string>("-o");
std::cout << "Output filename: " << output_filename << std::endl;

try {
  auto files = program.get<std::vector<std::string>>("files");
  std::cout << files.size() << " files provided" << std::endl;
  for (auto& file : files)
    std::cout << file << std::endl;
} catch (std::logic_error& e) {
  std::cout << "No files provided" << std::endl;
}

```

```console
foo@bar:/home/dev/$ ./compiler -o main foo.cpp bar.cpp baz.cpp
Output filename: main
3 files provided
foo.cpp
bar.cpp
baz.cpp
```

***NOTE***: Remember to place all optional arguments BEFORE the remaining argument. If the optional argument is placed after the remaining arguments, it too will be deemed remaining:

```console
foo@bar:/home/dev/$ ./compiler foo.cpp bar.cpp baz.cpp -o main
5 arguments provided
foo.cpp
bar.cpp
baz.cpp
-o
main
```

### Parent Parsers

A parser may use arguments that could be used by other parsers.

These shared arguments can be added to a parser which is then used as a "parent" for parsers which also need those arguments. One or more parent parsers may be added to a parser with `.add_parents`. The positional and optional arguments in each parent is added to the child parser.

```cpp
argparse::ArgumentParser surface_parser("surface", "1.0", argparse::default_arguments::none);
surface_parser.add_argument("--area")
  .default_value(0)
  .scan<'i', int>();

argparse::ArgumentParser floor_parser("floor");
floor_parser.add_argument("tile_size").scan<'i', int>();
floor_parser.add_parents(surface_parser);
floor_parser.parse_args({ "./main", "--area", "200", "12" });  // --area = 200, tile_size = 12

argparse::ArgumentParser ceiling_parser("ceiling");
ceiling_parser.add_argument("--color");
ceiling_parser.add_parents(surface_parser);
ceiling_parser.parse_args({ "./main", "--color", "gray" });  // --area = 0, --color = "gray"
```

Changes made to parents after they are added to a parser are not reflected in any child parsers. Completely initialize parent parsers before adding them to a parser.

Each parser will have the standard set of default arguments. Disable the default arguments in parent parsers to avoid duplicate help output.

### Subcommands

Many programs split up their functionality into a number of sub-commands, for example, the `git` program can invoke sub-commands like `git checkout`, `git add`, and `git commit`. Splitting up functionality this way can be a particularly good idea when a program performs several different functions which require different kinds of command-line arguments. `ArgumentParser` supports the creation of such sub-commands with the `add_subparser()` member function.

```cpp
#include <argparse/argparse.hpp>

int main(int argc, char *argv[]) {
  argparse::ArgumentParser program("git");

  // git add subparser
  argparse::ArgumentParser add_command("add");
  add_command.add_description("Add file contents to the index");
  add_command.add_argument("files")
    .help("Files to add content from. Fileglobs (e.g.  *.c) can be given to add all matching files.")
    .remaining();

  // git commit subparser
  argparse::ArgumentParser commit_command("commit");
  commit_command.add_description("Record changes to the repository");
  commit_command.add_argument("-a", "--all")
    .help("Tell the command to automatically stage files that have been modified and deleted.")
    .default_value(false)
    .implicit_value(true);

  commit_command.add_argument("-m", "--message")
    .help("Use the given <msg> as the commit message.");

  // git cat-file subparser
  argparse::ArgumentParser catfile_command("cat-file");
  catfile_command.add_description("Provide content or type and size information for repository objects");
  catfile_command.add_argument("-t")
    .help("Instead of the content, show the object type identified by <object>.");

  catfile_command.add_argument("-p")
    .help("Pretty-print the contents of <object> based on its type.");

  // git submodule subparser
  argparse::ArgumentParser submodule_command("submodule");
  submodule_command.add_description("Initialize, update or inspect submodules");
  argparse::ArgumentParser submodule_update_command("update");
  submodule_update_command.add_description("Update the registered submodules to match what the superproject expects");
  submodule_update_command.add_argument("--init")
    .default_value(false)
    .implicit_value(true);
  submodule_update_command.add_argument("--recursive")
    .default_value(false)
    .implicit_value(true);
  submodule_command.add_subparser(submodule_update_command);

  program.add_subparser(add_command);
  program.add_subparser(commit_command);
  program.add_subparser(catfile_command);
  program.add_subparser(submodule_command);

  try {
    program.parse_args(argc, argv);
  }
  catch (const std::exception& err) {
    std::cerr << err.what() << std::endl;
    std::cerr << program;
    return 1;
  }

  // Use arguments
}
```

```console
foo@bar:/home/dev/$ ./git --help
Usage: git [-h] {add,cat-file,commit,submodule}

Optional arguments:
  -h, --help   	shows help message and exits
  -v, --version	prints version information and exits

Subcommands:
  add           Add file contents to the index
  cat-file      Provide content or type and size information for repository objects
  commit        Record changes to the repository
  submodule     Initialize, update or inspect submodules

foo@bar:/home/dev/$ ./git add --help
Usage: add [-h] files

Add file contents to the index

Positional arguments:
  files        	Files to add content from. Fileglobs (e.g.  *.c) can be given to add all matching files.

Optional arguments:
  -h, --help   	shows help message and exits
  -v, --version	prints version information and exits

foo@bar:/home/dev/$ ./git commit --help
Usage: commit [-h] [--all] [--message VAR]

Record changes to the repository

Optional arguments:
  -h, --help   	shows help message and exits
  -v, --version	prints version information and exits
  -a, --all    	Tell the command to automatically stage files that have been modified and deleted.
  -m, --message	Use the given <msg> as the commit message.

foo@bar:/home/dev/$ ./git submodule --help
Usage: submodule [-h] {update}

Initialize, update or inspect submodules

Optional arguments:
  -h, --help   	shows help message and exits
  -v, --version	prints version information and exits

Subcommands:
  update        Update the registered submodules to match what the superproject expects
```

When a help message is requested from a subparser, only the help for that particular parser will be printed. The help message will not include parent parser or sibling parser messages.

Additionally, every parser has the `.is_subcommand_used("<command_name>")` and `.is_subcommand_used(subparser)` member functions to check if a subcommand was used. 

Sometimes there may be a need to hide part of the subcommands from the user
by suppressing information about them in an help message. To do this,
```ArgumentParser``` contains the method ```.set_suppress(bool suppress)```:

```cpp
argparse::ArgumentParser program("test");

argparse::ArgumentParser hidden_cmd("hidden");
hidden_cmd.add_argument("files").remaining();
hidden_cmd.set_suppress(true);

program.add_subparser(hidden_cmd);
```

```console
foo@bar:/home/dev/$ ./main -h
Usage: test [--help] [--version] {}

Optional arguments:
  -h, --help    shows help message and exits
  -v, --version prints version information and exits

foo@bar:/home/dev/$ ./main hidden -h
Usage: hidden [--help] [--version] files

Positional arguments:
  files         [nargs: 0 or more]

Optional arguments:
  -h, --help    shows help message and exits
  -v, --version prints version information and exits
```

### Getting Argument and Subparser Instances

```Argument``` and ```ArgumentParser``` instances added to an ```ArgumentParser``` can be retrieved with ```.at<T>()```. The default return type is ```Argument```.

```cpp
argparse::ArgumentParser program("test");

program.add_argument("--dir");
program.at("--dir").default_value(std::string("/home/user"));

program.add_subparser(argparse::ArgumentParser{"walk"});
program.at<argparse::ArgumentParser>("walk").add_argument("depth");
```

### Parse Known Args

Sometimes a program may only parse a few of the command-line arguments, passing the remaining arguments on to another script or program. In these cases, the `parse_known_args()` function can be useful. It works much like `parse_args()` except that it does not produce an error when extra arguments are present. Instead, it returns a list of remaining argument strings.

```cpp
#include <argparse/argparse.hpp>
#include <cassert>

int main(int argc, char *argv[]) {
  argparse::ArgumentParser program("test");
  program.add_argument("--foo").implicit_value(true).default_value(false);
  program.add_argument("bar");

  auto unknown_args =
    program.parse_known_args({"test", "--foo", "--badger", "BAR", "spam"});

  assert(program.get<bool>("--foo") == true);
  assert(program.get<std::string>("bar") == std::string{"BAR"});
  assert((unknown_args == std::vector<std::string>{"--badger", "spam"}));
}
```

### Hidden argument and alias

It is sometimes desirable to offer an alias for an argument, but without it
appearing it in the usage. For example, to phase out a deprecated wording of
an argument while not breaking backwards compatible. This can be done with
the ``ArgumentParser::add_hidden_alias_for()` method.

```cpp
argparse::ArgumentParser program("test");

auto &arg = program.add_argument("--suppress").flag();
program.add_hidden_alias_for(arg, "--supress"); // old misspelled alias
```

The ``Argument::hidden()`` method can also be used to prevent a (generally
optional) argument from appearing in the usage or help.

```cpp
argparse::ArgumentParser program("test");

program.add_argument("--non-documented").flag().hidden();
```

This can also be used on positional arguments, but in that later case it only
makes sense in practice for the last ones.

### ArgumentParser in bool Context

An `ArgumentParser` is `false` until it (or one of its subparsers) have extracted
known value(s) with `.parse_args` or `.parse_known_args`. When using `.parse_known_args`,
unknown arguments will not make a parser `true`.

### Custom Prefix Characters

Most command-line options will use `-` as the prefix, e.g. `-f/--foo`. Parsers that need to support different or additional prefix characters, e.g. for options like `+f` or `/foo`, may specify them using the `set_prefix_chars()`.

The default prefix character is `-`.

```cpp
#include <argparse/argparse.hpp>
#include <cassert>

int main(int argc, char *argv[]) {
  argparse::ArgumentParser program("test");
  program.set_prefix_chars("-+/");

  program.add_argument("+f");
  program.add_argument("--bar");
  program.add_argument("/foo");

  try {
    program.parse_args(argc, argv);
  }
  catch (const std::exception& err) {
    std::cerr << err.what() << std::endl;
    std::cerr << program;
    return 1;
  }

  if (program.is_used("+f")) {
    std::cout << "+f    : " << program.get("+f") << "\n";
  }

  if (program.is_used("--bar")) {
    std::cout << "--bar : " << program.get("--bar") << "\n";
  }

  if (program.is_used("/foo")) {
    std::cout << "/foo  : " << program.get("/foo") << "\n";
  }  
}
```

```console
foo@bar:/home/dev/$ ./main +f 5 --bar 3.14f /foo "Hello"
+f    : 5
--bar : 3.14f
/foo  : Hello
```

### Custom Assignment Characters 

In addition to prefix characters, custom 'assign' characters can be set. This setting is used to allow invocations like `./test --foo=Foo /B:Bar`.

The default assign character is `=`.

```cpp
#include <argparse/argparse.hpp>
#include <cassert>

int main(int argc, char *argv[]) {
  argparse::ArgumentParser program("test");
  program.set_prefix_chars("-+/");
  program.set_assign_chars("=:");

  program.add_argument("--foo");
  program.add_argument("/B");

  try {
    program.parse_args(argc, argv);
  }
  catch (const std::exception& err) {
    std::cerr << err.what() << std::endl;
    std::cerr << program;
    return 1;
  }

  if (program.is_used("--foo")) {
    std::cout << "--foo : " << program.get("--foo") << "\n";
  }

  if (program.is_used("/B")) {
    std::cout << "/B    : " << program.get("/B") << "\n";
  }
}
```

```console
foo@bar:/home/dev/$ ./main --foo=Foo /B:Bar
--foo : Foo
/B    : Bar
```

## Further Examples

### Construct a JSON object from a filename argument

```cpp
argparse::ArgumentParser program("json_test");

program.add_argument("config")
  .action([](const std::string& value) {
    // read a JSON file
    std::ifstream stream(value);
    nlohmann::json config_json;
    stream >> config_json;
    return config_json;
  });

try {
  program.parse_args({"./test", "config.json"});
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

nlohmann::json config = program.get<nlohmann::json>("config");
```

### Positional Arguments with Compound Toggle Arguments

```cpp
argparse::ArgumentParser program("test");

program.add_argument("numbers")
  .nargs(3)
  .scan<'i', int>();

program.add_argument("-a")
  .default_value(false)
  .implicit_value(true);

program.add_argument("-b")
  .default_value(false)
  .implicit_value(true);

program.add_argument("-c")
  .nargs(2)
  .scan<'g', float>();

program.add_argument("--files")
  .nargs(3);

try {
  program.parse_args(argc, argv);
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

auto numbers = program.get<std::vector<int>>("numbers");        // {1, 2, 3}
auto a = program.get<bool>("-a");                               // true
auto b = program.get<bool>("-b");                               // true
auto c = program.get<std::vector<float>>("-c");                 // {3.14f, 2.718f}
auto files = program.get<std::vector<std::string>>("--files");  // {"a.txt", "b.txt", "c.txt"}

/// Some code that prints parsed arguments
```

```console
foo@bar:/home/dev/$ ./main 1 2 3 -abc 3.14 2.718 --files a.txt b.txt c.txt
numbers = {1, 2, 3}
a = true
b = true
c = {3.14, 2.718}
files = {"a.txt", "b.txt", "c.txt"}
```

### Restricting the set of values for an argument

```cpp
argparse::ArgumentParser program("test");

program.add_argument("input")
  .default_value(std::string{"baz"})
  .choices("foo", "bar", "baz");

try {
  program.parse_args(argc, argv);
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

auto input = program.get("input");
std::cout << input << std::endl;
```

```console
foo@bar:/home/dev/$ ./main fex
Invalid argument "fex" - allowed options: {foo, bar, baz}
```

Using choices also works with integer types, e.g.,

```cpp
argparse::ArgumentParser program("test");

program.add_argument("input")
  .default_value(0)
  .choices(0, 1, 2, 3, 4, 5);

try {
  program.parse_args(argc, argv);
}
catch (const std::exception& err) {
  std::cerr << err.what() << std::endl;
  std::cerr << program;
  std::exit(1);
}

auto input = program.get("input");
std::cout << input << std::endl;
```

```console
foo@bar:/home/dev/$ ./main 6
Invalid argument "6" - allowed options: {0, 1, 2, 3, 4, 5}
```

### Using `option=value` syntax

```cpp
#include "argparse.hpp"
#include <cassert>

int main(int argc, char *argv[]) {
  argparse::ArgumentParser program("test");
  program.add_argument("--foo").implicit_value(true).default_value(false);
  program.add_argument("--bar");

  try {
    program.parse_args(argc, argv);
  }
  catch (const std::exception& err) {
    std::cerr << err.what() << std::endl;
    std::cerr << program;
    return 1;
  }

  if (program.is_used("--foo")) {
    std::cout << "--foo: " << std::boolalpha << program.get<bool>("--foo") << "\n";
  }

  if (program.is_used("--bar")) {
    std::cout << "--bar: " << program.get("--bar") << "\n";
  }  
}
```

```console
foo@bar:/home/dev/$ ./test --bar=BAR --foo
--foo: true
--bar: BAR
```

### Advanced usage formatting

By default usage is reported on a single line.

The ``ArgumentParser::set_usage_max_line_width(width)`` method can be used
to display the usage() on multiple lines, by defining the maximum line width.

It can be combined with a call to ``ArgumentParser::set_usage_break_on_mutex()``
to ask grouped mutually exclusive arguments to be displayed on a separate line.

``ArgumentParser::add_usage_newline()`` can also be used to force the next
argument to be displayed on a new line in the usage output.

The following snippet

```cpp
    argparse::ArgumentParser program("program");
    program.set_usage_max_line_width(80);
    program.set_usage_break_on_mutex();
    program.add_argument("--quite-long-option-name").flag();
    auto &group = program.add_mutually_exclusive_group();
    group.add_argument("-a").flag();
    group.add_argument("-b").flag();
    program.add_argument("-c").flag();
    program.add_argument("--another-one").flag();
    program.add_argument("-d").flag();
    program.add_argument("--yet-another-long-one").flag();
    program.add_argument("--will-go-on-new-line").flag();
    program.add_usage_newline();
    program.add_argument("--new-line").flag();
    std::cout << program.usage() << std::endl;
```

will display:
```console
Usage: program [--help] [--version] [--quite-long-option-name]
               [[-a]|[-b]]
               [-c] [--another-one] [-d] [--yet-another-long-one]
               [--will-go-on-new-line]
               [--new-line]
```

Furthermore arguments can be separated into several groups by calling
``ArgumentParser::add_group(group_name)``. Only optional arguments should
be specified after the first call to add_group().

```cpp
    argparse::ArgumentParser program("program");
    program.set_usage_max_line_width(80);
    program.add_argument("-a").flag().help("help_a");
    program.add_group("Advanced options");
    program.add_argument("-b").flag().help("help_b");
```

will display:
```console
Usage: program [--help] [--version] [-a]

Advanced options:
               [-b]
```

## Developer Notes

### Copying and Moving

`argparse::ArgumentParser` is intended to be used in a single function - setup everything and parse arguments in one place. Attempting to move or copy invalidates internal references (issue #260). Thus, starting with v3.0, `argparse::ArgumentParser` copy and move constructors are marked as `delete`.

## CMake Integration 

Use the latest argparse in your CMake project without copying any content.  

```cmake
cmake_minimum_required(VERSION 3.14)

PROJECT(myproject)

# fetch latest argparse
include(FetchContent)
FetchContent_Declare(
    argparse
    GIT_REPOSITORY https://github.com/p-ranav/argparse.git
)
FetchContent_MakeAvailable(argparse)

add_executable(myproject main.cpp)
target_link_libraries(myproject argparse)
```

## Bazel Integration

Add an `http_archive` in WORKSPACE.bazel, for example

```starlark
http_archive(
    name = "argparse",
    sha256 = "674e724c2702f0bfef1619161815257a407e1babce30d908327729fba6ce4124",
    strip_prefix = "argparse-3.1",
    url = "https://github.com/p-ranav/argparse/archive/refs/tags/v3.1.zip",
)
```

## Building, Installing, and Testing

```bash
# Clone the repository
git clone https://github.com/p-ranav/argparse
cd argparse

# Build the tests
mkdir build
cd build
cmake -DARGPARSE_BUILD_SAMPLES=on -DARGPARSE_BUILD_TESTS=on ..
make

# Run tests
./test/tests

# Install the library
sudo make install
```

## Supported Toolchains

| Compiler             | Standard Library | Test Environment   |
| :------------------- | :--------------- | :----------------- |
| GCC >= 8.3.0         | libstdc++        | Ubuntu 18.04       |
| Clang >= 7.0.0       | libc++           | Xcode 10.2         |
| MSVC >= 16.8         | Microsoft STL    | Visual Studio 2019 |

## Contributing
Contributions are welcome, have a look at the [CONTRIBUTING.md](CONTRIBUTING.md) document for more information.

## License
The project is available under the [MIT](https://opensource.org/licenses/MIT) license.
