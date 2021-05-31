# Arithmetics

Let's take a break and do something simpler in this exercise: math.  So far we learned how to
declare stuff and move data around.  But what if we wanted to actually do something?  Luckily doing
things is simpler than setting things up!

In this exercise, you are pretty much free to do whatever you want.  In `exercise.spvasm` you can
find a fragment shader to work with that produces the following code inside `main()`:

```c
beautiful_color = fancy_varying;
physicsy_properties = props.smoothness;
```

I suggest modifying the SPIR-V to produce the following instead:

```c
beautiful_color = fancy_varying * (vec4(1) - props.shininess);
physicsy_properties = -props.smoothness / props.shininess;
```

Here are a few pointers:

- `OpConstant` is used to create constants,
- `OpFNegate` can be used to negate floating point scalars and vectors
- `OpFSub` subtracts floating point scalars and vectors,
- `OpFMul` multiplies floating point scalars and vectors (per component),
- `OpFDiv` divides floating point scalars and vectors (per component),

Fancy doing something more complicated?  By all means go for it.  Here are a couple more pointers
to create something even more interesting:

- `OpTypeMatrix` is used to create matrix types; why not add a matrix field to our buffer object?
- `OpMatrixTimesVector` and `OpVectorTimesMatrix` multiply a floating point matrix and a vector,
- `OpDot` produces the dot product of two floating point vectors,
