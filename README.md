# Project template for the projects based on reflex-frp

[reflex-frp](https://reflex-frp.org) is a functional reactive programming framework for Haskell.

## prerequisites

- see reflex-platform notes on [os-compatibility](https://github.com/reflex-frp/reflex-platform#os-compatibility)
  - also, it works on Ubuntu and should work on any Linux
- for any OS except [NixOS](https://nixos.org/nixos/) you'll need [Nix](https://nixos.org/nix/manual/#ch-installing-binary) package manager
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
  - [bin/open-page-cabal](bin/open-page-cabal) and [bin/run-cabal](bin/run-cabal) (fix paths)
    - TODO: is there a way to detect these paths from shell automatically?
- optional:
  - configure tools:
    - `stylish-haskell -d > .stylish-haskell.yaml` and update the file
    - `hlint -d > .hlint.yaml` and update the file
  - [update](#update-reflex-platform) `reflex-platform`
  - if you don't use Visual Studio Code: `rm -rf .vscode`
  - rename `frontend` package:
    - in [frontend.cabal](frontend/frontend.cabal) change `name` and `executable` as desired
    - in [bin/open-page](bin/open-page) change `frontend.jsexe` to proper name
    - in [bin/run](bin/run) change `frontend` file name to your executable name
    - in [bin/open-page-cabal](bin/open-page-cabal) and [bin/run-cabal](bin/run-cabal) fix paths
  - recreate `frontend` package completely:
    - `bin/shell` (since you'll need `cabal` to create package)
    - `rm -rf frontend`
    - [add new package](#add-package)

## bin/

- `build` = `nix-build -o result/frontend -A ghcjs.frontend`
- `build-cabal` = `nix-shell -A shells.ghcjs --run "cabal --project-file=cabal-ghcjs.project --builddir=dist-ghcjs new-build all"`
- `build-native` = `nix-build -o result/frontend-native -A ghc.frontend --arg useWarp false`
  - for warp-based version remove `--arg useWarp false` piece
- `build-native-cabal` = `nix-shell -A shells.ghc --run "cabal new-build all"`
- `clean` = `rm -rf result dist-*`
- `ghcid` = `nix-shell -A shells.ghc --run 'ghcid -W -c "cabal new-repl frontend" -T Main.main'`
- `hoogle` = `nix-shell -A shells.ghc --run "hoogle server --local --port=8080" --arg withHoogle true`
- `open` = `xdg-open http://localhost:3003`
- `open-page` = ``xdg-open file://`pwd`/result/frontend/bin/frontend.jsexe/index.html``
- `open-page-cabal` = ``xdg-open file://`pwd`/dist-ghcjs/build/x86_64-linux/ghcjs-8.4.0.1/frontend-0.1.0.0/x/frontend/build/frontend/frontend.jsexe/index.html``
- `open-hoogle` = `xdg-open http://localhost:8080`
- `repl` = `cabal new-repl frontend` (run under `bin/shell`)
- `run` = `result/frontend-native/bin/frontend`
- `run-cabal` = `dist-newstyle/build/x86_64-linux/ghc-8.4.3/frontend-0.1.0.0/x/frontend/build/frontend/frontend`
- `shell` = `nix-shell -A shells.ghc`
- `shelljs` = `nix-shell -A shells.ghcjs`
- `style` = `find src -name "*.hs" | xargs stylish-haskell -i`

## dev

There are 3 ways of building project:

- GHC with jsaddle, result = web server executable (warp)
- GHC with WebKit, result = WebKit application (native)
  - still uses jsaddle under the hood to communicate with page hosted by WebKit
- GHCJS, result = JS scripts / HTML page

There are also options for building iOS/Android apps, but they aren't included into this template.

Usually you use warp way for development (with `bin/ghcid`) because it recompiles and reloads the app automatically. So, for development you do:

- open terminal for each of:
  - `bin/hoogle`
  - `bin/ghcid`
  - `bin/shell`
- run `bin/open` to open warp url in browser
- run your favorite editor under `bin/shell` (e.g. `code .`)
- optional:
  - run `bin/repl` under `bin/shell`

## cabal

Your project may have JS FFI that doesn't have jsaddle implementation. Good example is a browser extension API which is not available for the code running on a regular HTML page. In this case you cannot use GHC. You cannot use ghcid as well because GHCJS doesn't support interactive mode yet. You still can build JS with `bin/build`, but it uses Nix and cannot build incrementally. Instead you may want to use:

- `bin/build-cabal` to build with cabal and GHCJS
- `bin/open-page-cabal` to open HTML page created by the build

## build

- run `bin/build`
- see result in `result/frontend/bin/frontend.jsexe`
  - use `bin/open-page` to open it

native:

- run `bin/build-native`
- see result in `result/frontend-native/bin`
  - use `bin/run` to run it

## misc

- `bin/clean` - remove all build results
- `bin/style` - run stylish-haskell (run under `bin/shell`)
  - WARN: updates files in place, use with care!
  - it's recommended to commit the changes first, then run `bin/style` and make sure you like the diff

## update reflex-platform

- open [reflex-platform](https://github.com/reflex-frp/reflex-platform) and find commit you want
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
- update [bin/style](bin/style)

## credits

This template is based on:

- [Exploring Nix & Haskell Part 3: Less Nix, More Reflex](https://cah6.github.io/technology/nix-haskell-3/) article
  - the code is in [haskell-nix-skeleton-2](https://github.com/cah6/haskell-nix-skeleton-2)
    - which is based on [reflex-project-skeleton](https://github.com/ElvishJerricco/reflex-project-skeleton)
      - which uses [reflex-platform](https://github.com/reflex-frp/reflex-platform)

## license

License is MIT
