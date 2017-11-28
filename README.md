
# Table of contents

1. Introduction
2. Basic types
3. Basic operators
4. Pattern matching
5. case, cond and if
6. Binaries, strings and char lists
7. Keywords and maps
8. Modules and Functions
9. Recursion
10. Enumerables and streams
11. Processes
12. IO and the file system
13. alias, require and import
14. Module attributes
15. Structs
16. Protocols
17. Comprehension
18. Sigils
19. try, catch and rescue
20. Typespecs and behaviours
21. Erlang libraries
22. Where to go next




# Basic types

## Basics
``` elixir
1            # integer
0x1F         # integer
1.0          # float
true
:atom
"elixir"
[1, 2, 3]    # list
{1, 2, 3}    # tuple

10 / 2       # returns a float
div(10, 2)   # int
rem 10, 3

# function arity matters
> h myfunction/1
> h myfunction/2
```


## Booleans
```elixir
# bool
> true
true

> true == false
false

> is_boolean true
true

> is_boolean 1
false
```


## Atoms

```elixir
# an atom is its own value
> :hello
:hello

> is_atom :hello
true

# aliases start in upper case are also atoms
> is_atom Hello
true

> :hello == :world
false

> true == :true
true

> is_boolean :false
true

```


## Strings

``` elixir
> "hellö"
"hellö"

# interpolation
> "hellö #{:world}"
"hellö world"


# escape
> IO.puts "hello\nworld"

# strings are binaries
> is_binary("hellö")
true

> byte_size "hellö"
6

> String.length "hellö"
5

> String.upcase "hellö"
"HELLÖ"
```


## Anonymous functions

```elixir
> add = fn a, b -> a + b end

> add. 1 2
3

> is_function add
true

> is_function add 2
true

# the & shorthand
> add = &(&1 + &2)

# clause and guards
> f = fn
>   x, y when x > 0  -> x + y
>   x, y -> x * y
> end

```


## (linked) Lists

```elixir
> [1, 2, true, :hello]
[1, 2, true, :hello]


# concatenation
> [1,2,3] ++ [4,5,6]
[1,2,3,4,5,6]

#hd and tl
> list = [1,2,3]
> hd list
1

> tl list
[2,3]


# single quotes VS double-quotes
'hello' == "hello"
false

> [104, 101, 108, 108, 111]
'hello'

> i 'hello'
... data type List ...
```


Lists are stored in memory as linked lists,

* list length computation is linear
* performance of list concatenation depends of left-hand list length


```elixir
> list = [1,2,3]
> [0] ++ list      # traverse only 1 element before append
> list ++ 4        # traverse 4 elements...
```


## Tuples

```elixir
> {1,2, :hello, false}
{1,2, :hello, false}

> tuple_size {:ok, "hello"}
2

> mytuple = {:ok, "hello"}
{:ok, "hello"}

> elem(tuple, 1)
"hello"

> tuple_size tuple
2


# immutability
> put_elem tuple 1 "world"
{:ok, "world"}

> tuple
{:ok, 'hello'}
```

Tuples are stored contiguously in memory:

* getting tuple size is fast
* accessing element by index is fast
* update or adding new element requires new tuple, it's an expensive operation


Elixir guidance

* there is `elem\2` for tuples but not for lists
* `size` means constant time
* `length` means linear time


# Basic operators

```elixir
+, -, *, /, div/2, rem/2   # arithmetics

++, --  # list concatenation, dissociation

<>      # string concatenation

and, or, not  # only booleans

||, &&, !     # boolean and other (all values except false and nil evaluate to true)

== vs ===, the latter is more strict when comparing integers and float
```


# Pattern matching

`=` is actually called _the match operator_

```elixir
> x = 1
1

> x
1

> 1 = x
1

> 2 = x
... match error ...

> {a,b,c} = {:hello, "world", 42}
{:hello, "world", 42}

> a
:hello

> b
"world"

> {:ok, result} = {:ok, 13}
> result
13

> [a,b,c] = [1,2,3]
> [head|tail] = [1,2,3]

> list = [1,2,3]
> [0|list]
[0,1,2,3]

# pin operator
> x = 1
> x = 2
> x
2
> ^x = 1
... match error...
```


# case, cond and if

## case

```elixir
> case {1,2,3} do
>      {1,2,3} -> "won't match"
>      {1,x,3} -> "match and bind x to 2"
>            _ -> "match any value"
> end    
"match and bind x to 2"


> case 10 do
>      ^x -> "match if x is 10, otherwise no bound"
>       _ -> "will match"
> end
"will match"


# guard
> case {1,2,3} do
>      {1,x,3} when x>0 -> "will match"
>                     _ -> "would match if guard was not satisfied"
> end
"will match"


# error in guard wont bubble
> case 1 do
>      x when raise_error() -> "won't match"
>      x -> "Got #{x}"
> end     


# CaseClauseError
> case :ok do
>      :error -> "won't match"
> end     
... CaseClauseError ...
```


## cond

`cond` check different conditions and find the first one that evaluates to `true`.

```elixir

> cond do
>     2 + 2 == 5 -> "this will not be true"
>     2 * 2 == 3 -> "nor this"
>     1 + 1 == 2 -> "this will"
> end    
"this will"


# CondClauseError
> cond do
>     false -> "won't be true"
> end    
... CondClauseError ...


# catch all clause
> cond do
>     2 + 2 == 5 -> "this will not be true"
>     2 * 2 == 3 -> "nor this"
>     true -> "this is always true (equivalent to else)"
> end    
"this is always true (equivalent to else)"


# any value besides nil and false are true
> cond do
>     hd [1,2,3] -> "1 is considered true"
> end    
"1 is considered true"
```


## if and unless

```elixir
> if true do
>     "this works"
> end    
"this works"


> unless true do
>     "this will never happen"
> end    


> if nil do
>     "this won't happen"
> else
>     "this will"
> end    
"this will"
```

## do / end blocks

`if/2` and `unless\2` are macros...

```elixir
# passing arguments using keywords list
> if true, do: 1 + 2
3

> if false, do: :this, else: :that
:that

> if true, do: (
>     a = 1 + 2
>     a + 10
> )
13
```


# Binaries, strings and char lists

## UTF-8 and Unicode

```elixir
> string = "hełło"
"hełło"
> byte_size(string)
7
> String.length string
5

# get character code point
> ?a
97        # 1 byte
> ?ł
322       # 2 bytes


# split by codepoints
> String.codepoints "hełło"
["h", "e", "ł", "ł", "o"]
```

## Binaries (bitstrings)

a binary is defined using `<<>>`
```elixir
> <<0, 1, 2, 3>>
<<0, 1, 2, 3>>

> byte_size <<0, 1, 2, 3>>
4

> String.valid?(<<239,191,19>>)
false

# string concatenation is actually a binary operation
> <<0,1>> <> <<2,3>>
<<0,1,2,3>>

# common trick...
> "hełło" <> <<0>>
<<104, 101, 197, 130, 197, 130, 111, 0>>


# each number given to a binary is meant to represent a byte
> <<255>>
<<255>>

> <<256>>
<<0>>      # truncated to byte

> <<256 :: size(16)>> # use 16 bits (2 bytes) to store the number
<<1,0>>

> <<256 :: utf8 >>   # the number is a code code point
"Ā"

> <<256 :: utf8, 0>>
<<196, 128, 0>>
```

stop page 51, to be continued


# Keywords and maps

todo


# Modules and Functions

A module groups several function, e.g. the `String` module.


```elixir
# create a module
> defmodule Math do
>     def sum(a, b) do
>         a + b
>     end
> end

> Math.sum 1 2
3
```

## Compilation

```elixir
$ cat math.ex

defmodule Math do
    def sum(a, b) do
        a + b
    end
end


$ elixirc math.ex
$ iex
> Math.sum 1 2
3
```

Elixir project structure :
* `ebin/` contains the compiled bytecode
* `lib/` contains elixir code (`.ex` files)
* `test/` contains tests (`.exs` files)


## Scripted mode

While `.ex` files are meant to be compiled, `.exs` files are for scripting.

```
$ cat math.exs

def module Math do
    def sum(a, b) do
        a + b
    end
end

IO.puts Math.sum 1 2

$ elixir math.exs
3 # the file compiles in memory and executed, no bytecode file is generated
```


## Named functions

Private versus public function (`defp` vs. `def`)

```elixir

defmodule Math do

    def sum(a, b) do      # public
        do_sum(a, b)
    end

    defp do_sum(a, b) do   # private
        a + b
    end
end

IO.puts Math.sum(1,2)      # 3
IO.puts Math.do_sum(1,2)   # UndefinedFunctionError
```


Using `clause` and `guard`

```elixir
defmodule Math do

    def zero?(0) do
        true
    end

    def zero?(x) when is_integer(x) do
        false
    end
end    

IO.puts Math.zero?(0)         #=> true
IO.puts Math.zero?(1)         #=> false
IO.puts Math.zero?([1, 2, 3]) #=> ** (FunctionClauseError)
IO.puts Math.zero?(0.0)       #=> ** (FunctionClauseError)
```


Using keyword list format...

```elixir
defmodule Math do
    def zero?(0), do: true
    def zero?(x) when is_integer(x), do: false
end    
```


## Function capturing
Using the arity notation - `name/arity`, to retrieve named function as function type.

```elixir
$ iex math.exs
> Math.zero?(0)
true

> fun = &Math.zero?/1
> is_function(fun)
true
> fun.(0)
true
```

Captured named functions behave like anonymous function, they can be assigned to
variables and passed as arguments.

```elixir
> (&is_function/1).(fun)
true

> fun = &(&1 + 1   # equivalent of : fn x -> x + 1 end.
> fun.(1)
2


> fun = &List.flatten(&1, &2)
> fun.([1, [[2], 3]], [4, 5])
[1, 2, 3, 4, 5]


# the following 3 are equivalent
> fun = &List.flatten(&1, &2)
> fun = fn (list, tail) -> List.flatten (list, tail) end
> fun = &List.flatten/2
```


## Default arguments

```elixir
defmodule Concat do
    def join (a, b, sep \\ "") do
        a <> sep <> b
    end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
```


Default values are not evaluated during function definition, only when call.

```elixir
> defmodule DefaultTest do
>     def dowork(x \\ "hello") do
>         x
>     end
> end

> DefaultTest.dowork
"hello"

> DefaultTest.dowork 123
123

> DefaultTest.dowork
"hello"
```

If a function with default values has multiple clauses, a function head is required...

```elixir
defmodule Concat do
    def join (a, b \\ nil, sep \\ "")

    def join (a, b, _sep) when is_nil(b) do
        a
    end

    def join(a, b, sep) do
        a <> sep <> b
    end
end

```


When using default values, be carefull of overlapping function definition...

```elixir
defmodule Concat do
    def join(a, b) do
        IO.puts "...first join"
        a <> b
    end

    def join(a, b, sep \\ "") do
        IO.puts "...second join"
        a <> sep <> b
    end
end

# at compile time :
warning: this clause cannot match because a previous clause at line 2 always matches
```



# Recursion

Because of immutability recursion is needed...


```elixir
defmodule Recursion do

    # the base case
    def print_multiple_times(msg, n) when n <= 1 do
        IO.puts msg
    end

    # second definition makes sure we get exactly one step closer to base case
    def print_multiple_times(msg, n) do
        IO.puts msg
        print_multiple_times(msg, n-1)
    end
end    
```


## Reduce and map algorithms

```elixir
# sum a list of numbers ... reduce algorithm
defmodule Math do
    def sum_list([], accumulator) do
        accumulator
    end

    def sum_list([head | tail], accumulator) do
        sum_list(tail, head + accumulator)
    end
end    

# double all values in list ... map algorithm
defmodule Math do
    def double_each([]) do
        []
    end
    def double_each([head | tail ]) do
        [head * 2 | double_each(tail)]
    end
end
```


In practice, recursion is not used in Elixir...
```elixir
> Enum.reduce([1,2,3], 0, fn(x, acc) -> x + acc end)
6

> Enum.map([1,2,3], fn(x) -> x * 2 end)
[2,4,6]
```

# Enumerables and streams

## Enumerables

Elixir provides the concept of enumerables and the `Enum` module to
work with them. `list` and `map` are examples of enumerables, however
the `Enum` module can work with any data type that implements the **Enumerable protocol**

```elixir
> Enum.map([1, 2, 3], &(&1 * 2))
[2, 4, 6]

> Enum.map(%{1 => 2, 3 => 4}, fn {k, v} -> k * v end)
[2, 12]


# with range
> Enum.map(1..3, &(&1 * 2))
[2, 4, 6]
> Enum.reduce(1..3, &+/2)
6
```

## Eager vs Lazy

All the functions in the ```Enum``` are eager, functions expect enumerable and return list back. When performing multiple operations, each operation generate an intermediate list...

```elixir
> odd? = &(rem(&1, 2) != 0)
> Enum.filter(1..3, odd?)
[1, 3]

# intermediate lists are generated...
> 1..100_000 |> Enum.map( &(&1 * 3)) |> Enum.filter(odd?) |> Enum.sum
```


## Streams

`Stream`, a lazy alternative to `Enum`

```elixir

> 1..100_000 |> Stream.map(&(&1 * 3))
... stream...

# composable
> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?)

# creating streams
> stream = Stream.cycle([1,2,3])
> Enum.take(stream, 10)
[1, 2, 3, 1, 2, 3, 1, 2, 3, 1]

# generate values from initial given value
> stream = Stream.unfold("hełło", &String.next_code-point\1)
> Enum.take(stream, 3)
["h", "e", "ł"]

# streamify a resource...
> stream = File.stream("/path/to/file")
> Enum.take(stream, 10)
# fetch the first 10 lines...
```


# Processes

Elixir's processes are extremely lightweight in terms of memory and CPU, it's not
uncommon to have tens of thousand of processes running simultaneously.

## spawn

```elixir
> spawn fn -> 1 + 2 end
#PID<0.43.0>


> pid = spawn fn -> 1 + 2 end
> Process.alive?(pid)
false


# get the pid of current process
> self()
> Process.alive?(self())
true
```



## send and receive

```elixir

> send self(), {:hello, "world"}
{:hello, "world"}

> receive do
>   {:hello, msg} -> msg
>   {:world, msg} -> msg
> end
"world"
```

Note:

* `send/2` is not blocking
* when a message is sent it goes to the process mailbox
* `receive/1` goes through the mailbox and wait until a msg matches its guard
* `receive/1` supports guards and many clauses
* if there is no msg matching `receive/2` waits indefinitely, unless a timeout was specified.


```elixir
# receive/1 with timeout
> receive do
>       {:hello, msg} -> msg
> after
        # a timeout can be 0 if the msg is expected
>       1_000 -> "nothing after 1 second"    expected to be in inbox.
> end
```

Complete example...
```elixir
> parent = self()
> spawn fn -> send(parent, {:hello, self()}) end
> receive do
>   {:hello, pid} -> "Got hello from #{inspect pid}"
> end
"Got hello from #PID<0.48.0>"

# flushes and print all messages in mailbox
> send self(), :hello
:hello
> flush()
:hello
:ok
```


## Links

Links are usefull when the failure of one process needs to propagate to another one.


```elixir
> self()
#PID<0.41.0>

> spawn_link fn -> raise "oops" end
... exit from PID 41 ...
... process PID X raised an exception
```


## Tasks

`spawn/1` and `spawn_link/1` are basic primitives for creating processes, `task`
is the most common abstractions that build on top of them...


```elixir
> Task.start fn -> raise "oops" end
{:ok, #PID<0.55.0>}
# better error report
```


## State

Where to keep application configuration ? Processes are the most common answer.


```elixir
# a module that starts new processes as a key-value store
defmodule KV do

    # start the loop process
    def start_link do
        Task.start_link(fn -> loop(%{}) end)
    end

    # wait for msg and perform appropriate action
    defp loop(map) do
        receive do
            {:get, key, caller} ->
                send caller, Map.get(map, key)
                loop(map)

            {:put, key, value} ->
                loop(Map.put(map, key, value))
        end
    end
end

> {:ok, pid} = KV.start_link
{:ok, #PID<0.62.0>}

> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}

> flush()
nil  # no key yet
:ok

# let's put in a key
> send pid, {:put, :hello, :world}
{:put, :hello, :world}

> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}

> flush()
:world
:ok

# the process keeps a state, and sending message update the state

# let's give the PID an explicit name
> Process.register(pid, :kv)
true

> send :kv, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}

> flush()
:world
:ok
```

## Agent

`Agent` is abstraction around state, it provides state, and name registration

```elixir
> {:ok, pid} = Agent.start_link(fn -> %{} end)
{:ok, #PID<0.41.0>}

> Agent.update(pid, fn map -> Map.put(map, :hello, :world) end)
:ok

> Agent.get(pid, fn map -> Map.get(map, :hello) end)
:world
```


# IO and the file system

## The IO module

```elixir
> IO.puts "hello world"
hello world
:ok

> IO.gets "yes or no?"
yes or no? yes
"yes\n"

# defaults are :stdin / :stdout, but can be changed
> IO.puts :stderr, "hello world"
hello world
:ok
```


## The File module

```elixir
> {:ok, file} = File.open "Hello", [:write]
{:ok, #PID<...>}

> IO.binwrite file, "world"
:ok

> File.close file
:ok

> File.read "hello"
{:ok, "world"}

# Unix style file operation
> h File.rm/1
> h File.mkdir/1
> h File.mkdir_p/1
> h File.cp_r/2
> h File.rm_rf/1

# return the contents, and fail spectacularly in case
> File.read! "hello"
"world"   # instead of {:ok, "world"}

> File.read "unlnown"
{:error, :enoent}

> File.read! "unknown"
..."unknown": no such file or directory


# handle different scenario using pattern matching
case File.read(file) do
    {:ok, body} -> # do sthg with body
    {:error, reason } -> # handle error
end


# use (!) if file is known to be there
{:ok, body} = File.read(file) # don't do this

```


## The Path module

```elixir
> Path.join("foo", "bar")
"foo/bar"

> Path.expand("~/hello")
"Users/jose/hello"

```


## Processes and group leaders

`IO` devices are modelled as process in Erlang, that allows different nodes in the
same network to exchange file process in order to read/write between nodes.

```elixir
# File.open/2 returns un tuple, IO module works with process
> {:ok, file} = File.open "hello", [:write]
{:ok, #PID<0.47.0>}

# IO operations are processes operations
# IO.write => IO will send a message to a process
> pid = spawn fn -> receive do: (msg -> IO.inspect msg) end
#PID<0.57.0>
> IO.write(pid, "hello")
{:io_request, #PID<0.41.0>, #Reference<0.0.8.91>, {:put_chars, :unicode, "hello"}}
** (ErlangError) erlang error: :terminated

# ^ failure because IO is expecting an unreturned result

StringIO provides an implementation of the IO device message on top of strings:
> {:ok, pid} = StringIO.open("hello")
{:ok, #PID<0.43.0>}
> IO.read(pid, 2)
 "he"
 ```

Of all `IO` devices, there is one that is special to each process: the **group
leader**. The GL writes to `:stdio`, it can be configured per process and can be
used for example to forward console output.


```elixir
> IO.puts :stdio, "hello"
hello
:ok
> IO.puts Process.group_leader, "hello"
hello
:ok
```


## `iodata` and `chardata`

Functions IO and File accept list (in addition to binary), in that case special
care is needed specially if the file is opened without encoding. A list may
either represent a bunch of bytes or a bunch of characters and which one to use
depends on the  encoding of the IO device.

```elixir
> IO.puts 'hello world'   # note the single quotes
hello world
:ok
> IO.puts ['hello', ?\s, "world"]
hello world
:ok

```




# alias, require and import

```elixir
# alias the module so it can be called as Bar instead of Foo.Bar
alias Foo.Bar, as: Bar

# require the module in order to use its macros
require Foo

# Import functions from Foo so they can be called without the `Foo.` prefix
import Foo

# Invoke the custom code defined in Foo as an extension point
use Foo
```


Note:

* `alias`, `require` and `import` are called directives because they have **lexical scope**
* `use` is a common extension point



## alias

```elixir
> alias Math.List, as: List

# the following 2 are equivalent
> alias Math.List
> alias Math.List, as: List

# alias is lexically scoped...

defmodule Math do
    def plus(a, b) do
        alias Math.List   # the alias is valid only inside plus/2
        # ...
    end

    def minus(a, b) do
        # ...
    end
end
```


`alias` behind the scene
```elixir
# an alias is a capitalized identifier which is converted to an atom at compilation time

> is_atom(String)
true

> to_string(String)
"Elixir.String"

> :"Elixir.String" == String
true
```


## require

In order to use macros, you need to opt-in by requiring the module they
are defined in.

```elixir
> Integer.is_odd(3)
... undefined function error ...

> require Integer
Integer

> Integer.is_odd(3)
true
```


## import

```elixir
# only: is a best practice, equivalent of avoiding Python's import *
> import List, only: [duplicate: 2]
List

> duplicate :ok, 3
[:ok, :ok, :ok]

# to import all macros
import Integer, only: :macros

# to import all functions
import Integer, only: :functions


# import is lexically scoped too
defmodule Math do
    def f do
        import List, only: [duplicate: 2]
        # ...
    end
end
```

Note: `import`ing a module automatically `require`s it.


## use

to bring external functionality into the current lexical scope.

```elixir
# write unit tests using the ExUnit framework
defmodule AssertionTest do
    use ExUnit.Case, async: true

    test "always pass" do
        assert true
    end
end
```

using behind the scene

```elixir
# this module...
defmodule Example do
    use Feature, option: :value
end

# compiles to...
defmodule Example do
    require Feature
    Feature.__using__( option: :value )
end
```


## Module nesting

```elixir
# defining 2 modules: Foo and Foo.Bar
defmodule Foo do
    defmodule Bar do
        # ...
    end
end

# is equivalent to
defmodule Elixir.Foo do
    defmodule Elixir.Foo.Bar do
        # ...
    end

    alias Elixir.Foo.Bar, as: Bar
end
```

## Multi alias / immport / require / use

```elixir
# alias MyApp.Foo, MyApp.Bar and MyApp.Baz at once
> alias MyApp.{Foo, Bar, Baz}

```


# Module attributes

Module attributes serve 3 purposes:

1. annotate the module with info to be used by user or vm
2. work as constant
3. work as temporary module storage to be used during compilation


## As annotations

Some reserved attributes:

* `@moduledoc` doc of current module
* `doc` doc for functions or macros that follows the attribute
* `@vsn` version of module
* `@behaviour` (british spelling) to specify OTP and user-defined behaviour
* `@before_compile` hook that will be invoked before compilation

```elixir
# explicitly set the version attribute of module
defmodule MyServer do
    @vsn 2
end

# fyi @vsn is used by code reloading mecanism, if not provided is set to md5 of module code


# add some documentation
defmodule Math do
    @moduledoc"""
        Provides math-related functions.

        ## Examples

        iex> Math.sum(1, 2)
        3

    """

    @doc """
        Calculates the sum of two numbers.
    """
    def sum(a, b), do: a + b
end    
```


## As constants

```elixir
defmodule MyServer do
    @initial_state %{ host: "127.0.0.1", port: 3456 }
    IO.inspect @initial_state
end

# undefined attribute will raise warning
> defmodule MyServer do
>     @unknown
> end
warning: ... undefined attr... explicitly set before access


# attributes can also be read inside functions
defmodule MyServer do

    @my_data 14
    def first_data, do: @my_data

    @my_data 13
    def second_data, do: @my_data
end    

> MyServer.first_data
14
> MyServer.second_data
13
> MyServer.first_data
14
```


## As temporary storage

`Plug` is a common foundation for building web libraries and framework.
`Plug` allows developers to define their own plugs which can be run in a web server.

```elixir
defmodule MyPlug do

    use Plug.Builder


    # use plug/1 macro to connect function set_header
    # under the hood function is stored in @plug attribute
    plug :set_header
    plug :send_ok

    def set_header(conn, _opts) do
        put_resp_header(conn, "x-header", "set")
    end

    def sen_ok(conn, _opts) do
        send(conn, 200, "ok")
    end
end

IO.puts "Running MyPlug with Cowboy on http://localhost:4000"
Plug.Adapters.Cowboy.http MyPlug, []
```

Tags in `ExUnit` are used to annotate tests...

```elixir
defmodule MyTest do

    use ExUnit.Case

    @tag :external
    test "contact external service" do
        # ...
    end
end
```

## Conclusion

Attributes are fundamental, they will really shine once combine with macros capability to do meta-programming.


# Structs

Remember about maps:

```elixir
> map = %{a: 1, b: 2}
%{a: 1, b: 2}

> map[:a]
1

> %{map | a :3}
%{a: 3, b: 2}
```

... `structs` are extensions built on top of maps that provide compile-time checks and default values.


```elixir
defmodule User do

    defstruct name: "John", age: 27  # defining fields and default value
end

> %User{}
%User{age: 27, name: "John"}

> %User{name: "Meg"}
%User{age: 27, name: "Meg"}

# compile-time guarantee: only the field defined are allowed to exist
> %User{oops: :field}
... KeyError ...
```

## Accessing and updating structs

```elixir
> john = %User{}
> john.name
"John"

> meg = %{john | name: "Meg"}

# pattern matching
> %User{name: name} = john
> name
"John"
```


## Structs are bare maps underneath

``` elixir
> is_map(john)
true

> john.__struct__
User

# however none of the maps protocols are available for structs
> john[:name]
... UndefinedFunctionError ...

> Enum.each john, fn({field, value}) -> IO.puts(value) end
... Protocol.UndefinedError ...

# structs work with the functions from the Map module
> kurt = Map.put(%User{}, :name, "Kurt")
%User{age: 27, name: "Kurt"}

> Map.merge(kurt, %User{name: "Yakashi"})
%User{age: 27, name: "Takashi"}

> Map.keys(john)
[:__struct__, :age, :name]
```

## Default values and required keys

```elixir
# nil is assumed when no default is provided
> defmodule Product do
>     defstruct [:name]
> end

> %Product{}
%Product{name: nil}


# enforce keys definition
> defmodule Car do
>
>     @enforce_keys [:make]
>     defstruct [:model, :make]
> end

> %Car{}
... ArgumentError the following keys must also be given ... Car: [:make]    
```


# Protocols

Protocols are mechanism to achieve polymorphism. Let's implement a generic `size` protocols...


```elixir
defprotocol Size do

    @doc "Calculate the size (and not the length!) of a data structure"

    def size(data)
    end

    defimpl Size, for: BitString do
        def size(string), do: byte_size(string)
    end

    defimpl Size, for: Map do
        def size(map), do: map_size(map)
    end

    defimpl Size, for: Tuple do
        def size(tuple), do: tuple_size(tuple)
    end

end

> Size.size("foo")
3

# passing data that does not implement the protocol raises error
> Size.size([1,2,3])
... Protocol.UndefinedError ...
>
```

It is possible to implement protocols for all Elixir data types:
`Atom`, `BitString`, `Float`, `Function`, `Integer`, `List`, `Map`, `PID`, `Port`, `Reference`, `Tuple`


## Protocols and structs

The power of Elixir extensibility comes when protocols and structs are used together.

Structs do not share protocol with maps...

```elixir
defimpl Size, for: Mapset do
    def size(set), do: Mapset.size(set)
end
```

Structs can be used to defined new data types, they ca implement all relevants protocols...

```elixir
defmodule User do
    defstruct [:name, :age]
end

...
defimpl Size, for: User do
    def size(_user), do: 2
end
```


## Implementing Any

Implementing protocols for many data types can be tedious.
Elixir provides 2 options:

1. derive the protocol implementation for our types
2. automatically implement the protocol for all types.

either case the protocol has to be implemented for `Any`.


```elixir
# APPROACH 1 - deriving
defimpl Size, for: Any do
    def size(_), fo: 0    # ahem this not reasonable though!
end

# we need to tell our struct to explicitly derive the Size protocol
defmodule OtherUser do
    @derive [Size]
    defstruct [:name, :age]
end


# APPROACH 2 - fallback to any
# fallback to Any only when an implementation cannot be found
defprotocol Size do

    @fallback_to_any true
    def size(data)
end


defimpl Size, for: Any do
    def size(_) do: 0
end
```


## Built-in protocols

```elixir
# the Enumerable protocol
> Enum.map [1,2,3], fn(x) -> x * 2 end
> Enum.reduce 1..3, 0, fn(x, acc) -> x + acc end

# the String.Chars protocol, exposed via to_string function
> to_string :hello
> "age: #{25}"   # calls to_string behind the scene

# tuple do not implement the String.Chars protocol
> tuple = {1,2,3}
> "tuple: #{tuple}"
... Protocol.UndefinedError ...

# solution is the Inspect protocol
# the Inspect protocol transforms any data type to readable textual
> "tuple: #{inspect tuple}"
```


# Comprehensions

```elixir
> for n <- [1,2,3,4,5], do: n * n
```

A comprehension is made of 3 parts: generators, filters and collectables.


## Generators and filters

```elixir
# in the following...
> for n <- [1,2,3,4,5], do: n * n

# the generator is
n <- [1,2,3,4,5]


# generator can support pattern matching...
> values = [good: 1, good: 2, bad: 3, good: 4]
> for {:good, n} <- values, do: n * n
[1, 4, 16]


# filter ... as alternative to pattern matching
> multiple_of_3? = fn(n) -> rem(n, 3) == 0 end
> for n <- 0..5, multiple_of_3?.(n), do: n* n
[0, 9]


# multiple generators...
> dirs = ['/home/mikey/', '/home/james/']
> for dir <- dirs,
>     file <- File.ls!(dir),
>     path = Path.join(dir, file),
>     File.regular?(path) do
>   File.stat!(path).size


# multiple generator for cartesian product
> for i <- [:a, :b, :c],
>     j <- [1, 2],
>     do: {i, j}


# Pythagorean triples:
# a more advanced example of multi gen/filter
defmodule Triple do

    def pythagorean(n) when n > 0 do
        for a <- 1..n,
            b <- 1..n,
            c <- 1..n,
            a + b + c <= n,
            a * a + b * b == c * c,
            do: {a, b, c}
    end
end

# Pythagorean triples:
# an optimized version
defmodule Triple do

    def pythagorean(n) when n > 0 do
        for a <- 1 .. n - 2,
            b <- a + 1 .. n - 1,
            c <- b + 1 .. n,
            a + b >= c,
            a * a + b * b == c * c,
            do: {a, b, c}
    end
end


# Bitstring generators
> pixels = << 213, 45, 132, 64, 76, 32, 76, 0, 0, 234, 32, 15 >>
> for << r::8, g::8, b::8 <- pixels >>, do: {r, g, b}
[{213, 45, 132}, {64, 76, 32}, {76, 0, 0}, {234, 32, 15}]
```


## The `:into` option

Pass generator into different data structures than list.

```elixir
# :into accepts any structure that implements the Collectable protocol
> for << c <- "hello world" >>, c != ?/s, into: "", do: <<c>>
"helloworld"


# update value in map, without touching the key
> for {key, val} <- %{"a" => 1, "b" -> 2}, into: %{}, do: {key, val * val}
%{"a" => 1, "b" => 4}


# using streams
# the following prints everything from stdin in upper case
# ctrl-c to exit
> stream = IO.stream(:stdio, :line)
> for line <- stream, into: stream stream do
>     String.upcase(line) <> "\n"
> end
```


# Sigils

## Regular expressions
Sigils are one of the mecanism for working with textual representations.

```elixir
# the most common sigil is ~r (regex)
> regex = ~r/foo|bar/
> "foo" =~ regex
true
> "bat" =~ regex
false

# i modifier makes case insensitive
> "HELLO" =~ ~r/hello/
false
> "HELLO" =~ ~r/hello/i
true

# sigil support 8 different delimiters
~r/hello/
~r|hello|
~r"hello"
~r'hello'
~r(hello)
~r[hello]
~r{hello}
~r<hello>

# depending on what you want to escape, yo can choose the appropriate delimiter.
```

## Strings, char lists, and word lists sigils

Beside regular expressions, there are 3 other sigils:


```elixir

# String
# ~s is used to generate string
# ~s is useful when a string contains double quotes
> ~s(this is a string with "double" quotes, not 'single' ones)
this is a string with \"double\" quotes, not 'single' ones"


# Char lists
# ~c is usefull for generating char list that contains single quotes
> ~c(this is a char list containing 'single quotes')
'this a char list containing \'signle quotes\''


# Word lists
# ~w is used to generate lists of words
> ~w(foo bar bat)
["foo", "bar", "bat"]
# accept modifiers c (char), s (string) and a (atom)
> ~w(foo bar bat)a
[:foo, :bar, :bat]


```


























#
