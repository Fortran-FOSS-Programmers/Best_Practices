# best practices and styles guidelines of Fortran FOSS Programmers

> an opionated list of *best practices* to be a good Fortran FOSS Programmers

1. [standard compliance](#standardization) counts... but *practicality* counts more;
1. [beautiful](#beautiful) is better than *ugly*:
    1. [naming convention](#naming) counts;

### References

Sources of inspiration:

<a name="modern-fortran-style"></a>[1] [Modern Fortran: Style and Usage](http://www.amazon.com/Modern-Fortran-Norman-S-Clerman/dp/052173052X), *Norman S. Clerman and Walter Spector*.

# Motivations

## <a name="beautiful"></a> Beautiful is better than *ugly*
There is a common wrong feeling about computer codes: *only the results count, no matter the form* of your code is. This is wrong from many points of view, but mostly remember that

> in addition to writing code for the computer, you are also writing code for humans, yourself included. The purpose of the calculations and the methods used to do so must be clear.

see [[1]](#modern-fortran-book). This means that results count, but also *readability* is a value: a readable and clear (and possible concise) code can be easily debugged, improved and *maintained* (e.g. ported on new architecture). As a consequence, the time you spend on beautify your code is not wasted: it will reduce future works, it will allow easy collaboration with new developers, it will make easy to evolve the code itself.

The followings are some guidelines to try to develop *beautiful* (in the sense of readable/clear) codes:

1. [naming convention](#naming) counts;

### <a name="naming"></a> Naming convention
The naming convention is a very personal choice. The following suggestions should be considered as they are... just *suggestions*.

> + adopt your *opinionated* naming convention and use it **consistently**;
> + prefer lowercase: Fortran is a case-insensitive language, thus you could not rely on lower-UPPER case distinction for syntax checks; however, `CamelCase` style could help code readability, thus select your way:
>     + if you go with strict lowercase style, consider to exploit *underscores* (`lower_case`) to improve your names clearness (especially for multi-word names);
> + names should be *descriptive*, names should have *meaning*: do not strive to much in *local shortening*, adopt names that are *auto-explicative*; in particular, singular character names should be admissible for **only** indexes/counters;
> + take care of *entities disambiguation*; often you would have the same name for different entities, e.g. a module name and the name of the *main public derived type* it defines:
```fortran
module ambiguous
    type, public :: ambiguous
    endtype ambiguous
endmodule ambiguous
```
This is not allowed! Consider to adopt a non ambiguous convention to distinguish different kind of entities.

#### General consideration

##### Entities disambiguation

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
