# Introduction to SPIR-V

SPIR-V is the language to describe shaders used with Vulkan.  Normally, these shaders are written in
a high-level language such as GLSL, but it may occasionally be necessary to look at, debug or modify
the corresponding SPIR-V.  These tutorials are aimed at introducing this language bit by bit and
making it easier to read.

The SPIR-V language is in SSA form, and is very much an Abstract Syntax Tree (with a heading).  This
means that every intermediate value is written to only once.  Think of it as a language where every
variable is `const`, much like how functional languages are.  For the first few tutorials, we will
be focusing on the heading.

During all these tutorials, please have [the SPIR-V specification][SPIRV-spec] handy.

[SPIRV-spec]: https://www.khronos.org/registry/spir-v/specs/unified1/SPIRV.html

## A Minimal SPIR-V

Let's take the boilerplate out of the way.  Here's the minimum required to have a valid shader (that
does nothing):

```swift
     OpCapability Shader
     OpMemoryModel Logical GLSL450
     OpEntryPoint Vertex %3 "main"
%1 = OpTypeVoid
%2 = OpTypeFunction %1
%3 = OpFunction %1 None %2
%4 = OpLabel
     OpReturn
     OpFunctionEnd
```

Let's go through this line by line:

```swift
     OpCapability Shader
```

It's a shader!  There are other capabilities, that the SPIR-V could declare up front, like the fact
that it may use 16-bit float instructions, or that it uses transform feedback.

```swift
     OpMemoryModel Logical GLSL450
```

Consider this boilerplate.  It declares that the shader uses logical addresses (as opposed to
physical) and that it uses the GLSL memory model.

```swift
     OpEntryPoint Vertex %3 "main"
```

This instruction (`OpEntryPoint`) declares an "entry point" in the shader module.  In this case, the
entry point is for a Vertex Shader, the function is identified by `%3` (more on this below), and the
name is `"main"`.  The name is used with `VkPipelineShaderStageCreateInfo` to identify this entry
point.

The SPIR-V may very well contain code for multiple shaders, possibly sharing some functions, and it
can have multiple entry points.

```swift
%1 = OpTypeVoid
```

SPIR-V uses "ids" to refer to everything that is declared, be it types, functions, variables,
intermediate values etc.  These ids are written as `%id` where `id` can be a number or a c-style
variable name.

This instruction is declaring the `void` type and giving it id `%1`.  As you can observe, SPIR-V
itself doesn't predefine any types.

As an exercise, look up `OpTypeVoid` in the SPIR-V spec.  Simply search for `OpTypeVoid` in the spec
until you find a link to the instruction, or the table that defines the instruction itself.  No need
to bother with the binary representation of the instruction.

```swift
%2 = OpTypeFunction %1
```

This instruction declares the function type `void (*)()` in C parlance.  `%1` was just declared
above as the `void` type, and that's the return type.  The function type itself is stored in a new
id `%2`.

What if a function type needed parameters?  Look up `OpTypeFunction` in the SPIR-V spec.

```swift
%3 = OpFunction %1 None %2
```

Finally, declare the function itself.  Observe the function type (`%2`).  The return type (`%1`) is
redundantly specified here.  This is a common pattern in SPIR-V where the "result type" of every
instruction is redundantly specified.

What is `None`?  Look up `OpFunction` in the spec, then click on "Function Control".


```swift
%4 = OpLabel
```

A label, marking the beginning of a code block.  Used for "jump" instructions.  You can ignore this
for now.

```swift
     OpReturn
     OpFunctionEnd
```

Self-explanatory instructions!

## Try it out

The above SPIR-V can be found in `exercise.spvasm`.  Try validating it (and generating the
corresponding GLSL) by running:

    $ ./validate

Neat!  If you want to see the output colored by `spirv-dis`, try:

    $ ../scripts/names_to_id exercise.spvasm

If you want to see (some of) the ids use "friendly names" instead of numbers, try:

    $ ../scripts/id_to_names exercise.spvasm
