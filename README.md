# php74

[![Build Images](https://github.com/specsnl/php74/actions/workflows/main.yml/badge.svg)](https://github.com/specsnl/php74/actions/workflows/main.yml)

A PHP 7.4 (FPM and Apache) based Docker base image.

## Pulling the images

```
# FPM
docker pull ghcr.io/specsnl/php74:latest
docker pull ghcr.io/specsnl/php74/builder:latest
docker pull ghcr.io/specsnl/php74/builder_nodejs:latest

# Apache
docker pull ghcr.io/specsnl/php74/apache:latest
docker pull ghcr.io/specsnl/php74/apache/builder:latest
docker pull ghcr.io/specsnl/php74/apache/builder_nodejs:latest
```

The image name scheme: `ghcr.io/specsnl/php74[/{VARIANT}][/{TARGET}]:{VERSION}`

- **{VARIANT}**: omitted for FPM, otherwise `apache`
- **{TARGET}**: omitted for `runtime`, otherwise `builder` or `builder_nodejs` (also published as `node`)
- **{VERSION}**: `latest`, a release tag (i.e. `0.5.5`), `main` or `pr-<number>`

## Building the docker image(s)

There are multiple targets:

  - **runtime**: this is for *production*. It does not contain any development tools like Composer and Xdebug.
  - **builder**: this is for *development*. This is based on the runtime-target and it adds Composer, Xdebug etc.
  - **builder_nodejs**: this is for *development*. This is based on the builder-target and it adds NodeJS.

Building `runtime`-target:

```
docker build --tag ghcr.io/specsnl/php74:latest --file fpm/Dockerfile --target runtime .
```

Building `builder`-target:

```
docker build --tag ghcr.io/specsnl/php74/builder:latest --file fpm/Dockerfile --target builder .
```

Building `builder_nodejs`-target:

```
docker build --tag ghcr.io/specsnl/php74/builder_nodejs:latest --file fpm/Dockerfile --target builder_nodejs .
```

## Debian bullseye is EOL (`DEBIAN_SECURITY_SNAPSHOT`)

PHP 7.4 images are only published for Debian 11 (bullseye), which is past end-of-life. Its security
suite has stopped being maintained, and the mirrors have moved on in a way that breaks `apt-get`:

- `deb.debian.org` still serves the `bullseye-security` package index, but the `.deb` pool behind it
  has been purged — every package download returns a 404.
- `archive.debian.org`, where EOL suites normally end up, does not have `bullseye-security` yet. As
  of this writing it only goes up to `buster`.
- The final `Release` file (published 2026-08-31) carries a 7-day `Valid-Until` that has long since
  lapsed, so `apt` rejects the repository as expired even where it is reachable.

To still get the last published security updates, the runtime stage rewrites the `bullseye-security`
entry in `/etc/apt/sources.list` to point at [snapshot.debian.org](https://snapshot.debian.org),
which keeps every historical mirror state permanently. The build arg pins which state is used:

```
ARG DEBIAN_SECURITY_SNAPSHOT=20260901T000000Z
```

That timestamp sits just after the final security publish, so the image gets the complete, final set
of bullseye security updates. Because the snapshot is immutable, this does not break when Debian
finishes moving bullseye to `archive.debian.org`. The rewrite also disables `apt`'s freshness check
for that one repository — signatures are still verified, only the "is this recent?" assertion is
skipped, which is unavoidable for a deliberately frozen suite.

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
