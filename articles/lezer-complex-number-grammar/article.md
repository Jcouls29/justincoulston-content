---
title: "Lezer: A Complex Number Grammar"
summary: A walkthrough of building a Lezer parser grammar for complex numbers, explaining the shift/reduce LR algorithm, token definitions, and the whitespace handling trade-offs involved.
category: "Language Design"
publishedDate: 2023-01-04
tags:
  - parsing
  - language-design
  - code-quality
keyTakeaways:
  - Lezer's LR parsing algorithm is strict about shift/reduce conflicts; any ambiguity halts parsing, so grammar rules must be unambiguous by design.
  - Lowercase node names are invisible in the output tree, which lets you define intermediate rules for readability without polluting the parsed result.
  - The @tokens block uses a Deterministic Finite Automaton under the hood, making recursion very limited and requiring explicit whitespace tokens where @skip cannot be used.
  - Working through a simple domain (complex numbers) is an effective way to learn grammar authoring before tackling a full programming language.
draft: false
---

Writing grammars is harder than it looks.

![Screen Capture by Author](https://cdn-images-1.medium.com/max/800/1*8nTcmDFsMLVQXwUx3VXm6A.png)
*Screen Capture by Author*

[Lezer](https://lezer.codemirror.net/) is a parsing system developed, and maintained, by the [CodeMirror](https://codemirror.net/) team. It can take a defined grammar and parse a textual input as a tree of nodes, which in turn can be used to highlight text in an editor. You could even use the tree semantically to run analysis or execute a function based on the input, if you were so inclined.

It is an incredibly powerful tool, used primarily to handle use cases within the CodeMirror ecosystem. It does have some limitations, mainly around the strict requirements of the grammar. Any changes of shift/reduce issues will halt the parsing algorithm. These are well documented in the [System Guide](https://lezer.codemirror.net/docs/guide/). That said, it has been a trying time understanding the nuances of the parser algorithm.

## Complex Grammar

With power comes significant complexity. It requires a different mindset to handle grammars of any complexity. So, I have decided to start with a simple use case: Complex Numbers.

Below is a simple grammar I put together to parse out a tree for a single complex number, where a complex number is defined as a `[Real Part]? +/- [Imaginary Part]?`.

An example of an imaginary number is `25.4E9-65.02j`.

The only limitation I put on the parser is that the `i|j` must come after the number, instead of allowing it in front of the number. Otherwise, any number of spaces within the complex number works and new lines at the end parse correctly.

Here is the grammar, as it sits today,

```
@top ComplexNumber { whitespace* complex } 

complex { 
  RealPart whitespace* | 
  ImaginaryPart whitespace* | 
  RealPart whitespace* "+" whitespace* ImaginaryPart whitespace* |
  RealPart whitespace* ImaginaryPart whitespace*
}

RealPart { Number } 
ImaginaryPart { Number ("j" | "i") }

@tokens {  
  Number { "-" whitespace* int frac? exp? | int frac? exp? } 
  int  { '0' | $[1-9] @digit* }
  frac { '.' @digit+ }
  exp  { $[eE] $[+\-]? @digit+ }
  whitespace { $[ \t]  }
  lineEnd { $[\n\r]+ }
}
 
@skip { lineEnd } 
```

The grammar uses a regex-esque syntax to define `or` operations on node types and uses various conventions to determine if nodes are provided within the tree.

As an example, let's parse

```
0.0195 -  10.09E16j
```

We will get an output tree of the following,

```
ComplexNumber (0.0195 -  10.09E16j\n\n)
    RealPart (0.0195)
        Number (0.0195)
    ImaginaryPart (-  10.09E16j)
        Number (-  10.09E16)
```

Let's walk through this step by step.

#### @top ComplexNumber { whitespace* complex }

This defines the entry point of the grammar. It will match the entire input.

In my case, I am only matching a single `complex` number class with the possibility of 0 or more whitespace characters before it. The `ComplexNumber` node type is capitalized, so it will be rendered in the tree.

#### complex

```
complex { 
  RealPart whitespace* | 
  ImaginaryPart whitespace* | 
  RealPart whitespace* "+" whitespace* ImaginaryPart whitespace* |
  RealPart whitespace* ImaginaryPart whitespace*
}
```

Here we are defining a node type of `complex`. However, lowercase node types do not end up in the tree. This is beneficial because we already have the highest-level `ComplexNumber` defined. So having a nested node, doesn't make sense.

The type can be 1 of 4 different matches within the grammar,

1. `RealPart whitespace*`: Will match an input with only a real part (and no imaginary part) with any number of whitespace characters trailing.
2. `ImaginaryPart whitespace*`: Will match an input with only an imaginary part (and no real part) with any number of whitespace characters trailing.
3. `RealPart whitespace* "+" whitespace* ImaginaryPart whitespace*`: This matches a scenario where a `+` is included within the input. For instance, `1 + -5j` is a valid complex number. For this reason, we separated out this use case. Also, notice the `whitespace` additions around the parts of the number.
4. `RealPart whitespace* ImaginaryPart whitespace*`: This matches a scenario where the imaginary part is negative, without the addition symbol. For example, `1 - 5j`.
*Note: This will also match `13 8i` as a valid complex number. Probably need to correct this at some point.*

There is a lot of `whitespace` tokens strewn about. This is due to the nature of the shift/reduce LR algorithm being used. I was unable to get the `@skip` to properly ignore whitespace while pushing the `-` sign to the imaginary part of the number properly. *(If anyone can make it better, teach me!)*

#### RealPart { Number }

Defines the real part of the equation as ONLY a valid number.

#### `ImaginaryPart { Number ("j" | "i") }`

Defines the imaginary part as a number with a required `j` or `i` postfixed to the number directly.

#### @tokens

```
@tokens {  
  Number { "-" whitespace* int frac? exp? | int frac? exp? } 
  int  { '0' | $[1-9] @digit* }
  frac { '.' @digit+ }
  exp  { $[eE] $[+\-]? @digit+ }
  whitespace { $[ \t]  }
  lineEnd { $[\n\r]+ }
}
```

Here we define the common tokens. This section is special within a grammar in terms of the [Deterministic Finite Automaton](https://en.wikipedia.org/wiki/Deterministic_finite_automaton) that is generated, so recursion is very limited.

We define the primary `Number` token as an `int frac? exp?` with two use cases: A negative case and a positive case.

There is special syntax utilized throughout all of these tokens,

- `$[1-9]`: A regex-esque syntax looking for a single character from `1` to `9`.
- `@digit+`: Match 1 or more digits (`$[0–9]`).
- `$[eE]`: Match a lowercase `e` or a capital `E`.
- `$[+\-]?`: Match a `+` or a `-` sign. The `\` is used as the escape character. The `?` is used to denote `0` or `1` instance of the character class.

*As a note, `*` is used to match `0` or more of a character class.*

#### @skip { lineEnd }

Lastly, we have a skip expression where we ignore line endings `$[\n\r]+`. This means they will not match throughout the expression. This is useful if we NEVER care about these matches. In this case, new lines have no value within the tree.

## Output

The output is expressed as a tree,

```
ComplexNumber (0.0195 -  10.09E16j\n\n)
    RealPart (0.0195)
        Number (0.0195)
    ImaginaryPart (-  10.09E16j)
        Number (-  10.09E16)
```

From an analysis standpoint, we can quickly use this to find the `RealPart` and `ImaginaryPart` of the number, grab its corresponding `Number` node text, remove any whitespace, then parse the value to a float.

And since this is a tree, we will only get a single node from the `Tree` object during parsing. It will have a top-level node with a type of `ComplexNumber`. Then we simply dig into the tree from there.

## Whitespace Issue

Admittedly, I spent most of my time trying to work through the whitespace issue. It was exceedingly frustrating trying to understand how `@skip` worked and how overlapping non-terminals worked. Although, the documentation is well-written, examples are sparse.

So, I ended up putting `whitespace*` where I expected there to be potential spaces or tabs. And after some experimentation, I ended up with the grammar above. It does seem like parsing whitespace and the overlap with tokens in this LR parser can get complex and nuanced.

There is definitely more study needed here.

## Conclusion

I wanted to present a basic example of a grammar, as they are sparse. I am working towards creating some other grammars and highlighting for the CodeMirror editor, specifically around .NET languages. However, I suspect that I have a long road ahead. Once I get past the grammar design, the extension-based setup of CodeMirror 6 has its own complications. Being more advanced within .NET, and a JavaScript novice, I have some work to do.

If you're curious, I used an online tool called [Lezer Playground on littletools.app](https://littletools.app/lezer) I found that allows you to quickly test your grammars. This site has other interesting, and useful, tools for developers. Check it out and let me know what you think!

Hope this quick overview helps you develop a simple grammar for yourself. More complex grammars coming soon!
