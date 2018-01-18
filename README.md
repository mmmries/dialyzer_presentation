# success typing

* Not trying to prove correctness.
* Won't force you to handle all possible values
* Looking for cases that can't possibly work

# how to use it

Add `{:dialyxir, "~> 0.5", only: [:dev], runtime: false}` to your mix.exs.
Run `mix do deps.get, compile, dialyzer`

> Note: first run took 4min 19.6sec, but subsequent runs are pretty fast

# further reading

* http://learnyousomeerlang.com/dialyzer
* https://hexdocs.pm/elixir/typespecs.html#types-and-their-syntax
* http://erlang.org/doc/man/dialyzer.html
