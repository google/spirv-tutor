# SPIR-V Tutorials

This is a repository of SPIR-V tutorials starting from the very basics.  It is expected that the
reader is familiar with basics of Vulkan GLSL.

This is not an officially supported Google product.

## Initial Setup

These tutorials are currently only supported on Linux.  Please run `./scripts/setup` for initial
setup.  It is expected that `spirv-as` and `spirv-val` are installed on the system (usually in a
`spirv-tools` package).  Additionally, it is recommended to have the `highlight` command installed
(from a package with the same name).

## How to Follow the Tutorials

Navigate to the directory of each tutorial and follow the instructions in the `README.md` there.
The tutorial directories are tagged with a number to indicate the intended order, for example:

    01 - Introduction to SPIR-V/
    02 - My First Change/
    03 - My First Variable/

In the tutorials, `README.md` will generally introduce a new set of SPIR-V instructions and include
an exercise to modify a provided SPIR-V by hand and verify the results.  Solutions are provided, but
it's strongly recommended not to be consulted until the exercise is complete.

**Warning:** passively reading the tutorials will quickly become boring.  Do get your hands dirty.

When opening the SPIR-V files, it is recommended to enable some syntax highlighting for readability.
With vim for example, you can install [this excellent plugin][1], otherwise `set ft=lisp` is an
improvement over no highlighting.

[1]: https://github.com/kbenzie/vim-spirv

## Troubleshooting

If issues with `highlight` are encountered, edit `./scripts/test_spirv`:

- Try changing `xterm256` to `ansi` if your terminal is not an XTERM.
- Get rid of the `highlight` path and live with monochrome GLSL.
