Tools for remembering code projects
===

If there is one thing I'm really really good at, it's being unable to remember things. I often struggle to recall the name of the store I visited last week, the file I edited yesterday[^find-command], or the browser tab I re-opened minutes ago. Unfortunately, I still don't have real solutions to these challenges, but for code and programs, here are some tricks and tools for reminding myself what I need to know. Note that these codes and programs are usually contained in code projects, so you can consider this a subset of "personal knowledge management", whose discussion is unfortunately in a different castle.

[^find-command]: there are useful suggestions using `find` like in [this discussion](https://stackoverflow.com/a/16086041), but often I am not even sure if it was a file I edited, visited, or a website, or email, or.

# anticipating my future, dumber self

If there's one prediction I'm really really good at making, it's the prediction that future me is not going to remember. Since the user (later me) is slower and forgetfuler than the maker (now me), we need to minimize the number of commands to remember, and minimize the amount of searching needed to get answers. So the *project* should tell what it is, what it do, what it need, how it run, who my friend.

For tiny projects, file names like `main.py` may be suggestive enough of the program's entry point: you'd run `python main.py` and expect it to Do The Thing. But what about `java -Xms256m -Xmx2048m -Djava.library.path=. -classpath .:./mysterious.jar:./dependency.jar:./requirement.jar MainProgram`? Probably the only time I can reproduce a command like that is when I'm dreaming, which explains why I don't have memory of it happening. Naively then, I would put those long commands into a launch script like `run.sh` or `start.sh`

And who's going to tell me if the settings, the stage, and the supporting actors are at the ready?

Maybe I should add environment variable initializations into `run.sh`. Then add a few `if` statements verifying that paths exist. I'll run `ldd` and `grep` to check if the linked libraries exist and are the right version. Then extract these repeated operations into a separate function and put them somewhere else. Maybe I'll call it `doctor`!

The annoying thing about whatever [which](https://docs.brew.sh/Manpage#doctor-dr---list-checks---audit-debug-diagnostic_check-) [doctor](https://docs.flutter.dev/get-started/install/windows#run-flutter-doctor) you have, is that unlike human doctors that give you a prescription, these code doctors just tell you what might be broken, and then _you have to find the prescription_. In the end, I'm in the operating room. You should be called nurse.

#TODO (insert graphic here?)

It's more effective to describe how to reliably get to the correct state. And now, we have a dependency management problem, although it's a better problem: it's simpler to specify how to work forward from a kernel that grows into tree, than whacking moles playing hide and seek in the leaves of an already big tree.

## how to be healthy without the doctor

For persons, this is also known as preventive medicine. For projects, this is sometimes known as `docker`.

I lied. `docker` has good rerunnability, but hides the doctor's sweat and tears behind a tricked-out, uncomposable shell script and its own management suite [with](https://docs.docker.com/engine/install/ubuntu/) or [without](https://podman.io/getting-started/) a client-server model: two burdensomes for my tiny memory.

For projects, what we need is a foolproof guide to put us in the correct state where we can run our program and Do The Thing. Wait, what program? Do what thing? The foolproof guide should *also* tell us that.

# starting with the plainest of texts

## remembering the ~~BIG BANG~~ genesis

These days, many frameworks have scaffolding commands, like `npx create-next-app my-nuxt-app`, `lein new app my-boot-app`, `rails new my-sinatra-app`, that generate a runnable project skeleton. They also generate README files, but don't record _how_ the project was created (try `lein new luminus my-project +aleph +http-kit +h2 +sqlite +shadow-cljs +war +auth`). So the first thing I do is record the command somewhere. When revisiting a shelved project, the underlying framework versions and template commands may have changed, so it is important to record how the world appeared out of nowhere.

It's also sensible to add the BIG BANG command into a git commit, but sometimes I am so clever and write something like `initial commit`. Sometimes I make modifications before first light, maybe because I'm following a guide, I cloned a sample repo, or because the generator creates junk I don't want in the commit: multi-platform scripts, unusued database seed files, sample files with 🎉 emojis congratulating me in understanding instructions (everything is so complicated now, it's important to reward every step). If I forgot to commit the genesis command, I'd want to stuff it in a README or another text file.

Still here? Prize! 🎉🎉🎉

## the README

Since many projects are initialized with a README now, it seems natural to also put "how to run this" in there. It also seems like the least effective method: READMEs usually describe static or slow-changing information, such that for projects in flux, they undergo factuo-active decay. You might be a disciple of discipline and decide to "be disciplined" about disciplining the README.

That would certainly set a bar for quality. So the "README" name carries weight and I don't expect to see a memories of better days dumped in there. So before committing a README change you'd want to send it through the spell check, grammar check, grandma check, blank check, focus groups, legal, HR, subscribe like follow me on Twitstagram and all that. Soon, the mental red tape seeps into your subconscious, and past promises of the README asynchronously decay into future falsehoods.

## the run log

So now your README is Truth but minimal, and doesn't list all the possible commands that you need to know -- doing that for something like [awscli](https://github.com/aws/aws-cli#basic-commands) would be disasterotaculous (awscli 1.20.54 has 286 subcommands [^aws-cli-subcommands]). But you need to remember; what do?

[^aws-cli-subcommands]:
				interestingly, getting this number was not as straightforward as I expected, because `aws help` [prints funny stuff](https://github.com/aws/aws-cli/issues/5455). So ultimately, this is what I ended up with: `aws --no-paginate help | col -b | sed -e '1,/AVAILABLE SERVICES/d' | sed '/SEE ALSO/Q' | grep ' o ' | wc -l`, which translates to: spit the `aws` non-paginated help (else your pipe gets no output), `col` remove all control characters except the final column, `sed` remove all lines up until and including `AVAILABLE SERVICES`, `sed` remove all lines including and after `SEE ALSO`, `grep` all lines with the `o` entry marker, `wc` count lines.

For shell interaction, sometimes I use a run log: run some commands, check the output, and if everything looked good, copy-paste everything into a file called `run.log` in the project directory. I still do this when no tools are available. [^shell-recording-tools] I also know big-head people who just review `history`, but since I can't remember context, I prefer to record as I go.

[^shell-recording-tools]: `script` might be the most prevalent tool on bare machines, and there are more sophisticated tools like `asciinema`, but I have not found them to be the most suitable for self reminders:
				- the output format is not human-first
				- it captures everything: treasure and trash
				- I want to edit as I go (when I have the most mental context), instead of edit afterwards
				- that said, having playback is immensely useful, but having to watch a movie to find one dialogue is annoying

At one point I wrote a shell function `rr <command>` that automated recording the command and output into `run.log`, or if I just finished running a command, `rr` would append the previous command (via `fc -ln -1`) to the log. This was a failure: among other problems [^other-problems-with-rr], when I want to run `<something>`, I am not *inclined* to run `rr <something>`; on the other hand, after I run `<something>`, I won't *remember* to run `rr`.

[^other-problems-with-rr]:
				- a dumb command wrapper with pipes interferes with running character-mode programs
				- forgetting to run `rr` breaks the flow if the command modifies the system
				- wrong arguments, stale inputs, re-run of commands by mistake means cleaning up the log
				- sometimes you can't predict how much output the program will create
				- sometimes you can't predict how long the program will run
				- it interferes with a flow of background/resume/pkill

I also rarely bothered to revisit `run.log`. When you're adding to the log, everything makes sense! When you're revisiting a week later, nothing makes sense, so you start from the beginning. Now, when you're adding to the log, everything makes sense! Déjà vu?! Wait...

To really make the log file useful, you need to have discipline: clean it up diligently, send it through the spell check, grandma check, like and subscribe, etc. etc. so we're back to README with partially automated copy-paste.

# tool-enhanced interaction and history

## literate programming and org-mode + babel

Working with `org-mode` in Emacs means it only takes a few keystrokes to run a command and capture its output in the same file. This addresses the pain point of having to remember to record the contents and outputs of important commands. As a result, all my documentation was in org-mode for a time. Filtering files in bash, saving the results to a variable, and processing the result in python, then plotting the output in R, all in the same file, is a magical feeling.

[![asciicast](https://asciinema.org/a/BlN2izVoMJCtb83OzJTeFqMoE.svg)](https://asciinema.org/a/BlN2izVoMJCtb83OzJTeFqMoE)

To see org used effectively, see the demonstration in [literate devops with Emacs](http://howardism.org/Technical/Emacs/literate-devops.html) by Howard Abrams. As impressive as it is, I don't recommend this approach anymore.

The first two problems, in practice:
1. most people don't use Emacs
2. among people who use Emacs, less are proficient in org-mode, and of those who are proficient, less are as proficient as Howard.

We can easily export org files into nicely-typesetted, syntax highlighted, publication-quality documents, but the exported files, be them HTML or PDF documents, are optimized for archival and consumption, not interaction and modification. The moment you send an export to a colleague, you'll see them extending it in the company-sanctioned WYSIWYG editor, leading to cosmic visual torture of code blocks in Sans-Serif font. The moment you send the original org source to a colleague, you'll hear them converting it to their favorite flavor of markdown, and you wonder why you didn't just send the output of `org-md-export-to-markdown` (but the voice on your shoulder whispers _just use markdown_).

In the end, you got the information across, so where's the problem?

Firstly, the commands are part of an interactive, living system. Your org file and your colleague's markdown, while structured around two different brains, should model the same truth. The representatation of truth changed from an org-wrapped representation to an org-plus-markdown-wrapped representation. Let's say a 3rd colleague joins the team with their 3 Letter-sized pages of 12pt 1" margin Arial font notes. Now you have to co-maintain the shared truth, and more layers of wrapping means more effort.

Less impedance of tools means faster output. At this point, markdown tooling is so much more extensive and widespread that I hesitate to recommend `org-mode` for the locus of documentation, including to myself. While structured plain-text markup files should easily [convert between each other](https://pandoc.org/), maintaining true compatibility means avoiding advanced features like code blocks in `org-mode`.

That said, there are still some situations where org-mode can be the best tool at hand. For example, when HashiCorp's HCL didn't support composition, I used `noweb` in org-babel to maintain global resource constants, and tangled out updated `hcl` files for `terraform`. I then executed all `terraform` (with `-auto-approve`) commands from the org file with all outputs saved. It works best within a small, contained level of complexity, and a small number of brains.

### a few more limitations with Emacs + org-mode
- for shell commands in babel, the first problem is syncronous evaluation. This isn't a bottleneck for simple commands, but for long-running commands, we quickly enter an engineering vortex.
	- to un-block emacs, we need to run a separate process
	- we want to see shell output as it appears
	- we need to capture outputs back into org-mode
	- I wrote [ob-shstream](https://github.com/whacked/ob-shstream) for working with asynchronous / long-running processes, but all things considered, there are better methods.
- code block line numbers don't match line error messages
- commands with massive output or long lines can cripple Emacs
- remoting requires configuring the remote shell, special header directives in shell blocks, running a separate shell process (`M-x shell` or maybe `vterm`, as separate challenge), and various other things you figure out on a Saturday afternoon when you realize maybe you've really gone too far. But wait, it looks pretty close, one more fix!

## jupyter notebooks

Circa 2017, IPython notebooks started replacing Emacs for a lot of my executable documentation. The bash kernel <https://github.com/takluyver/bash_kernel> addresses most of the problems with command and output management for shell scripts, including those with long-running processes and large output. I tried keeping text in Emacs using [ein](https://github.com/millejoh/emacs-ipython-notebook), but in the end, server + browser remains the easiest to remember.

IPython and Jupyter are excellent tools that have changed the industry; the great features of notebooks are widely covered elsewhere, so I will focus on limitations, particularly as they relate to a system for remembering how to execute code and programs within a project.

### limitations
-   one language at a time [^multi-kernel-solutions]
- backwards compatibility issues between IPython notebook and Jupyter lab [^ipython-jupyter-incompatibility]
-  notebooks are stored in JSON, and JSON:
	- is not human friendly, and is, for all intents and purposes, not grep-friendly
	- not a good fit for output log storage
	- hard to diff readably
	- as an aside, using Emacs EIN _does_ allow for a human-friendly text file notebook, but the pros from having features in the browser-based notebook outweigh the pros of having a human-friendly file
- refactoring is a hassle, and it's trivially easy to end up with linearly organized notes but a non-linear execution order; [gather](https://github.com/microsoft/gather) aims to solve this problem, but I have reservations about this approach: it appears to alleviate the burden of code organization for runnability, but the core issue is actually understandability.
-  For local rendering, ipynb files look best in ipython notebook/jupyter lab. Sometimes nTeract, sometimes VS code, in that order, but viewing the notebooks just for reference can be a hassle.
-  CodeMirror is a very good editor, but is not a purebred text manipulator like Emacs nor Vim
-  using `bash_kernel` on to reach a remote machine requires fooling the kernel by hacking `PS1` [^bash-kernel-ssh]
-  as of this writing, no STDIN support for `bash_kernel`, so processes reading from STDIN will hang on input
-  sometimes the process output breaks and restarting the kernel is the only way out
-  shared evaluation session allows the undisciplined (me) user to easily lose track of execution dependencies and evaluation order
-  no simple way to edit outputs for generalization / obfuscation purposes
-  only a single evaluation is kept, which makes it difficult to compare time-specific outputs in-notebook [^multi-outputs-plugin]

[^multi-kernel-solutions]: multi-kernel solutions like [SoS](https://vatlab.github.io/sos-docs/), [polynote](https://polynote.org), and [beakerx](https://github.com/twosigma/beakerx) remind me of the Emacs situation: powerful but niche tooling. For that matter, Org babel supports at least [30 languages](https://orgmode.org/worg/org-contrib/babel/languages/index.html)
[^ipython-jupyter-incompatibility]: I believe Jupyter Lab is the de-facto solution for ipynb files these days, but since I built a [repeatable workflow](https://github.com/whacked/demodemodemodemo/blob/master/jupyter-notebook-sandbox-with-extensions/default.nix) around several plugins, which were unavailable (incompatible) with early Jupyter, I have been dragging my feet. Years ago, you had to choose between either having `ipython-contrib` extensions (use IPython) or other features like draw.io support (use Jupyter).
[^bash-kernel-ssh]: using `bash_kernel` on a remote host requires overriding `PS1`: `ssh target-hostname -t 'PS1="[PEXPECT_PROMPT>" bash --norc --noprofile'`
[^multi-outputs-plugin]: see [this plugin](https://github.com/NII-cloud-operation/Jupyter-multi_outputs) and [my modifications](https://github.com/whacked/Jupyter-multi_outputs) for a solution

# managing requirements and remembering how to run programs

## Makefile

My experience with `make` is almost from entirely building software someone else wrote, and I understand maybe 5% of the syntax. But for running lightweight, well-defined, cacheable pipelines, like codegen and asset syncing, `make` is the best tool I know. I don't think the syntax is very pleasant (I found [tup](https://gittup.org/tup/ex_multiple_directories.html) easier to reason about), but after looking around tools like tup, [dvc] (https://dvc.org), [ninja](https://ninja-build.org), and a few others, there doesn't seem to be anything as lightweight, fast, and practically ubiquitous, for managing local, low-complexity program pipelines.

For example, I use `make` to generate project skeletons from a shared upstream directory. When the directory changes, I use a syncing target to update compatible changes into the downstream repository. For comparison, I don't know how to do this nicely in a `leiningen` project with shared, non-packaged dependencies, so syncing co-dependent projects is either a manual process, or cobbled together with nested macros in `project.clj`.

How do I remember what targets in `make` to run? I always include from a shared (upstream) Makefile with a `help` target:

`/path/to/shared/Makefile`:

```makefile
# helper target modified from: https://stackoverflow.com/a/59087509
help:
	@grep -B1 -h -E "^[a-zA-Z0-9_-]+\:([^\=]|$$)" $(MAKEFILE_LIST) \
     | grep -v -- -- \
     | sed 'N;s/\n/###/' \
     | sed -n 's/^# \(.*\)###\(.*\):.*/\2###\1/p' \
     | column -t  -s '###' \
     | grep -v '^help '


# a very useful sentence describing the process   
shared-process:
	run something useful
```

`project/Makefile`:

```makefile
include /path/to/shared/Makefile

### don't show this target
hidden-target:
	execute something else

# run this to do the thing
do-stuff:
	do the thing
```

then, running `make` or `make help` from `project`:

```
$ make
do-stuff        run this to do the thing
shared-process  a very useful sentence describing the process
```

What's more, zsh understands Makefiles out of the box, so tab completion comes for free!

This usage may be antithetical to the common usage in building software via `./configure; make; make install`. Perhaps if there's a build specification file -- `Makefile`, `Tupfile`, `build.ninja` -- running the build program already means you want to build _right now_. But some other "builder programs" like `cargo`, `lein`, `go`, `flutter`, when called without a `build` or `compile` argument, print a help message with a list of available commands. It seems like these programs don't infer intent without an explicit verb, and they will tell you what verb to use. Some manager programs even detect typos:

- `pip` produces suggestions [by text similarity with valid commands](https://github.com/pypa/pip/blob/7f8a6844037fb7255cfd0d34ff8e8cf44f2598d4/src/pip/_internal/commands/__init__.py#L122)

```
$ pip isntal typo
ERROR: unknown command "isntal" - maybe you meant "install"
```

- `npm` [actually understands "instal" among several other typos](https://github.com/npm/cli/blob/f17aca5cdf355aaa7e1f517d1b3bb4213f4df092/lib/utils/cmd-list.js#L39)

```
$ npm isntalll typo   

Usage: npm <command>

... lots of halp ...

Did you mean this?
    install
```

## `project.json`

`npm`'s `scripts` is a nice feature for reminding how to Do The Things. It allows you to store essential commands into `package.json`:

```json
{
	...
	"scripts": {
		"collect-dependencies": "node main-script.js --collect-dependencies --with=complex-args",
		"build-bridge": "build --structure=bridge --type=suspension --from=castle --to=sky"
	},
	...
}
```

with this in the `package.json`, we get this kind of helpful interaction

```
$ npm run
Scripts available in  via `npm run-script`:
  collect-dependencies
    node main-script.js --collect-dependencies --with=complex-args
  build-bridge
    build --structure=bridge --type=suspension --from=castle --to=sky
$
```

### limitations
- no structured comments: this may be by design, but as a memory aid, it helps to have context
- npm-native: doesn't make sense for non-node projects; otherwise, it creates temptation to stash management commands into the npm toolchain. If you have a Django backend with `django` management commands, you probably don't add django commands into `package.json`, so you still have to keep track of at least 2 command entrypoints: `npm run` and `python manage.py`.
- package.json is [not designed for composability]([https://github.com/npm/npm/issues/8112#issuecomment-192489694](https://github.com/npm/npm/issues/8112#issuecomment-192489694); 10 projects with the same `npm run X` command would have 10 identical `"scripts": { "X": "same special command" }` entries. If I update `special command`, I would need to remember to update 9 other projects (I won't).
- `package.json` is now a kitchensink metadata file, containing dependencies, to delcarations of [plugin](https://code.visualstudio.com/api/references/extension-manifest) [capabilities](https://flight-manual.atom.io/hacking-atom/sections/package-word-count/), to commands for for testing and building. It's sensible in that it's all data _about the package_ -- but so is the README. In the end, the distinction seems to be `package.json` for electrons and README for neurons. Since I'm losing neurons but not electrons, I have to play dangerously in machine land.

### recollection bad? composition good!

If you have a bunch of Node projects that follow a similar base dependency structure, and suddenly dependabot suggests updating dependencies all at once, one way to ease remembering which projects to update is by restructuring `package.json` files into an inheritance hierarchy; several tools exist with this functionality: [dhall](https://dhall-lang.org/), [cue](https://github.com/cue-lang/cue), straight javascript or typescript, but in my experience, [jsonnet](https://jsonnet.org/) offers the best feature set of a pure and complete JSON substitute [^jsonnet-quirks]

- Jsonnet is a strict superset of JSON, which allows low-friction ease-in (as TypeScript does for JavaScript)
- comes with syntax formatter (which works excellently in [Vim](https://github.com/google/vim-jsonnet)) and more regular syntax (that leads to cleaner diffs)
- loose importing means any jsonnet file can inherit from any other jsonnet/json file, which makes setting up composition very easy
- easy [bindings for several popular languages](https://jsonnet.org/ref/bindings.html) including python and Node

As an example for package inheritence then, I would have a shared package file:

```
$ cat super/package.jsonnet
{
  dependencies: {
    '@quali/fied': '^1.2.3',
    pi: '^3.0.0',  // library for approximation
  },
  devDependencies: {
    quoteLessKeywords: '^10.12.14',
    trailingComma: '^20.40.60',
  },
}

```

then the satellite projects

```
$ head hobble/package.jsonnet web/package.jsonnet
==> hobble/package.jsonnet <==
(import '../super/package.jsonnet') {
  dependencies+: {
    pi: '^3.1.0',  // output looks better
  },
}

==> web/package.jsonnet <==
local Base = import '../super/package.jsonnet';

{
  dependencies: Base.dependencies {
    donut: '^1.0.1',
  },
}
```

Then after we update `pi` to `3.1.4`, use a `Makefile` to regenerate the `package.json` target:

```
[web] $ cat Makefile
package.json: package.jsonnet
	jsonnet $< | jq | tee $@
[web] $ make
jsonnet package.jsonnet | jq | tee package.json
{
  "dependencies": {
    "@quali/fied": "^1.2.3",
    "donut": "^1.0.1",
    "pi": "^3.1.4"
  }
}
```

[^jsonnet-quirks]: due to an odd quirk of jsonnet [producing 3-space-indented JSON](https://github.com/google/jsonnet/issues/547), such that I almost always pipe the output through `jq` for re-indentation.

This automates syncing the package information in one direction, but if we run an `npm install` or `yarn add` in the satellite project, syncing back upstream becomes a separate problem. But before any of that: how do I remember to run `make` at all? Ideally, I would enter the project, and the project will tell me.

## enter the project, get a tour guide

Over the past several years I found the [nix package manager](https://nixos.org/guides/install-nix.html) very effective for these features:
- works on at least Linux and mac, often _identically_ (reliable!)
- tight control over software dependencies, down to the compiler (_what it need!_)
- self-contained, per-project shell environment, where I can write **reminders when the shell starts**! (_what it do and how it run!_)
- who my friend? `nix-shell`

Now, I am not an expert nix user, and the workflow described here may be non-standard, but it has worked better than anything else I have tried to date (if you know a better solution, I'd love to know [^nix-alternatives])

[^nix-alternatives]: [Guix](https://guix.gnu.org/) comes up, but Nix has far more packages. [Spack](https://spack.io/) is also very interesting but I found Nix better for system-level integration, plus it has far more packages. Also, `ripgrep` recognizes the `.nix` filetype natively

### reminding myself what I can do in a project

I add aliases for the most useful commands into the `shellHook` property inside the `shell.nix` or `default.nix` file, which nix looks for by default.

then, enter the nix-defined environment by running `nix-shell` (aliased to `nsh`)

```
$ nsh
/tmp/demodemodemodemo/schema-server/node_modules/.bin/shadow-cljs
shadow-cljs is in /tmp/demodemodemodemo/schema-server/node_modules/.bin/shadow-cljs
      alias watch='shadow-cljs watch main server'
      alias wserver="watchexec --restart --no-ignore --watch app/ node app/server.js"
```

the `shellHook` ends with a `grep` for all aliases in the nix file, so once I enter the shell environment, I am reminded of useful things I can do, all the commands I don't remember how to use, and any other messages I think I should pay attention to (with the `pastel` package for attention-grabbing alerts).

with some extra support in bash, here's the greeting when I enter `nix-shell` in this [jupyter notebook environment](https://github.com/whacked/demodemodemodemo/tree/master/jupyter-notebook-sandbox-with-extensions):

```
$ nsh

...

=== shortcuts from default.nix ===
    function enable-jupyter-extension() {
    function disable-jupyter-extension() {
    function setup-tslab() {
    function setup-first-run() {
    function setup-additional-packages() { 
    function run-unprotected-server() {
    alias build-docker-container='sudo $(which docker) build . -t emacs-with-nix'
    alias run-nix-docker-container='sudo $(which docker) run -p 8888:8888 -v $PWD:/opt/demo -w /opt/demo --rm -it emacs-with-nix /bin/bash'
    alias run='jupyter notebook --no-browser'
(jupyter-notebook-sandbox-with-extensions-venv) 
[nix-shell:/tmp/demodemodemodemo/jupyter-notebook-sandbox-with-extensions]$ 
```

a huge benefit of defining environments this way is that composing different environments together is simple:

```nix
{ pkgs ? import <nixpkgs> {} }:

let
  # use the webserver environment as a base
  webserver = (pkgs.callPackage (import ./nix/webserver/default.nix) {});
in pkgs.mkShell {
  name = "composition-example";

  buildInputs = [
    pkgs.jq
    pkgs.nodejs
	pkgs.pastel
  ]
  ++ webserver.buildInputs
  ;

  nativeBuildInputs = [
    ~/setup/bash/nix_shortcuts.sh
  ];

  shellHook =
    ''
	pastel paint -n cyan "including the webserver shell env..."
    ''
    + webserver.shellHook + ''
    if (which shadow-cljs 2> /dev/null); then
        echo "shadow-cljs is in $(which shadow-cljs)"
    else
	    pastel paint -n white --on magenta "setting up a new shadow-cljs environment"
        npm install shadow-cljs react create-react-class react-dom
    fi

    function generate-cljs-build-report() {
        shadow-cljs run shadow.cljs.build-report $1 report-$1.html
    }
    
    alias wserver="watchexec --restart --no-ignore --watch app/ node app/server.js"
    alias dev='npm run dev'
    alias release='npm run release'
    alias pull-data='python pull_data.py'
	
    echo-shortcuts ${__curPos.file}
  '';
}
```

### how to remember nix syntax

I confess, I don't remember nix syntax. Compared with the [top 10 languages on github](https://madnight.github.io/githut/#/pull_requests/2021/4), Nix -- apparently 11th on the list [^package-language] -- probably has the most unusual syntax. But I also don't actually feel the need to write it often enough for the syntax to become ingrained in memory, so I rely on a shell function to generate a nix skeleton when I create a new project:

```bash
function create-nix-shell-skeleton() {
    # https://nixos.wiki/wiki/Development_environment_with_nix-shell
    if [ -e shell.nix ]; then
        echo "ERROR: shell.nix already exists; doing nothing"
        return
    fi
    cat > shell.nix<<'EOF'
{ pkgs ? import <nixpkgs> {} }:
pkgs.mkShell {
  buildInputs = [
  ]; # join lists with ++

  shellHook = ''
  ''; # join strings with +
}
EOF
}
```

Then I add project package dependencies to `buildInputs`, add shared scripts to `nativeBuildInputs`, and shared shell initializations into `shellHook`. This works for most projects I create or import.

If you do this, you'll proably be using less than 5% of nix's functionality. But there aren't many tools where 5% functionality gives you something close to a superpower. [^high-leverage-tools] Just imagine how popular you'll be by trivially running 5 different postgreSQL servers!

```bash
$ which postgres 
postgres not found
$ for v in 10 11 12 13 14; do nix-shell -p postgresql_$v --run 'postgres --version'; done 
postgres (PostgreSQL) 10.19
postgres (PostgreSQL) 11.14
postgres (PostgreSQL) 12.9
postgres (PostgreSQL) 13.5
postgres (PostgreSQL) 14.1
$ 
```

Well, I certainly can say from experience, it doesn't make you popular at all. But if you're not popular anyway, why not pick up a superpower while you're at it?

[^package-language]: to be clear, this is an artifact of being a package langauge, and [according to repology.org](https://repology.org/repositories/statistics), `nix_unstable` currently has over 60k packages, the highest of all repositories tracked
[^high-leverage-tools]: what else is there? smartphones? POSIX pipes? programming languages? spreadsheets?

### what if no pre-defined packages exist?

Every once in a while, something more special is needed (like [package](https://eipi.xyz/blog/simple-single-package-pinning-on-nixos/) [pinning](https://gist.github.com/zimbatm/de5350245874361762b6a4dfe5366530)) or compiling a package whose dependencies are not in nixpkgs already. Then I am resigned to spend hours praying to Google, chasing geese, inventing new expletives, and wondering if it was all worth it.

Fortunately, after having gone through many of these chases, I think on balance, this has brought me more time, happiness, and peace of mind than it has frustration. I was able pick up an old python2.7 project, activate its shell environment, and have it Do The Thing and that Blowed The Mind.

Since Nix outputs are highly reusable, a one-time toil often leads to N-time productivity. I am happy standing on the shoulders of giants and walking on the backs of geese chasers, and adding `shell.nix` to code projects.

## remembering what web apps do

Sometimes you might make a single-purpose webserver to interact with information using the browser. A while ago I was [experimenting](https://github.com/whacked/demodemodemodemo/tree/master/schema-server) with json schemas using a little web server. Revisiting it after a few months, I no longer remember what it does.

As a reminder, I expose the sitemap at the `/help` endpoint: [^help-endpoint]

```
ROUTES:

    /
    /schemas/
    /schemas/:name
    /schemas/:name/generate-samples
    /schemas/:name/validate
    /js/*.js
```

It's a simple thing, and depending on the framework, this is a [solved](https://stackoverflow.com/questions/1275486/django-how-can-i-see-a-list-of-urlpatterns) [problem](https://guides.rubyonrails.org/routing.html#listing-existing-routes) in the command line or otherwise, but when I am interacting with the app through a browser, I like to receive the information also through the browser.

[^help-endpoint]: `schema-server` uses a [data-as-routes](https://github.com/metosin/reitit) library for routing, so we create the help route by [creating a route handler with the routes mapping](https://github.com/whacked/demodemodemodemo/blob/e635dec2de9dd5f5279798129b375f73dd7e215f/schema-server/src/schema_server/server.cljs#L120) and adding that back into the routes. In `flask`, this can be achieved using `url_map` for a specific app

```python
@app.route('/help')
def show_route_list():
	from flask import render_template_string
	route_list = []
	for rule in app.url_map.iter_rules():
		if rule.endpoint != 'static' and '<' not in rule.rule:
			route_list.append((rule.rule, app.view_functions[rule.endpoint].__doc__ or ''))
	route_list.sort()
	return render_template_string('''
	<ol>
	{% for rule, doc in route_list %}
	<li>
		<a href="{{ rule }}">{{ rule }}</a> :: {{ doc }}
	</li>
	{% endfor %}
	</ol>
	''', route_list=route_list)
```

in `express`

```javascript
expressApp.get('/help', (req, res) => {
	let ol: Array<any> = ["ol"]
	expressApp._router.stack.forEach((layer) => {
		if (!isEmpty(layer.route)) {
			let li = ["li",
				["div",
					["a", { href: layer.route.path }, ["code", layer.route.path]],
					Object.keys(layer.route.methods != null ? layer.route.methods : {}).map(
						method => ["code", method]
					),
				]
			]
			ol.push(li)
		}
	})

	return respondHiccup(res, ["body", ol])
})
```

# conclusion

I share my experience in working with personal projects over the years, with strategies to help future me know what to do. If you liked it, please remember to HIT THAT LIKE BUTTON. Oh wait, there's no like button. I'll be happy if you find some of these experiences useful too.
