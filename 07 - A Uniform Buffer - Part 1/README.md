# A Uniform Buffer (Part 1)

Let's be honest, `vec4(0.25, 0.25, 0.75, 1.0)` in the previous exercise did not look like crucial
data to pass to the fragment shader.  Let's ditch that in this exercise, and build towards a uniform
buffer.  We will dedicate this exercise to just declaring one, and have the next exercise use it.

## Declaring a Structure

In SPIR-V, uniform buffers, storage buffers and I/O blocks are just structures that are decorated
especially.

Let's first then just declare a structure:

```c
struct object_properties
{
    vec4 shininess;
    vec4 smoothness;
};
```

We already have the `vec4` type handy, marvelous.  Now we just need to create a new struct type that
holds two `vec4`s.  Uninspiringly (and that's a good thing), the necessary instruction is called
`OpTypeStruct`.  Look it up in the spec, and add the type declaration to the SPIR-V.  I recommend
placing it right after the `vec4` type declaration.

As always, verify your code with `./validate`.

> Waiting for you to complete this part of the exercise ...

You can probably guess how to set the debug name `object_properties` for the struct.  But, how would
you set debug names to its members?  That's `OpMemberName`, do give it a brief look in the spec.
For example:

```swift
OpMemberName %struct_type 1 "smoothness"
```

## Turning the Struct Into a Uniform Buffer

Declaring the struct was easy, but we really want a uniform buffer.  What's importantly different
between the two is that the uniform buffer has a memory backing, so the SPIR-V should know exactly
where to find each member of the uniform buffer.  If you have heard of `std140` and `std430`, you
know what I'm talking about.  If not, you really should look that up!

So we need a host of new decorations:

- Use `OpDecorate` with the `Block` decoration to indicate that the struct type is special
- Use `OpMemberDecorate` with the `Offset` decoration to lay each member out in memory by giving
  them byte offsets.  I suggest offset 0 for the first member and offset 16 for the second.  In case
  it's not obvious, you should place `OpMemberDecorate` instructions near where `OpDecorate`
  instructions are.

We also need a new variable of this type, otherwise the declaration is pointless.  Declare a new
variable, much like `useful_output` and `crucial_data`, except they are of this struct type **and**
their _Storage Class_ is `Uniform` (instead of `Output` or `Input` as we previously used).

Give that a whirl with `./validate`.

> Waiting for you to complete this part of the exercise ...

If you are stuck, you can take a peek at `solution.spvasm`, but don't cheat or you think you are
learning but you are not.

> Waiting for you to really complete this part of the exercise ...

Let's experiment a bit too.  What happens if you set both offsets (in `OpMemberDecorate`
instructions) to 0?  What error does `spirv-val` (the command used to validate the SPIR-V in
`exercise.spvasm`, called from `./validate`) generate?  What if you set the first offset to 0 and
the second to 10?  What about 0 and 32?

## Set and Binding Qualifications

Just like interface variables require a `location` qualification (GLSL's term for SPIR-V's
decorations) for the shader to be usable with Vulkan, so do uniform buffers require `set` and
`binding` qualifications.  If you don't know what descriptor sets and bindings are, look them up in
the Vulkan spec.

How about we make the uniform buffer look like this?

```c
layout(set = 2, binding = 3) uniform object_properties
{
    vec4 shininess;
    vec4 smoothness;
} props;
```

As you might have guessed since "decorations" were mentioned, these are set with an `OpDecorate`
instruction.  For GLSL's `set`, use the `DescriptorSet` decoration and for `binding`, use `Binding`.
Otherwise the instructions look pretty similar to the ones we already have for `Location`.  Also,
remember that it's the variable that's being decorated here not the type!

Congratulations, `./validate` should now show this decoration.

Did you happen to encounter a validation error with `OpEntryPoint`?  Before SPIR-V 1.4 (we are using
SPIR-V 1.0 here) only the `Input` and `Output` variables need to be declared in `OpEntryPoint`, so
the uniform buffer variable shouldn't be placed there!

> Waiting for you to complete this part of the exercise ...
