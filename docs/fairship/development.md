# Development

## Git workflow

We use a fork-based workflow. If you are new to Git or GitHub, read the
[Git tutorial for SHiP](https://github.com/ShipSoft/FairShip/wiki/Git-Tutorial-for-SHiP)
first. The steps are:

1. **Fork** [ShipSoft/FairShip](https://github.com/ShipSoft/FairShip) on GitHub.
2. **Clone** your fork locally:

    ```bash
    git clone git@github.com:YOUR_USERNAME/FairShip.git
    cd FairShip
    ```

3. **Add the official repository as a remote:**

    ```bash
    git remote add official https://github.com/ShipSoft/FairShip.git
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

    Then open a PR from your fork to `ShipSoft/FairShip` on GitHub.

### Commit messages

Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/),
for example `fix(veto): correct the SBT segment numbering` or
`docs: update the README`. The `commit-msg` hook installed below checks
this when you commit, and CI checks it again on the pull request.

- Keep the summary line short (< 72 characters).
- Optionally add a blank line followed by a more detailed explanation.
- Code must at least compile before you open a PR.
- Add an entry to the `Unreleased` section of `CHANGELOG.md` for any
  change users or developers will notice. The pull request template asks
  for it.

### Linting and git hooks

FairShip runs its linters through [prek](https://github.com/j178/prek)
with the tools from the pixi `lint` environment: ruff, clang-format,
cpplint, codespell, pyupgrade, mypy, reuse, gersemi, yamlfmt and
commitizen. Install the hooks once per clone and run them on the whole
tree with:

```bash
pixi run install-hooks
pixi run lint
```

See [Linting & git hooks](../dev-guide/linting-and-hooks.md) for details.

### AI-assisted contributions

AI-assisted contributions are welcome under the rules in
[`AI_POLICY.md`](https://github.com/ShipSoft/FairShip/blob/main/AI_POLICY.md).
In short: say which tool or model you used, review and understand
everything you submit, and answer reviewers yourself. AI tools cannot be
co-authors; credit them with a trailer instead:

```text
Assisted-by: <harness>:<model>
```

## C++ guidelines

The FairShip wiki has the full [C++ guidelines](https://github.com/ShipSoft/FairShip/wiki/CPP-guidelines).
See also the [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines).

### General

- **NEVER** use `using namespace` in headers.
- Prefer `static_cast`, `dynamic_cast`, or `reinterpret_cast` over C-style casts.
- Prefer range-based for loops.
- Prefer STL containers over ROOT containers.
- Prefer smart pointers over raw pointers.
- Consider using `auto` to reduce type verbosity.

### FairRoot/FairShip specific

- Use `LOG(SEVERITY)` macros instead of `std::cout` or legacy `FairLogger` methods.
- Use `TFile::Open` rather than direct `TFile` constructors.
- Prefer STL vectors and `RVec` over `TVectorT`.
- Prefer `std::string` over `const char*` and `TString`.

## Python guidelines

The FairShip wiki has the full [Python guidelines](https://github.com/ShipSoft/FairShip/wiki/Python-guidelines).
See also the [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html).

### General

- **ALWAYS** use `ruff format` (for new code) and `ruff check`.
- **ALWAYS** place import statements at the top of the file.
- **NEVER** run non-trivial code at module top-level scope; hide it behind `if __name__ == "__main__":`.
- Prefer iterating over containers to manual indexing.
- Prefer `argparse` over `getopt`.
- Use f-strings or `str.format()` rather than `%`-formatting.

### FairRoot/FairShip specific

- Prefer STL vectors, `RVec`, or `GenVector` over `TVectorT`.
- Prefer STL containers or `RTensor` over `TMatrix`.
- Use `len()` instead of `.size()` and `.GetEntries()` (supported for STL containers, `TCA`s, `TTree`s, Pythia8 events, and many other data types).

## Getting help

Contact [ship-software@cern.ch](mailto:ship-software@cern.ch).
