# What ABI Version Should Be Used for `parser.c`?

As mentioned in the discussion for ["Should Generated Parser Source Be
Committed?"](../should-parser-source-be-committed/README.md), most
parser / grammar repositories currently do.

It turns out that the tree-sitter library has a notion of an ABI:

```c
/**
 * The latest ABI version that is supported by the current version of the
 * library. When Languages are generated by the Tree-sitter CLI, they are
 * assigned an ABI version number that corresponds to the current CLI version.
 * The Tree-sitter library is generally backwards-compatible with languages
 * generated using older CLI versions, but is not forwards-compatible.
 */
```

The cli is typically built around the tree-sitter library that's in
the same monorepos, and the [above
comment](https://github.com/tree-sitter/tree-sitter/blob/5766b8a0a785ea34fceb479a94f7fe24c9daae2f/lib/include/tree_sitter/api.h#L17-L23)
is from within the source of the library.

Thus, which ABI is used for one's generated parser source (typically
wired into `src/parser.c`), usually depends on specifically which
version (or build) of the `tree-sitter` cli was used when invoking the
`generate` subcommand.

For release versions of the `tree-sitter` cli, the [default ABI became
14](https://github.com/tree-sitter/tree-sitter/commit/e2fe380a08408ff42eada21f8723f653e6da6606)
in version 0.20.7.  In version 0.20.3, it became possible to specify
generation of at least one level lower via the [`--abi` option to the
`generate`
subcommand](https://github.com/tree-sitter/tree-sitter/pull/1599/commits/516fd6f6def1615cb5dc004ab41c348c7de6d182).

Note that these changes occurred in 2022.

The following is an incomplete table of which versions used which ABI levels:

```
ABI   Version   Release
---   -------   -------
  9    0.14.0   2019-02
 10    0.15.2   2019-06
 11    0.16.0   2019-12
 12    0.17.0   2020-09
 12    0.18.0   2021-01
 13    0.19.0   2021-03
 13    0.20.0   2021-06
 14    0.20.7   2022-09
 14    0.20.8   2023-04
```

There's a demo below whose purpose is to print out a table of which
ABI versions appear in locally fetched `src/parser.c` files.

## Prerequisites for Demo

See the section of the corresponding name in the [repository
README](../../README.md).

## Demo Steps

* Ensure the current working directory is the repository root directory.
* `cd questions/what-abi-level-should-be-used`
* Invoke `sh ./script/gen-abi-stats.sh`
* Observe output like:

```
ABI: Count

9: 12
10: 3
11: 6
12: 4
13: 105
14: 125

Minimum number of repositories with parser.c: 251
```

At least for what was collected about 42% use 13, while about 50% use
14.

Perhaps it's not unreasonable to assume a sufficient sample of
repositories had been collected though.

As the ability to explicitly specify an ABI level was only made
available and advertised in 2022, it may be that these results may be
more a reflection of which cli happened to be used rather than a
conscious decision on the part of individual grammar repository
maintainers.

Tip: to get a list of which repositories use which ABI version, invoke:

```
sh ./script/list-parser-c-abi-nums.sh | sort -n
```

This should produce output that starts something like:

```
9 tree-sitter-abnf.jmitchell
9 tree-sitter-clojure.Tavistock
9 tree-sitter-dhall.jmitchell
9 tree-sitter-email.maxnordlund
9 tree-sitter-graphql.dralletje
9 tree-sitter-qml.rschiang
9 tree-sitter-todo.Aerijo
9 tree-sitter-xml.unhammer
10 tree-sitter-biber.Aerijo
10 tree-sitter-eno.eno-lang
10 tree-sitter-sml.stonebuddha
11 tree-sitter-agda.tree-sitter
11 tree-sitter-carp.GrayJack
11 tree-sitter-move.move-hub
11 tree-sitter-ocamllex.atom-ocaml
11 tree-sitter-odin.lucypero
11 tree-sitter-souffle.julienhenry
12 tree-sitter-diff.vigoux
12 tree-sitter-sexp.AbstractMachinesLab
12 tree-sitter-twitchchat.rockerBOO
12 tree-sitter-xml.dorgnarg
```

Note that the actual repository name for a result with something like:

```
tree-sitter-carp.GrayJack
```

is actually likely to be more like:

```
https://github.com/GrayJack/tree-sitter-carp
```
