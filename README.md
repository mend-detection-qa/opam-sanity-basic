# opam-sanity-basic

This is a Mend SCA detection sanity probe. It contains the minimal valid opam 2.0
project needed to verify that Mend's OCaml resolver detects a flat `depends:` list
from a single `.opam` manifest and produces a deterministic dependency tree.

## Feature exercised

Plain `depends:` list with no filter annotations, no `depopts`, no `dune-project` —
the simplest possible opam 2.0 project that resolves cleanly against opam-repository.

## Pattern

`basic-registry` — direct registry resolution from opam-repository with pinned
lower-bound versions on 3 stable, widely-used packages.

## Dependencies chosen

| Package    | Pinned lower-bound | Resolved exact | Rationale                                 |
|------------|-------------------|----------------|-------------------------------------------|
| dune       | >= 3.23.0         | 3.23.0         | Most common OCaml build system            |
| cmdliner   | >= 2.1.1          | 2.1.1          | Widely-used CLI arg parser, tiny dep tree |
| fmt        | >= 0.11.0         | 0.11.0         | Common formatting library                 |

Transitive (pulled in by dune and fmt):

| Package     | Resolved exact | Pulled in by      |
|-------------|----------------|-------------------|
| base-threads | base          | dune              |
| base-unix    | base          | dune              |
| ocamlfind    | 1.9.8         | fmt (build)       |
| ocamlbuild   | 0.16.1        | fmt (build)       |
| topkg        | 1.1.1         | fmt (build)       |

## Mend config

No `.whitesource` is emitted for this probe. The opam resolver is driven entirely by
`ocaml.resolveDependencies=true` (auto-recommended by Mend when a `*.opam` file is
found) and reads from the active opam switch.

`ocaml.runPreStep` is NOT set. The probe assumes the target scan environment already
has `opam install .` run prior to scanning (i.e., packages are present in the switch).
If the scan environment does not pre-install, set `ocaml.runPreStep=true` in
`whitesource.config` so Mend runs `opam install opam-sanity-basic.opam` as a pre-step.

## Resolver behavior notes (from UA ocaml.md)

- Mend detects `*.opam` files; no lockfile parsing — resolver-driven from installed switch.
- Disabled in Quick Mode. Not supported on Windows.
- If `ocaml.switchName` is set and the switch is not found, UA falls back to active switch.
- DependencyType reported: `OPAM`.

## Probe metadata

```json
{
  "probe_id": "opam-sanity-basic",
  "pattern": "basic-registry",
  "pm": "opam",
  "generated_at": "2026-05-27T10:50:52Z",
  "target": "remote",
  "remote_url": "https://github.com/mend-detection-qa/opam-sanity-basic",
  "ocaml_constraint": ">= 4.14",
  "pm_version_tested": null
}
```

## Expected dependency tree

See `expected-tree.json` at the repo root.