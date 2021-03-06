# Elixir Style Guide

> A programmer does not primarily write code; rather, he primarily writes to another programmer about his problem solution. The understanding of this fact is the final step in his maturation as technician.
>
> — <cite>What a Programmer Does, 1967</cite>

## Table of Contents

* [Linting](#linting)
  * [Naming](#naming)
  * [Comments](#comments)
  * [Modules](#modules)
  * [Regular Expressions](#regular-expressions)
  * [Structs](#structs)
  * [Exceptions](#exceptions)
  * [ExUnit](#exunit)

The following section are automatically applied by the code formatter in Elixir v1.6 and listed here only for documentation purposes:

* [Formatting](#formatting)
  * [Whitespace](#whitespace)
  * [Indentation](#indentation)
  * [Term representation](#term-representation)
  * [Parentheses](#parentheses)
  * [Layout](#layout)

## Linting

* <a name="pipeline-operator"></a>
  Favor the pipeline operator `|>` to chain function calls together.
  <sup>[[link](#pipeline-operator)]</sup>

  ```elixir
  # Bad
  String.downcase(String.strip(input))

  # Good
  input |> String.strip() |> String.downcase()
  ```

  For a multi-line pipeline, place each function call on a new line, and retain the level of indentation.

  ```elixir
  input
  |> String.strip()
  |> String.downcase()
  |> String.slice(1, 3)

* <a name="needless-pipeline"></a>
  Avoid needless pipelines like the plague.
  <sup>[[link](#needless-pipeline)]</sup>

  ```elixir
  # Bad
  result = input |> String.strip()

  # Good
  result = String.strip(input)
  ```

* <a name="anonymous-pipeline"></a>
  Don't use anonymous functions in pipelines.
  <sup>[[link](#anonymous-pipeline)]</sup>

  ```elixir
  # Bad
  sentence
  |> String.split(~r/\s/)
  |> (fn words -> [@sentence_start | words] end).()
  |> Enum.join(" ")

  # Good
  split_sentence = String.split(sentence, ~r/\s/)
  Enum.join([@sentence_start | split_sentence], " ")
  ```

  Consider defining private helper function when appropriate:

  ```elixir
  # Good
  sentence
  |> String.split(~r/\s/)
  |> prepend(@sentence_start)
  |> Enum.join(" ")
  ```

* <a name="no-else-with-unless"></a>
  Never use `unless` with `else`. Rewrite these with the positive case first.
  <sup>[[link](#no-else-with-unless)]</sup>

  ```elixir
  # Bad
  unless Enum.empty?(coll) do
    :ok
  else
    :error
  end

  # Good
  if Enum.empty?(coll) do
    :error
  else
    :ok
  end
  ```

* <a name="no-nil-else"></a>
  Omit the `else` option in `if` and `unless` constructs if `else` returns `nil`.
  <sup>[[link](#no-nil-else)]</sup>

  ```elixir
  # Bad
  if byte_size(data) > 0, do: data, else: nil

  # Good
  if byte_size(data) > 0, do: data
  ```

* <a name="true-in-cond"></a>
  If you have an always-matching clause in the `cond` special form, use `true` as its condition.
  <sup>[[link](#true-in-cond)]</sup>

  ```elixir
  # Bad
  cond do
    char in ?0..?9 ->
      char - ?0

    char in ?A..?Z ->
      char - ?A + 10

    :other ->
      char - ?a + 10
  end

  # Good
  cond do
    char in ?0..?9 ->
      char - ?0

    char in ?A..?Z ->
      char - ?A + 10

    true ->
      char - ?a + 10
  end
  ```

* <a name="boolean-operators"></a>
  Never use `||`, `&&`, and `!` for strictly boolean checks. Use these operators only if any of the arguments are non-boolean.
  <sup>[[link](#boolean-operators)]</sup>

  ```elixir
  # Bad
  is_atom(name) && name != nil
  is_binary(task) || is_atom(task)

  # Good
  is_atom(name) and name != nil
  is_binary(task) or is_atom(task)
  line && line != 0
  file || "sample.exs"
  ```

* <a name="patterns-matching-binaries"></a>
  Favor the binary concatenation operator `<>` over bitstring syntax for patterns matching binaries.
  <sup>[[link](#patterns-matching-binaries)]</sup>

  ```elixir
  # Bad
  <<"http://", _rest::bytes>> = input
  <<first::utf8, rest::bytes>> = input

  # Good
  "http://" <> _rest = input
  <<first::utf8>> <> rest = input
  ```

### Naming

* <a name="snake-case-atoms-funs-vars-attrs"></a>
  Use `snake_case` for functions, variables, module attributes, and atoms.
  <sup>[[link](#snake-case-atoms-funs-vars-attrs)]</sup>

  ```elixir
  # Bad
  :"no match"
  :Error
  :badReturn

  fileName = "sample.txt"

  @_VERSION "0.0.1"

  def readFile(path) do
    # ...
  end

  # Good
  :no_match
  :error
  :bad_return

  file_name = "sample.txt"

  @version "0.0.1"

  def read_file(path) do
    # ...
  end
  ```

* <a name="camelcase-modules"></a>
  Use `CamelCase` for module names. Keep uppercase acronyms as uppercase.
  Module names should be descriptive and as unique as possible to make code navigation easy for developers.  The `path-to` does not need to be included in the name as elixir treats the dots in a difference fashion than other languages.
  <sup>[[link](#camelcase-modules)]</sup>
  

  ```elixir
  # Bad
  defmodule :appStack do
    # ...
  end

  defmodule App_Stack do
    # ...
  end

  defmodule Appstack do
    # ...
  end

  defmodule Html do
    # ...
  end

  # Good
  defmodule AppStack do
    # ...
  end

  defmodule HTML do
    # ...
  end
  ```

* <a name="predicate-funs-name"></a>
  The names of predicate functions (functions that return a boolean value) should have a trailing question mark `?` rather than a leading `has_` or similar.
  <sup>[[link](#predicate-funs-name)]</sup>

  ```elixir
  # Bad
  def is_leap(year) do
    # ...
  end

  # Good
  def leap?(year) do
    # ...
  end
  ```

  Always use a leading `is_` when naming guard-safe predicate macros.

  ```elixir
  defmacro is_date(month, day) do
    # ...
  end
  ```

* <a name="snake-case-dirs-files"></a>
  Use `snake_case` for naming directories and files, for example `lib/my_app/task_server.ex`.
  <sup>[[link](#snake-case-dirs-files)]</sup>

* <a name="one-letter-var"></a>
  Avoid using one-letter variable names.
  <sup>[[link](#one-letter-var)]</sup>

### Comments

> Remember, good code is like a good joke: It needs no explanation.
>
> — <cite>Russ Olsen</cite>

* <a name="critical-comments"></a>
  Use code comments only to communicate important details to another person reading the code.
  For example, a high-level description of the algorithm being implemented or why certain
  critical decisions, such as optimization or business rules, were made.
  TODO's should be used to track the work a developer is currently working on and should not appear in a comment
  <sup>[[link](#critical-comments)]</sup>

```elixir
# Bad
  #TODO: fix this up when the opportunity arrises
```

* <a name="no-superfluous-comments"></a>
  Avoid superfluous comments.
  <sup>[[link](#no-superfluous-comments)]</sup>

  ```elixir
  # Bad
  String.first(input) # Get first grapheme.  
  ```

### Modules

* <a name="module-layout"></a>
  Use a consistent structure when calling `use`/`import`/`alias`/`require`: call them in this order and group multiple calls to each of them.
  <sup>[[link](#module-layout)]</sup>

  ```elixir
  use GenServer

  import Bitwise
  import Kernel, except: [length: 1]

  alias Mix.Utils
  alias MapSet, as: Set

  require Logger
  ```

* <a name="current-module-reference"></a>
  Use the `__MODULE__` pseudo-variable to reference the current module.
  <sup>[[link](#current-module-reference)]</sup>

  ```elixir
  # Bad
  :ets.new(Kernel.LexicalTracker, [:named_table])
  GenServer.start_link(Module.LocalsTracker, nil, [])

  # Good
  :ets.new(__MODULE__, [:named_table])
  GenServer.start_link(__MODULE__, nil, [])
  ```

### Regular Expressions

* <a name="pattern-matching-over-regexp"></a>
  Regular expressions are the last resort. Pattern matching and the `String` module are things to start with.
  <sup>[[link](#pattern-matching-over-regexp)]</sup>

  ```elixir
  # Bad
  Regex.run(~r/#(\d{2})(\d{2})(\d{2})/, color)
  Regex.match?(~r/(email|password)/, input)

  # Good
  <<?#, p1::2-bytes, p2::2-bytes, p3::2-bytes>> = color
  String.contains?(input, ["email", "password"])
  ```

* <a name="non-capturing-regexp"></a>
  Use non-capturing groups when you don't use the captured result.
  <sup>[[link](#non-capturing-regexp)]</sup>

  ```elixir
  ~r/(?:post|zip )code: (\d+)/
  ```

* <a name="caret-and-dollar-regexp"></a>
  Be careful with `^` and `$` as they match start and end of the __line__ respectively. If you want to match the __whole__ string use: `\A` and `\z` (not to be confused with `\Z` which is the equivalent of `\n?\z`).
  <sup>[[link](#caret-and-dollar-regexp)]</sup>

### Structs

* <a name="defstruct-fields-default"></a>
  When calling `defstruct/1`, don't explicitly specify `nil` for fields that default to `nil`.
  <sup>[[link](#defstruct-fields-default)]</sup>

  ```elixir
  # Bad
  defstruct first_name: nil, last_name: nil, admin?: false

  # Good
  defstruct [:first_name, :last_name, admin?: false]
  ```

### Exceptions

* <a name="exception-naming"></a>
  Make exception names end with a trailing `Error`.
  <sup>[[link](#exception-naming)]</sup>

  ```elixir
  # Bad
  BadResponse
  ResponseException

  # Good
  ResponseError
  ```

* <a name="exception-message"></a>
  Use non-capitalized error messages when raising exceptions, with no trailing punctuation.
  <sup>[[link](#exception-message)]</sup>

  ```elixir
  # Bad
  raise ArgumentError, "Malformed payload."

  # Good
  raise ArgumentError, "malformed payload"
  ```

  There is one exception to the rule - always capitalize Mix error messages.

  ```elixir
  Mix.raise("Could not find dependency")
  ```

### ExUnit

* <a name="exunit-assertion-side"></a>
  When asserting (or refuting) something with comparison operators (such as `==`, `<`, `>=`, and similar), put the expression being tested on the left-hand side of the operator and the value you're testing against on the right-hand side.
  <sup>[[link](#exunit-assertion-side)]</sup>

  ```elixir
  # Bad
  assert "héllo" == Atom.to_string(:"héllo")

  # Good
  assert Atom.to_string(:"héllo") == "héllo"
  ```

  When using the match operator `=`, put the pattern on the left-hand side (as it won't work otherwise).

  ```elixir
  assert {:error, _reason} = File.stat("./non_existent_file")
  ```

## Formatting

The rules below are automatically applied by the code formatter in Elixir v1.6.
They are provided here for documentation purposes and for those maintaining older codebases.

### Whitespace

>  Whitespace might be (mostly) irrelevant to the Elixir compiler, but its proper use is the key to writing easily readable code.

* <a name="no-trailing-whitespaces"></a>
  Avoid trailing whitespaces.
  <sup>[[link](#no-trailing-whitespaces)]</sup>

* <a name="newline-eof"></a>
  End each file with a newline.
  <sup>[[link](#newline-eof)]</sup>

* <a name="spaces-indentation"></a>
  Use two __spaces__ per indentation level. No hard tabs.
  <sup>[[link](#spaces-indentation)]</sup>

  ```elixir
  # Bad
  def register_attribute(name, opts) do
      register_attribute(__MODULE__, name, opts)
  end

  # Good
  def register_attribute(name, opts) do
    register_attribute(__MODULE__, name, opts)
  end
  ```

* <a name="spaces-in-code"></a>
  Use a space before and after binary operators. Use a space after commas `,`, colons `:`, and semicolons `;`. Do not put spaces around matched pairs like brackets `[]`, braces `{}`, and so on.
  <sup>[[link](#spaces-in-code)]</sup>

  ```elixir
  # Bad
  sum = 1+1
  [first|rest] = 'three'
  {a1,a2} = {2 ,3}
  Enum.join( [ "one" , << "two" >>, sum ])

  # Good
  sum = 1 + 2
  [first | rest] = 'three'
  {a1, a2} = {2, 3}
  Enum.join(["one", <<"two">>, sum])
  ```

* <a name="no-spaces-in-code"></a>
  Use no spaces after unary operators and inside range literals. The only exception is the `not` operator: use a space after it.
  <sup>[[link](#no-spaces-in-code)]</sup>

  ```elixir
  # Bad
  angle = - 45
  ^ result = Float.parse("42.01")

  # Good
  angle = -45
  ^result = Float.parse("42.01")
  2 in 1..5
  not File.exists?(path)
  ```

* <a name="default-arguments"></a>
  Use spaces around default arguments `\\` definition.
  <sup>[[link](#default-arguments)]</sup>

  ```elixir
  # Bad
  def start_link(fun, options\\[])

  # Good
  def start_link(fun, options \\ [])
  ```

* <a name="bitstring-segment-options"></a>
  Do not put spaces around segment options definition in bitstrings.
  <sup>[[link](#bitstring-segment-options)]</sup>

  ```elixir
  # Bad
  <<102 :: unsigned-big-integer, rest :: binary>>
  <<102::unsigned - big - integer, rest::binary>>

  # Good
  <<102::unsigned-big-integer, rest::binary>>
  ```

* <a name="leading-space-comment"></a>
  Use one space between the leading `#` character of the comment and the text of the comment.
  <sup>[[link](#leading-space-comment)]</sup>

  ```elixir
  # Bad
  #Amount to take is greater than the number of elements

  # Good
  # Amount to take is greater than the number of elements
  ```

* <a name="space-before-anonymous-fun-arrow"></a>
  Always use a space before `->` in 0-arity anonymous functions.
  <sup>[[link](#space-before-anonymous-fun-arrow)]</sup>

  ```elixir
  # Bad
  Task.async(fn->
    ExUnit.Diff.script(left, right)
  end)

  # Good
  Task.async(fn ->
    ExUnit.Diff.script(left, right)
  end)
  ```

### Indentation

* <a name="binary-ops-indentation"></a>
  <a name="guard-clauses"></a>
  Indent the right-hand side of a binary operator one level more than the left-hand side if left-hand side and right-hand side are on different lines. The only exceptions are `when` in guards and `|>`, which go on the beginning of the line and should be indented at the same level as their left-hand side. Do this also for binary operators when assigning.
  <sup>[[link](#binary-ops-indentation)]</sup>

  ```elixir
  # Bad

  "No matching message.\n" <>
  "Process mailbox:\n" <>
  mailbox

  message =
    "No matching message.\n" <>
    "Process mailbox:\n" <>
    mailbox

  input
    |> String.strip()
    |> String.downcase()

  defp valid_identifier_char?(char)
    when char in ?a..?z
      when char in ?A..?Z
      when char in ?0..?9
      when char == ?_ do
    true
  end

  defp parenless_capture?({op, _meta, _args})
       when is_atom(op) and
       atom not in @unary_ops and
       atom not in @binary_ops do
    true
  end

  # Good

  "No matching message.\n" <>
    "Process mailbox:\n" <>
    mailbox

  message =
    "No matching message.\n" <>
      "Process mailbox:\n" <>
      mailbox

  input
  |> String.strip()
  |> String.downcase()

  defp valid_identifier_char?(char)
       when char in ?a..?z
       when char in ?A..?Z
       when char in ?0..?9
       when char == ?_ do
    true
  end

  defp parenless_capture?({op, _meta, _args})
       when is_atom(op) and
              atom not in @unary_ops and
              atom not in @binary_ops do
    true
  end
  ```

* <a name="with-indentation"></a>
  Use the indentation shown below for the `with` special form:
  <sup>[[link](#with-indentation)]</sup>

  ```elixir
  with {year, ""} <- Integer.parse(year),
       {month, ""} <- Integer.parse(month),
       {day, ""} <- Integer.parse(day) do
    new(year, month, day)
  else
    _ ->
      {:error, :invalid_format}
  end
  ```

  Always use the indentation above if there's an `else` option. If there isn't, the following indentation works as well:

  ```elixir
  with {:ok, date} <- Calendar.ISO.date(year, month, day),
       {:ok, time} <- Time.new(hour, minute, second, microsecond),
       do: new(date, time)
  ```

* <a name="for-indentation"></a>
  Use the indentation shown below for the `for` special form:
  <sup>[[link](#for-indentation)]</sup>

  ```elixir
  for {alias, _module} <- aliases_from_env(server),
      [name] = Module.split(alias),
      starts_with?(name, hint),
      into: [] do
    %{kind: :module, type: :alias, name: name}
  end
  ```

  If the body of the `do` block is short, the following indentation works as well:

  ```elixir
  for partition <- 0..(partitions - 1),
      pair <- safe_lookup(registry, partition, key),
      into: [],
      do: pair
  ```

* <a name="expression-group-alignment"></a>
  Avoid aligning expression groups:
  <sup>[[link](#expression-group-alignment)]</sup>

  ```elixir
  # Bad
  module = env.module
  arity  = length(args)

  def inspect(false), do: "false"
  def inspect(true),  do: "true"
  def inspect(nil),   do: "nil"

  # Good
  module = env.module
  arity = length(args)

  def inspect(false), do: "false"
  def inspect(true), do: "true"
  def inspect(nil), do: "nil"
  ```

  The same non-alignment rule applies to `<-` and `->` clauses as well.

* <a name="pipeline-indentation"></a>
  Use a single level of indentation for multi-line pipelines.
  <sup>[[link](#pipeline-indentation)]</sup>

  ```elixir
  input
  |> String.strip()
  |> String.downcase()
  |> String.slice(1, 3)
  ```

### Term representation

* <a name="underscores-in-numerics"></a>
  Add underscores to decimal literals that have six or more digits.
  <sup>[[link](#underscores-in-numerics)]</sup>

  ```elixir
  # Bad
  num = 1000000
  num = 1_500

  # Good
  num = 1_000_000
  num = 1500
  ```

* <a name="hex-literals"></a>
  Use uppercase letters when using hex literals.
  <sup>[[link](#hex-literals)]</sup>

  ```elixir
  # Bad
  <<0xef, 0xbb, 0xbf>>

  # Good
  <<0xEF, 0xBB, 0xBF>>
  ```

* <a name="quotes-around-atoms"></a>
  When using atom literals that need to be quoted because they contain characters that are invalid in atoms (such as `:"foo-bar"`), use double quotes around the atom name:
  <sup>[[link](#quotes-around-atoms)]</sup>

  ```elixir
  # Bad
  :'foo-bar'
  :'atom number #{index}'

  # Good
  :"foo-bar"
  :"atom number #{index}"
  ```

* <a name="trailing-comma"></a>
  When dealing with lists, maps, structs, or tuples whose elements span over multiple lines and are on separate lines with regard to the enclosing brackets, it's advised to *not* use a trailing comma on the last element:
  <sup>[[link](#trailing-comma)]</sup>

  ```elixir
  [
    :foo,
    :bar,
    :baz
  ]
  ```

### Parentheses

* <a name="zero-arity-parens"></a>
  Parentheses are a must for __local__ or __imported__ zero-arity function calls.
  <sup>[[link](#zero-arity-parens)]</sup>

  ```elixir
  # Bad
  pid = self
  import System, only: [schedulers_online: 0]
  schedulers_online

  # Good
  pid = self()
  import System, only: [schedulers_online: 0]
  schedulers_online()
  ```

  The same should be done for __remote__ zero-arity function calls:

  ```elixir
  # Bad
  Mix.env

  # Good
  Mix.env()
  ```

  This rule also applies to one-arity function calls (both local and remote) in pipelines:

  ```elixir
  # Bad
  input
  |> String.strip
  |> decode

  # Good
  input
  |> String.strip()
  |> decode()
  ```

* <a name="anonymous-fun-parens"></a>
  Never wrap the arguments of anonymous functions in parentheses.
  <sup>[[link](#anonymous-fun-parens)]</sup>

  ```elixir
  # Bad
  Agent.get(pid, fn(state) -> state end)
  Enum.reduce(numbers, fn(number, acc) ->
    acc + number
  end)

  # Good
  Agent.get(pid, fn state -> state end)
  Enum.reduce(numbers, fn number, acc ->
    acc + number
  end)
  ```

* <a name="fun-parens"></a>
  Always use parentheses around arguments to definitions (such as `def`, `defp`, `defmacro`, `defmacrop`, `defdelegate`). Don't omit them even when a function has no arguments.
  <sup>[[link](#fun-parens)]</sup>

  ```elixir
  # Bad
  def main arg1, arg2 do
    # ...
  end

  defmacro env do
    # ...
  end

  # Good
  def main(arg1, arg2) do
    # ...
  end

  defmacro env() do
    # ...
  end
  ```

* <a name="parens-in-zero-arity-types"></a>
  Always use parens on zero-arity types.
  <sup>[[link](#parens-in-zero-arity-types)]</sup>

  ```elixir
  # Bad
  @spec start_link(module, term, Keyword.t) :: on_start

  # Good
  @spec start_link(module(), term(), Keyword.t()) :: on_start()
  ```

### Layout

* <a name="no-semicolon"></a>
  Use one expression per line. Don't use semicolons (`;`) to separate statements and expressions.
  <sup>[[link](#no-semicolon)]</sup>

  ```elixir
  # Bad
  stacktrace = System.stacktrace(); fun.(stacktrace)

  # Good
  stacktrace = System.stacktrace()
  fun.(stacktrace)
  ```

* <a name="multi-line-expr-assignment"></a>
  When assigning the result of a multi-line expression, begin the expression on a new line.
  <sup>[[link](#multi-line-expr-assignment)]</sup>

  ```elixir
  # Bad
  {found, not_found} = files
                       |> Enum.map(&Path.expand(&1, path))
                       |> Enum.partition(&File.exists?/1)

  prefix = case base do
             :binary -> "0b"
             :octal -> "0o"
             :hex -> "0x"
           end

  # Good
  {found, not_found} =
    files
    |> Enum.map(&Path.expand(&1, path))
    |> Enum.partition(&File.exists?/1)

  prefix =
    case base do
      :binary -> "0b"
      :octal -> "0o"
      :hex -> "0x"
    end
  ```

* <a name="binary-operators-at-eols"></a>
  When writing a multi-line expression, keep binary operators at the end of each line. The only exception is the `|>` operator (which goes at the beginning of the line).
  <sup>[[link](#binary-operators-at-eols)]</sup>

  ```elixir
  # Bad

  "No matching message.\n"
    <> "Process mailbox:\n"
    <> mailbox

  input |>
    String.strip() |>
    decode()

  # Good
  "No matching message.\n" <>
    "Process mailbox:\n" <>
    mailbox

  input
  |> String.strip()
  |> decode()
  ```

## License

This work was created by Aleksei Magusev and is licensed under [the CC BY 4.0 license](https://creativecommons.org/licenses/by/4.0).

![Creative Commons License](http://i.creativecommons.org/l/by/4.0/88x31.png)

## Credits

The structure of the guide and some points that are applicable to Elixir were taken from [the community-driven Ruby coding style guide](https://github.com/bbatsov/ruby-style-guide).
