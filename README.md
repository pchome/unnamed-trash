# The Meson Build System Survival Horror Game

A robot trying to modify/mutate your project. If your code will survive, it will be completely different code ...

## About
As-is, incomplete, useless but fun ... :)

Using [clang-tydy](https://clang.llvm.org/extra/clang-tidy/) linter tool's `-fix` option, the script running every [check](https://clang.llvm.org/extra/clang-tidy/checks/list.html) from provided set separately, auto-commit changes (if any) and auto-generate final patch-set.

Afterwards, try to reanimate your code and apply another set of checks. Better works on big projects, resuling patch-set can weight several MB, contain insane changes and still compile.

Make sure you moved final result to /dev/null, if you not the author of the code.

## How to use
**NOTE**: Use on separate copy, which you can safely remove/modify!

1. Copy ontents of `meson.build.template` and `meson_options.txt.template` files into projects `meson.build` and `meson_options.txt` accordingly.
2. Create `compile_flags.txt.in` file with very basic compiler flags list (`@PROJECT_PATH@` will be replaced with absolute path to source root).
3. Copy `clang-tidy-genpatch.sh.in` and `clang-tidy.sh.in` into projects source root.
4. Commit changes, or add excludes to `.gitignore` file.
5. Init build directory (`meson build`) (and maybe make build at least once to generate all required sources).
6. Goto build directory and run e.g. `sh clang-tidy-genpatch.sh modernize` (this will run all `*modernize*` checks).
7. Wait for `clang-tidy-patches/*.patch` and `clang-tidy-logs/*.log`.

## Additional
`compile_flags.txt` when placed in sources root directory can be used by `clang-tidy` lauched directly e.g.`clang-tidy src/filename.cpp`.
As well as `.clang-tidy` file to control used "checks".
Same for `clangd` and Language Server plugin in IDE.

## Requirements
1. clang (clang-tidy/clangd) >= 8.0

## Example for D9VK (big enough project, should produce more output/results)
**NOTE**: Wine >= 3.14 required for winelib build!

For steps:
2. Copy `compile_flags.txt.in.winelib` example file into `compile_flags.txt.in`
5. Init script example:

```sh
#!/bin/sh

LINT_PATH="/tmp/d9vk-mutate"

# Unity builds are faster, regular builds can be more accurate
meson "${LINT_PATH}" --cross-file build-wine64.txt \
  --unity=on \
  -Denable_d3d10=false \
  -Denable_d3d11=false \
  -Denable_dxgi=false \
  -Denable_lint=true \
  -Dbuildtype=plain

cd "${LINT_PATH}"

# Fix for clang-tidy
# --no-gnu-unique flag forced for GCC by the project, but unsupported by Clang
sed -i -r -e 's/--no-gnu-unique//' "$TESTING_PATH/compile_commands.json"

# Firs build required to generate d3d*/dxgi shader headers
ninja

echo -e "\n\n"
echo "Done. NOTE: You can manually modify ${LINT_PATH}/compile_commands.json to reduce check time."
```
