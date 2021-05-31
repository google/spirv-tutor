# A Useful Shader

Let's take the exercise in the previous tutorial one step further and create a shader that does
something useful (albeit mundane).

## An Output Variable

In the previous exercise, we added an attribute to the vertex shader in `exercise.spvasm`.  In this
exercise, we will add an output of the same type:

```c
in float fancy_attribute;  // What we had
out float useful_output;   // What we are adding
```

Recall that basic types must be unique in SPIR-V, so the `OpTypeFloat 32` instruction shouldn't be
duplicated.  However, we need a new `OpTypePointer` and `OpVariable` for this variable.  We need a
new `OpTypePointer` because the storage class is different.  We need a new `OpVariable` because well
of course we are adding a new variable!

> Waiting for you to complete this part of the exercise ...

Is the output of `./validate` satisfactory?  Did you forget something?

## Load and Store

We briefly saw `OpLoad` in the previous exercise.  Now is the time to look at it, along with
`OpStore` in more depth.  First, look these two up in the spec.

> Waiting for you to consult the spec ...

Let's ignore the optional _Memory Operands_ parameter, though you could click on it if you are
curious.

`OpLoad` is one of many instructions you will encounter that redundantly specify the _Result Type_.
In this case, it must match the type pointed to by the _Pointer_ being loaded.  In fact, you have
already seen such an instruction before; `OpFunction`.  Maybe take a look at `OpFunction` in the
spec again, now that you know SPIR-V better.

So we have two variables in our SPIR-V at this point (defined with `OpVariable`), and they are both
pointers to the `float` type (one `Input`, one `Output`).  You should now be able to implement the
following by adding an `OpLoad` instruction (using a new id as result) followed by an `OpStore`
instruction, inserted right after the `OpLabel` instruction (i.e. inside our `main` function):

```c
useful_output = fancy_attribute;
```

Is `./validate` happy?  If you forgot something in the previous section (but there were no errors),
you should see an error message now.  Where did we see a similar one before?

> (If error:) Waiting for you to figure it out ...

See `solution.spvasm` if you are not successful.

## Locations

If you have ever written shader code for Vulkan, you might have noticed something missing in the
GLSL output from `./validate`.  Unlike OpenGL, where the driver assigns _locations_ to attributes
and varyings (shader interface variables), Vulkan requires that they be "qualified" in the shader
source (well, SPIR-V in the end).  In GLSL, this is written as:

```c
layout(location=4) in float fancy_attribute;
layout(location=2) out float useful_output;
```

In SPIR-V, the location "decoration" (SPIR-V parlance for GLSL's "qualification") is specified with
`OpDecorate`.

> Waiting for you to consult the spec ...

Click on _Decoration_ to see what decorations there are.  Unsurprisingly, the location decoration is
called `Location`.  As `OpDecorate` doesn't produce a result, it looks like this in human-readable
SPIR-V:

```swift
     OpDecorate %variable Location Value
```

Try adding a `Location` decoration for each variable in the shader (`fancy_attribute` and
`useful_output`).  These instructions must go after the debug instructions (e.g. `OpName`) but
before type/variable declarations (e.g. `OpFloatType`, `OpVariable` etc).

How does the generated GLSL (again, through `./validate`) look like?
