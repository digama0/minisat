Adding FRAT style proofs to Minisat
===============================================================================

This is a modification of the Minisat SAT solver (<http://minisat.se/>)
which adds support for FRAT proofs, which can be postprocessed by the [FRAT toolchain](https://github.com/digama0/frat).
See [README](README) for install instructions.

## Implementation notes

The modification is based on the DRUP proof format modification of Minisat v2.2, and the first
few commits reflect that. [This diff](https://github.com/digama0/minisat/compare/f9e6c43...digama0:master) contains most of the real changes. The changes break down into the following parts:

* The variable `chain` in `addClause_()` and elsewhere keeps the reversed list of clause IDs in the unit propagation proof of the current conflict clause.

* All clause IDs are computed. See the `CLAUSE_ID(cr)`, `LITERAL_ID(lit)` and `EMPTY_ID` macros. This means that we don't need to store any extra data to produce chains, but the downside is that we have to explicitly put a `r` step in the proof whenever we are going to do a garbage collection, which changes `CRef` values of clauses and thus also the computed clause IDs. This is done in `Solver::relocAll()`.

* While we make an attempt to keep chains (reverse) ordered such that each unit is justified before it is used, in some passes, in particular in `litRedundant()`, the information on this is not readily available without another pass. In these cases we just output the units out of order (but still justifying everything) and let the FRAT elaborator put things back together.

* Unlike the [CaDiCaL modification](https://github.com/digama0/cadical), there are no missing proofs in this modification. (It helps that Minisat only has one kind of step.) Every step gets a `chain` proof, although as mentioned it may be out of order.

* FRAT proofs are output only in ASCII form, for simplicity and to keep it similar to the DRUP modification. These are done in `fprintf(output, ...)` lines that can be seen in a few places.