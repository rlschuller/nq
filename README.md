# nq

Natural queries, or nq for short, is a line filter with natural language as the
programming interface. Given a program, e.g. 'sum the odd numbers', it prompts
an LLM to produce a self-contained Python script that takes data from stdin and
prints the answers to stdout. Drawing from classical commands such as awk and
jq, nq assumes that each line is either a raw value or a JSON formatted entry,
e.g.

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

nq has 0 dependencies besides Python (and optionally bubblewrap, see Security),
since the model queries are POST requests implemented with builtin libraries.

## Examples

![](https://github.com/user-attachments/assets/fe5461c9-3df0-476b-ae49-9220bb9d9f4a)

![](https://github.com/user-attachments/assets/18cf3c85-d1d5-4169-a3f2-f6761fd86a1f)
