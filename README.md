# diff_rast

Our version of differentiable gaussian rasterizer

# Installation
Clone the repository and submodules with

```
git clone --recurse-submodules URL
```

For CUDA development, it is recommend to install with `BUILD_NO_CUDA=1`, which
will disable compiling during pip install, and instead use JIT compiling on your
first run. The benefit of JIT compiling is that it does incremental compiling as
you modify your cuda code so it is much faster than re-compile through pip. Note
the JIT compiled library can be found under `~/.cache/torch_extensions/py*-cu*/`.

```
BUILD_NO_CUDA=1 pip install -e .[dev]
```

If you won't touch the underlying CUDA code, you can just install with compiling:

```
pip install -e .[dev]
```

# Development

## Protect Main Branch over Pull Request.
It is recommend to commit the code into the main branch as a PR over a hard push, as the
PR would protect the main branch if the code break tests but a hard push won't.

The curret tests that will be triggered by PR:
- `.github/workflows/core_tests.yml`: Black formating.
- `.github/workflows/doc.yml`: Doc build.

Because we check for black formatting, it is recommend to run black before commit in the code:
```
black . diff_rast/ tests/ examples/ --check
```

## Build the Doc Locally
If you want to contribute to the doc, here is the way to build it locally. The source code of the doc can be found in `docs/source` and the built doc will be in `_build/`. Here are some examples on documentation: [viser](https://github.com/nerfstudio-project/viser/tree/main/docs/source), [nerfstudio](https://github.com/nerfstudio-project/nerfstudio/tree/main/docs), [nerfacc](https://github.com/KAIR-BAIR/nerfacc/tree/master/docs/source).

```
pip install -e .[dev]
pip install -r docs/requirements.txt
sphinx-build docs/source _build 
```



# Brief walkthrough

The main python bindings for rasterization are found by importing diff_rast

```
import diff_rast
help(diff_rast)
```

# clangd setup (for Neovim)

[clangd](https://clangd.llvm.org/) is a nice tool for providing completions,
type checking, and other helpful features in C++. It requires some extra effort
to get set up for CUDA development, but there are fortunately only three steps
here.

**First,** we should install a `clangd` extension for our IDE/editor.

For Neovim+lspconfig users, this is very easy, we can simply install `clangd`
via Mason and add a few setup lines in Lua:

```lua
require("lspconfig").clangd.setup{
    capabilities = capabilities
}
```

**Second,** we need to generate a `.clangd` configuration file with the current
CUDA path argument.

Make sure you're in the right environment (with CUDA installed), and then from
the root of the repository, you can run:

```sh
echo "# Autogenerated, see .clangd_template\!" > .clangd && sed -e "/^#/d" -e "s|YOUR_CUDA_PATH|$(dirname $(dirname $(which nvcc)))|" .clangd_template >> .clangd
```

**Third,** we'll need a
[`compile_comands.json`](https://clang.llvm.org/docs/JSONCompilationDatabase.html)
file.

If we're working on PyTorch bindings, one option is to generate this using
[`bear`](https://github.com/rizsotto/Bear):

```sh
sudo apt update
sudo apt install bear

# From the repository root, 3dgs-exercise/.
#
# This will save a file at 3dgs-exercise/compile_commands.json, which clangd
# should be able to detect.
bear -- pip install -e diff_rast/

# Make sure the file is not empty!
cat compile_commands.json
```

Alternatively: if we're working directly in C (and don't need any PyTorch
binding stuff), we can generate via CMake:

```sh
# From 3dgs-exercise/csrc/build.
#
# This will save a file at 3dgs-exercise/csrc/build/compile_commands.json, which
# clangd should be able to detect.
cmake .. -DCMAKE_EXPORT_COMPILE_COMMANDS=on

# Make sure the file is not empty!
cat compile_commands.json
```

**Known issues**

- The torch extensions include currently raises an error:
  `In included file: use of undeclared identifier 'noinline'; did you mean 'inline'?`
