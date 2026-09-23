# intel-brew

Builds unofficial Homebrew bottles on Intel (`x86_64`) macOS 15 Sequoia using
GitHub-hosted Intel runners.

The default build list contains locally used formulae that are outdated and do
not currently have an official Intel bottle:

- `aliyun-cli`
- `redis`
- `rsync`
- `uv`
- `wget`

The workflow input accepts a JSON array, so a manual run can build a different
set without changing the workflow file.

## Release process

The manually triggered GitHub Actions workflow:

1. updates Homebrew;
2. builds every requested formula from source with `--build-bottle` in a
   separate matrix job;
3. runs the formula test;
4. creates the bottle and its JSON metadata;
5. verifies the generated files and writes `SHA256SUMS`;
6. uninstalls the source build, reinstalls the generated bottle, and tests it;
7. publishes one verified GitHub Release per formula, tied to the workflow
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
