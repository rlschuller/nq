# nq

Disclaimer: this project is experimental.

Natural queries, or nq for short, is a line filter with natural language as the
interface. It works by prompting a large language model via an openai / ollama
API, which means that you can use it with self-hosted LLMs. There are no
dependencies besides Python (and optionally bubblewrap, see Security), since
the queries are POST requests implemented with builtin libraries.

## Installation

Since there are no pip dependencies, you can install nq by adding

```
export PATH="$HOME/.local/bin/:$PATH"
```

to your bashrc (if PATH does not contain "$HOME/.local/bin" already) and
running

```
git clone 'https://github.com/rlschuller/nq'
cp -r nq/skel/.config/nq ~/.config
mkdir -p ~/.local/bin
cp nq/nq ~/.local/bin
```

nq communicates with the model via an openai REST API, and you can configure
addresses and authorizations in ~/.config/nq/api.json.  Users of UNIX systems
not based on Linux (such as BSD or macOS) must disable the sandboxing feature
by setting sandbox to false in ~/.config/nq/config.json.

## Flags

```
usage: nq [-h] [-m {rick,minicpm5,remote,deepseek,gemma}] [-t PATH] [-v] [-0]
          [-j] [-n N] [-p] [-s] [-d] [-nl] [-f FILE [FILE ...]]
          [prompt ...]

make queries with natural language

options:
  -h, --help            show this help message and exit

general:
  -m, --model {rick,minicpm5,remote,deepseek,gemma}
  -t, --template PATH
  prompt

language mode (default):
  -v, --verbose         instead of echoing the response, outputs a dictionary
                        with more information
  -0, --null            use NULL as the entry splitter instead of newline
  -j, --json            parse each line as a JSON before forwarding to the
                        model
  -n, --max-threads N   maximum number of simultaneous connections to the API

programmatic mode:
  -p, --prog            enter programmatic mode
  -s, --sample          use the first line in the prompt
  -d, --debug           print generated code to stderr (set always_debug to
                        true in ~/.config/nq/config.json to make it default)
  -nl, --no-log         disable log, even if log_path is not null in
                        ~/.config/nq/config.json
  -f, --jsonl-files FILE [FILE ...]
                        given JSONL files, merge them line-by-line into arrays

environment variables:
  NQ_CONFIG_FOLDER      config directory (default: ~/.config/nq)
```

## Language mode

By default, nq uses the language_template.md to send each line from stdin to be
processed by the language model. There are no real minimum requirements for
this mode, but in my anecdotal experience models adept in tool use are better
at following precise output formats. This can be useful for subsequent
statistical analyses.

### Examples

```
$ cat books
The Linux Programming Interface
The Human Condition
1984
Ensaio Sobre a Cegueira
$ cat books | nq translate to portuguese
"A Interface de Programação do Linux"
"A Condição Humana"
"1984"
"Ensaio Sobre a Cegueira"
```

```
$ cat books | nq classify into fiction / nonfiction
"nonfiction"
"nonfiction"
"fiction"
"fiction"
```
## Programmatic mode

With the -p / --prog flag, nq prompts an LLM to produce a Python script that
takes input from stdin and prints the answers to stdout. Drawing from classical
commands such as awk and jq, nq assumes that each line is either a raw value or
a JSON formatted entry, e.g.

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
{ "year" : 1968, "title": "Planet of the Apes" }
```

### Examples

In the examples, we use the -d / --debug flag, to make nq print the generated
code to stderr. You can set always_debug to true in ~/.config/nq/config.json to
show the code by default.

#### Default input format

If the instruction does not name a field or use a generic name, the model
assumes raw line-separated values

```
$ seq 20 | nq -dp select fibonacci numbers
```

**stdout**

```
1
2
3
5
8
13
```

**stderr**

```python
#!/usr/bin/env python3
import sys

def is_perfect_square(n):
    if n < 0:
        return False
    sqrt_n = int(n**0.5)
    return sqrt_n * sqrt_n == n

def is_fibonacci(n):
    # A number is Fibonacci if and only if (5*n^2 + 4) or (5*n^2 - 4) is a perfect square
    if n < 0:
        return False
    return is_perfect_square(5 * n**2 + 4) or is_perfect_square(5 * n**2 - 4)

for line in sys.stdin:
    try:
        val = int(line.strip())
        if is_fibonacci(val):
            print(val)
    except ValueError:
        continue
```

#### JSONL input format

For named entries it interprets the input as JSONL (i.e., one valid JSON per
line)

the contents of movies.jsonl:

```
{ "year" : 1968, "title": "2001: A Space Odyssey" }
{ "year" : 1982, "title": "Blade Runner" }
{ "year" : 1968, "title": "Planet of the Apes" }
```

```
$ cat movies.json | nq -dp get title
```

**stdout**

```
2001: A Space Odyssey
Blade Runner
Planet of the Apes
```

**stderr**

```python
#!/usr/bin/env python3
import sys
import json

for line in sys.stdin:
    try:
        data = json.loads(line)
        if "title" in data:
            print(data["title"])
    except json.JSONDecodeError:
        continue
```

Note: to avoid ambiguities, you can explicitly name a field, e.g. 'mean of the
number field'.

### Minimum requirements

With Gemma4 12B (4 bits) the tool works most of the time with English prompts,
bug gets lost in Portuguese. Since the runtime is reasonably fast (3-4s in an
RTX 3060, with ollama), the occasional syntax error might still be acceptable.
This is the lowest tier of general purpose language model that I would use, and
if you have more VRAM I'd recommend a better one to iron out the use of Python
syntax.

Since bigger models such as Gemma4 31B (fp16) are producing reliable code for
unambiguous nq prompts, creating a fine-tuning dataset to specialize smaller
models and either lower the minimum requirements or improve the performance of
Gemma4 12B (4 bits) might be possible. The relatively narrow application scope
seems conducive to such endeavour.

### Security

By default, nq launches the Python script generated by the model in an isolated
environment, using an unprivileged low-level sandboxing tool named
[bubblewrap](https://github.com/containers/bubblewrap). The tool is a CLI
wrapper of a Linux kernel feature called namespaces, and is available on most
distributions.

Other UNIX systems (such as BSDs and macOS) do not support namespaces nor
bubblewrap, hence users of these operating systems must disable rootless
isolation by setting sandbox to false in ~/.config/nq/config.json. While jails
could be a viable alternative, I'm not sure it's well suited for dynamic
sandboxing due to it's privilege escalation requirements.

Note that even without the guardrails of bubblewrap the attacking surface is
limited, as the stdin data only interacts with the Python program produced by
the LLM, and thus the code itself cannot be directly altered by it. With that
said, always be cautious when processing untrusted data.

