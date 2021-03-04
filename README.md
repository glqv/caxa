<h1 align="center">caxa</h1>
<h3 align="center">📦 Package Node.js applications into executable binaries 📦</h3>
<p align="center">
<a href="https://github.com/leafac/caxa"><img src="https://img.shields.io/badge/Source---" alt="Source"></a>
<a href="https://www.npmjs.com/package/caxa"><img alt="Package" src="https://badge.fury.io/js/caxa.svg"></a>
<a href="https://github.com/leafac/caxa/actions"><img src="https://github.com/leafac/caxa/workflows/.github/workflows/main.yml/badge.svg" alt="Continuous Integration"></a>
</p>

### Support

- Recurring support on Patreon: <https://patreon.com/leafac>
- One-time support on PayPal: <https://paypal.me/LeandroFacchinetti>

### Why Package Node.js Applications into Executable Binaries?

- Simple deploys. Transfer the binary into a machine and run it.
- Let users test an application even if they don’t have Node.js installed.
- Simple installation story for command-line applications.
- It’s like the much-praised distribution story of [Go](https://golang.org) programs, but for Node.js.

### Features

- Works on Windows, macOS, and Linux.
- Simple to use. `npm install caxa` and call `caxa` from the command line. No need to declare which files to include; no need to bundle the application into a single file.
- Supports any kind of Node.js project, including those with native modules (for example, [sharp](https://npm.im/sharp), [@leafac/sqlite](https://npm.im/@leafac/sqlite) (shameless plug!), [and others](https://www.npmjs.com/package/windows-build-tools#examples-of-modules-supported)).
- Works with any Node.js version.
- Packages in seconds.
- Relatively small binaries. A “Hello World!” application is ~30MB, which is terrible if compared to Go’s ~2MB, and worse still if compared to C’s ~50KB, but best-in-class if compared to other packaging solutions for Node.js.
- Produces `.exe`s for Windows, simple binaries for macOS/Linux, and macOS Application Bundles (`.app`).
- Based on a simple but powerful idea. Written in ~200 lines of code.
- No magic. No traversal of `require()`s trying to find which files to include; no patches to Node.js source.

### Anti-Features

- Doesn’t patch the Node.js source code.
- Doesn’t build Node.js from source.
- Doesn’t support cross-compilation (for example, building a Windows executable from a macOS development machine).
- Doesn’t support packaging with a Node.js version different from the one that’s running caxa.
- Doesn’t hide your JavaScript source code in any way.

### Installation

```console
$ npm install --save-dev caxa
```

### Usage

#### Prepare the Project for Packaging

- Install any dependencies with `npm install` or `npm ci`.
- Build. For example, compile TypeScript with `tsc`, bundle with webpack, and whatever else you need to get the project ready to start. Typically this is the kind of thing that goes into an [npm `prepare` script](https://docs.npmjs.com/cli/v7/using-npm/scripts#prepare-and-prepublish), so the `npm ci` from the previous point may have taken care of this already.
- If there are files that shouldn’t be in the package, remove them from the directory. For example, you may wish to remove the `.git` directory.
- It’s recommended that you run this on a Continuous Integration server. (GitHub Actions, for example, does a shallow fetch of the repository, so removing the `.git` directory becomes superfluous.)

#### Call caxa

```console
$ npx caxa --help
Usage: caxa [options]

Options:
  -d, --directory <directory>               The directory to package.
  -c, --command <command-and-arguments...>  The command to run and optional arguments to pass to the command every time the executable is called. Paths must be absolute. The ‘{{caxa}}’ placeholder is substituted for the folder from which the package runs. The
                                            ‘node’ executable is available at ‘{{caxa}}/node_modules/.bin/node’. Use double quotes to delimit the command and each argument.
  -o, --output <output>                     The path at which to produce the executable. Overwrites existing files/folders. On Windows must end in ‘.exe’. On macOS may end in ‘.app’ to generate a macOS Application Bundle.
  -V, --version                             output the version number
  -h, --help                                display help for command

Examples:

  Windows:
  > caxa --directory "examples/echo-command-line-parameters" --command "{{caxa}}/node_modules/.bin/node" "{{caxa}}/index.js" "some" "embedded arguments" --output "echo-command-line-parameters.exe"

  macOS/Linux:
  $ caxa --directory "examples/echo-command-line-parameters" --command "{{caxa}}/node_modules/.bin/node" "{{caxa}}/index.js" "some" "embedded arguments" --output "echo-command-line-parameters"

  macOS (Application Bundle):
  $ caxa --directory "examples/echo-command-line-parameters" --command "{{caxa}}/node_modules/.bin/node" "{{caxa}}/index.js" "some" "embedded arguments" --output "Echo Command Line Parameters.app"
```

Here’s [a real-world example of using caxa](https://github.com/courselore/courselore/blob/master/.github/workflows/main.yml). This example includes packaging for Windows, macOS, and Linux; distributing tags with GitHub Releases Assets; distributing Insiders Builds for every push with GitHub Actions Artifacts; and deploying a binary to a server with `rsync` (and publishing an npm package as well, but that’s out of the scope).

#### Call caxa from TypeScript/JavaScript

For example:

```typescript
import caxa from "caxa";

(async () => {
  await caxa({
    directory: "examples/echo-command-line-parameters",
    command: [
      "{{caxa}}/node_modules/.bin/node",
      "{{caxa}}/index.js",
      "some",
      "embedded arguments",
    ],
    output: "echo-command-line-parameters",
  });
})();
```

You may need to inspect `process.platform` to determine in which operating system you’re running and come up with the appropriate parameters.

### Fine Points

#### Calling an Executable That Isn’t `node`

If you wish to run a command that isn’t `node`, for example, `ts-node`, you may do so by extending the `PATH`. For example, you may run the following on macOS/Linux:

```console
$ caxa --directory "examples/echo-command-line-parameters" --command "env" "PATH={{caxa}}/node_modules/.bin/:\$PATH" "ts-node" "{{caxa}}/index.js" "some" "embedded arguments" --output "echo-command-line-parameters"
```

#### Preserving the Executable Mode of the Binary

This is only an issue on macOS/Linux. In these operating systems a binary must have the executable mode enabled in order to run. Here’s what you may do when you distribute the binary to ensure that the file mode is preserved:

1. Create a tarball or zip. macOS/Linux (most distributions, anyway) come with software to uncompress tarballs and zips. The user can just double-click on the file and it’ll mostly work. You may generate a tarball with, for example, the following command:

   ```console
   $ tar -czf <caxa-output>.tgz <caxa-output>
   ```

   **Fun fact:** Windows 10 also comes with the `tar` executable, so the command above works on Windows as well. Unfortunately the File Explorer on Windows doesn’t support uncompressing the `.tgz` with a double-click (it supports uncompressing `.zip`, however). Fortunately, Windows doesn’t have issues with file modes to begin with (it simply looks for the `.exe` extension) so distributing the caxa output directly is appropriate.

2. Tell your users to run the following command after downloading:

   ```console
   $ chmod +x <caxa-output>
   ```

   In some contexts this may make more sense, but it requires your users to use the command line.

#### Detect Whether the Application Is Running from the Packaged Version

caxa doesn’t do anything special to your application, so there’s no intrinsic way of telling whether the application is running from the packaged version. It’s part of caxa’s ethos of being as out of the way as possible. Also, I consider it to be a bad practice: an application that is so self-aware is more difficult to test and reason about.

That, if you really need to know whether the application is running from the packaged versions, here are some possible workarounds in increasing levels of badness:

1. Set an environment variable in the `--command`, for example, `--command "env" "CAXA=true" "{{caxa}}/node_modules/.bin/node" "..."`.
2. Have a different entrypoint for the packaged application, for example, `--command "{{caxa}}/node_modules/.bin/node" "caxa-entrypoint.js"`.
3. Receive a command-line argument that you embed in the packaging process, for example, `--command "{{caxa}}/node_modules/.bin/node" "application.js" "--caxa"`.
4. Check whether `__dirname.startsWith(path.join(os.tmpdir(), "caxa"))`.

#### The Current Working Directory

Even though the code for the application is in a temporary directory, the current working directory when calling the packaged application is preserved, and you may inspect it with `process.cwd()`. This is probably not something you have to think about—caxa just gets it right.

### How It Works

#### The Issue

As far as I can understand, the root of the problem with creating binaries for Node.js projects is native modules. Native modules are libraries written at least partly in C/C++, for example, [sharp](https://npm.im/sharp) and [@leafac/sqlite](https://npm.im/@leafac/sqlite) (shameless plug!). There are at least three issues with native modules that are relevant here:

1. You must have a working C/C++ build system to install these libraries (C/C++ compiler, `make`, Python, and so forth). On Windows, you must install [windows-build-tools](https://npm.im/windows-build-tools). On macOS, you must install the Command-Line Tools (CLT) with `xcode-select --install`. On Linux, it depends on the distribution, but on Ubuntu `sudo apt install build-essential` is enough.

2. The installation of native modules isn’t cross-platform. Unlike JavaScript dependencies, which you may copy from an operating system to another, native modules produce compiled C/C++ code that’s specific to the operating system on which the dependency was installed. This compiled code appears in your `node_modules` directory in the form of `.node` files.

3. As far as I understand, **Node.js insists on loading native modules from files in the disk**. Other Node.js packaging solutions get around this limitation in one of two ways: They either patch Node.js to trick it into loading native modules differently; or they put `.node` files somewhere before starting your program.

#### The Solution

caxa builds on the idea of putting `.node` files in a temporary location, but takes it to ultimate consequence: a caxa executable is a form of [self-extracting archive](https://en.wikipedia.org/wiki/Self-extracting_archive) containing your whole project along with the `node` executable. When you first run a binary produced by caxa, it **extracts the source the whole project (and the bundled `node` executable) into a temporary location**. From there, it simply calls whatever command you told it to run when you packaged the project (via the `--command` command-line argument).

At first, this may seem too costly, but in practice it’s mostly okay: It doesn’t take too long to uncompress a project in the first place, and caxa doesn’t clean the temporary directory after running your program, so subsequent calls are effectively cached and run without overhead.

This idea is simple, but it’s super powerful! caxa supports any kind of project, including those with native dependencies, because running a caxa executable amounts to the same as installing Node.js on the user’s machine. caxa compiles binaries fast, because generating a self-extracting archive is a simple matter of concatenating some files. caxa supports any version of Node.js, because it simply copies the `node` executable with which it was called into the self-extracting archive.

**Fun fact:** By virtue of compressing the archive, caxa produces naturally smaller binaries in comparison to other packaging solutions. Obviously, you could achieve the same effect by compressing the output of these other tools.

#### How the Self-Extracting Archive Works

Did you know that you may append anything to a binary and it’ll continue to work? This is true of binaries for Windows, macOS, and Linux. Here’s an example to try out on macOS/Linux:

```console
$ cp $(which ls) ./ls  # Copy the ‘ls’ binary into the current directory to play with it
$ ./ls                 # List the files, proving the that the binary works
$ echo ANYTHING >> ls  # Append material to the binary
$ tail ./ls            # You should see ‘ANYTHING’ at the end of the output
$ ./ls                 # The output should be same as before!
$ rm ls                # Okay, the test is over
```

The caxa self-extracting archives work by putting together three parts: 1. a stub; 2. an archive; and 3. a footer. This is the layout of these parts in the binary produced by caxa:

```
STUB
### CAXA ###
ARCHIVE
FOOTER
```

The `STUB` and the `ARCHIVE` are separated by the `### CAXA ###` string. And the `ARCHIVE` and the `FOOTER` are separated by a newline. This layout allows caxa to find the footer simply by looking backward from the end of the file until reaching a newline. And if this is the first time you’re running the caxa executable and the archive needs to be uncompressed, then caxa may find the beginning of the `ARCHIVE` by looking for the `### CAXA ###` separator forward from the beginning.

Build a binary with caxa and inspect it yourself in a text editor (Visual Studio Code asks you to confirm that you want to open a binary as text, but it works fine). You should be able to find the `### CAXA ###` separator between the `STUB` and the `ARCHIVE`, as well as the `FOOTER` at the end.

Let’s examine each of the parts in detail:

**Part 1: Stub**

This is a program written in Go that, when executed, reads itself as a file, finds the archive, extracts it, finds the footer, and runs whatever command it finds there. You may find the source code for the stub in `stubs/stub.go`, and the compiled stubs live in `stubs`. The stubs are distributed with caxa in compiled form so you don’t need a Go build system to use caxa. You may build the stubs yourself with `npm run stubs`. This Go program has no dependencies beyond the Go standard library, so simply installing Go is enough, there’s no need to setup Go modules or configure a `$GOPATH`.

This is beautiful in a way: We’re using Go’s ability to produce binaries to bootstrap Node.js’s new ability to produce binaries.

**Part 2: Archive**

This is a tarball of the directory with your project.

**Part 3: Footer**

This is JSON containing the extra information that caxa needs to run your project. Most importantly, the command that you want to run, but also an identifier for where to uncompress the archive.

#### Using the Self-Extracting Archive without caxa

**Fun fact:** There’s nothing Node.js-specific about the stubs. You may use them to uncompress any kind of archive and run any arbitrary command on the output! And it’s relatively straightforward to build a self-extracting archive from scratch. For example, you may run the following in macOS:

```console
$ cp stubs/macos an-ls-caxa
$ tar -czf - README.md >> an-ls-caxa
$ printf "\n{ \"identifier\": \"an-ls-caxa/AN-ARBITRARY-STRING-THATS-DIFFERENT-EVERY-TIME\", \"command\": [\"ls\", \"{{caxa}}\"] }" >> an-ls-caxa
$ ./an-ls-caxa
README.md
```

#### To Where Are the Packages Uncompressed at Runtime?

It depends on the operating system. You may find the location on your system with:

```console
$ node -p "require(\"os\").tmpdir()"
```

Look for a directory named `caxa` in there.

#### Why No Cross-Compilation? Why No Different Versions of Node.js besides the Version with Which caxa Was Called?

Two reasons:

1. I believe you should have environments to work with all the operating systems you plan on supporting. They may not be your main development environment, but they should be able to build your project and let you test things. At the very least, you should use a service like GitHub Actions which let you run building tasks and tests on Windows, macOS, and Linux.

   (I, for one, bought a PC to work on caxa. Yet another reason to [support my work](#support)!)

2. The principle of least surprise. When cross-compiling (for example, building a Windows executable from a macOS development machine), or when bundling different versions of Node.js (for example, bundling Node.js 15 while running caxa with Node.js 14), there’s no straightforward way to guarantee that the packaged project will run the same as the unpackaged version. If you aren’t using any native modules then things **may** work, but as soon as you introduce a new dependency that you didn’t know was native your packages may break. Not only are native dependencies different on the operating systems, but they may also be different between different versions of Node.js if these versions aren’t ABI-compatible (which is why sometimes when you update Node.js you must run `npm install` again).

**Fun fact:** The gold-standard for easy cross-compilation these days is Go. But even in Go cross-compilation goes out the window as soon as you introduce C dependencies (something called CGO). It appears that many people in the Go community try to solve the issue by avoiding CGO dependencies, sometimes going to great lengths to reinvent everything in pure Go. On the one hand, this sounds like fun when it works out. On the other hand, it’s a huge case of not-invented-here syndrome. In any case, native modules are much more prevalent in Node.js than CGO is in Go, so I think that cross-compilation in caxa would be a fool’s errand.

If you still insist on cross-compiling or compiling for different versions of Node.js, you can still use the stub to build a self-extracting archive by hand (see [§ Using the Self-Extracting Archive without caxa](#using-the-self-extracting-archive-without-caxa)). You may even use <https://www.npmjs.com/package/node> to more easily bundle different versions of Node.js.

### Features to Consider Implementing in the Future

If you’re interested in one of these features, please send a Pull Request if you can, or at least reach out to me and mention your interest, and I may get to them.

1. Other compression algorithms. Currently caxa uses tarballs, which are ubiquitous and reasonably efficient in terms of compression/uncompression times and archive size. But there are better algorithms out there… (See <https://github.com/leafac/caxa/issues/1>.)

2. Add support for signing the executables. There are limitations on the kinds of executables that are signable, and a self-extracting archive of the kind that caxa produces may be unsignable (I know very little about this…). A solution could be use Go’s support for embedding data in the binary (which landed in Go 1.16). Of course this would require the person packaging a project to have a working Go build system.

3. Add support for custom icons and other package metadata. This should be relatively straightforward by using [rcedit](https://github.com/electron/rcedit) for `.exe`s and by adding `.plist` files to `.app`s (we may copy whatever Electron is doing here as well).

### Prior Art

#### [Deno](https://deno.land/manual@v1.8.0/tools/compiler)

Deno has experimental support for producing binaries. I haven’t tried it myself, but maybe one day it catches on and caxa becomes obsolete. Let’s hope for that!

#### <https://github.com/vercel/pkg>

pkg is great, and it’s where I first learned that you could think about compiling Node.js projects this way. It’s [the most popular packaging solution for Node.js by a long shot](https://www.npmtrends.com/pkg-vs-boxednode-vs-js2bin-vs-nexe-vs-enclose-vs-nar).

It works by patching the Node.js executable with a proxy around [`fs`](https://github.com/vercel/pkg/blob/7a9257ac91efaddc90c558173af1a6bec5da6cdd/prelude/bootstrap.js#L378). This proxy adds the ability to look into something called a **snapshot** file system, which is where your project is stored. Also, it doesn’t store your source JavaScript directly. It runs your JavaScript through the V8 compiler and produces a V8 snapshot, which has two nice consequences: 1. Your code will start marginally faster, because all the work of parsing the JavaScript source and so forth is already done; and 2. Your code doesn’t live in the clear in the binary, which may be advantageous if you want to hide it.

Unfortunately, this approach has a few issues:

1. The Node.js patches must be kept up-to-date. For example, when `fs/promises` became a thing, the `fs` proxy broke. It was a subtle and surprising issue that only arises in the packaged version of the application. (See my soon-to-be-archived-now-that-caxa-is-out fork of pkg, [@leafac/pkg](https://github.com/leafac/pkg) for a workaround, as well as the issues linked there for more information).

2. The patched Node.js distributions must be updated with each new Node.js release. At the time of this writing they’re [lagging behind by half an year](https://github.com/vercel/pkg-fetch/releases) (v14.4.0, while the latest LTS is v14.16.0). That’s new features and security updates you may not be getting. (See https://github.com/yao-pkg/pkg-binaries for a seemingly abandoned attempt at automating the patching process that could improve on this situation. Of course, manual intervention would still be required every time the patches become incompatible with Node.js upstream.)

3. Native modules work [by the way of a self-extracting archive](https://github.com/vercel/pkg/tree/7a9257ac91efaddc90c558173af1a6bec5da6cdd#native-addons).

Also, pkg tries to prune your directory to try and find only the files that matter. This is good if you want to optimize for small binaries with little effort. But often this process goes wrong, specially when something like TypeScript produces JavaScript that throws off pkg’s heuristics. In that case you have to intervene and list the files that should be included by hand.

Not to mention that the maintainers of pkg haven’t been super responsive this past year. (And who can blame them? Open-source is **hard**. No shade thrown here; pkg is **awesome**! And speaking of “open-source is hard,” [support my work](#support)!)

#### <https://github.com/nexe/nexe>

The second most popular packaging solution in Node.js, right behind pkg. nexe works by a similar strategy, and suffers from some of the same issues. But `fs/promises` work, [newer Node.js versions are available](https://github.com/nexe/nexe/releases/tag/v3.3.3), and the project seems to be more actively maintained.

Native modules don’t work, but there’s a workaround based on the self-extracting archive idea: https://github.com/nmarus/nexe-natives

#### <https://github.com/mongodb-js/boxednode>

This works with a different strategy. Node.js has a bunch of standard libraries written in JavaScript itself, and when Node.js is built, this JavaScript ends up embedded as part of the `node` executable. boxednode works by recompiling Node.js from source with your project embedded as if it were just another standard library. On the upside, this supports native extensions and whatever new `fs/promises` situation comes up in the future. The down side is that compiling Node.js takes hours (the first time, and still a couple minutes after the subsequent times) and 10+GB of disk(!) Also, boxednode only works with a single JavaScript file, so you must bundle with something like ncc or webpack before packaging. And I don’t think it could handle assets like images along with the code (think of packaging a web application).

#### <https://github.com/pmq20/node-packer>

This works with an idea of a **snapshot** file system (à la pkg), but it uses a more principled approach for that, something called Squashfs. To the best of my knowledge the native-extensions story in node-packer is the same self-extracting archive from most packaging solutions. The downside of node-packer is that installing and setting it up is a bit more involved than a simple `npm install`. For that reason I ended up not really giving it a try, so I’ll say no further…

#### <https://github.com/criblio/js2bin>

This should work with a strategy similar to boxednode, but with a pre-compiled binary including some pre-allocated space to save you from having to compile Node.js from source. Like boxednode, it also works with only a single JavaScript file at a time, requiring a bundler like ncc or webpack. I tried js2bin out and it produced binaries that didn’t work at all. I have no idea why…

#### <http://enclosejs.com>

The predecessor of pkg. Worked with the same idea. I believe it has been deprecated in favor of pkg. To the best of my knowledge it was closed source and paid.

#### <https://github.com/h2non/nar>

This is the project that gave me the idea for caxa! It’s more obscure, so at first I gave it little attention in my investigation. But then it handled native extensions and the latest Node.js versions out-of-the-box despite haven’t been updated in 4 years! I was intrigued!

In principle, it works the same as caxa, using the idea of a self-extracting archive. There are some important differences, though:

1. nar doesn’t support Windows. That’s because nar’s stub is a bash script instead of the Go binary that we use in caxa.
2. nar gets some small details wrong. For example, it changes your current working directory to the temporary directory in which the archive is uncompressed. This breaks some assumptions about how command-line tools should work; for example, if you’re project implements `ls` in Node.js, then when running it from nar it’d always list the files in the temporary directory.
3. It’s no longer maintained. They recommend pkg instead.
4. It was written in LiveScript, which is significantly more obscure than TypeScript/Go, in which caxa is implemented.

#### <https://github.com/jedi4ever/bashpack>

Similar to nar. Hasn’t seen activity in 8 years.

#### Other Packages

If you dig through npm, GitHub, and Google, you’ll find other projects in this space, but I couldn’t find one that had a good combination of working well, being well documented, being well maintained, and so forth.

#### References About Self-Extracting Archives

Creating a self-extracting archive with a bash script for the stub (only works on macOS/Linux, and depends on things like `tar` being available—which they probably are):

- <https://peter-west.uk/blog/2019/making-node-script-binaries.html>: This is specific to Node.js applications. It’s similar in spirit to nar and bashpack.
- <https://github.com/megastep/makeself>
- <https://www.linuxjournal.com/node/1005818>
- <https://community.linuxmint.com/tutorial/view/1998>
- <http://alexradzin.blogspot.com/2015/12/creating-self-extracting-targz.html?m=1>
- <https://www.matteomattei.com/create-self-contained-installer-in-bash-that-extracts-archives-and-perform-actitions/>
- <https://www.codeproject.com/Articles/7053/Pure-WIN32-Self-Extract-EXE-Builder>
- <https://sysplay.in/blog/linux/2019/12/self-extracting-shell-script/>
- <https://gist.github.com/gregjhogan/bfcffe88ac9d6865efc5>

Creating a self-extracting batch file for Windows (an idea I didn’t pursue, going for the Go stub instead):

- <https://www.dostips.com/forum/viewtopic.php?f=3&t=4842>
- <http://www.piclist.com/techref//dos/binbat.htm>

Other tools that create self-extracting archives:

- 7-Zip. It was studying 7-Zip that I learned that you can append data to a binary, and the concept of how to create a self-extracting archive sunk in. Unfortunately I couldn’t use 7-Zip itself because it’s tailored for **installers** as opposed to **applications**, so some assumptions break.
  - <https://sevenzip.osdn.jp/chm/start.htm>
  - <https://sevenzip.osdn.jp/chm/cmdline/switches/sfx.htm>
  - <https://superuser.com/questions/42788/is-it-possible-to-execute-a-file-after-extraction-from-a-7-zip-self-extracting-a>
  - <http://ntsblog.homedev.com.au/index.php/2015/05/14/self-extracting-archive-runs-setup-exe-7zip-sfx-switch/>
  - <https://nsis.sourceforge.io/Main_Page>
  - <https://www.7-zip.org/sdk.html>
  - <https://superuser.com/questions/1048866/creating-7zip-sfx-installer>
  - <https://netbeansscribbles.wordpress.com/2015/01/30/creating-a-self-extracting-7zip-archive-multi-platform/>
  - If 7-Zip had worked, I’d try using one of the following packages to control 7-Zip from Node.js:
    - <https://www.npmjs.com/package/node-7z-archive>
    - <https://www.npmjs.com/package/7zip-bin>
    - <https://www.npmjs.com/package/7zip-bin-wrapper>
    - <https://www.npmjs.com/package/p7zip>
    - <https://www.npmjs.com/package/7zip>
    - <https://www.npmjs.com/package/node-7z>
    - <https://www.npmjs.com/package/7zip-min>
- WinZip Self-Extractor: <https://www.winzip.com/> (I didn’t try this.)
- <https://documentation.help/WinRAR/HELPArcSFX.htm>
- <https://en.wikipedia.org/wiki/IExpress> (I tried this and it didn’t work. Also I think it’d be difficult to automate. And it’s old technology.)

#### References about Building the Stub in C

Besides Go, I also considered writing the stub in C. Ultimately Go won because it’s less prone to errors and has a better cross-compilation/standard-library story. But C has the advantage of being setup in the machines of Node.js developers because of native dependencies. Anyway, here’s what you could use to build a stub in C:

- Just use `system()` and rely on `tar` being installed (it most certainly is, anyway): <https://www.tutorialspoint.com/c_standard_library/c_function_system.htm>
- <http://www.libarchive.org>: This is the implementation of `tar` that you find in Windows 10 and many other places.
  - <https://github.com/libarchive/libarchive/wiki/LibarchiveFormats>
- <https://zlib.net>: This is the underlying implementation of gzip that [may be the most deployed software library in existence](https://www.sqlite.org/mostdeployed.html). It’s lower level than libarchive.
- <https://bitbucket.org/proxymlr/bsdtar/src/xtar/contrib/xtar.c>
- <https://opensource.apple.com/source/libarchive/libarchive-29/libarchive/examples/untar.c.auto.html>
- <https://github.com/calccrypto/tar>
- <https://repo.or.cz/libtar.git>

#### References about Creating Self-Extracting Archives in Node.js

- <https://github.com/vk-twiconnect/sfx-creator-service>
- <https://stackoverflow.com/questions/27904532/how-do-i-make-a-self-extract-and-running-installer>
- <https://www.npmjs.com/package/sfxbundler>
  - <https://github.com/touchifyapp/sfx>

#### References about the Structure of Executables

A more principled way of building the self-extracting archive is to not append data at the end of the file, but manipulate the stub binary as a data structure. It’s actually three data structures: Portable Executables (Windows), Mach-O (macOS), and ELF (Linux). This idea was abandoned because it’s more work for the packager and for the stub—the `### CAXA ###` separator is a hack that works well enough. But we may have to revisit this to make the executables signable. You can even manipulate binaries with Go standard libraries.

Anyway, here are some references on the subject:

- <https://github.com/AlexanderOMara/portable-executable-signature>
- <https://github.com/anders-liu/pe-struct>
- <https://github.com/ironSource/portable-executable>
- <https://github.com/bennyhat/peid-finder>
- <https://github.com/lief-project/LIEF>

#### References about Just Appending Data to an Executable Works

The data that you append is sometimes called an **overlay**.

- <https://stackoverflow.com/questions/5795446/appending-data-to-an-exe>
- <https://blog.barthe.ph/2009/02/22/change-signed-executable/>
- <https://edn.embarcadero.com/article/27979>
- <https://security.stackexchange.com/questions/77336/how-is-the-file-overlay-read-by-an-exe-virus>
- <https://github.com/jason-klein/signed-nsis-exe-append-payload>
- <https://stackoverflow.com/questions/5316152/store-data-in-executable>

#### References About Cross-Compilation of CGO

- <https://github.com/karalabe/xgo>
- <https://github.com/mattn/go-sqlite3>: A popular project using CGO.

#### References on Building macOS Application Bundles (`.app`)

- <https://mathiasbynens.be/notes/shell-script-mac-apps>
- <http://www.wsanchez.net/software/>
- <https://sveinbjorn.org/platypus>
- <https://github.com/subtleGradient/tilde-bin/blob/master/appify>
- <https://github.com/subtleGradient/Appify-UI>
- <https://stackoverflow.com/questions/281372/executing-shell-scripts-from-the-os-x-dock>
- <https://gist.github.com/mathiasbynens/674099>
- <https://github.com/KosalaHerath/macos-installer-builder>
- macOS’s Automator can generate `.app`.
- <https://www.electronjs.org/docs/tutorial/application-distribution>

#### References about How to Untar in Go

The Go standard library has low-level utilities for handling tarballs. I could have used a higher-level library, but I couldn’t get them to work with an archive that’s in memory (having been extracted from the binary). Besides, relying only on the standard library is good for an easy compilation story. In the end, the solution was to copy and paste a bunch.

- <https://github.com/golang/build/blob/db2c93053bcd6b944723c262828c90af91b0477a/internal/untar/untar.go>
- <https://github.com/mholt/archiver/tree/v3.5.0>
- <https://github.com/mholt/archiver/blob/v1.1.2/targz.go>
- <https://pkg.go.dev/golang.org/x/build/internal/untar> / <https://github.com/k3s-io/k3s/blob/v1.0.1/pkg/untar/untar.go> / <https://pkg.go.dev/github.com/rancher/k3s/pkg/untar>
- <https://github.com/cloudfoundry/archiver/blob/master/extractor/tgz_extractor.go>
- <https://gist.github.com/mimoo/25fc9716e0f1353791f5908f94d6e726>
- <https://stackoverflow.com/questions/57639648/how-to-decompress-tar-gz-file-in-go>
- <https://gist.github.com/indraniel/1a91458984179ab4cf80>
- <https://medium.com/@skdomino/taring-untaring-files-in-go-6b07cf56bc07>
- <https://medium.com/learning-the-go-programming-language/working-with-compressed-tar-files-in-go-e6fe9ce4f51d>
- <https://github.com/codeclysm/extract>

#### References about How to Execute a Command from Go

It’d have been nice to use `syscall.Exec()`, which replaces the currently running binary (the stub) with another one (the command you want to run for your application), but `syscall.Exec()` is macOS/Linux-only. So we use `os.Exec()` instead, paying attention to wiring `stdin/stdout/stderr`, and forwarding the command-line arguments on the way and the status code on the way out. The downside is that there’s an extra process in the process tree.

- <https://github.com/golang/go/issues/30662>
- <https://golang.org/pkg/os/exec/>
- <https://golang.org/pkg/syscall/#Exec>
- <https://pkg.go.dev/golang.org/x/sys>
- <https://pkg.go.dev/golang.org/x/sys@v0.0.0-20210226181700-f36f78243c0c/unix#Exec>
- <https://pkg.go.dev/golang.org/x/sys@v0.0.0-20210226181700-f36f78243c0c/windows/mkwinsyscall>
- <https://stackoverflow.com/questions/10385551/get-exit-code-go>

#### References about the Layout of the Data in the Self-Extracting Archive

- <https://stackoverflow.com/questions/1443158/binary-data-in-json-string-something-better-than-base64>: I played with the idea of including the `ARCHIVE` in the JSON that ended up becoming the `FOOTER`. It’d have been simpler conceptually, but embedding binary data in JSON has an overhead.
- Maybe I could have used multipart/form-data, which is a standard way of interleaving binary data and text. But the layout was so simple that I decided against it. Being able to generate a binary by appending stuff on the command line is handy and cute.
- <https://github.com/electron/asar>
