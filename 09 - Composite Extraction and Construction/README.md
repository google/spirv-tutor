# Composite Extraction and Construction

In this exercise, we'll learn how to extract components of a composite and stitch them together to
make new ones.  What is a composite?  It's a piece of data that's more complex than a basic type
(`int`, `float`); so structs, arrays, vectors and matrices and any combination of them are
composites.

In the previous exercise, we generated the following code:

```c
crucial_data = props.smoothness;
```

We're going to change that here, and generate:

```c
crucial_data = vec4(props.shininess.xy, props.smoothness.xy);
```

Which is really:

```c
crucial_data = vec4(props.shininess.x,
                    props.shininess.y,
                    props.smoothness.x);
                    props.smoothness.y);
```

There are two parts to this; extracting the `x` and `y` components out of the uniform values, and
compositing them to make a new `vec4`.

## Extraction

To extract a component out of a composite, you can use `OpCompositeExtract`.  This instruction is
somewhat similar to `OpAccessChain`, in that it takes a list of indices to drill down the composite
and reach the component of interest.

> Waiting for you to consult the spec ...

Did you notice the difference between the _Indexes_ of `OpCompositeExtract` and `OpAccessChain`?

> Waiting for you to check again ...

That's right, `OpCompositeExtract` takes literals as indices instead of ids.  That may seem
insignificant at first, because so far we have only used constants (`OpConstant`) as indices.
However, as you might have guessed by now, this means that you can use variables for example to
index an array with `OpAccessChain`, but not with `OpCompositeExtract`.  Take a look at the
description of _Indexes_ in `OpAccessChain` to see what restrictions apply!  If you are curious to
know how `someVec4Var[i]` can be implemented, take a look at `OpVectorExtractDynamic`.

Alright, we know the instruction now.  Let's get to coding.  First, modify `exercise.spvasm` to load
(`OpLoad`) both `props.shininess` and `props.smoothness`.  `props.smoothness` is already loaded, so
loading the other is a matter of copy paste.  Next, use `OpCompositeExtract` four times to extract
the `x` and `y` components of both `props.shininess` and `props.smoothness`.  In case it's not
obvious, `x` is component 0 and `y` is component 1!  You should have four ids now for these
extracted components.

> Waiting for you to complete this part of the exercise ...

## Construction

We have four `float` values now, extracted from the uniform buffer members.  We can construct a
`vec4` out of these using the `OpCompositeConstruct` instruction.  You should be pretty familiar by
now with the syntax, so this should be a breeze once you consult the spec.

Once the `vec4` is constructed, store (`OpStore`) it in `crucial_data` as usual.

> Waiting for you to complete this part of the exercise ...

The solution can be found in `solution1.spvasm`.  There's an alternative instruction that can be
used here, presented in `solution2.spvasm`.  Read the next section if you are interested.

## (Optional) An Alternative; Vector Shuffling

In the special case of a vector swizzle, or constructing one vector out of two other vectors where
components of one come before the components of another (as in this exercise), there's another
instruction that can be used to achieve the same results as all the instructions above.  That
instruction is `OpVectorShuffle`.  This instruction takes two vectors, logically concatenates them
to be a larger vector (of up to 8 components), then takes a list of literals as components to choose
from this logical vector.  This is _the_ way to implement swizzle (where both vectors are given the
same), but can come in handy in situations like in this exercise.

Try removing all the `OpCompositeExtract` and `OpCompositeConstruct` instructions and replacing them
all with a single `OpVectorShuffle` instruction.  `solution2.spvasm` has the answer.
