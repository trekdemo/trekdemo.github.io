---
categories: [Testing, CI, GitHub Actions]
date: 2024-07-05
title: A primer to act
excerpt: |-
  Running your GitHub Actions locally, can speed up than having to commit/push every time you want to test
---

Every now and then, I have to tweak some <abbr title="Continuous Integration">CI</abbr>
workflows. In my last two jobs we use GitHub Actions to implement them.

## How to install `act`?

```sh
brew install act
```

## General usage

The help is pretty short and concise. See `act -h`.

```sh
act [event name to run] [flags]
```

## Validating the workflow file

If you want to make sure that, horizontal rule,

```sh
act pull_request --dryrun
```

## Running a single workflow file

```sh
act pull_request --workflows .github/workflows/file.yml
```

## Running a single job

Be careful, you can help multiple jobs with the same name. In that case `act` will run all of them.
To run a specific step from a specific workflow you can combine the `--workflows` and `--job` flags.

```yaml
# .github/workflows/file.yml
name: Pull Request
on:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: ruby/setup-ruby@v1
    - run: bundle install
    - run: bundle exec rake
```

```sh
act pull_request --workflows .github/workflows/file.yml --job test
```
