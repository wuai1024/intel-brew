# intel-brew

Builds an unofficial Homebrew bottle for `tree` on Intel (`x86_64`) macOS 15
Sequoia using a GitHub-hosted Intel runner.

## Release process

The manually triggered GitHub Actions workflow:

1. updates Homebrew;
2. builds `tree` from source with `--build-bottle`;
3. runs the formula test;
4. creates the bottle and its JSON metadata;
5. verifies the generated files and writes `SHA256SUMS`;
6. uninstalls the source build, reinstalls the generated bottle, and tests it;
7. publishes the verified files in a GitHub Release tied to the workflow commit.

Release tags include the formula version, platform, workflow run number, and
attempt number, for example:

```text
tree-2.3.2-x86_64-sequoia-build.1.1
```

## Install a released bottle

Download the `.bottle.tar.gz` file and `SHA256SUMS` from the same
[GitHub Release](https://github.com/wuai1024/intel-brew/releases), then run:

```bash
shasum -a 256 -c SHA256SUMS
brew install ./tree--*.bottle*.tar.gz
```

This repository is currently a bottle build and release project, not a
Homebrew tap. The released bottle therefore needs to be installed from its
downloaded file instead of with `brew tap`.
