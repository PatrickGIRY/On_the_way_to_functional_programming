# Setup Haskell project

2025-08-03

```sh
stack new attendees-finder-hs new-template
cd attendees-finder-hs
stack install
stack test
```

- Upgrade `package.yaml` to add `hspec`dependency

```yaml
name:                attendees-finder-hs
version:             0.1.0.0
github:              "githubuser/attendees-finder-hs"
license:             BSD-3-Clause
author:              "Author name here"
maintainer:          "example@example.com"
copyright:           "2025 Author name here"

extra-source-files:
- README.md
- CHANGELOG.md

# Metadata used when publishing your package
# synopsis:            Short description of your package
# category:            Web

# To avoid duplicated efforts in documentation and dealing with the
# complications of embedding Haddock markup inside cabal files, it is
# common to point users to the README.md file.
description:         Please see the README on GitHub at <https://github.com/githubuser/attendees-finder-hs#readme>

dependencies:
- base >= 4.7 && < 5

ghc-options:
- -Wall
- -Wcompat
- -Widentities
- -Wincomplete-record-updates
- -Wincomplete-uni-patterns
- -Wmissing-export-lists
- -Wmissing-home-modules
- -Wpartial-fields
- -Wredundant-constraints

library:
  source-dirs: src

executables:
  attendees-finder-hs-exe:
    main:                Main.hs
    source-dirs:         app
    ghc-options:
    - -threaded
    - -rtsopts
    - -with-rtsopts=-N
    dependencies:
    - attendees-finder-hs

tests:
  attendees-finder-hs-test:
    main:                Spec.hs
    source-dirs:         test
    ghc-options:
    - -threaded
    - -rtsopts
    - -with-rtsopts=-N
    dependencies:
    - attendees-finder-hs
    - hspec
```
