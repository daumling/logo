# Discussion

This file is intended to be the basis to discuss various language features, mainly those who change the current Logo implementations.

## Main Directive

**Do not touch the core principles**. We'll try to be careful to not break backwards compatibility by adding features.

## Line Continuation Characters

UCBLogo defines the following line termination characters:

1. The backslash, meaning that the lexer inserts a newline character and continues scanning:

```
MAKE "ROTTER\
DAM "EUROPE
```

Creates a symbol with an embedded newline character.

2. The tilde, meading that the lexer continues scanning:

```
MAKE "ROTTER~
DAM "EUROPE
```

Creates a symbol ROTTERDAM.

I propose to drop these continuation characters in favor of multiple strings and symbols.

## Symbols

Valid symbol characters are any legal Unicode character, numbers, and a few other extra characters: `._$`. Anything else?

Symbols in vertical bars as in `|hello world|` are legal symbols without case conversion. Such a symbol may also span multiple lines.

## Case Conversions

Logo converts all unquoted symbols to upper case. Strings are not case converted, except for the single double quote string as in "STRING.

## Quoting

Currently, major Logo dialects required that constants are quoted. `MAKE "A 5` is a typical example. It is very difficult for young students to understand the meaning behind this.

I suggest that certain Logo commands not require quoting. `MAKE A 5` is easier to write, to read, and to understand. This may introduce a new level of complexity for commands that may or may not accept strings as inputs.

Example: This is good code:

```
MAKE "A "B
TO C
    OP "D
END
MAKE :A 5 ; assign 5 to B
MAKE C 6  ; assign 6 to D, not C
```

If MAKE would accept unquoted symbols, then the latter assignment would be clear (assign 6 to C). See also below for the proposal to remove colons. See also Strings.

## Numbers

Currently, all Logo implementations that I know use the IEEE-754 floating point format, which introduces hard-to-explain rounding errors. Why is 0.1 + 0.2 = 0.300000000000004? OK, this could also be an implementation issue, but maybe we could agree on a BigInt format with, say, 64 bits with a fixed number of decimals, maybe configurable.

Terrapin Logo, for example tries to avoid confusion by defining a global :EPSILON; when comparing two floating-point numbers, they are considered equal if their absolute difference is <= the value of :EPSILON. Together with the ability to define the number of decimals in numeric output, this may be enough to hide these representational errors caused by the IEEE-754 format.

## Variables

Goal: Remove the colon notation.

Reason: Especially with teacher and younger students, there is a lot of confusion about when to use colons in front of a symbol, and what the difference between a symbol with and without a colon is. Here is an example from a question I had recently:

```
TO SQUARE :SIZE
    REPEAT 4 [FD SIZE RT 90]
END
```

The error message "SIZE is not a procedure" is not very helpful for a young student.

Pro: Remove unnecessary notation. Align with the majority of other programming languages that do not require a special character to declare variables.

Contra: Some may argue that other languages like PHP also have their unique designator for variables. In current Logo implementations, a symbol can be one of three, a property list, a variable, or a procedure. This will limit a symbol to be a property list, or either a variable or a procedure.

In the example above, 5 would be assigned to B (indirect assignment). Removing colons would remove that feature; so maybe colons could be made optional?

## MAKE vs LOCALMAKE

LOCALMAKE is actually syntactic sugar for `LOCAL "VAR MAKE "VAR value`.

This is an open topic for me. In a procedure context, the meaning of MAKE is ambiguous, because MAKE could reference both global and local variables. It may be easier to understand to restrict MAKE to global assignments, and LOCALMAKE to local assignments.

## Strings

Currently, Logo has up to three different string notations:

1. Single double quote: The "STRING notation does not allow special characters, such as spaces, without having to escape them. This makes it hard to create strings, and makes it difficult to determine where strings end.
2. Backtick notation: Terrapin Logo offers the backtick character as a string delimiter as in \`hello\`. This permits embedded spaces and other separators, and preserves case.
3. Vertical bars. This is actually not a real string delimiter. Vertical bars allow the definition of non-standard symbols like |hello world|. Some Logo dialects preserve case, while others don't. 
   
This is perfectly legal Logo:

```
TO |hello world|
    PR "HI\ TO\ YOU\ ALL!
END
```

Proposed change:

Add a new string delimiter, two singe quotes (Terrapin could add the backtick as well). Should we allow both single quotes and backticks?

```
'I said "two hamburgers"'
`I said 'two hamburgers'`
```

The end of a line is not treated as a string delimiter, permitting multiline strings.

For multiline strings, these two statements are equivalent:

```
MAKE "A 'One
Two'
MAKE "A "|One
Two|
```

### Embedded Variables

Strings can have embedded variables in the form `{NAME}` so people can output data more easily, instead of having to create strings with WORD or SENTENCE:

```
MAKE FIRST "Michael"
PR 'Good morning, {FIRST}!'
```

Alternatively, we can adopt the Javascript syntax, with a dollar sign preceeding the curly braces as in `PR 'Good morning, ${NAME}!'` to avoid confusion with the usage of curly brackets in a string.

## Variable Scope

Currently, local variables are available to called procedures:

```
TO A
    LOCALMAKE "AA "|Defined in A|
    B
END

TO B
    PR :AA
END
A
Defined in A
```

This is not only confusing, but also introduces strange and hard-to-find bugs. Also, it adds a level of complexity for compilers.

Solution: Limit variable lookups to the local procedure and the global namespace. Javascript does this; PHP goes even further, having to declare global variables as global before accessing them. This should not have any impact on current code, as this feature is widely unknown.

## Runlists

Changing the way runlists work is a difficult decision, and I am not sure that we actually need to do this.

Currently, this is legal code:

```
TO NOT.IF :cond :runlist
    IF NOT :cond :runlist
END
```

Runlists can be stored in variables and fed to branching code like IF. This prevents higher-level compiler optimization. The runlist must be compiled immediately before execution, because it may reference local variables, for example.

Pro: Easier compilation.

Contra: This touches a core feature of the Logo language, and I am not sure whether removing this feature is worth the change.

## DEFINE and TEXT

The DEFINE command lets you define a procedure on the fly; its input is a structured list of lists. The first sub-list is the list of formal inputs, while the others are the procedure lines.

Unfortuately, this makes compilation very complex:

```
TO RUN.ME :instructions
    DEFINE "TEMP LIST [] :instructions
    TEMP
    ERASE "TEMP
END
```

Action: Remove DEFINE

Pro: Easier compilation.

Contra: This is again a core feature of the Logo language, but probably rarely used.

Similarly, the TEXT command outputs the list notation of a defined procedure. People expect that TEXT outputs the same list as entered:

```
TO SQUARE :side
    REPEAT 4 [FD :side RT 90]
END
TEXT "SQUARE
Result: [[:side][REPEAT 4 [FD :side RT 90]]]
```

This requirement prevents quick procedure optimizations, like the replacement of inline operators with Logo commands:

```
TO FORMULA :a :b
    OP :a + 2 * :b
END
TEXT "FORMULA
Result: [[:A :B][OP :A + 2 * :B]]
```

If the TEXT command could be dropped, an analogous command could return different information, like, for example:

```
PROCDEF "FORMULA
Result: [:A :B][(OP (SUM :A (TIMES 2 :B)))]
```

Or whatever the implementation chooses to display.
