# The Phi Function

You may recall (as if!) from exercise 01 that SPIR-V is in SSA form.  What does that mean?  TL;DR,
it means that every id is written to in exactly one place.  In C, you can modify a variable like
`x = x + 1`, which is not allowed in SSA form.  In SSA form instead, you can think of the `x` being
written to as a new variable; `x2 = x1 + 1`.

SSA form makes a lot of things easier for the optimizer, but we aren't going to get bogged down on
the details.  If you want to, you can read a bit about it [on Wikipedia][SSA].  Funnily enough,
SPIR-V isn't entirely in SSA form.  You might recall from the previous exercise that we wrote to the
same output variable in both the true and false blocks of an `if`!

So how is SPIR-V in SSA form?  It's true that every "intermediate" result is written to in exactly
one place; that would be the ids.  Writes to memory however (through `OpVariable`s) are allowed to
be done in multiple places.  This could be largely avoided using a special instruction called
`OpPhi` which is what this exercise is all about.  That said, you can write SPIR-V without ever
using `OpPhi`, so feel free to skip learning about the most interesting part of SPIR-V.

[SSA]: https://en.wikipedia.org/wiki/Static_single_assignment_form

## Local Variables

Before we get to `OpPhi`, let's actually see how we can avoid it.  Maybe then you could appreciate
its value.  Admittedly this will be a simple exercise, so I suggest an `x20` multiplier on the
appreciation.

In `exercise.spvasm`, you can find the solution to the previous exercise, which basically generates
the following code in `main()`:

```
if (fancy_attribute < props.threshold)
{
    useful_output = 0.0;
    crucial_data = vec4(0.0);
}
else
{
    useful_output = 1.0;
    crucial_data = props.highly_exclusive_bits;
}
```

As you can see, write to `crucial_data` is being done in both the true and false blocks of the `if`.
Now that's a simple `OpStore` instruction, but for a moment let's pretend something complicated was
supposed to happen to the data before it gets written to `crucial_data`.  Obviously we don't want to
repeat that in both blocks.  So let's try to assign to `crucial_data` only once outside the if-else
blocks (and pretend it was complicated).  In GLSL, there's really only one way to do that; use a
temporary variable:

```
vec4 temp;
if (fancy_attribute < props.threshold)
{
    useful_output = 0.0;
    temp = vec4(0.0);
}
else
{
    useful_output = 1.0;
    temp = props.highly_exclusive_bits;
}
crucial_data = temp;
```

Let's get that implemented.  We need to declare a new variable, which as you already know is done
with `OpVariable`.  Let's be civil and not declare a global variable here.  To declare a
function-local variable, you need to know the following:

- The storage class of a function-local variable is `Function`
- Function-local variables must be declared right after `OpLabel` at the beginning of the function.
  Note that it's not the beginning of the block that uses it, but the beginning of the function!  In
  terms of scope, thus, there are only two scopes in SPIR-V, the global scope and the function
  scope (unlike C or GLSL where every block introduces a new scope).

To summarize:

- Declare a function local variable `temp` (where does the corresponding `OpTypePointer` go?),
- Write the values that were previously written to `crucial_data` to `temp` instead,
- In the "merge" block of the `if`, load from `temp` and write to `crucial_data`.

> Waiting for you to complete this part of the exercise ...

If you have trouble completing this part, I suggest revisiting previous exercises.  If not, good job
champion!  You can find the solution to this part in `solution1.spvasm` for reference (and not
cheating).

## `OpPhi`

Now let's make things interesting.  We are going to remove the `temp` variable now and use the
SSA `OpPhi` instruction to do something you can't do in GLSL.

> Waiting for you to consult the spec even though I'm sure you already did out of habit ...

So `OpPhi` says "select a value based on which of the parent blocks was taken to reach this
instruction".  Mind = blown?

Attempting to put this in GLSL-ish, here's how we can make the code look like with `OpPhi`:

```
if (fancy_attribute < props.threshold)
{
    useful_output = 0.0;
}
else
{
    useful_output = 1.0;
    temp = props.highly_exclusive_bits
}
crucial_data = true_block_taken  ? vec4(0)
	       false_block_taken ? temp;
```

That last made-up syntax with question marks is what `OpPhi` does.  Let's do that now:

- Remove the `temp` variable you added in the previous section (and the `OpStore`s to it),
- In the "merge" block, instead of `OpLoad` from `temp`, use `OpPhi` to select one value or another
  based on which block was taken.  In this case, one of the values is a constant (which is declared
  globally, and not calculated in the if-else blocks, so yeah, that works)

It doesn't help that the spec says `OpPhi` selects a variable.  It's *not* selecting a variable (as
in something defined by `OpVariable`), but rather a value.

> Waiting for you to complete this part of the exercise ...

Now do the same for `useful_output`:

```
if (fancy_attribute < props.threshold)
{
}
else
{
    temp = props.highly_exclusive_bits
}
useful_output = true_block_taken  ? 0.0
	        false_block_taken ? 1.0;
crucial_data = true_block_taken  ? vec4(0)
	       false_block_taken ? temp;
```

> Waiting for you to complete this part of the exercise ...

Good job!  We saved a number of instructions this way, avoiding `OpStore` and `OpLoad` instructions.
Now imagine how many `OpStore` instructions you could save by using that on a huge switch case!

Most importantly though, we delivered on the SSA promise that everything is written to in exactly
one place.  The optimizer shall rejoice in this knowledge.

You can find the solution to this part in `solution2.spvasm` (cheating allowed, but only this time).

## (Optional) Eliminating The Empty Block

It's pretty annoying that we now have an empty block.  Can we eliminate that?  Why would I ask if
the answer was "no"?

Here are a couple of hints:

- `OpBranchConditional` takes two blocks to jump to in case of the condition being true or false.
  Either of those can be the merge block!  In this example, you can have the true case select the
  merge block as the jump target and delete the empty true block altogether.  Notice how there's
  only an "else" block now (which is impossible in GLSL)?  That's a clever way of avoiding one
  `OpLogicalNot` on the condition!
- `OpPhi` considers the block before `OpBranchConditional` (which is called the "header" block) as a
  parent too.  Using the id of that block effectively means "if none of the other parent blocks were
  taken".

> Waiting for you to complete this part of the exercise ...

Nice!  The solution to this part can be found in `solution3.spvasm`.  Did you notice how no matter
what we do here, `./validate` ends up producing the same GLSL?  This stuff is just something GLSL
can't do.
