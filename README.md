# success typing

* Not trying to prove correctness.
* Won't force you to handle all possible values
* Looking for cases that can't possibly work
* Good for catching
  * passing arguments in the wrong order
  * typos, forgetting how many elements in a tuple
  * changing a function and forgetting to change how you call it
* No type checking at all for sending and receiving messages
  * processes are objects with extreme late-binding
  * places where you receive data are the most helpful places to hint at types
  * wrap `send`s in a function that can hint at types

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

# example: changing a function signature

Change `Sqlitex.Server.prepare_impl/2` with `when is_integer(sql)`.

> `The call 'Elixir.Sqlitex.Server.StatementCache':prepare(stmt_cache@1::any(),sql@1::integer()) will never return since it differs in the 2nd argument from the success typing arguments: (#{'__struct__':='Elixir.Sqlitex.Server.StatementCache', 'cached_stmts':=map(), _=>_},binary())`

This error actually implies that we are calling `Sqlitex.Server.StatementCache.prepare` incorrectly because it now knows that `sql` will be an integer at that point.
It would be great if `dialyzer` could tell us that we are calling `prepare_impl/2` incorrectly.
If we look up to where it gets called in a `handle_call` the reason it doesn't know is that it has no idea what type the `sql` argument has.

Add a `when is_binary(sql)`

> `lib/sqlitex/server.ex:176: Function prepare_impl/2 has no local return`

> `lib/sqlitex/server.ex:176: Guard test is_integer(sql@1::binary()) can never succeed`

Now it can tell us that there is a type mismatch because it knows that `sql` is a `binary()` and then it knows we expect an `integer()`.

# example: providing an incorrect type spec

Change `Sqlitex.Query.query_rows!` so the typespec says that it returns `[[]]`

> `Invalid type specification for function 'Elixir.Sqlitex.Query':'query_rows!'/2. The success typing is ({'connection',reference(),binary()},binary() | string()) -> #{}`

`dialyzer` is telling us that it thinks the function will return a map which doesn't match our typespec.

# exmaple: breaking a behaviour contract

Change `Sqlitex.Server.init/1` to return an atom.

> `The inferred return type of init/1 ('sweet_bruh') has nothing in common with 'ignore' | {'ok',_} | {'stop',_} | {'ok',_,'hibernate' | 'infinity' | non_neg_integer()}, which is the expected return type for the callback of the 'Elixir.GenServer' behaviour`

We can see that `GenServer` expects the `init/1` function to return something like `{:ok, _}` or `{:stop, _}`, but since we return something different we get this warning.

# further reading

* http://learnyousomeerlang.com/dialyzer
* https://hexdocs.pm/elixir/typespecs.html#types-and-their-syntax
* http://erlang.org/doc/man/dialyzer.html
