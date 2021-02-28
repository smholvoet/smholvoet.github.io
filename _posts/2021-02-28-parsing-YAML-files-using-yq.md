---
published: true
title: "Parsing YAML files using yq"
excerpt: "Parsing YAML files using yq"
date: 2021-02-28T00:00:00-04:00
show_date: true
tags:
  - YAML
  - parser
  - yq
---

Looking for a lightweight tool to parse YAML files? Look no further...  
I recently bumped into **`yq`**, which is command-line YAML processor that is able to traverse YAML, sort by keys, update values, export to JSON, and much more...

- <i class="fab fa-github"></i> **GitHub repo:** [mikefarah/yq](https://github.com/mikefarah/yq)
- 📚 **Docs:** [link](https://mikefarah.gitbook.io/yq/)
- 🛠️ **Language:** [Go](https://github.com/golang/go)

```console
Usage:
  yq [flags]
  yq [command]

Available Commands:
  eval             Apply expression to each document in each yaml file given in sequence
  eval-all         Loads _all_ yaml documents of _all_ yaml files and runs expression once
  help             Help about any command
  shell-completion Generate completion script

Flags:
  -C, --colors        force print with colors
  -e, --exit-status   set exit status if there are no matches or null or false is returned
  -h, --help          help for yq
  -I, --indent int    sets indent level for output (default 2)
  -i, --inplace       update the yaml file inplace of first yaml file given.
  -M, --no-colors     force print with no colors
  -N, --no-doc        Don't print document separators (---)
  -n, --null-input    Don't read input, simply evaluate the expression given. Useful for creating yaml docs from scratch.
  -P, --prettyPrint    pretty print, shorthand for '... style = ""' 
  -j, --tojson        output as json. Set indent to 0 to print json in one line.
  -v, --verbose       verbose mode
  -V, --version       Print version information and quit

Use "yq [command] --help" for more information about a command.
```

## Installation

### Windows

I suggest using a package manager like [Chocolatey](https://chocolatey.org/install):

```console
choco install yq
```

### Linux / macOS

Use [Homebrew](https://brew.sh/) to install on Linux or macOS:

```bash
brew install yq
```

## Usage

Sample `fruits.yml` file:

```yaml
fruits:
  apple: red
  pear: green
  banana: yellow
```

### Print out file

The `eval` or `e` command, followed by the path to a YAML file will print out its entire contents:

```bash
yq e fruits.yml
```

```yaml
fruits:
  apple: red
  pear: green
  banana: yellow
```

### Print out a specific value

Target a specific key:

```bash
yq e '.fruits.apple' fruits.yml
```

```yaml
red
```

### Update file

You can also manipulate files directly:

```bash
yq e '.fruits.apple = "orange"' -i fruits.yml
```

⚠️ If you're running `yq` in PowerShell and run into a parsing error, you might need to escape the quotation marks with a backslash: `yq e '.fruits.apple = \"orange"' -i fruits.yml`
{: .notice--warning}

Will update `fruits.yml`:

```yaml
fruits:
  apple: orange 👈
  pear: green
  banana: yellow
```

### Using variables

You can use [variable operators](https://mikefarah.gitbook.io/yq/operators/env-variable-operators) and use them to update a key:

```bash
color="green" yq e '.fruits.banana = env(color)' -i fruits.yml
```

```yaml
fruits:
  apple: red
  pear: green
  banana: green 👈
```

Taking things further, you could save the output of a command and assign it to the environment variable:

```bash
colorArray=("purple" "blue" "cyan")
color=${colorArray[1]} yq e '.fruits.pear = env(color)' -i fruits.yml
```

```yaml
fruits:
  apple: red
  pear: blue 👈
  banana: yellow
```
