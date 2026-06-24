# nq

`nq` is a line filter that uses natural language as the programming interface,
designed as an alternative to `jq` / `awk`. Given a prompt, e.g. 'sum the odd
numbers', it prompts an LLM to produce a self-contained Python script that
takes data from stdin and prints the answers to stdout. It assumes that each
line is either a raw entry or a JSON formatted entry, e.g.

one name per line

```
Grace Hopper
Edsger Dijkstra
Alan Turing
...
```

or

```
{ "year" : 1968, "title": "2001: A Space Odyssey" }
{ "year" : 1982, "title": "Blade Runner" }
...
```


It has 0 dependencies besides Python and bubblewrap (see Security), since it
queries LLM's via urllib POST requests.


## Examples

