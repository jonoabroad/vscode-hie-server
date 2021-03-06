# Haskell Language Server Client

Client interface to the Language Server Protocol server for Haskell, as provided by the Haskell IDE Engine. Check the [requirements](#user-content-requirements) for dependencies.

**It is still under development!** If you want to help, get started by reading [Contributing](https://github.com/alanz/vscode-hie-server/blob/master/Contributing.md) for more details.

## Requirements

The language client requires you to manually install the [HIE](https://github.com/haskell/haskell-ide-engine) language server,

```bash
$ git clone https://github.com/haskell/haskell-ide-engine --recursive
$ cd haskell-ide-engine && make build
```

Alternatively you can just `stack install`, but `make build` will give you the best setup.

## Features

Language server client for haskell using the [HIE](https://github.com/haskell/haskell-ide-engine) language server. Supports,

* Diagnostics via HLint and GHC warnings/errors
* Code actions and quick-fixes via [`apply-refact`](https://github.com/mpickering/apply-refact) (click the lightbulb)
* Type information and documentation (via hoogle) on hover
* Jump to definition (`F12` or `Go to Definition` in command palette)
* List all top level definitions
* Highlight references in document
* Completion
* Formatting via [`brittany`](https://github.com/lspitzner/brittany) (`^ ⌥ B` or `Format Document` in command palette)
* Renaming via [`HaRe`](https://github.com/alanz/HaRe) (`F2` or `Rename Symbol` in command palette)
* [Multi-root workspace](https://code.visualstudio.com/docs/editor/multi-root-workspaces) support

Additionally the language server itself features,

* Supports plain GHC projects, cabal projects (sandboxed and non sandboxed) and stack projects
* Fast due to caching of compile info

## Extension Settings

You can disable HLint and also control the maximum number of reported problems,

```json
"languageServerHaskell.hlintOn": true,
"languageServerHaskell.maxNumberOfProblems": 100,
```

#### HIE Wrapper

Furthermore, the extension supports multiple ways of initializing hie, depending on your needs. The first one is to use the hie-wrapper that follows this extension, and tries to pick the right hie for your GHC version. The following,

```json
"languageServerHaskell.useHieWrapper": true,
```

makes VSCode use the `hie-wrapper.sh` file to start hie through. This does assume that you built the hie executable using make build, but will fall back to plain hie. This will take precedence over `hieExecutablePath`.

#### Custom Wrapper

If you need more control, and want to have a custom wrapper, either in your specific project or somewhere else on your computer, you can set a custom wrapper via,

```json
"languageServerHaskell.useCustomHieWrapper": true,
"languageServerHaskell.useCustomHieWrapperPath": "~/wrapper-in-home.sh",
```

There are a few placeholders which will be expanded:

* `~`, `${HOME}` and `${home}` will be expanded into your users' home folder.
* `${workspaceFolder}` and `${workspaceRoot}` will expand into your current project root.

This can be beneficial if you are using something like nix, to have a wrapper script tailored to your setup. This will take precedence over `useHieWrapper` and `hieExecutablePath`.

#### Enable/disable HIE

You can enable or disable HIE via configuration. This is useful, because multi-root workspaces do not yet allow you to manage extensions at the folder level, which can be necessary.

```json
"languageServerHaskell.enableHIE": true
```

#### Path for hie executable

If you `hie` executable is not on your path, you can manually set it,

```json
"languageServerHaskell.hieExecutablePath": "~/.local/bin/hie"
```

The path placeholders work here as well. Note that this adds the `--lsp` argument to the call of this executable.

## Docs on Hover/Generating Hoogle DB

For the most current documentation on this, see [Docs on Hover/Completion](https://github.com/haskell/haskell-ide-engine#docs-on-hovercompletion).

HIE supports fetching docs from haddock on hover. It will fallback on using a hoogle db(generally located in ~/.hoogle on linux)
if no haddock documentation is found.

To generate haddock documentation for stack projects:

```bash
$ cd your-project-directory
$ stack haddock --keep-going
```

To enable documentation generation for cabal projects, add the following to your ~/.cabal/config

```
documentation: True
```

To generate a hoogle database that hie can use

```bash
$ cd haskell-ide-engine
$ stack --stack-yaml=<stack.yaml you used to build hie> exec hoogle generate
```

## Manual Installation

Either install the extension via the marketplace (preferred), or if you are testing an unreleased version by,

```bash
$ npm install -g vsce
$ git clone https://github.com/alanz/vscode-hie-server
$ cd vscode-hie-server
$ npm install
$ vsce package
```

This will create a file something like `vscode-hie-server-<version>.vsix`
according to the current version.

In VS Code, open the extensions tab, and click on the `...` at the top right of it,
and use the `Install from VSIX...` option to locate and install the generated file.

## Using multi-root workspaces

First, check out [what multi-root workspaces](https://code.visualstudio.com/docs/editor/multi-root-workspaces) are. The idea of using multi-root workspaces, is to be able to work on several different Haskell projects, where the GHC version or stackage LTS could differ, and have it work smoothly.

HIE is now started for each workspace folder you have in your multi-root workspace, and several configurations are on a resource (i.e. folder) scope, instead of window (i.e. global) scope.

To showcase the utility of this, let's imagine that we are writing a full-stack Haskell website, and we have three main components:

* a backend using LTS 10.10 and
* a frontend using GHCJS and LTS 8.11,
* a common part that is shared between the two, using LTS 8.11.

One way to be able to work on all these in the same VSCode project is to create a new workspace, and then add each folder to the workspace. This way, they are considered separate entities, and HIE will start separately for each.

You can then define how to start HIE for each of these folders, by going into `Settings` and then choosing `Folder Settings -> <the folder you want>`, or just configure the workspace. E.g. the _backend_ and _common_ projects could have a folder settting,

```json
{
  "languageServerHaskell.useCustomHieWrapper": true,
  "languageServerHaskell.useCustomHieWrapperPath": "${workspaceFolder}/hie.sh"
}
```

to launch HIE via `hie.sh` inside the _backend_ and _common_ folder, while the _frontend_, because of using GHCJS, might not want to use HIE, and therefore needs to disable HIE,

```json
{
  "languageServerHaskell.useHieWrapper": false
}
```

This provides a very flexible way of customizing your setup.

## Release Notes

See the [Changelog](https://github.com/alanz/vscode-hie-server/blob/master/Changelog.md) for more details.
