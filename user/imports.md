---
title: Importing shared build configuration

layout: en
---

The main source of configuration for your build is the `.travis.yml` file stored in your repository. You can import up to five configuration snippets into your `.travis.yml`, so you can update multiple repositories making only one change.

> BETA Configuration imports is currently in beta. Please leave feedback on the [Community forum](https://travis-ci.community/c/early-releases).
{: .beta }

## Example

Instead of specifying which versions of Ruby to test against in multiple files, you can define them in a snippet:

```yaml
rvm:
  - 2.5
  - 2.6
```
{: data-file="rubies.yml"}

You can then `import` that snippet into your `.travis.yml`. The following configuration imports the file `rubies.yml` from the `main` branch of the `shared-configs` repository of the `travis-ci` account:

```yaml
import: travis-ci/shared-configs/rubies.yml@main
script: bundle exec rake
```
{: data-file=".travis.yml"}

Resulting in the following configuration:

```yaml
rvm:
  - 2.5
  - 2.6
script: bundle exec rake
```
{: data-file=".travis.yml"}

## Import key

You can import up to 5 YAML sources/snippets into one `.travis.yml`, and you can of course reuse those snippets in multiple repositories.

The format of the `import` source is `<account>/<repository>/<path>@<ref>` in whic `<ref>` can be any valid Git reference, such as a commit sha, branch name, or tag name. If you do not specify `<ref>`, Travis CI will fetch the latest version of the default branch in the repository.

> Public repositories can import sources from public repositories, but not private repositories. Private repositories can import sources from both public and private repositories. See [private repositories](#private-repositories) for more information.

Import a single snippet:

```yaml
#import a single file
import: travis-ci/build-configs/rubies.yml@main
```
{: data-file=".travis.yml"}

Import multiple snippets:

```yaml
# import multiple files
import:
  - travis-ci/build-configs/rubies.yml@adf1235
  - travis-ci/build-configs/other.yml@v1
```
{: data-file=".travis.yml"}

By default, if the same keys are present in multiple imported snippets, they are overwritten with the last configuration imported. But you can also specify [deep_merge](#merge-mode).

## Merge mode

The default merge `mode` is `merge` which performs a shallow merge.
This means that root level sections (keys) defined in your `.travis.yml` will be overwritten by root level sections (keys) that are also present in the imported yaml file.
Merge modes are specific to each source.

You can also use `deep_merge`:

* `deep_merge` recursively merges sections (keys) that hold maps.

```yaml

# shallow merge - this one is the default if no merge mode is specified
- source: travis-ci/shared-config/hola.yml@main
  mode: merge

#
- source: travis-ci/shared-configs/other.yml@main
  mode: deep_merge
```
{: data-file=".travis.yml"}


## Importing configurations from the same repository

When importing configurations stored in the same repository as your `travis.yml`, you can ommit the `<account>/<repository>` and use only the path preceeded by `./`:

```yaml
# local imports fetch the same git commit ref
import:
- ./one.yml
- ./other.yml
```
{: data-file=".travis.yml"}

By default, configurations imported from the same repository, fetch the commit you’re currently building. This is intended to help while you are  creating and testing the shared configurations.
You can override this value by specifying a valid Git reference.

## Private repositories

Before sharing configurations **from** a public repository you need to toggle the *Allow importing config files from this repository.* setting in *More options* > *Settings* > *Config Import*. Only repositories owned by the same organisation will be able to access the configuration snippets.

## Include precedence 

When triggering a build through the Travis API or the web UI, the order of ascending precedence is:

- Config from the API build request payload, if given
- Config from `.travis.yml`
- Imported configs from the API build request payload, if given, in the order listed
- Imported configs from `.travis.yml`, in the order as listed