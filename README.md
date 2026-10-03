# intel-brew

Builds unofficial Homebrew bottles on Intel (`x86_64`) macOS 15 Sequoia using
GitHub-hosted Intel runners.

The default build list is stored in
[`.github/intel-bottles.json`](.github/intel-bottles.json). Edit its
`formulae` array to change what a normal manual run builds; the Actions workflow
does not need to be edited.

Formula-specific build behavior also lives in the configuration file under
`options`. Supported options are:

- `force_toolchains`: expose the runner-provided `go` or `rust` toolchain;
- `skip_dependencies`: do not install named Homebrew dependencies;
- `patches`: enable a known formula-source patch;
- `post_build`: run a known post-build repair before testing.

The workflow detects transitive Go and Rust build dependencies automatically.
It resolves every dependency in topological order and validates the exact keg,
Homebrew `opt` link, dynamic-library linkage, and Python `pip` entry point before
building the requested formula. This prevents a runner's preinstalled tools or
stale links from silently selecting the wrong dependency.

Validated bottles already published by this repository also act as the durable
dependency store. When an exact-version dependency bottle exists, the workflow
downloads it, verifies `SHA256SUMS`, installs it, and skips recompiling that
dependency. A source build is used only when no matching bottle exists. The
workflow intentionally does not cache Homebrew downloads because that would
save network transfer without avoiding the expensive compilation work.

The optional workflow input still accepts a JSON array for a one-off subset.
For example, `["certbot","harfbuzz"]` builds only those two formulae while
retaining any matching options from the configuration file.

## Release process

The manually triggered GitHub Actions workflow:

1. updates Homebrew;
2. restores matching dependency bottles from existing Releases, then builds
   only the remaining dependencies from source in topological order;
3. repairs and validates every dependency's keg and Homebrew-managed paths, and
   exposes the runner's Go or Rust toolchain to Homebrew's isolated build
   environment when required;
4. builds every requested formula from source with `--build-bottle` in a
   separate matrix job;
5. runs the formula test;
6. creates the bottle and its JSON metadata;
7. verifies the generated files and writes `SHA256SUMS`;
8. uninstalls the source build, reinstalls the generated bottle, and tests it;
9. publishes one verified GitHub Release per formula, tied to the workflow
   commit.

Release tags include the formula version, platform, workflow run number, and
attempt number, for example:

```text
redis-8.10.2-x86_64-sequoia-build.5.1
```

## Install a released bottle

Download the `.bottle.tar.gz` file and `SHA256SUMS` from the same
[GitHub Release](https://github.com/wuai1024/intel-brew/releases), then run:

```bash
shasum -a 256 -c SHA256SUMS
HOMEBREW_DEVELOPER=1 brew install ./*.bottle*.tar.gz
```

This repository is currently a bottle build and release project, not a
Homebrew tap. The released bottle therefore needs to be installed from its
downloaded file instead of with `brew tap`.
