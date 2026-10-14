# Development

## Git workflow

We use a fork-based workflow. The steps are:

1. **Fork** [ShipSoft/aegir](https://github.com/ShipSoft/aegir) on GitHub.
2. **Clone** your fork locally:

    ```bash
    git clone git@github.com:YOUR_USERNAME/aegir.git
    cd aegir
    ```

3. **Add the official repository as a remote:**

    ```bash
    git remote add official https://github.com/ShipSoft/aegir.git
    ```

4. **Keep your fork up to date:**

    ```bash
    git fetch official
    git rebase official/main
    ```

5. **Create a branch for your work:**

    ```bash
    git checkout -b my-feature
    ```

6. **Push and open a pull request:**

    ```bash
    git push origin my-feature
    ```

    Then open a PR from your fork to `ShipSoft/aegir` on GitHub.

### Commit messages

- Write a short (< 72 characters) summary on the first line.
- Optionally add a blank line followed by a more detailed explanation.
- Code must at least compile before you open a PR.

See also the [commit conventions](https://www.conventionalcommits.org).

The commit will be used to automatically generate the changelog for all projects of the new software framework, including aegir, such that the commit conventions will be strictly enforced. 
Comments using the chore scope are excluded from the CHANGELOG.md by default.

## Hooks and linting

aegir uses [prek](https://github.com/j178/prek) hooks, with the tools taken
from the pixi `lint` environment. Install them once per clone:

```bash
pixi run install-hooks
```

This sets up a pre-commit hook for formatting and linting, and a commit-msg
hook that checks the Conventional Commits format. Run all hooks by hand with
`pixi run lint`.

C++ is also checked with clang-tidy. CI reports findings only on the lines a
PR changes. Check your changes locally with `pixi run clang-tidy-diff`. After
editing a header, run `pixi run clang-tidy` on the whole tree, because the
diff task cannot see header-only changes.

See [Linting & git hooks](../dev-guide/linting-and-hooks.md) for the setup
shared by all SHiP repositories, and
[CONTRIBUTING.md](https://github.com/ShipSoft/aegir/blob/main/CONTRIBUTING.md)
for the full contribution rules.


## C++ guidelines

See also the [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines).

### General

- **NEVER** use `using namespace` in headers.
- Prefer `static_cast`, `dynamic_cast`, or `reinterpret_cast` over C-style casts.
- Prefer range-based for loops.
- Prefer STL containers over ROOT containers.
- Prefer smart pointers over raw pointers.
- Consider using `auto` to reduce type verbosity.


## Python guidelines

See also the [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html).

### General

- **ALWAYS** use `ruff format` (for new code) and `ruff check`.
- **ALWAYS** place import statements at the top of the file.
- **NEVER** run non-trivial code at module top-level scope; hide it behind `if __name__ == "__main__":`.
- Prefer iterating over containers to manual indexing.
- Prefer `argparse` over `getopt`.
- Use f-strings or `str.format()` rather than `%`-formatting.
