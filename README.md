# Browseraptor Default Channel

The Browseraptor Channel is a Git-based plugin registry that distributes metadata about available plugins.

## Submitting your plugin

- Fork this repository.
- Add the details of your plugin, in alphabetical order,
  to the correct JSON file in the [repository directory][repodir].
- Create a PR and make sure you check [all the boxes][prtemplate].

[repodir]: https://github.com/livrasand/browseraptor_channel/tree/master/repository
[prtemplate]: https://github.com/livrasand/browseraptor_channel/blob/master/.github/PULL_REQUEST_TEMPLATE.md

## About

The `index.json` and `repository/*/*.json` files
contain lists of repositories and plugins for use with
[Browseraptor](https://github.com/livrasand/Browseraptor).
These source files are processed by a crawler
and compiled into a single file:
[channel_v1.json](https://github.com/livrasand/browseraptor_channel/channel_v1.json).
This is the source of all plugin available for installation via Browseraptor.