# Changelog

## Unreleased

- Bugfix: Fixed CMake not installing the executable into the correct directory. (#65)
- Dev: `crashpad-handler` is now installed when running `cmake --install` (into `<prefix-dir>/crashpad/`). (#14)
- Dev: Added support and CI for building with `clang-cl`. (#61)
