# If Else

In this exercise, we'll learn all about code blocks and see how to write conditional code, sticking
to if-else for the time being.

In `exercise.spvasm`, we have the usual setup with `fancy_attribute`, `useful_output` and
`crucial_data`:

```c
useful_output = fancy_attribute;
crucial_data = props.highly_exclusive_bits;
```

We're going to turn that into the following:

```c
if (fancy_attribute < props.threshold)
{
    useful_output = 0;
    crucial_data = vec4(0);
}
else
{
    useful_output = 1.0;
    crucial_data = props.highly_exclusive_bits;
}
```

## Blocks

You have already seen the `OpLabel` instruction after which we have so far been writing code that
appears inside `main()`.  You have surely also noticed the `OpReturn` instruction at the end of the
function.  Everything from the `OpLabel` instruction to the `OpReturn` one has so far been one
"block".  A block is defined as a set of instructions starting with `OpLabel` and some form of
"jump" instruction like return, branch, switch, etc.

So first of all, `OpLabel` is the first instruction of every block.  This instruction is not just an
identifier for the block, but is also used as the jump target of other blocks.

Thinking of the blocks as nodes and the jumps as (directed) connections between them, a graph is
formed called the Control-Flow Graph.  SPIR-V also defines a "dominates" relationship between the
blocks which basically tells whether a block needs to execute before another.  This is nothing too
important for us to know at this point (or maybe ever), other than that SPIR-V requires the blocks
to be specified in order such that a block that dominates another (i.e. is executed before) is
written before it.  In simple terms, no backward jumps (other than for loops obviously).

In this exercise, we will have four blocks.  If you thought blocks are just going to be like in C,
here's where you know that's not true.  Recall that every block must start with an `OpLabel`
instruction and that's where other blocks can jump to.  Our four blocks will look like the
following (written in made-up pseudo-code):

```c
// Block 1: everything leading up to the branch, ending in branch.
block1:
condition = fancy_attribute < props.threshold;
if (condition) jump block2; else jump block3;

// Block 2: contents of the `if` case, ending in jump.
block2:
useful_output = 0;
crucial_data = vec4(0);
jump block4;

// Block 3: contents of the `else` case, ending in jump.
block3:
useful_output = 1.0;
crucial_data = props.highly_exclusive_bits;
jump block4;

// Block 4: everything after the if-else blocks, in this case just containing return.
block4:
return;
```

## An If-Else Construct

Now let's see.  We know how to write an `OpLabel` instruction (if not, look it up right now).  We
also know how to write the actual contents of the blocks (you _did_ do the previous exercises,
didn't you?).  We actually need only a handful of instructions to arrive at the full solution.

First off, the comparison operation used in the condition of the branch (floating point less than)
is implemented with the `OpFOrdLessThan` instruction.

> Waiting for you to consult the spec ...

What's the return type of that instruction, and how do we declare it?

> Waiting for you to check again ...

And what's the meaning of "Ord" in the instruction?  You might have noticed there's an "Unord"
version of the same instruction (and other similar ones).  The difference between the Ord and Unord
instructions is in the handling of NaN.  Ordered floating points can't be NaN, and we're satisfied
with that.

You can go ahead and implement the four blocks in order in `exercise.spvasm`, leaving the `if` and
`jump` instructions out.  You will end up with four blocks each starting with one `OpLabel`
instruction, but missing their terminating jump (except maybe the last block that already has
`OpReturn`)

> Waiting for you to complete this part of the exercise ...

Before implementing the if-else construct, let's look at the control flow again:

```
    block1
    /     \
   |       |
 true    false
   |       |
   V       V
block2   block3
     \   /
      \ /
       |
       V
    block4
```

To implement the if-else branch, SPIR-V provides the above graph through two instructions:

- One instruction specifies what block is reached after the if-else flow is finished; this block is
  referred to as the "merge" block,
- Another instruction specifies the if condition and the true and false blocks.

Note again that the blocks are identified by the id of their `OpLabel` instruction.  The first
instruction above is `OpSelectionMerge` and the second instruction is `OpBranchConditional`.  Go
ahead and look these up in the spec before implementing them in the exercise.

> Waiting for you to complete this part of the exercise ...

Finally, all that's left is to tell blocks 2 and 3 to jump to block 4 at the end.  This is done with
the `OpBranch` instruction.  You can now finish the exercise.

As always, remember to use `./validate` to validate your hand-written SPIR-V and see the
corresponding GLSL.  You can also see a summary of what each exercise does with:

```bash
$ diff exercise.spvasm solution.spvasm
```
