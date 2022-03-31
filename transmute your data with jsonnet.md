# data scientists, data alchemists

If we pause and think about it, "data" is an extremely general term, like "feelings" and "information". Everyone has feelings! Everything contains information! When someone says, "I feel like there's information in there," it feels like you gained no information. Maybe information gained feelings?

Since data is so nonspecific, it's somewhat unfortunate that "data scientist" nowadays implies some kind of statisticker or machine learner or number muncher or spreadsheet pivoter. What about _configurations_? Configs are data! And there's also science in configuration. It's about time configs gained some feelings.

This is certainly not a new viewpoint, because some folks at Google infected with an affection for effective data came up with the open-source [Jsonnet](https://jsonnet.org/) language. I must say, whatever science there was in working with Json files, Jsonnet has turned it into art. If "data scientists" transform data as structured tables, then "data poets" transform written data into structured poetry. Wait, that doesn't sound right. We're not discussing iambic parameters; we're transmuting JSON. So let's say "data alchemist" instead.

# why Jsonnet, for you

- 8+ years old (since 2014) and still actively updated
- cross-platform, single binary
- fast (enough) to run regenerate interactively
- official syntax formatter built-in
- reasonably informative error messages
- straightforward file importing
- trailing commas makes reordering lists simple and clean
- it does nothing aside of JSON generation
- allows refactoring of JSON
- allows data tests to be included in-file
- decent language bindings

# why Jsonnet, for your team

If you ever edit JSON by hand and are one of the [lucky 10000](https://xkcd.com/1053/) who hasn't used Jsonnet, here are 3 reasons to consider adding it to your (and your team's) toolbelt:

- **easy to adopt**: To get started, [get a compiled binary](https://github.com/google/go-jsonnet/releases) for your platform and you're ready to go.
- **easy to dedopt**: If you decide to get off the Jsonnet train, simply save the rendered output and you're back to ~~square one~~ plain-old-JSON-objects.
- **easy to ramp up**: Since Jsonnet is a strict superset of JSON, any valid JSON is valid Jsonnet. If you don't want to learn any new syntax, you don't need to. Different people have different tastes for repetition, modularization, and abstraction. Gradual adoption allows people to adjust at their own pace. This is the same reason why I _don't_ prefer cue / dhall / TS / py / clj / edn / awk / wardever for sculpting data.

# what are the cons?

obviously I wish more people to use Jsonnet, because it's excellent, but there's **one not nice thing**: it has a bizarre [3-space default indentation](https://github.com/google/jsonnet/issues/547) for its generated JSON. In practice, this means I almost always pipe jsonnet output through a formatter (`jq -S` in my case), and in my experience, this arrangement is more powerful than using jsonnet alone, which usually lets me sidestep the indentation.

## conflict of interest?

The author of this article declares interest in working with declarative, composable data using a functional language, which conflicts with non-declarative, non-composable data using dysfunctional languages.

More Jsonnet transmuting means less JSON twiddling. Transmute happy, twiddle sad.

# learning

it's hard to do better than these existing guides:
- [Learn Jsonnet probably less than 10 minutes](https://learnxinyminutes.com/docs/jsonnet/) (syntax)
- [official (live interactive) tutorial](https://jsonnet.org/learning/tutorial.html) (syntax + standard library, basic usage and tricks)
- [Databricks Jsonnet Guide](https://github.com/databricks/jsonnet-style-guide) (recommended practices)

# working with Jsonnet ~~code~~ (J-sonnets? JSON-sonnets? JSON-nets?)

Since Jsonnet outputs to STDOUT (and because of its strange 3-character indentation), I almost always pipe `jsonnet` to [jq](https://stedolan.github.io/jq/). If the target format is YAML, I use [yq](https://github.com/mikefarah/yq) for conversion.

## nice editor plugins

- [for vscode](https://github.com/Sebbia/vscode-jsonnet-ng)
- [for vim](https://github.com/google/vim-jsonnet)

Jsonnet doesn't have a lot of syntax, so it doesn't take much to get productive. In my experience, by far the most important feature of the editor plugins is auto-applying `jsonnetfmt` on save; if it fails to re-format, I know my code has errors.

# transmutation testing and verification

Being a functional language with a single output, your options are to write assertions (bail on fail) or capture test results as outputs. For testing consistent ~~transformations~~ transmutation, this goes pretty far! A simple version is shown here:

```jsonnet
# test-demo.jsonnet

local unit = {
  runTest(expected, received, message=null):: (
    assert (expected == received)
           : (if message != null then
                message else
                '\nexpected %s\nreceived %s' % [expected, received]);
    true
  ),
};

local lib = {
  dissoc(object, mixedKeys)::
    local keys = (if std.type(mixedKeys) == 'array' then mixedKeys else [mixedKeys]);
    std.mergePatch(object, {
      [key]: null
      for key in keys
    }),
};

[
  unit.runTest(
    lib.dissoc(
      { a: 1, b: 2, c: 3 },
      ['a', 'b'],
    ),
    { c: 3 } + std.extVar('stdinObject')  # override for demo
  ),
]
```

then running this

```bash
$ jsonnet --ext-code stdinObject='{}' test-demo.jsonnet  # PASS
# > [
# >     true
# > ]
$ jsonnet --ext-code stdinObject='{c: 999}' test-demo.jsonnet  # FAIL
# > RUNTIME ERROR: 
# > expected {"c": 3}
# > received {"c": 999}
# > 	demo.jsonnet:(3:5)-(7:9)	function <anonymous>
# > 	demo.jsonnet:(21:3)-(27:4)	thunk <array_element>
# > 	During manifestation	
```

# adding guards for configs

For illustration, here's an example that checks if the `os` fields in `package.json` are among a set of allowed OSes, and tries to catch common mistakes

```jsonnet
# guard-demo.jsonnet

local helpers = {
  // turns { k: [v1, v2, ...] } into { v1: k, v2: k, ... }
  makeLookupTable(o):: {
    [value]: key
    for key in std.objectFields(o)
    for value in o[key]
  },
  suggestions: $.makeLookupTable({
    darwin: ['apple', 'mac', 'osx'],
    win32: ['windows', 'win', 'win64', 'win3.1'],
  }),
  suggestCorrection(candidate):: (
    local key = std.asciiLower(std.stripChars(candidate, ' '));
    if std.objectHas($.suggestions, key)
    then $.suggestions[key]
    else 'None!'
  ),
};

local package = (import 'package.json');
local constants = {
  allowedOses: ['darwin', 'win32', 'linux'],
};

[
  std.trace(
    'checking OS is allowed: %s' % [os],
    (
      assert (std.member(constants.allowedOses, os) == true)
             : 'OS "%s" is not allowed; suggestion: %s' % [
        os,
        helpers.suggestCorrection(os),
      ];
      true
    )
  )
  for os in package.os
]
```

with an input of

```json
{
    "os": [
        "darwin",
        "linux",
        "windows"
    ]
}
```

this gives an output of

```bash
$ jsonnet guard-demo.jsonnet
# > TRACE: guard-demo.jsonnet:28 checking OS is allowed: darwin
# > TRACE: guard-demo.jsonnet:28 checking OS is allowed: linux
# > RUNTIME ERROR: OS "windows" is not allowed; suggestion: win32
# > 	guard-demo.jsonnet:(31:7)-(36:11)	
# > 	guard-demo.jsonnet:(28:3)-(38:4)	thunk <array_element>
# > 	During manifestation	
```

This is a contrived way to add validation to package.json. Unless the validation is highly localized, you're probably better off using a [json schema](https://json-schema.org/)-based validator.

# importing external sources

## external files, on- and off- disk

Jsonnet's `import` works by file path. This means that if the parent file resides in a remote git repository, and you want to inherit it using `local Upstream = import "./upstream/parent.jsonnet";`, here are 3 ways to make the import file visible
1. clone / subtree / submodule the repository to `./upstream`
2. if the repository is already on the local filesystem at `/path/to/upstream`, you can symlink with `ln -s /path/to/upstream ./upstream`
3. if the source is a remote, publicly accessible repository, you can mount the repository using a tool like [hubfs](https://github.com/billziss-gh/hubfs/tree/master/src), e.g. `mkdir upstream; hubfs -auth none https://path-to-upstream.tld/upstream.git ./upstream` and reference the file by git ref, like `local Upstream = import "./upstream/master/parent.jsonnet";`, or even `./upstream/0123456789abcdef0123456789abcdef01234567/parent.jsonnet` for the specific commit; the obvious drawback is that once you terminate the hubfs process, the remote source is unmounted, and Jsonnet will fail to render. I have only used this method for one-off testing.

## github repositories or any JSON-ifiable remote source

Jsonnet does not support importing files over http/s, so if you want to dynamically evaluate an expression from a remote source, one method is to load it from STDIN with `--ext-code`

```sh
echo '{locallyDefined: "some-value"} + {external: std.extVar("stdinJsonArray")}' |
    jsonnet --ext-code stdinJsonArray="$(curl -s https://raw.githubusercontent.com/json-api/json-api/gh-pages/_config.yml | yq -o json '.quicklinks' -)" -

```

this method would work similarly for any data source that outputs valid Jsonnet or JSON

# problems and extensions

Teams that adopt an extra layer of code-driven configuration would have an extra source of conflicts when working together in a version-controlled codebase. It helps to aggressively and frequently sync generators and generator data in the code cycle. Adding tools like standardized scripts and git hooks can help simplify the process.

As a full-fledged programming language, Jsonnet opens up the door for hacks and tricks for config generation, and thus requires programmers to practice good judgment (or hacksters to practice moderation) in the goals of simplifying and modularizing configuration. On the other hand, it also enables embedding self-describing and self-validating logic into the configuration files themselves.

## how about YAML?

Jsonnet easily extends to Yaml configurations (kubernetes and others) by using [yq](https://github.com/mikefarah/yq) as a bridge between JSON and YAML. This convenience, plus `jsonnet`'s ease of composition, and Yaml's lack of it, has made me replace all YAML configurations with Jsonnet-based generators as well. Kubernetes? Jsonnet. Cloudformation? Jsonnet it. Terraform? Jsonnet, damnet.

# supporting a bidirectional jsonnet <--> json workflow

## remembering to generate up-to-date JSON

I usually use `make` for this, and I haven't seen a more suitable tool for the job:

```makefile
target-file.json: generator-file.jsonnet
	jsonnet $< | jq -S | tee $@
```

## tools for working in the shell

Assuming you are working with JSON files generated from Jsonnet, there are 2 main diffs you care about on every effective transmutation: one in the Jsonnet, and one in the JSON. Jsonnet diffing is like any other code diff, but JSON diffing can sometimes be more annoying than it should be. One reason is JSON's disallowance of the trailing comma. Another reason is that JSON data is not well-suited for line-diffing.

Here are some interesting tools for JSON diffing that provide more useful context than plain line diffs:
- https://github.com/andreyvit/json-diff: structural diff for JSON files
- https://github.com/trailofbits/graphtage: semantic diff

After having tried several of these libraries, and also writing several structural and visual tools for tracking JSON change management, when it comes to working in version-controlled JSON changesets, the Jsonnet-centric workflow I found most effective is still terminal-based, producing real-time feedback using a collection of fast, single-purpose tools. Currently, this involves using [icdiff](https://github.com/jeffkaufman/icdiff) for side-by-side, colorized diffs, and a bunch of bash functions.

For example, to get a nice diff:

```bash
icdiff <(cat package.jsonnet|jsonnet -|jq -S) package.json
```

the side-by-side, colorized output shows up like so

[![asciicast](https://asciinema.org/a/475031.svg)](https://asciinema.org/a/475031)

## a terminal-based editing / syncing flow

To illustrate how this works, let's take a process where I'm generating the package.json file above from Jsonnet. The left side shows what the Jsonnet generates, and the right side shows what is currently in package.json (the ground truth). In order to make the Jsonnet source correctly generate and match what's in package.json, we use `watchexec` to trigger `icdiff` to print the colorized diff every time the generator file is updated.

A vim-integrated setup is shown in this asciicast:

[![asciicast](https://asciinema.org/a/475041.svg)](https://asciinema.org/a/475041)

The logic behind this flow is implemented in [this bilingual script](https://github.com/whacked/setup/blob/master/bash/package-jsonnet-composition.nix.sh), which can be sourced in a BASH-compatible shell or imported into a nix expression (see [this article](./nix%20shellHack.md) for an explanation of the bilingual construction); the required programs are declared in the line below `buildInputs`: pastel, gron, fswatch, icdiff, jsonnet, watchexec, vim with +terminal.

# package management

The most highly visible attempt at creating a package management solution for Jsonnet appears to be [jsonnet-bundler](https://github.com/jsonnet-bundler/jsonnet-bundler), whose main binary is a short `jb` command. There are friend repositories such as [jsonnet-libs](https://github.com/jsonnet-libs/xtd) that specifically target distribution using `jb`, so it probably commands the most mindshare right now.

`jsonnet-bundler` basically operates off the assumption that at run-time, you call `jsonnet` with the `--jpath` flag and specify the location of the `jb`-managed libraries, which defaults to `vendor`. So instead of running `jsonnet <path-to-file>`, you'd run `jsonnet --jpath vendor <path-to-file>` or equivalently, `jsonnet -J vendor <path-to-file>`.

While you can override `jsonnet-bundler`'s default using the `--jsonnetpkg-home` flag, it operates in the style of e.g. `npm`, where it wants to put the vendor libraries and the package lockfile in the current directory of execution. There are pros and cons to this approach, but for my personal use, I am much more partial to the style of  maven, which pulls libraries from a centralized location on the system (usually `$HOME/.m2`).

Another problem is that `jb` has expectations of how the `jb`-managed libraries are structured, which adds a level of annoyance when mixing "dumb" libraries like one-off files. There's always the option of directly importing the external file by relative or absolute path, but now you have to context switch between external-and-jb-managed and external-but-not-jb-managed files.

## massaging `JSONNET_PATH` to have it both ways

By setting the `JSONNET_PATH` environment variable to include both `jb`-managed libraries at a centralized location, and installing other external files / repositories to the central location using the same structure, we can mix libraries easily.

The method is encoded in this [nix-bash bilingual file](https://github.com/whacked/setup/blob/ffdfc4aeebf9573ba604e5280a802a5816609cb6/bash/jsonnet_shortcuts.sh). It supplies 2 functions, one that wraps `jb` so that it always installs to the central repository location, and another that wraps `git clone` so it installs non-jsonnet-bundler repositories to the central location using the same layout. Using these functions then, I would do

```bash
# for jsonnet-bundler style repos
# instead of: jb install github.com/jsonnet-libs/xtd
jsonnet-bundler-install github.com/jsonnet-libs/xtd

# for other repos
jsonnet-repo-install github.com/example/other-library

```

Then in my jsonnet file:

```jsonnet
// jsonnet-bundler
local ascii = import 'github.com/jsonnet-libs/xtd/ascii.libsonnet'

// non-jsonnet-bundler
local otherLib = import 'github.com/example/other-library/templates/jsonschema.jsonnet'
```

Note that while `jsonnet-bundler` creates a symlink within the `--jsonnetpkg-home` directory so that you can import it using e.g. `import 'xtd'` above, I prefer to ignore it and use fully qualified paths both to keep consistency across `jb` and non-`jb` libraries, but also to signify that these are external imports.

Another downside is that we completely ignore jsonnet-bundler's lockfile, which encodes the dependency tree. I have not found this to be a problem whatsoever, and I suspect that since jsonnet functions entirely in data space, the room for "spooky state changes at a distance" is greatly reduced, such that having the dependency tree is less important than having the _output schema_.

# misc notes

## using language bindings

see https://jsonnet.org/ref/bindings.html; but for NodeJS I find [@hanazuki/node-jsonnet](https://github.com/hanazuki/node-jsonnet#synopsis) a better experience.
