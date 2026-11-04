# php74

[![Build Images](https://github.com/specsnl/php74/actions/workflows/main.yml/badge.svg)](https://github.com/specsnl/php74/actions/workflows/main.yml)

A PHP 7.4 (FPM and Apache) based Docker base image.

## Pulling the images

```
# FPM
docker pull ghcr.io/specsnl/php74:latest
docker pull ghcr.io/specsnl/php74/builder:latest

# Apache
docker pull ghcr.io/specsnl/php74/apache:latest
docker pull ghcr.io/specsnl/php74/apache/builder:latest
```

The image name scheme: `ghcr.io/specsnl/php74[/{VARIANT}][/{TARGET}]:{VERSION}`

- **{VARIANT}**: omitted for FPM, otherwise `apache`
- **{TARGET}**: omitted for `runtime`, otherwise `builder`
- **{VERSION}**: `latest`, a release tag (i.e. `0.5.5`), `main` or `pr-<number>`

The `builder` image is also published under its old names, `builder_nodejs` and `node`, from the time Node.js had its own
target. These names are deprecated and will be removed ([specsnl/github-actions#76](https://github.com/specsnl/github-actions/issues/76));
use `builder` instead.

## Building the docker image(s)

There are multiple targets:

  - **runtime**: this is for *production*. It does not contain any development tools like Composer and Xdebug.
  - **builder**: this is for *development*. This is based on the runtime-target and it adds Composer, Xdebug, Node.js etc.

Building `runtime`-target:

```
docker build --tag ghcr.io/specsnl/php74:latest --file fpm/Dockerfile --target runtime .
```

Building `builder`-target:

```
docker build --tag ghcr.io/specsnl/php74/builder:latest --file fpm/Dockerfile --target builder .
```

## Debian bullseye is EOL

PHP 7.4 images are only published for Debian 11 (bullseye), which is past end-of-life. Its security
suite has stopped being maintained, and `deb.debian.org` still serves the `bullseye-security` package
index while the `.deb` pool behind it has been purged — every package download returns a 404.

To still get the final published security updates, the runtime stage rewrites the `bullseye-security`
entry in `/etc/apt/sources.list` to point at `archive.debian.org`, where Debian keeps EOL suites
permanently. The archived `Release` file carries no `Valid-Until`, so no `apt` freshness override is
needed.

A `grep` guard follows the rewrite so the build fails loudly if the base image ever changes its
`sources.list` format, rather than silently falling back to building without security updates.

Do not remove this in a cleanup: without it, `apt-get install` fails outright.

## Task commands

Available [Task](https://taskfile.dev/#/) commands:

```
* build:              Build all PHP Docker image targets of the FPM and Apache
* generate:           Generate all Dockerfiles from Dockerfile.tmpl
* lint:               Apply a Dockerfile linter to all Dockerfiles
* build:apache:       Build all PHP Docker image targets of the Apache variant
* build:fpm:          Build all PHP Docker image targets of the FPM variant
* install:hooks:      Install git hooks (pre-commit regenerates and stages Dockerfiles from template)
* lint:apache:        Apply a Dockerfile linter (https://github.com/hadolint/hadolint)
* lint:fpm:           Apply a Dockerfile linter (https://github.com/hadolint/hadolint)
* remove:hooks:       Remove installed git hooks
* shell:apache:       Interactive shell
* shell:fpm:          Interactive shell
```
