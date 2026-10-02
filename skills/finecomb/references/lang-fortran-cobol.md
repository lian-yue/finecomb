# A.29 Fortran and COBOL

Applies to: Fortran numerical programs and COBOL data-processing programs. These are separate languages; use only the named rows and the actual compiler dialect.

| Checkpoint | What counts as a problem |
| --- | --- |
| Fortran interfaces | Missing or mismatched explicit interfaces hide argument kind, rank, shape or calling-convention errors |
| Fortran bounds and storage | Non-default lower bounds, array sections, aliasing or copy-in/copy-out temporaries invalidate bounds and ownership assumptions |
| Fortran initialization | Uninitialized locals or implicitly persistent `SAVE` state leak across calls or race between workers |
| Fortran compiler modes | Checks used in testing disappear in deployed builds; fast-math assumptions change handling of non-finite values |
| COBOL numeric fields | `PIC`, scale, signedness, truncation or rounding changes amounts and identifiers without handling the size error |
| COBOL records and tables | `REDEFINES`, variable record lengths or `OCCURS DEPENDING ON` let a count disagree with the available storage |
| COBOL file outcomes | File status, end-of-file, duplicate-key or locking outcomes are ignored, losing or repeating updates |
| External representation | Character sets, fixed-width padding, packed decimals or native calling conventions disagree across systems |

Semantic references: [GNU Fortran runtime checks and code generation](https://gcc.gnu.org/onlinedocs/gfortran/Code-Gen-Options.html), [GnuCOBOL programmer's guide](https://gnucobol.sourceforge.io/HTML/gnucobpg.html). Apply [24](dimensions/24-encoding-and-persistent-formats.md) and [4.32](specialties/4.32-cross-language-boundaries-and-native-extensions.md) at data boundaries.
