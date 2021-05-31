# My First Variable

In this exercise, we will modify the SPIR-V by adding "variables".  This will have a visible effect
on the generated GLSL.

As a reminder, please have [the SPIR-V specification][SPIRV-spec] handy.

[SPIRV-spec]: https://www.khronos.org/registry/spir-v/specs/unified1/SPIRV.html

## A Global Variable

First, let's start with a simple global variable.  If you recall from the first exercise, SPIR-V
doesn't predefine any types, so we need to also define a type for this global variable.  Let's say
we want to have the following GLSL generated:

```c
float var;
```

We first need to define the float type itself.

### The Float Type

Find the definition of `OpTypeFloat` in the SPIR-V spec.

> Waiting for you to consult the spec ...

The table for `OpTypeFloat` says: `3 | 22 | Result <id> | Literal Width`.

- What are `3` and `22`?  Those are the length and opcode of the instruction.  See section _2.3.
  Physical Layout of a SPIR-V Module and Instruction_, _Table 2. Instruction Physical Layout_ if you
  are interested, but you can generally ignore those during these tutorials.
- When a field mentions `<id>`, it means that an id is expected (rather than some literal).
  Likewise, literals are marked with `Literal`.  You can click on these for their definitions!
- `Result <id>` is what gets written before `=` in the human-readable SPIR-V.  Note that this may
  not always be the first parameter.  If there is a `Result Type <id>`, that always comes first in
  the instruction layout.

Throughout these tutorials, you will encounter many instructions.  Translating from the SPIR-V spec
to human-readable SPIR-V is as such:

#### No Result

```
length | opcode | Param1 | ... -> OpX Param1 ...
```

#### Result, No Type

```
length | opcode | Result <id> | Param1 | ... -> %id = OpX Param1 ...
```

#### Result and Type

```
length | opcode | Result Type <tid> | Result <id> | Param1 | ... -> %id = OpX %tid Param1 ...
```

Back to `OpTypeFloat`, the human-readable form of the instruction is:

```swift
%float = OpTypeFloat 32    ; or %5 instead of %float, 5 being a new id
```

Place this right after (or right before) the `OpTypeVoid` line in `exercise.spvasm`.  Notes:

- You can consult `solution.spvasm` if you get stuck, though it's recommended that you learn through
  experimentation.
- The placement of the instruction matters.  SPIR-V requires an ordering of instructions, grouped in
  sections.  You can read about this in section _2.3. Physical Layout of a SPIR-V Module and
  Instruction_.
- You can always use `./validate` to verify your code.

Note: Basic types in SPIR-V must have unique ids, as different ids indicate different types.  There
cannot be two `OpTypeFloat 32` instructions for example.

### The Type Pointer

Basic types are useful in SPIR-V to indicate the type of intermediate results.  Variables are not of
a type, but rather are a "pointer" to a type.  A type pointer carries an extra piece of information
aside from the type it is pointing to; a _Storage Class_.  Look up `OpTypePointer` in the SPIR-V
spec.

> Waiting for you to consult the spec ...

Click on _Storage Class_ to see what values it can take.  For a global variable, we want the
`Private` storage class.  Hopefully, by now you should be able to figure out how to write the
instruction in human-readable SPIR-V.  Place this after the `OpTypeFloat` instruction:

```swift
%floatptr = OpTypePointer Private %float    ; again, feel free to use numerical ids
```

We will see more of type pointers in future tutorials.

### The Variable

We are now ready to define a variable.  Look for `OpVariable` in the SPIR-V spec.

> Waiting for you to consult the spec ...

This instruction has an optional argument (and thus a variable length).  We won't specify this
argument in this tutorial.  You should be able to write the instruction on your own now.  Place it
right after the `OpTypePointer` instruction.

Note: Consult `solution.spvasm` if you get stuck, but do so as a last resort.  You will learn much
better by try and error.

### Better GLSL

Did you remember to use `./validate` along the way to verify your code?  If not, now is the time to
do so.  Nice work declaring a variable.  Is the generated GLSL not fantastic?  No, the variable name
is basically the id, like `_7`, that's not nice.  Let's use `OpName` as seen in the previous
tutorial to give this variable a readable name.

Debug instructions are placed right after the "boilerplate" header, in this case `OpEntryPoint`.

```swift
     OpName %variable "fancy_variable"
```

Try `./validate` again and enjoy this little victory.
