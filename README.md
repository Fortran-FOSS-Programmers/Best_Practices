# best practices and styles guidelines of Fortran FOSS Programmers

> an opionated list of *best practices* to be a good Fortran FOSS Programmers

1. [standard compliance counts... but practicality counts more](#standardization)

# Motivations

## <a name="standardization"></a> standard compliance counts... but practicality counts more

Standards conformance should be considered of paramount relevance:

> be standards compliant as much as possible

However, follow this guideline with *counsciousness*, be ready to be *flexible*, many are the reasons why you could prefer to break the standards compliance. **Practicality** imposes a compromise generating a *hierarchy* of standards conformities:

> + be standards compliant as much as possible...
>   + except when practicality needs extensions:
>     + prefer extensions that are widely supported by different (possible free) compilers.

Among the common reasons to break standards compliance, the following are worth to be mentioned

+ exploit parallel architectures: Fortran is widely used in High Performance Computing (HPC) where parallel hardware is omnipresent; it is very common the necessity to adopt *third party solution* (generally a library) to exploit such a hardware; rather than a very *promising*, but uncommon solutions, prefer: [OpenMP](), [OpenACC](), *to be completed*;
+ *consume third party library*: some problems thst are not strictly related to mathematical computations (e.g. OS-level operations or runtime errors handling) are often solved using mixed-languages, non standard libraries (typycaly C based, but Python is also commonn nowadays); in such a case consider to exploit `ISO_C_BINDING` to develop standards compliance codes as much as possible.

Finally, be conscious that there are other (subtle) aspects that could make your code non standard: for example the usage of a *preprocessor* makes your code not strictly standard just because Fortran has not defined a standard preprocessor; Fortraners tipically use the C preprocessor that has different features, for example, with respect the Intel Fortran preprocessor `fpp`.
