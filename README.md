# Project template for the projects based on reflex-frp

[reflex-frp](https://reflex-frp.org) is a functional reactive programming framework for Haskell.

## prerequisites

- see reflex-platform notes on [os-compatibility](https://github.com/reflex-frp/reflex-platform#os-compatibility)
  - also, it works on Ubuntu and should work on any Linux
- for OS except [NixOS](https://nixos.org/nixos/) you'll need [Nix](https://nixos.org/nix/manual/#ch-installing-binary) package manager
  - single user installation is the simplest thing that will work, but for experience closer to NixOS you may want to use multi-user installation

If you use NixOS, you may want to add [binary cache](https://github.com/reflex-frp/reflex-platform/blob/develop/notes/NixOS.md); for any other OS the same can be done for Nix package manager by adding to `/etc/nix/nix.conf` the following:

```
substituters = https://cache.nixos.org https://nixcache.reflex-frp.org
trusted-public-keys = cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY= ryantrinkle.com-1:JJiAKaRv9mWgpVAz8dwewnZe0AzzEAzPkagE9SP5NWI=
binary-caches = https://cache.nixos.org https://nixcache.reflex-frp.org
binary-cache-public-keys = cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY= ryantrinkle.com-1:JJiAKaRv9mWgpVAz8dwewnZe0AzzEAzPkagE9SP5NWI=
binary-caches-parallel-connections = 40
```

## reflex docs

- [An introduction to reflex](https://qfpl.io/posts/reflex/basics/introduction/) by Queensland FP Lab
- [reflex-dom-inbits](https://github.com/hansroland/reflex-dom-inbits)
- [Reflex docs](http://docs.reflex-frp.org/en/latest/reflex_docs.html)
- [Reflex Quick(ish) Reference](https://github.com/reflex-frp/reflex/blob/develop/Quickref.md)
- [Reflex-Dom Quick(ish) Reference](https://github.com/reflex-frp/reflex-dom/blob/develop/Quickref.md)

## start

- clone template to desired dir and cd there
- `rm -rf .git && git init`
- update:
  - [LICENSE](LICENSE) (at least, replace `<name>`)
  - [frontend/LICENSE](frontend/LICENSE) (at least, replace `<name>`)
  - [frontend.cabal](frontend/frontend.cabal) (replace `<name>`, `<email>`)
- optional:
  - [update](#update-reflex-platform) `reflex-platform`
  - if you don't use Visual Studio Code: `rm -rf .vscode`
  - rename `frontend` package:
    - in [frontend.cabal](frontend/frontend.cabal) change `name` and `executable` as desired
    - in [bin/open-file](bin/open-file) change `frontend.jsexe` to proper name
    - in [bin/run](bin/open-file) change `frontend` file name to your executable name
  - recreate `frontend` package completely:
    - `bin/shell` (since you'll need `cabal` to create package)
    - `rm -rf frontend`
    - [add new package](#add-package)

## bin/

- `build` = `nix-build -o result/frontend -A ghcjs.frontend`
- `build-native` = `nix-build -o result/frontend-native -A ghc.frontend --arg useWarp false`
  - for warp-based version remove `--arg useWarp false` piece
- `ghcid` = `nix-shell -A shells.ghc --run 'ghcid -W -c "cabal new-repl frontend" -T Main.main'`
- `hoogle` = `nix-shell -A shells.ghc --run "hoogle server --local --port=8080" --arg withHoogle true`
- `open` = `xdg-open http://localhost:3003`
- `open-file` = ``xdg-open file://`pwd`/result/frontend/bin/frontend.jsexe/index.html``
- `open-hoogle` = `xdg-open http://localhost:8080`
- `repl` = `cabal new-repl frontend` (run under `bin/shell`)
- `run` = `result/frontend-native/bin/frontend`
- `shell` = `nix-shell -A shells.ghc`
- `shelljs` = `nix-shell -A shells.ghcjs`

## dev

- open terminal for each of:
  - `bin/hoogle`
  - `bin/ghcid`
  - `bin/shell`
  - `bin/shelljs`
- run `bin/open` to open app running with warp
  - `bin/ghcid` is one of the ways to start app with warp
- run your favorite editor under `bin/shell` (e.g. `code .`)
- optional:
  - run `bin/repl` under `bin/shell`

## build

- run `bin/build`
- see result in `result/frontend/bin/frontend.jsexe`
  - use `bin/open-file` to open it

native:

- run `bin/build-native`
- see result in `result/frontend-native/bin`
  - use `bin/run` to run it

## update reflex-platform

- go to [reflex-platform](https://github.com/reflex-frp/reflex-platform) and find commit you want
- `nix-prefetch-git https://github.com/reflex-frp/reflex-platform.git` _commit_
- use `rev` and `sha256` from output to update [nix/reflex-platform.nix](nix/reflex-platform.nix)

## add dependency

- [Adding some dependencies](https://cah6.github.io/technology/nix-haskell-3/#adding-some-dependencies)

## add package

- `bin/shell` (if not under it already)
- `mkdir` _package_
- `cd` _package_
- `cabal init`
- add `reflex-dom` to cabal file `build-depends` section if needed
- add _package_ to [cabal.project](cabal.project)
- add _package_ to [default.nix](default.nix) sections `packages` and `shells`
- add _package_ src dir to [.vscode/settings.json](.vscode/settings.json) under `ghcSimple.startupCommands.custom` item that starts with `:set -i`
  - paths are separated by `:` like `-ifrontend/src:somepkg/src`

## credits

This template is based on:

- [Exploring Nix & Haskell Part 3: Less Nix, More Reflex](https://cah6.github.io/technology/nix-haskell-3/) article
  - the code is in [haskell-nix-skeleton-2](https://github.com/cah6/haskell-nix-skeleton-2)
    - which is based on [reflex-project-skeleton](https://github.com/ElvishJerricco/reflex-project-skeleton)
      - which uses [reflex-platform](https://github.com/reflex-frp/reflex-platform)

## license

License is MIT
