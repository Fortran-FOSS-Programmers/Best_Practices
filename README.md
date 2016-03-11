# <a name="top"></a> best practices and styles guidelines of Fortran FOSS Programmers

> an opinionated list of *best practices* to be a good Fortran FOSS Programmer

+ [standard compliance](#standardization) counts... but *practicality* counts more;
+ [beautiful](#beautiful) is better than *ugly*:
    + [naming convention](#naming) counts;
    + [explicit](#explicit) is better than *impl.*;

---

[Guide](#guide) | [Motivations](#motivations) | [References](#references) | [Copyrights](#copyrights)

---

# <a name="guide"></a> Guide to the document

Before read this document, please understand the following conventions.

## General rules

General rules are emphasized by *blockquotes*, e.g.

> be standards compliant as much as possible

## Examples

In order to clearly explain our motivations we often report examples, both *positive* and *negative*. The examples snippets are syntax-highlighted and *decorated*, e.g.

##### :x:
```fortran
type foo
  contains
    procedure :: negate_foo
    generic :: operator(.not_foo.) => negate_foo ! .not_foo. is not a valid name
end type
```
##### :white_check_mark:
```fortran
type foo
  contains
    procedure :: negate_foo
    generic :: operator(.notFoo.) => negate_foo ! .notFoo. is a valid name
end type
```

## Compilers support

Not all *best practices* are supported by all compilers, e.g. if a suggested practice relies on a modern Fortran standard it is likely that few compilers support it. When it is useful, the compilers support is emphasized by a check list like

| Compiler       | Version | Support                   |
|----------------|---------|---------------------------|
| GNU gfortran   | 5.3+    |  :heavy_check_mark:       |
| Intel Fortran  | 15.03   |  :heavy_exclamation_mark: |
| IBM XL Fortran | 14.1    |  :question:               |

---

# <a name="motivations"></a> Motivations

## <a name="standardization"></a> standard compliance counts... but practicality counts more

Standards conformance should be considered of paramount relevance:

> be standards compliant as much as possible

However, follow this guideline with *counsciousness*, be ready to be *flexible*, many are the reasons why you could prefer to break the standard compliance. **Practicality** imposes a compromise generating a *hierarchy* of standards conformities:

> + be standards compliant as much as possible...
>   + except when practicality needs extensions:
>     + prefer extensions that are widely supported by different (possible free) compilers.

Among the common reasons to break standards compliance, the following are worth to be mentioned

+ exploit parallel architectures: Fortran is widely used in High Performance Computing (HPC) where parallel hardware is omnipresent; it is very common the necessity to adopt *third party solution* (generally a library) to exploit such a hardware; rather than a very *promising*, but uncommon solutions, prefer: [OpenMP](), [OpenACC](), *to be completed*;
+ *consume third party library*: some problems thst are not strictly related to mathematical computations (e.g. OS-level operations or runtime errors handling) are often solved using mixed-languages, non standard libraries (typycaly C based, but Python is also commonn nowadays); in such a case consider exploiting `ISO_C_BINDING` to develop standards compliance codes as much as possible.

Finally, be conscious that there are other (subtle) aspects that could make your code non standard: for example the usage of a *preprocessor* makes your code not strictly standard just because Fortran has not defined a standard preprocessor; Fortraners tipically use the C preprocessor that has different features, for example, with respect the Intel Fortran preprocessor `fpp`.

## <a name="beautiful"></a> Beautiful is better than *ugly*
There is a common wrong feeling about computer codes: *only the results count, no matter the form* of your code is. This is wrong from many points of view, but mostly remember that

> in addition to writing code for the computer, you are also writing code for humans, yourself included. The purpose of the calculations and the methods used to do so must be clear

see [[1](#modern-fortran-style)]. This means that results count, but also *readability* is a value: a readable and clear (and possible concise) code can be easily debugged, improved and *maintained* (e.g. ported on new architecture). As a consequence, the time you spend on beautify your code is not wasted: it will reduce future works, it will allow easy collaboration with new developers, it will make easy to evolve the code itself.

The followings are some guidelines to try to develop *beautiful* (in the sense of readable/clear) codes:

+ [naming convention](#naming) counts;
+ [explicit](#explicit) is better than *impl.*;

### <a name="naming"></a> Naming convention counts
The naming convention is a very personal choice. The following suggestions should be considered as they are... just *suggestions*.

> + adopt your *opinionated* naming convention and use it **consistently**;
> + prefer lowercase: Fortran is a case-insensitive language, thus you cannot rely on lower-UPPER case distinction for syntax checks; however, `CamelCase` style could help code readability, thus select your way:
>     + if you go with strict lowercase style, consider to exploit *underscores* (`lower_case`) to improve your names clearness (especially for multi-word names);
> + names should be *descriptive*, names should have *meaning*: do not strive too much in *local shortening*, adopt names that are *auto-explicative*; in particular, singular character names should be admissible for **only** indexes/counters;
>     + avoid *magic numbers*, use *parameters*, i.e. named constants;
>     + logical variables should be named to reflect the state they are used to conveying, e.g.
>         + prefer `lib_is_initialized` vs `lib_init`;
>         + prefer `obj_has_parent` vs `obj_parent`;
> + take care of *entities disambiguation*; often you would have the same name for different entities, e.g. a module name and the name of the *main public derived type* it defines.
> + the name of source files should reflect the name of the main entity they are defining;

Note that there is at least one case where *underscores* style cannot be used, i.e. naming *defined operators*, e.g.
##### :x:
```fortran
type foo
  contains
    procedure :: negate_foo
    generic :: operator(.not_foo.) => negate_foo ! .not_foo. is not a valid name
end type
```
In such a case, consider to exploit `CamelCase` style, e.g.
##### :white_check_mark:
```fortran
type foo
  contains
    procedure :: negate_foo
    generic :: operator(.notFoo.) => negate_foo ! .notFoo. is a valid name
end type
```

#### General considerations
The following general (incomplete) considerations are **not** *rule of thumb*, but they provide interesting details on some common scenario.

+ [Names readability](#names-readability)
+ [Names meaning](#names-meaning)
+ [Entities disambiguation](#entities-disambiguation)
+ [Procedures disambiguation](#procedures-disambiguation)

##### <a name="names-readability"></a> Names readability
The following are some interesting *suggestions* to improve readability. Some of them could be considered *anachronistic*:

+ write Fortran keywords in all *UPPERCASE*: this guideline was often adopted (in the near past) to distinguish reserved Fortran keywords from user's defined names; presently, almost all widely used editors provide a fairly complete syntax highlighting support by means of which the Fortran keywords are immediately distinguished form others; as a consequence, before adopt this style, consider to exploit editors highlighting thus saving your *typing efforts* necessary to capitalized all Fortran keywords;
    + exception could be the naming of *global accessible* variables (that, indeed should be avoided if they are not parameters): all *UPPERCASE* names could help to distinguish global scope entities (e.g. module or program variables) from local ones (e.g. procedures variables)
##### :white_check_mark:
```fortran
```fortran
module global_scope
integer :: GLOBAL
contains
  subroutine unsafe(is_local, local_global)
    logical, intent(in)  :: is_local
    integer, intent(out) :: local_global

    local_global = 0
    if (.not.is_local) local_global = GLOBAL
    return
  end subroutine unsafe
end module global_scope
```
+ avoid naming variables with single-character `l`, i.e. lower case `L`: often the lower case name `l` can be confused with the number constant `1`, especially when some mono-spaced/sans-serif fonts are used, e.g. in the statement `do l=1, N` do you easily distinguish the loop counter from the loop starting value? Again, editors syntax highlighting helps to distinguish variables from numeric constants; note that similar considerations could be made also for the single-character name `O`, i.e. upper case `o` that could be confused with the number constant `0`;

##### <a name="names-meaning"></a> Names meaning
As a *rule of thumb* (yes, we aforementioned that these are not rule of thumb...) consider that

> all names must have a meaning.

Current (modern) Fortran standards (90 and later) allow *long* names (up to 63 characters). Consider exploiting all the characters you need, do not strive too much on *local shortening*.

Let us consider the definition of the air *speed of sound*: naming it as `a`, as it is common in some fields, is not a good choice, it being unclear for who is not familiar with this implicit convention; even worst could be to name it as `sos` for the sake of conciseness: does this variable contain the value of the speed of sound or a help request? Prefer a more clear name like `speed_of_sound`, it being clear for all.

Verbosity or names-lengthy could be effectively limited with other helper constructs when/where conciseness helps readability. Let us assume that our speed of sound variable is a member of a derived type representing a gas
##### :white_check_mark:
```fortran
```fortran
type :: gas
  real :: gamma = 0.0
  real :: pressure = 0.0
  real :: speed_of_sound = 0.0
endtype gas
```
Local names shortening could be easily achieved exploiting the `associate` construct, e.g.
##### :white_check_mark:
```fortran
```fortran
function density(low_pressure_gas)
  type(gas), intent(in) :: low_pressure_gas
  real                  :: density

  associate(gamma => low_pressure_gas%gamma,       &
            pressure => low_pressure_gas%pressure, &
            speed_of_sound => low_pressure_gas%speed_of_sound)
    density = gamma * pressure / (speed_of_sound**2)
  end associate
  return
end function density
```

Consider exploiting *meaningful names* to their *extreme*. Let us assume we are developing a large library that must deal with numerous *objects*, and we need to perform the same set of *actions* on these data, i.e. we need to develop a set of procedures performing the *same conceptual* action, but on different objects. In order to improve the readability of this large library, it could be helpful to provide to such a procedures set with a *complete meaningful and descriptive* name, e.g.
##### :white_check_mark:
```fortran
subroutine compute_transpose_of_sparse_approximate_matrix(...)
...
subroutine compute_transpose_of_dense_approximate_matrix(...)
...
subroutine compute_transpose_of_sparse_exact_matrix(...)
...
```

| Compiler | Version | Support                   |
|----------|---------|---------------------------|
| all      | all     |  :heavy_check_mark:       |

##### <a name="entities-disambiguation"></a> Entities disambiguation
It could happen that you would like to attribute the same name to different *entities* in the same *scope*. An emblematic example is the definition of a derived type. Typically, for important *classes*, a derived type is defined into its own module. Consequently, you have to select a name (with a meaning) for both the module and the derived type. You could be tempted to write something like

##### :x:
```fortran
module shape_sphere
  type, public :: shape_sphere ! not allowed
  endtype shape_sphere
endmodule shape_sphere
```
This is not allowed because two different *entities*, namely the module and the derived type, have the same name in the same scope. In such a situation, you should consider adopting a convention to disambiguate names. Note that a similar ambiguity happens when you would like to name a variable with the same of its type, e.g.:

##### :x:
```fortran
type(shape_sphere) :: shape_sphere ! not allowed
```

Among the others, the following are widely-used disambiguation techniques:

###### use *prefix* and or *suffix*
+ suffix modules with `_m` and derived type with `_t`:
##### :white_check_mark:
```fortran
module shape_sphere_m
  type, public :: shape_sphere_t
  endtype shape_sphere_t
end module shape_sphere_m
```
+ prefix *all* modules building up a library (or a package) with a common tag, e.g. `pkg_`:
##### :white_check_mark:
```fortran
module pkg_shape_sphere
  type, public :: shape_sphere
  endtype shape_sphere
end module pkg_shape_sphere
```

###### use different styles concurrently
Use mixed mode `CamelCase/under_score` style, one for entity:
##### :white_check_mark:
```fortran
module shape_sphere ! underscores for module name
  type, public :: ShapeSphere ! CamelCase for derive type
  endtype ShapeSphere
end module shape_sphere
```

**Please note:** This style doesn't work in case you have module and/or type names that only consist of a single word. `Car` and `car` are the same thing to Fortran compilers, so it will generate namespace conflicts.

###### exploit submodules (specific for module/derived type disambiguation)
exploit the (new) `submodule` construct to separate the API definition of the derived type and its actual implementation and, consequently, to distinguish the module name from the derived type being defined, e.g.:
##### :white_check_mark:
```fortran
module speaker_interface
  type :: speaker
  contains
    procedure, nopass :: speak
  end type speaker

  interface
    module subroutine speak
    end subroutine speak
  end interface
end module

submodule (speaker_interface) speaker_implementation
contains
  subroutine speak
    print "(A)", "Hello, world!"
  end subroutine speak
end submodule speaker_implementation
```
Here `_interface` suffix is used to distinguish the API interface definition of the module from its actual implementation done into the submodule suffixed by `_implementation`, the derived type `speaker` being disambiguated at well.

| Compiler       | Version | Support     |
|----------------|---------|-------------|
| GNU gfortran   | 5.3+    |  :question: |
| Intel Fortran  | 15.03   |  :question: |
| IBM XL Fortran | 14.1    |  :question: |

##### <a name="procedures-disambiguation"></a> Procedures disambiguation
The disambiguation and readability improvement of procedures naming merit some specific considerations:

> + insert an explicit reference to the class name into each type bound procedure name, e.g.
##### :white_check_mark:
```fortran
type :: object
  contains
    procedure :: stuff_1 => perform_stuff_1_on_object
    procedure :: stuff_2 => perform_stuff_2_on_object
    ...
end type object
```
this allow you to easy move the object and its implementation into other modules preserving namespaces integrity.
> + adopt mixed style `CamelCase/under_score` to identify procedures with respect other entities, e.g.
##### :white_check_mark:
```fortran
type :: gas
  real :: speed_of_sound ! underscores style for variables
  contains
    procedure :: ProjectOnCharacteristics ! CamelCase for procedures
end type gas
```
> + procedures should have an *an action **verb*** into their name ...
> + however, consider to distinguish `subroutine` from `function` exploiting the presence (or the absence) of the action verb, i.e. use **verb form** for subroutine and **noun form** for function, e.g.
##### :white_check_mark:
```fortran
! verb form
subroutine get_useful_thing
end subroutine get_useful_thing
! noun form
function useful_thing
end function useful_thing
```

Go to [TOP](#top) or [Up](#beautiful)

### <a name="explicit"></a> Explicit is better than *impl.*
Implicit typing can *speed* the development of codes, but it is difficult to debug/read/understand **implicit-defined** codes, thus

> always exploit `implicit none` statement:
+ into *hosts*: place `implicit none` at the start of each `program`, `module` and `submodule`;
+ into procedures that are not contained into a *host*: place `implicit none` at the start of each `function` and `subroutine` defined outside a host;

A sane explicit style is

##### :white_check_mark:
```fortran
module sane_interface
  implicit none
  type sane_type
    contains
      procedure, nopass :: print_hello
  end type sane_type
end module sane_interface

submodule (sane_inteface) sane_implementation
  implicit none
  contains
    subroutine print_hello
      print "(A)", "Hello, world!"
    end subroutine print_hello
end submodule sane_implementation

program sane
  use sane_interface
  implicit none
  type(sane_type) :: greeter
  call greeter%print_hello
end program sane
```
Note that is not necessary to add `implicit none` into the subroutine `print_hello` because it inherits the clause from its host, namely the submodule `sane_implementation`.

It is also worth to mention that many compilers offer the possibility to enforce `implicit none` everywhere by means of dedicated options, e.g. GNU gfortran offers the option `-fimplicit-none`.

#### Unusual implicit typing usage
There are particular scenario where implicit typing could be admissible.

##### Implicit templating
Exploiting implicit typing and standard `include` statement could be used to *mimic* templating programming
##### :white_check_mark:
```fortran
subroutine abc_sub
  implicit type(abc) (v)
  include "sub_base-inc.f90"
end subroutine abc_sub
```
where `sub_base-inc.f90` contains just the usage of variable `var`.

Go to [TOP](#top) or [Up](#explicit)

### <a name="references"></a> References

Sources of inspiration:

[1] <a name="modern-fortran-style"></a> [Modern Fortran: Style and Usage](http://www.amazon.com/Modern-Fortran-Norman-S-Clerman/dp/052173052X), *Norman S. Clerman and Walter Spector*.

Go to [TOP](#top)

### <a name="copyrights"></a> Copyrights

This document is released under a Creative Commons Attribution-ShareAlike 4.0 International [License](https://github.com/Fortran-FOSS-Programmers/Best_Practices/blob/master/LICENSE-CC-by-sa.md).

All the code-snippets contained into this document are released under a Creative Commons CC0 1.0 Universal [License](https://github.com/Fortran-FOSS-Programmers/Best_Practices/blob/master/LICENSE-CC0.md).

Go to [TOP](#top)
