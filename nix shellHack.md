mix and mash nix and bash: a trick to fix the stack
===

_(if you read the title aloud, did you say "stash"?)_

`nix-shell` is immensely useful for building reproducible, [rememberable](./tools%20for%20remembering%20code%20projects.md) project environments. Not only does it allow you to precisely define the project dependencies and supporting tools, it also lets you customize the project-specific shell (bash) environment via the `shellHook` attribute.

Program dependencies compose easily in nix; they are first-class attributes. If there's a project `seedProject` that declares `jq` and `python39` as dependencies:

```nix
# `nix-shell seedProject.nix` prints "hello from seed" after entering the shell
with import <nixpkgs> {};
mkShell {
  buildInputs = [ jq python39 ];
  shellHook = "echo hello from seed";
}
```

and a `treeProject` that imports from `seedProject`, that declares an overlapping set of dependencies:

```nix
# `nix-shell treeProject.nix` prints "hello from tree" after entering the shell
with import <nixpkgs> {};
let seedProject = (import ./seedProject.nix);
in mkShell {
  buildInputs = [ jq jsonnet ] ++ seedProject.buildInputs;
  shellHook = ''
  some-function() {
      echo "hello from tree"
  }

  some-function
  '';
}
```

the redundant `jq` input does not trip the `nix-shell` into loading the `jq` package twice.

But what about `shellHook`, which is a string? After the `shellHook` grows and sprawls, we are tempted to refactor the shell code to a separate bash script and `source` it, doing something like

```nix
shellHook = ''
  some-function() {
      echo "hello some function"
  }

  . scripts/more-functions.sh  # source the code that we refactored out
'';
```

Moving the code to `scripts/more-functions.sh` also simplifies testing and debugging because we can re-source the file directly from the shell environment, instead of having to reload the shell or set up another tool like [lorri](https://github.com/target/lorri).

This works well while everything in `more-functions.sh` is _pure_ bash, and we have a high confidence of it running the same way in, say, bash 3+ (for the wellbeing of vanilla macs).

But restricting ourselves to pure bash is like dressing up using only bamboo leaves. Sometimes it's easier with banana leaves: more flexibility, greater surface area, smoothie interaction. When I write "bash" I always write as if I can search files with `find`, that I can snatch files with `curl`, that I can 'splain files with `file`. This is only _usually true_. Once you catch a failure from shell commands in a docker container, then you remember: oh, right, these commands _don't_ come standard issue! We need to add them into `buildInputs`.

We now have a new itch to scratch. We missed some dependencies in the `shellHook` script and fixed them by manually adding `file` into the `buildInputs` for the environment. If we are importing this expression in 10 different projects, we need to update the nix file for all 10 projects and add `file` to _their_ `buildInputs`. By doing this, we've lost the easy composability that nix provided in the first place!

How can we make the bash script declare the dependencies it wants so the environments using the script pull them automatically?

# method uno: separate nix file for every shellHook

the most straightforward, and perhaps _cleanest_ way to do this is to encapsulate every set of shell scripts into their own `.nix` files, which each declare their own `buildInputs` and `shellHook`.

```nix
# a shared expression
...
buildInputs = [
  file  # provides the `file` program
  unixtools.column  # provides the `column` program
];

shellHook = ''
  say-hello() {
      echo "I come with column, you can call 'em!"
  }
  
  tsv-to-columns() {
      file_path=$1
      cat $1 | column -t -s $'\t'
  }
'';
```

Then, the files that want to include this set of functions and its dependencies can add them as needed:

```nix
buildInputs = [
  brotli lazygit
] ++ shared.buildInputs;

shellHook = shared.shellHook + ''
  welcome() {
      echo "welcome to the bigger project that uses other-functions"
  }
  
  welcome
  say-hello
'';

```

this works as expected, but puts all the bash code back into the nix file. Since bash code itself is not first-class in nix, from the perspective of bash scripting, this is a drawback.

¿Puedo que hacer los dos? Can I do both?

![](./img/2022/02/664g3n.jpg)

# method por qué no los dos: shellHack

We can make a bash script in a way that lets us declare its program dependencies, and also make the script importable into nix. We will achieve this using a ~~nasty~~ nifty header, and a ~~sneaky~~ spiffy footer. From what I gather, this is known in Spanish as "un hack feo".

## el nix shellHack (feo)

```bash
/*/bin/true --BEGIN-polyglot-hack-- 2>/dev/null
# */ rec { buildInputs = (with import <nixpkgs>{}; [ unixtools.column ]); shellHook = ". ${__curPos.file}"; ignore = ''

# ...
# actual bash logic
# ...

# --END-polyglot-hack-- */ ''; }

```

in the first line for bash, we're not using a shebang, but invoke a (most likely existing under `/usr/bin/`) program that does nothing, does no interesting argument parsing, but exits without error, just so we can

1. start a nix multi-line comment beginning with `/* ...`
2. put an obvious header to remind ourselves later that the weird looking stuff serves a purpose

```nix
/*/bin/true --BEGIN-polyglot-hack-- 2>/dev/null
# */ rec { buildInputs = (with import <nixpkgs>{}; [ unixtools.column ]); shellHook = ". ${__curPos.file}"; ignore = ''

```

For nix, we end the multi-line comment on the next line and start the actual nix code, while bash sees another comment. Here, we add the shell script's program dependencies, and end the line by starting a multi-line nix string with `''`, and put what _used to be_ our `shellHook` contents in there.

To ensure the script works as a `shellHook`, we must remember that `${something}` actually evaluates to a _nix_ variable. This means that for bash variables that use the same expression, we must escape them for nix and write `''${something}`. There are probably pitfalls, like if `${something}` is passed through some empty-vs-unset check, but _for the most part_, it works in bash as-is.

There are other gotchas. For example, if you want to output nix multi-line strings (`'' ... ''`) within your shell code, you will have to avoid writing `''` directly. I use a script that contains a nix skeleton generator template function, which outputs nix multi-line strings, so for the template string, I set `II="'""'"` and use `$II` in place of `''` inside the heredoc:

```bash
    II="'""'"
    cat <<EOF
{ pkgs ? import <nixpkgs> {} }:
pkgs.mkShell {
  shellHook = $II
    echo "hello bash"
  $II;
}
EOF
```

Finally, at the end of our nix-bash script, we need to close the multi-line nix string with a bash non-op (we'll just use another bash comment), finish the `shellHook` attribute definition, and close the `nix` set:

```bash
# --END-polyglot-hack-- ''; /* <-- end of nix shellHook */  }
```

## using the bix expression

![](./img/2022/02/something-like-a-bix.svg)

Now we have our bilingual script, say we called it `bilingual.nix.sh`, we can `source bilingual.nix.sh` (or `. bilingual.nix.sh`) from a bash shell, or execute it directly with `bash bilingual.nix.sh`.

To use this in a nix expression, we can do it in 2 ways:

### via `nativeBuildInputs`

```nix
let
  bilingualScriptPath = /path/to/bilingual.nix.sh;
  bilingualScript = import bilingualScriptPath;
pkgs.mkShell {
  buildInputs = [ ... ] ++ bilingualScript.buildInputs;  # include dependencies
  nativeBuildInputs = [ ... bilingualScriptPath];
}
```

### via `shellHook`

```nix
let
  bilingualScriptPath = /path/to/bilingual.nix.sh;
  bilingualScript = import bilingualScriptPath;
pkgs.mkShell {
  buildInputs = [ ... ] ++ bilingualScript.buildInputs;  # include deps
  shellHook = bilingualScript.shellHook + ''
    # or include it directly
    ${bilingualScript.shellHook}
  '';
}
```

# extension

Since we can load remote sources in nix expressions using `builtins.fetchurl`, this gives us a nice way to load, try, or mix-and-match version-pinned and hash-verified sources. Say you wanted to try the `echo-shortcuts` function I use to tell me what aliases and functions are declared in a nix-shell file. The function is defined in [this bilingual script](https://github.com/whacked/setup/blob/a6327c55e7806e90b85555fd386e478e06fdeeda/bash/nix_shortcuts.sh) hosted on github.

To include this in a nix expression, we can use:

```nix
# shell.nix somewhere for one-off testing

with import <nixpkgs> {};

let
  remoteScript = (builtins.fetchurl {
    url = "https://raw.githubusercontent.com/whacked/setup/a6327c55e7806e90b85555fd386e478e06fdeeda/bash/nix_shortcuts.sh";
    sha256 = "0780cnzwlb9kvf7llqa32h8w92r5lk04rsnx9p3c7ihbqzrqgd2h";
  });
  scriptSet = import remoteScript;
in mkShell {
  buildInputs = [ visidata ] ++ scriptSet.buildInputs;

  nativeBuildInputs = [
    ## another way to include the script, but we'll use shellHook below
    # remoteScript
  ];

  shellHook = ''
    show-greeting() {  # a friendly function!
        echo "me gusta nix!"
    }
    
    # include the script
    ${scriptSet.shellHook}

    # this is provided by the remote script
    echo-shortcuts
  '';
}
```

Now, when we run `nix-shell`, it will call `echo-shortcuts`, which uses the `column` utility as declared in its `buildInputs` list to format the output.

[![asciicast](https://asciinema.org/a/SlHr0jRNe75b20rnhQc3NGiJu.svg)](https://asciinema.org/a/SlHr0jRNe75b20rnhQc3NGiJu)

If you were to extend the `shellHook`, or include other sources, and host that version online, someone else could then use that version, and include it in their project. Just don't forget the `buildInputs`!

# disclaimer

there is almost certainly a more better, more bueno, more bello way to do this. There are nice techniques covered in the [NixOS Wiki page on Shell Scripts](https://nixos.wiki/wiki/Shell_Scripts), but not bilingual feoHacks.
