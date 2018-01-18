# success typing

* Not trying to prove correctness.
* Won't force you to handle all possible values
* Looking for cases that can't possibly work
* Good for catching
  * passing arguments in the wrong order
  * typos, forgetting how many elements in a tuple
  * changing a function and forgetting to change how you call it

# what does it look like?

It specifies the input types and the return type of a function.

See https://hexdocs.pm/elixir/1.6.0/File.html#open/2 as an example

# how to use it

Add `{:dialyxir, "~> 0.5", only: [:dev], runtime: false}` to your mix.exs.
Run `mix do deps.get, dialyzer`

> Note: first run took 4min 19.6sec, but subsequent runs are pretty fast

# example: passing incorrect arguments

Switch `Sqlitex.open/1` arguments

> `Function with_db/2 has no local return` this is a common error and what it really means is that there was a problem resolving the types somewhere down-stream from here.
> `esqlite3:open({'path',_}) will never return since the success typing is (string()) -> {'error',_} | {'ok',{'connection',reference(),_}} and the contract is (FileName) -> {'ok',connection()} | {'error',_} when FileName :: string()`

The best way to look at this is to read it as "you tried to call it like this, but you need to call it like this".
So we can't pass `{:path, _}` to a function that is expecting `string()`.

# further reading

* http://learnyousomeerlang.com/dialyzer
* https://hexdocs.pm/elixir/typespecs.html#types-and-their-syntax
* http://erlang.org/doc/man/dialyzer.html
