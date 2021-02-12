# Welcome to the Next Gen of Logo!

This repo is an attempt to define the next generation of the Logo programming language.

Designed in 1967, Logo is still around, often as a teaching tool for students to learn programming. After all of these years, it has begun to show its age. Some design choices makes learning the language more difficult than it could be, and its inheritance from Lisp makes it not easy to compile.

Over the years, several Logo dialects have emerged, which often have incompatible implementations. I do not want to start mentioning all of the available dialects.

The only remaining commercial version is Terrapin Logo, which is loosely based on UCBLogo.

This relaunch of Logo should have these goals in mind:

1. Make it easier to learn
2. Make it easier to compile
3. Make it easier to integrate
4. Make it easier to use

The latter bullet point includes a unified definition of a standard runtime library covering the essential Logo commands.

## Repo Layout

This repo has the following layout:

### spec

The `spec` folder contains the language specification. This should not be a formal spec, but rather something that people can easily read and understand. Maybe with a separate file containing a more formal definition.

### lib

The `lib` folder contains the definition of the Logo standard library in reference format.

### src

The `src` folder will contain the reference implementation.

### logo

The `logo` folder will hopefully contain several sub-folders with example Logo programs.

## About Myself

I have been working with Logo ever since the August 1982 issue of BYTE magazine (which I still own). Since 1993, I have been the developer and maintainer of Terrapin Logo.
