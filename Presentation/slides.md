---
title: Efficient Compilation of Polymorphic Record Calculi
theme: apple-basic
layout: default
highlighter: shiki
transition: none
mdc: true
fonts:
    sans: Fira Sans
    serif: Robot Slab
    mono: Fira Code
favicon: 'https://sigarra.up.pt/favicon-96x96.png'
---

<style>
* {
    color: #595959;
}

h1, h2 {
    font-weight: normal !important;
    color: black !important;
}
</style>    

<h1 class="flex flex-col">
<span>Efficient Compilation of</span>
<span>Polymorphic Record Calculi</span>
</h1>

<style>
h1, h2 {
    text-align: center;
}

h1 {
    font-size: 52px;
    padding-top: 2em;
}

h1 span {
    line-height: 1.2;
    color: black;
}

h2 {
    font-size: 23px;
    color: #595959;
}

hr {
    margin-top: 2em;
    margin-bottom: 6em;
    width: 70%;
    margin-left: 15% !important; 
    margin-right: 15% !important;
}

.mermaid {
    text-align: center;
}
</style>

## Master's in Informatics and Computer Engineering 

<hr>

<div class="flex flex-col">
    <span><strong>Student:</strong> Eduardo Correia</span>
    <span><strong>Supervisor:</strong> Prof. Sandra Alves</span>
</div>

![](feup-logo.png){width=200px, style="position: absolute; bottom: 15px; right: 15px;"}

<i>18 July 2024</i>

---
layout: section
---

# Context

<!-- 
I'll start by presenting some of the key concepts necessary to better understand this thesis.
 -->

---

# Context

## $\lambda$-calculus

- First invented by **Alonzo Church** in the 1930s.
- It's a formal system in mathematical logic for expressing (Turing complete) computation, based on **function abstraction** and **application**.
- Due to its **simplicity** and **strong expressiveness**, it has been since widely used in the **study of programming languages**.

--- 

# Context 

## Typed $\lambda$-calculus

Untyped $\lambda$-calculus

$$
\lambda x . x
$$

Simply-typed $\lambda$-calculus

$$
\lambda x . x  : (Int \rightarrow Int)
$$

Polymorphic $\lambda$-calculus

$$
\lambda x . x  : \forall \alpha . \alpha \rightarrow \alpha
$$

---

# Context

## Records

- Labeled records are **data structures** that associate **labels** with **values**.
- Widely used in various data-intensive applications.
- Akin to ***dictionaries*** in Python, or ***maps*** in Haskell.
- The following example represents a record with a student's information:

$$
\{ \textit{Name}: ``\text{Eduardo}", \textit{Age}: 23, \textit{University}: ``\text{FEUP}" \}
$$

---

# Context

## Record Polymorphism

- In simply-typed systems, labeled records' allowable operations are restricted to **monomorphic** ones. That is, the type of a record must be specified in advance.
- **Record polymorphism** stems from the idea of having polymorphic operations over records. That is the same operation can be applied to records with different types.
- As an example, the following term, representing **label selection**, should work for any record type, containing the label $\textit{Name}$:

$$
\lambda x . x \textbf . \colorbox{#bceaeb}{$\textit{Name}$}
$$

$$
\begin{align*}
\{ \colorbox{#f5f2b5}{$\textit{Name}$}: ``\text{Eduardo}", \textit{Age}: 23 \} &\quad
\{ \colorbox{#f5f2b5}{$\textit{Name}$}: ``\text{Correia}", \textit{University}: ``\text{FEUP}" \}
\end{align*}
$$

---

# Motivation and Problem

To achieve record polymorphism, some possible strategies are:

- Subtyping;
- Row variables;
- Kinds.

But these solutions either:

- Lose typing information;
- Introduce over-head, or run-time failures, that should have been caught at compile time;
- Don’t allow for extensible records (add or remove labels).

Therefore, an **efficient compilation method** for a record calculus with extensible records would be a valuable contribution.

---
layout: section
--- 

# State of the Art

---

# State of the Art

## Polymorphic Record Calculus

- In 1995, **Atsushi Ohori**<sup>1</sup> introduced a *let*-polymorphic record calculus that extends a $\lambda$-calculus with **labeled records** and **polymorphic operations** over those records. 
- In Ohori’s system, the notion of a **kind** is used to specify the labels a record is expected to contain.
- An efficient compilation mechanism for this calculus was also provided.
- An **implementation** of this calculus, called **SML<sup>#</sup>**, was provided by extending the **Standard ML** language. 
- However, no operations for **extensible records** were defined (due to efficiency limits).

<Footnotes separator> 
    <Footnote number=1>
        Atsushi Ohori. A polymorphic record calculus and its compilation. ACM Trans. Program.Lang. Syst., 17(6):844–895, 1995. doi:10.1145/218570.218572.
    </Footnote>
</Footnotes>

--- 

# State of the Art

## Record Calculus with Extensible Records

- In 2021, **Miguel Ramos et al**<sup>2</sup> developed a record calculus with **extensible records** based on Ohori’s kinds.
- This calculus has a typing system, based on the notion of **kinded quantification** and a sound and complete **type inference algorithm**, based on **kinded unification**. 
- Negative information was added to kinds, allowing for the specification of fields that a record should not have.
- No compilation mechanism, however, was provided.

<Footnotes separator> 
    <Footnote number=2>
        Miguel Ramos, Sandra Alves. An ML-style Record Calculus with Extensible Records. Electronic Proceedings in Theoretical Computer Science, EPTCS, 1-17, 2021. <br>doi:10.4204/EPTCS.351.1
    </Footnote>
</Footnotes>

---
layout: section
---

# Developed Work

---
layout: image
image: flow.png
backgroundSize: 800px
---

# Flow

---
layout: section
---

# $\lambda^{\chi, \textbf .}$

## Record Calculus with Extensible Records

---

# Record Calculus with Extensible Records

## Terms

$$
\begin{array}{rll}
M \Coloneqq &\ c^b                         & (\text{constant})             \\
    |   &\ x                               & (\text{variable})             \\
    |   &\ \lambda x . M                   & (\text{function abstraction}) \\
    |   &\ M\ M                            & (\text{function application}) \\
    |   &\ \text{let}\ x = M\ \text{in}\ M & (\text{let expression})       \\
    |   &\ \{l = M, \dots, l = M\}         & (\textbf{record})               \\
    |   &\ M \textbf . l                   & (\textbf{field selection})      \\
    |   &\ \text{modify}(M, l, M)          & (\textbf{field update})         \\
    |   &\ M\ \backslash\!\backslash\ l    & (\textbf{contraction})          \\
    |   &\ \text{extend}(M, l, M)          & (\textbf{extension})
\end{array}
$$

---

# Record Calculus with Extensible Records

## Types

$$
\begin{array}{lcll}
        \tau & \Coloneqq & b                                                                  & \ \textrm{(base type)}        \\
             & \vert     & \alpha                                                             & \ \textrm{(variable type)}    \\
             & \vert     & \chi                                                               & \ \textrm{(extensible type)}  \\
             & \vert     & \tau \rightarrow \tau                                              & \ \textrm{(function type)}    \\
             \\
      \sigma & \Coloneqq & \tau                                                               & \ \textrm{(monomorphic type)} \\
             & \vert     & \forall \alpha \!::\! \kappa .\ \sigma                             & \ \textrm{(polymorphic type)} \\
             \\
      \chi   & \Coloneqq & \alpha                                                             & \ \textrm{(type variable)}    \\
             & \vert     & \{ l: \tau, \dots, l: \tau \}                                      & \ \textrm{(record type)}      \\
             & \vert     & \chi + \{ l: \tau \}                                               & \ \textrm{(field-addition type)}    \\
             & \vert     & \chi - \{ l: \tau \}                                               & \ \textrm{(field-removal type)}  \\
             \\
  \end{array}
$$

---

# Record Calculus with Extensible Records

## Kinds

- **Kinds** restrict possible instantiations of a type.
- There are two sorts of kinds: **universal kind** and **record kinds**.

$$
\mathcal U\ |\ \{\!\!\{ l_1^l: \tau_1^l, \dots, l_n^l: \tau_n^l \,\|\, l_1^r: \tau_1^r, \dots, l_n^r: \tau_n^r\}\!\!\}
$$

- In a record kind of the form $\{\!\!\{ F_l \,\|\, F_r \}\!\!\}$, $F_l$ denotes the fields a record should at least **have**, and $F_r$ denotes the fields a record should at least **not have**.

---

# Record Calculus with Extensible Records

## Example

- Given a term $M$ of type $\tau$, and kind $\{ \textit{Name}: \textit{String}, \textit{Age}: \textit{Int} \,\|\, \textit{University}: \textit{String} \}$, the following operations are typed as follows:

$$
\begin{align*}
M \textbf . \textit{Name}                             &: \textit{String} \\
\text{modify}(M, \textit{Name}, ``\text{Correia}")    &: \tau \\
M\ \backslash\!\backslash\ \textit{Age}               &: \tau - \{ \textit{Age}: \textit{Int} \} \\
\text{extend}(M, \textit{University}, ``\text{FEUP}") &: \tau + \{ \textit{University}: \textit{String} \}
\end{align*}
$$

---
layout: section
---

# $\dot \lambda^{\chi, \textbf .}$

## Intermediate Calculus

---

# Intermediate Calculus

- We define an **intermediate annotated calculus**, corresponding to an **explicitily typed** version of $\lambda^{\chi, \textbf .}$.
- The additional **type information** is used to facilitate the **compilation** process.
- The type system for this calculus remains unchanged from the original calculus.
- Two new terms are added to the calculus: **polymorphic instantiation**, $(x\ \tau_1 \cdots \tau_n)$, and **polymorphic generalization**, $\text{Poly}(M: \sigma)$.

$$
\begin{align*}
\text{let}\ x = \colorbox{#bceaeb}{$M$}\ &\text{in}\ \colorbox{#f5f2b5}{$x$} \\
& \hspace{-1.5em} \Downarrow  \\
\text{let}\ x = \colorbox{#bceaeb}{$\text{Poly}(M:\ \sigma)$}&\ \text{in}\ \colorbox{#f5f2b5}{$(x\ \tau_1 \cdots \tau_n)$} \\
\end{align*}
$$

---

# Intermediate Calculus

## Type inference

- A **type-inference algorithm** is provided that turns an untyped term into a typed one.
- As an example, the untyped $\lambda^{\chi, \textbf .}$-term

$$
\begin{align*}
    \text{let} &\ \textit{name} = \lambda x . x \textbf . \textit{Name}\\
    \text{in}  &\ \textit{name}\ \{ \textit{Name}: ``Eduardo", \textit{Age}: 23 \}
\end{align*}
$$

is inferred to

$$
\begin{align*}
    \text{let}\ & \textit{name} = \text{Poly}(\lambda x : \alpha_2 . x \textbf . \textit{Name}: 
     \colorbox{#f5f2b5}{$\forall \alpha_1$} \!::\! \mathcal U . 
     \colorbox{#bceaeb}{$\forall \alpha_2$} \!::\! \{\!\!\{ Name: \alpha_1 \,|\, \}\!\!\}
     . \colorbox{#bceaeb}{$\alpha_2$} \rightarrow \colorbox{#f5f2b5}{$\alpha_1$}) \\ 
    \text{in}\ & (\textit{name}\ \colorbox{#f5f2b5}{\textit{String}}\ \colorbox{#bceaeb}{$\{ \textit{Name}: \textit{String}, \textit{Age}: \textit{Int} \}$})\ \{ \textit{Name}: ``Eduardo", \textit{Age}: 23 \}
\end{align*}
$$

---
layout: section
---

# $\lambda^{\chi, [\,]}$

## Implementation Calculus

---

# Implementation Calculus

- We define an **implementation calculus** that will be used as an **abstract evaluation machine** for the calculus with extensible records.
- To represent labeled records in this calculus, we assume that the total **order** of the labels in a record (typically, the **lexicographic order**) is fixed.
- As such, the record type of the form $\{ l_1: \tau_1, \dots, l_n: \tau_n \}$ must satisfy the condition $l_1 \ll \cdots \ll l_n$.
- After this ordering, we can represent a record as a **list** of values, where the index of a value corresponds to the index of the label in the record type.

$$
\begin{align*}
\{ \textit{Name}: ``\text{Eduardo}", \textit{Age}: 23, \textit{University}: ``\text{FEUP}" \} & \Rightarrow 
[ 23, ``\text{Eduardo}", ``\text{FEUP}" ]
\end{align*}
$$

---

# Implementation Calculus

## Indexing

- Since records are represented as lists, previously defined operations must be adapted to work with **indexes** instead of **labels**.
- There are two types of **indexes**: **index variables**, $I$ (plus an offset $n$), and **index values**, $i$.

$$
\mathcal I = I + n \ |\ i 
$$

- **Label operations** are then converted to **index operations**, by replacing labels with their corresponding index values.

$$
\begin{align*}
    M \textbf . \colorbox{#f5f2b5}{$l$}                   & \Rightarrow M \lbrack \colorbox{#bceaeb}{$\mathcal I$} \rbrack          \\
    \text{modify}(M, \colorbox{#f5f2b5}{$l$}, M)          & \Rightarrow \text{modify}(M, \colorbox{#bceaeb}{$\mathcal I$}, M)       \\
    M\ \backslash\!\backslash\ \colorbox{#f5f2b5}{$l$}    & \Rightarrow M\ \backslash\!\backslash\ \colorbox{#bceaeb}{$\mathcal I$} \\
    \text{extend}(M, \colorbox{#f5f2b5}{$l$}, M)          & \Rightarrow \text{extend}(M, \colorbox{#bceaeb}{$\mathcal I$}, M)
\end{align*}
$$

---

# Implementation Calculus

## Index Abstraction and Application

- To deal with polymorphic record operations, we introduce **index abstraction**, $\lambda I_1 \cdots I_n$,  and **index application**, $x\ i_1 \cdots i_n$.

$$
\begin{align*}
\text{let}\ x = \colorbox{#bceaeb}{$\text{Poly}(M:\ \sigma)$}&\ \text{in}\ \colorbox{#f5f2b5}{$(x\ \tau_1 \cdots \tau_n)$} \\
& \hspace{-1.5em} \Downarrow  \\
\text{let}\ x = \colorbox{#bceaeb}{$\lambda I_1 \cdots I_n . M$}\ &\text{in}\ \colorbox{#f5f2b5}{$x\ i_1 \cdots i_n$}
\end{align*}
$$

- For each label in a given kind $\{\!\{ F_l \| F_r \}\!\}$ we have a corresponding index variable $I$.
- As an example, for a record with kind $\{\!\{ \textit{BirthYear}\ \| \textit{Age} \}\!\}$, where $I_1$ represents the position of $\textit{BirthYear}$ and $I_2$ the position of $\textit{Age}$, we have:

$$
\begin{align*}
\text{let}\ setBirthYear = \lambda \colorbox{#bceaeb}{$I_1$} . \lambda \colorbox{#f5f2b5}{$I_2$} . \lambda x . \text{extend}(x, I_1, 2024 - x \textbf \lbrack I_2 \rbrack) \ &\text{in}\ x\ \colorbox{#bceaeb}{$2$}\ \colorbox{#f5f2b5}{$1$}\ \{ 2000, ``\text{Eduardo}"\}
\end{align*}
$$

---

# Implementation Calculus

## Index Offset

- To account for **field-removal** and **field-addition** operations, an **index offset** is added to the operations with index variables.
- Given a record type of the form $\alpha \pm_1 \{ l_1 : \tau_1 \} \cdots \pm_n \{ l_n : \tau_n \}$, the index offset for a given label $l$ is defined as follows:

$$
\begin{align*}
    0 \pm_1 
    \begin{cases}
        0\ \text{if}\ l \leq l_1 \\
        1\ \text{if}\ l > l_1 
    \end{cases} 
    \cdots \pm_n 
    \begin{cases}
        0\ \text{if}\ l \leq l_n \\
        1\ \text{if}\ l > l_n
    \end{cases} 
\end{align*}
$$

- As an example, given a record $M$ of the type $\{ \textit{Age}: \textit{Int}, \textit{Name}: \textit{String} \}$, we have:

$$
\begin{align*}
(M\ \backslash\!\backslash\ \colorbox{#bceaeb}{$\textit{Age}$}) & \textbf . \colorbox{#f5f2b5}{$Name$} \\
& \hspace{-1em} \Downarrow \\
(M\ \backslash\!\backslash\ \colorbox{#bceaeb}{$I_1$}) & \lbrack \colorbox{#f5f2b5}{$I_2 - 1$} \rbrack \\
\end{align*}
$$

---

# Implementation

- A REPL for these calculi, named **Recording**, was implemented in **Haskell**.

![](recording-logo.png){width=500px, style="margin: 6.5em auto;"}

---

# Implementation 

## Architecture

![](code-diagram.png){width=500px, style="margin: 0 auto;"}

---
layout: section
---

# Implementation

## Demo

<!--
:h
{ Name = "Eduardo", Age = 23 }
:t { Name = "Eduardo", Age = 23 }
:e { Name = "Eduardo", Age = 23 }
let name = \x -> x . Name in name { Name = "Eduardo", Age = 23 }
:t let name = \x -> x . Name in name { Name = "Eduardo", Age = 23 }
:i let name = \x -> x . Name in name { Name = "Eduardo", Age = 23 }
:e let name = \x -> x . Name in name { Name = "Eduardo", Age = 23 }
:e let removeAge = \x -> x \\ Age in removeAge { Name = "Eduardo", Age = 23 }
:e let addUniv = \x -> extend(x, University, "FEUP") in addUniv { Name = "Eduardo", Age = 23 }
:c let updateStudent = \x -> extend(x \\ Age, University, "FEUP") in updateStudent { Name = "Eduardo", Age = 23 }
:e let updateStudent = \x -> extend(x \\ Age, University, "FEUP") in updateStudent { Name = "Eduardo", Age = 23 }
-->

--- 

# Conclusion

## Contributions

- We've managed to extend Ohori's calculus compilation relation with extensible records.
- We provide a pratical implementation of the calculus.
- Demonstration of soundness properties of the defined calculi.
- Adaptation of intermediate calculus with extended records.

## Future Work

- Set operations for records (general join, intersection, difference, ...).
- Labels as first-class citizens.
- Integration of developed work in SML#.

---
layout: section 
---

# Thank you!

## Any questions?
