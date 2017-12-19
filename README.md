# <a name="top"></a> Best practices and styles guidelines of Fortran FOSS Programmers

> an opinionated list of *best practices* to be a good Fortran FOSS Programmer

+ [standard compliance](#standardization) counts... but *practicality* counts more;
+ [beautiful](#beautiful) is better than *ugly*:
  + [naming convention](#naming) counts;
  + [explicit](#explicit) is better than *impl.*;
  + [simple](#simple) is better than *CoMpleX*:
    + [structured](#structured) is better than *unStRUcturED*;

---

[Guide](#guide) | [Motivations](#motivations) | [References](#references) | [Copyrights](#copyrights)

---

# <a name="guide"></a> Guide to the document

Before reading this document, please understand the following conventions.

## General rules

General rules are emphasized by *blockquotes*, e.g.

> be standards compliant as much as possible

## Examples

In order to clearly explain our motivations we often report examples, both *positive* and *negative*. The examples snippets are syntax-highlighted and *decorated*, e.g.

:x:
```fortran
type foo
  contains
    procedure :: negate_foo
    generic :: operator(.not_foo.) => negate_foo ! .not_foo. is not a valid name
end type
```

:white_check_mark:
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

+ exploit parallel architectures: Fortran is widely used in High Performance Computing (HPC) where parallel hardware is omnipresent; it is very common the necessity to adopt *third party solution* (generally a library) to exploit such a hardware; rather than a very *promising*, but uncommon solutions, prefer: [OpenMP](), [OpenACC](), ...
+ *consume third party library*: some problems thst are not strictly related to mathematical computations (e.g. OS-level operations or runtime errors handling) are often solved using mixed-languages, non standard libraries (typycaly C based, but Python is also commonn nowadays); in such a case consider exploiting `ISO_C_BINDING` to develop standards compliance codes as much as possible.

TODO: Complete discussion of 'exploit parallel architectures' (first bullet point)

Finally, be conscious that there are other (subtle) aspects that could make your code non-standard: for example, the use of *preprocessor* directives makes your code not strictly standard since there is no standard Fortran preprocessor. While many Fortraners use the `fpp` preprocessor which ships with Intel's Fortran compiler, others use the COCO preprocessor[[e]](#nagle-2012), and many use no preprocessor at all.

## <a name="beautiful"></a> Beautiful is better than *ugly*
There is a common misconception about computer codes: *only the results count, no matter the form* of your code. This is wrong from many points of view, but mostly remember that:

> in addition to writing code for the computer, you are also writing code for humans, yourself included. The purpose of the calculations and the methods used to do so must be clear

see [[1](#modern-fortran-style)]. This means that results count, but also *readability* is a value: a readable and clear (and possible concise) code can be easily debugged, improved and *maintained* (e.g. ported on new architecture). As a consequence, the time you spend on beautify your code is not wasted: it will reduce future works, it will allow easy collaboration with new developers, it will make easy to evolve the code itself.

In the case of code used in high-reliability environments (i.e. nuclear safety), source code may be subject to audits and peer review for *verification* purposes (i.e. ensuring the code correctly implements its functional specifications). Code which is obscure is difficult to review and verify and may harbor faults which testing does not reveal.

The followings are some guidelines to try to develop *beautiful* (in the sense of readable/clear) codes:

+ [naming convention](#naming) counts;
+ [explicit](#explicit) is better than *impl.*;

### <a name="naming"></a> Naming convention counts
The naming convention is a very personal choice. The following suggestions should be considered as they are... just *suggestions*.

> + adopt your *opinionated* naming convention and use it **consistently**;
> + prefer lowercase: Fortran is a case-insensitive language, thus you cannot rely on lower-UPPER case distinction for syntax checks; however, `CamelCase` style could help code readability, thus select your way:
>     + if you use a strict lowercase style, consider to exploit *underscores* (`lower_case`) to improve your names clearness (especially for multi-word names);
>     + try to use underscores consistently. For example, if the variables `temp_wall` and `temp-gas` have been defined, the floor temperature should be named `temp_floor`, not `tempfloor` or `floor_temp`.
> + names should be *descriptive*, names should have *meaning*: do not strive too much in *local shortening*, adopt names that are *auto-explicative*; in particular, singular character names should be admissible for **only** index/counter variables;
>     + avoid *magic numbers*, use *parameters*, i.e. named constants;
>     + logical variables should be named to reflect the state they are used to conveying, e.g.
>         + prefer `lib_is_initialized` vs `lib_init`;
>         + prefer `obj_has_parent` vs `obj_parent`;
> + take care of *entities disambiguation*; often you would have the same name for different entities, e.g. a module name and the name of the *main public derived type* it defines.
> + the name of source files should reflect the name of the main entity they are defining;

Note that there is at least one case where *underscores* style cannot be used, i.e. naming *defined operators*, e.g.

:x:
```fortran
type foo
  contains
    procedure :: negate_foo
    generic :: operator(.not_foo.) => negate_foo ! .not_foo. is not a valid name
end type
```
In such a case, consider to exploit `CamelCase` style, e.g.

:white_check_mark:
```fortran
type foo
  contains
    procedure :: negate_foo
    generic :: operator(.notFoo.) => negate_foo ! .notFoo. is a valid name
end type
```

#### General considerations
The following general considerations are **not** *rule of thumb*, but they provide interesting details on some common scenario.

+ [Names readability](#names-readability)
+ [Names meaning](#names-meaning)
+ [Entities disambiguation](#entities-disambiguation)
+ [Procedures disambiguation](#procedures-disambiguation)

TODO: Add more considerations.

##### <a name="names-readability"></a> Names readability
The following are some interesting *suggestions* to improve readability. Some of them could be considered *anachronistic*:

+ write Fortran keywords in all *UPPERCASE*: this guideline was often adopted (in the near past) to distinguish reserved Fortran keywords from user's defined names; presently, almost all widely used editors provide a fairly complete syntax highlighting support by means of which the Fortran keywords are immediately distinguished form others; as a consequence, before adopt this style, consider to exploit editors highlighting thus saving your *typing efforts* necessary to capitalized all Fortran keywords;
+ exception could be the naming of *global accessible* variables (that, indeed should be avoided if they are not parameters): all *UPPERCASE* names could help to distinguish global scope entities (e.g. module or program variables) from local ones (e.g. procedures variables)

:white_check_mark:
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
+ avoid naming variables with single-character `l`, i.e. lower case `L`: often the lower case name `l` can be confused with the number constant `1`, especially when some mono-spaced/sans-serif fonts are used, e.g. in the statement `do l=1, N` do you easily distinguish the loop counter from the loop starting value? Again, syntax highlighting sometimes helps to distinguish variables from numeric constants; note that similar considerations could be made also for the single-character name `O`, i.e. upper case `o` that could be confused with the number constant `0`;

##### <a name="names-meaning"></a> Names meaning
As a *rule of thumb* (yes, we aforementioned that these are not rule of thumb...) consider that

> all names must have a meaning.

Current (modern) Fortran standards (F90 and later) allow *long* names (up to 63 characters). Consider exploiting all the characters you need, do not strive too much on *local shortening*.

Let us consider the definition of the air *speed of sound*: naming it as `a`, as it is common in some fields, is not a good choice, it being unclear for who is not familiar with this implicit convention; even worst could be to name it as `sos` for the sake of conciseness: does this variable contain the value of the speed of sound or a help request?  A name like `speed_of_sound` is preferable since it is clear and of reasonable length.

If the speed of sound for multiple materials is needed, consider using an array with named constants representing the index for material, i.e. `speed_of_sound(AIR)`, `speed_of_sound(WATER)`, etc.

Verbosity or names-lengthy could be effectively limited with other helper constructs when/where conciseness helps readability. Let us assume that our speed of sound variable is a member of a derived type representing a gas

:white_check_mark:
```fortran
type :: gas
  real :: gamma = 0.0
  real :: pressure = 0.0
  real :: speed_of_sound = 0.0
end type gas
```

Local names shortening could be easily achieved exploiting the `associate` construct, e.g.

:white_check_mark:
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

NOTE: As of Fortran 2008, `gamma` is an intrinsic math function; a different variable name should be used to represent the isentropic expansion coefficient (ratio of heat capacities). This illustrates both the importance and (at times) difficulty of choosing good names for variables.

Consider exploiting *meaningful names* to their *extreme*. Let us assume we are developing a large library that must deal with numerous *objects*, and we need to perform the same set of *actions* on these data, i.e. we need to develop a set of procedures performing the *same conceptual* action, but on different objects. In order to improve the readability of this large library, it could be helpful to provide to such a procedures set with a *complete meaningful and descriptive* name, e.g.

:white_check_mark:
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

:x:
```fortran
module shape_sphere
  type, public :: shape_sphere ! not allowed
  end type shape_sphere
end module shape_sphere
```
This is not allowed because two different *entities*, namely the module and the derived type, have the same name in the same scope. In such a situation, you should consider adopting a convention to disambiguate names. Note that a similar ambiguity happens when you would like to name a variable with the same of its type, e.g.:

:x:
```fortran
type(shape_sphere) :: shape_sphere ! not allowed
```

Among the others, the following are widely-used disambiguation techniques:

###### use *prefix* and or *suffix*
+ suffix modules with `_m` and derived type with `_t`:

:white_check_mark:
```fortran
module shape_sphere_m
  type, public :: shape_sphere_t
  end type shape_sphere_t
end module shape_sphere_m
```
+ prefix *all* modules building up a library (or a package) with a common tag, e.g. `pkg_`:

:white_check_mark:
```fortran
module pkg_shape_sphere
  type, public :: shape_sphere
  end type shape_sphere
end module pkg_shape_sphere
```

###### use different styles concurrently
Use mixed mode `CamelCase/under_score` style, one for entity:

:white_check_mark:
```fortran
module shape_sphere ! underscores for module name
  type, public :: ShapeSphere ! CamelCase for derive type
  end type ShapeSphere
end module shape_sphere
```

**Please note:** This style doesn't work in case you have module and/or type names that only consist of a single word. `Car` and `car` are the same thing to Fortran compilers, so it will generate namespace conflicts.

###### exploit submodules (specific for module/derived type disambiguation)
exploit the (new) `submodule` construct to separate the API definition of the derived type and its actual implementation and, consequently, to distinguish the module name from the derived type being defined, e.g.:

:white_check_mark:
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

:white_check_mark:
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

:white_check_mark:
```fortran
type :: gas
  real :: speed_of_sound ! underscores style for variables
  contains
    procedure :: ProjectOnCharacteristics ! CamelCase for procedures
end type gas
```
> + procedures should have an *an action **verb*** into their name ...
> + however, consider to distinguish `subroutine` from `function` exploiting the presence (or the absence) of the action verb, i.e. use **verb form** for subroutine and **noun form** for function, e.g.

:white_check_mark:
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

:white_check_mark:
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
There are particular scenarios where implicit typing could be admissible.

##### Implicit templating
Exploiting implicit typing and standard `include` statement could be used to *mimic* templating programming

:white_check_mark:
```fortran
subroutine abc_sub
  implicit type(abc) (v)
  include "sub_base-inc.f90"
end subroutine abc_sub
```
where `sub_base-inc.f90` contains just the usage of variable `var`.

Go to [TOP](#top) or [Up](#explicit)

### <a name="simple"></a> Simple is better than *CoMpleX*

*Simplicity* is a fundamental key to writing readable and maintainable code. In general, a *task* can be completed by means of different approaches: depending on the particular application, different approaches result in simple or complex codes. In this regard, different programming *idioms* should be used for different aims, e.g. Object Oriented Programming is tailored to improve code maintainability and reausability for complex (possible *generic*) tasks, but it could add unnecessary complexity for simple applications.

Concerning simplicity, some aspects should be considered:
+ [structured](#structured) is better than *unStRUcturED*;

#### <a name="structured"></a> Structured is better than *unStRUcturED*

Commonly, a code could be viewed as sequence of statements that *flow* from one statement to the next one, but to allow the program to behave differently in response to different inputs, *jump* statements are often used, making the flow of control generally non-linear. Such flows can be distingushed in **structured** and **unstructured** (in the worst case leading to so called *spaghetti-code*):

+ structured flows are easy to read/improve/maintain because the jump conditions must strictly respect a *location constraint* allowing the flow to jump into only one prescribed location;
+ unstructured flows are more difficult to follow and thus more difficult to read/improve/maintain because the jump conditions are totally unrestricted allowing the flow to jump anywhere;

While the former is less flexible but more readable the latter may allow very efficient code which is very complex and unreadable. In Fortran, many constructs allow for structured flow, primarily `call/return`, `do/end do`, `exit`, `cycle` and the branching constructs like `select case` or `if elseif`. On the contrary, the most famous unstructured jump statement is `goto` (*unconditional*, *computed*, or *assigned*, or the related *arithmetic IF* statement) that allows flow jump to any labelled position (upward or downward) within a program unit.

> it is recommended to avoid, as much as possible, `goto` statement:
+ in almost all cases goto can be replaced by means of structured-safe constructs **without** any performance penalty while greatly improving readability and maintainability;
+ but exceptions should be allowed for very corner cases that, anyhow, should be in the hands of experts.

As a matter of fact, for very special cases, `goto` could be very helpful, see [[a]](#dijkstra-1968)[[b]](#rubin-1987). Commonly, such applications involves very low-level tasks that are more likely happening in C contest rather Fortran one, e.g. exceptions (errors) handling, resources cleaner/finalizer, ecc... The interested readers could read a recent study of Nagappan et. al. [[c]](#nagappan-2015) showing that `goto` is still used on many GitHub-hosted projects. Moreover, `goto` seems to be extensively used in the Linux kernel, e.g. see [[d]](#wilkens-2003).

> `goto` is not the evil *per se*, rather because it likely drives [programmers] to develop unstructured code that flows chaotically as the flight of a *butterfly*.

##### Examples

Fortran is a very high-level language, we *discourage* the usage of `goto`, but this is not a *rule of thumb*. The following examples show some cases where `goto` is commonly used and how to avoid it.

###### Exception/error handling

Sometimes, the *algorithm* must take into account the occurences of exceptions or errors and, for example, break a (series) of loop. In the old-good-days, such an exceptions handling was done by means of `goto`:

:x:
```fortran
do a=1,n
  do b=1,m
    ! do some stuff ...
    if (is_error_occurred) goto 10 ! break out of the outer loop
    ! ...
  end do
end do
10  continue
```

Although the above code is clear, for large loops it soon becomes *obscure*: the *jump* condition `goto 10` is not actually telling more than *jump on label 10*, but `label 10` can be anywhere in the current scope, even before the start of the outer loop. The resulting flow is *unstructered* and could lead to *spaghetti* code if not carefully handled. On the contrary, we suggest using **named** do loops:

:white_check_mark:
```fortran
a_loop : do a=1,n
  b_loop: do b=1,m
    !do some stuff ...
    if (done) exit a_loop ! break out of the outer loop
    !...
  end do b_loop
end do a_loop
```
The `exit` statement is more clear, it is jumping outside (to the end) of the outer (named) loop, that is a structured flow: exit can jump only foreward.

Similarly, there are conditions when one wants to skip to the end of the loop body and advance to the next iteration. The classic method of handling this case is also with a `goto`:

:x:
```fortran
do a=1,n
  ! do some stuff ...
  if (condition_detected) goto 10 ! skip remainder of loop body
  ! do a lot of other things
  x(a) = 0.0
10  continue
end do
```

The code is clear in this toy example but again, consider code with a complex loop body. Or worse, code structure such as the following:

:x:
```fortran
! This is occasionally seen in FORTRAN 77 code
do 10 a=1,n
  ! do some stuff ...
  if (condition_detected) goto 10 ! skip remainder of loop body
  ! do a lot of other things
  x(a) = 0.0
10 continue ! Please don't do this
```

or

:x:
```fortran
! This is seen more often in FORTRAN IV and FORTRAN 66 code
do 10 a=1,n
  ! do some stuff ...
  if (condition_detected) goto 10 ! skip remainder of loop body
  ! do a lot of other things
10 x(a) = 0.0 ! Please, please, PLEASE don't do this
```

The intent of the `goto` and behavior of the code can be badly obscured. Using both the `cycle` statement and named loops, the code can be greatly clearified:

:white_check_mark:
```fortran
a_loop: do a=1,n
  ! do some stuff ...
  if (condition_detected) cycle a_loop ! skip remainder of loop body
  ! do a lot of other things
  x(a) = 0.0
end do a_loop
```

Here it is clear that the loop `a_loop` is *short-circuited*; the intent is to skip the remainder of the loop body and advance to the next iteration of the loop. If `cycle` occurs on the last iteration of the loop, the loop terminates.

###### Alternate conditional branches

The *arithmetic IF* statement is an obsolete control structure which was used to implement alternate branches before the `if/elseif/else` construct was added to the Fortran standard.

:x:
```fortran
! This is seen often in FORTRAN IV and FORTRAN 66 code
if (j) 10, 20, 30
10 ! Do things if j < 0
  goto 40
20 ! Do things if j == 0
  goto 40
30 ! Do things if j > 0
40 continue
```

The logic of this example is clear however when branching logic becomes complex and nested and the labels references in the arithmetic `IF` statement are scattered throughout the code, the code quickly becomes unreadable and unmaintainable.

:white_check_mark:
```fortran
if (j > 0) then
  ! Do things if j < 0
elseif (j == 0) then
  ! Do things if j == 0
else
  ! Do things if j > 0
end if
```

###### Resources cleaner/finalizer

Often, the *algoritm* allocates resouces that need to be *cleaned/finalized* when the task is finished and, in general, resources are *case-dependent* thus different *kind* of cleaners are necessary.

TODO: Document finalization best practices

##### References

<a name="dijkstra-1968"></a>[[a]]() *Go To Statement Considered Harmful*, Edsger W. Dijkstra, ACM, Vol. 11, No. 3, March 1968, pp. 147-148.

<a name="rubin-1987"></a>[[b]]() *GOTO Considered Harmful*, Frank Rubin, ACM, Vol. 30, No. 3, March 1987, pp. 195-196.

<a name="nagappan-2015"></a>[[c]](http://www.se.rit.edu/~mei/publications/pdfs/An-Empirical-Study-of-Goto-in-C-Code-from-GitHub-Repositories.pdf) *An Empirical Study of Goto in C Code from GitHub Repositories*, M. Nagappan, et. al, Proceedings of the 2015 10th Joint Meeting on Foundations of Software Engineering, Pages 404-414, 2015.

<a name="wilkens-2003"></a>[[d]](http://koblents.com/Ches/Links/Month-Mar-2013/20-Using-Goto-in-Linux-Kernel-Code/) *USING GOTO IN LINUX KERNEL CODE*, R. Wilkens, *epistolary* discussion with Linus Torvalds, 2003.

<a name="nagle-2012"></a>[[e]](http://www.daniellnagle.com/coco.html) *Fortran Program coco*, D. Nagle, 2012.

Go to [TOP](#top) or [Simple](#simple)

### <a name="references"></a> References

Sources of inspiration:

 <a name="modern-fortran-style"></a>[[1]]() [Modern Fortran: Style and Usage](http://www.amazon.com/Modern-Fortran-Norman-S-Clerman/dp/052173052X), *Norman S. Clerman and Walter Spector*.

Go to [TOP](#top)

### <a name="copyrights"></a> Copyrights

This document is released under a Creative Commons Attribution-ShareAlike 4.0 International [License](https://github.com/Fortran-FOSS-Programmers/Best_Practices/blob/master/LICENSE-CC-by-sa.md).

All the code-snippets contained into this document are released under a Creative Commons CC0 1.0 Universal [License](https://github.com/Fortran-FOSS-Programmers/Best_Practices/blob/master/LICENSE-CC0.md).

Go to [TOP](#top)
