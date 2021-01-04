# A Vector

In this exercise, we will create a new type (`vec4`) out of an existing type (`float`) and learn
about constants.

## The Vector Operator

First, we are going to add a new output variable to our shader.

```c
layout(location = 1) out vec4 crucial_data;
```

We already know how to do this, except for the `vec4` type.  That is where `OpTypeVector` comes in.
This instruction takes a basic type, for example one defined with `OpTypeFloat 32` and turns it into
a vector.  Look it up in the SPIR-V specification.

> Waiting for you to consult the spec ...

I suggest placing this instruction (giving a new id as its result) right after the `OpTypeFloat`
instruction.  Once done, please go ahead and add the above declaration in SPIR-V.  This is going to
be very similar to how `useful_output` was declared.

> Waiting for you to complete this part of the exercise ...

Need a hint?  Recall from the previous exercise that declaring a new variable requires:

- Declaring a type, if not already available
- Declaring a type pointer (`OpTypePointer`), if not already available
- Declaring a variable (`OpVariable`)
- Specifying the variable id in `OpEntryPoint`
- Adding a `Location` decoration to the variable (`OpDecorate`)

You should be able to see the above declaration in the output of `./validate` now.

## Constants

Next, let's give this `crucial_data` some crucial data!  How about a dash of blue?  Let's add the
(equivalent of the) following line to `main`:

```c
crucial_data = vec4(0.25, 0.25, 0.75, 1.0);
```

The assignment part is familiar, `OpStore` will take care of that.  But `OpStore` takes an id to
store into `crucial_data`, so we must declare that constant vector somewhere and give it some id.
There are two pieces to this puzzle.  First, each element of the vector is a constant itself;
`0.25`, `0.75` and `1.0`.  Then the vector is a composite of those constants.

Use `OpConstant` to declare the elements of the above vector.  This should be placed in the
declarations section of the SPIR-V, i.e. after the decorations (with `OpDecorate`) and before the
function declarations (with `OpFunction`).  Naturally, they should necessarily be placed after the
`float` type is declared!

Next, use `OpConstantComposite` to use the above constants and create a composite `vec4` constant
out of them.  Same placement restriction applies as `OpConstant`.

Finally, add an `OpStore` instruction inside `main` (right after the other `OpStore` for example) to
assign the new constant to the new variable we introduced in this exercise.

As always, validate your code with `./validate`.

> Waiting for you to complete this part of the exercise ...
