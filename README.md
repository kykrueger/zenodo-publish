# Deprecated
This action was broken by the major Zenodo release of Oct 2023.

# zenodo-publish
Publish a zenodo draft with this github action.

This action:

1. Replaces the metadata of a deposition draft with the contents of .zenodo.metadata using the [json format from Zenodo's API](https://developers.zenodo.org/#deposit-metadata)
1. Removes files already attached to the deposition, since unwanted ones might have been carried from the last version.
1. Uploads files and paths found in the provided whitelist file; `whitelist_uploads` defaults to `.zenodo_whitelist_uploads`.
1. Publishes the draft

When first setting up this action, you will need an existing Deposition which belongs to your Zenodo account.
If you already have a Deposition that you were managing manually, or with the Github plugin, you should set up the [zenodo-new-version](https://github.com/kykrueger/zenodo-new-version) action in your workflow to run before this one. Creation of a new version is kept in a separate action to ensure that users of the action have a chance to use the pre-reserved DOI of the new version in other actions before publishing. An example use case is to 
1. Run the zenodo-new-version action which
    1. Creates a new draft of your deposition
    1. Prereserves a DOI
    1. Optionally replaces the prior DOI with the pre-reserved one
1. Create a PDF or other output format from the template.
1. Run this action

If you are setting up a new deposition, it is easiest to:
1. Start the creation of a new publication on zenodo.org
1. Add the required fields
1. Prereserve a DOI in the UI
1. Add the prereserved DOI where needed in your documents
1. Run this action
1. Add the zenodo-new-version action to your workflow to create a draft before running this action in following runs.

A basic example using this action is the following which creates a new version of a zenodo deposition, and publishes every time that a txt md or csv file are changed in the main branch of the repository.
``` yaml
name: Basic Workflow

on:
  push:
    branches: [ main ]
    paths:
      - '**.txt'
      - '**.md'
      - '**.csv'
  workflow_dispatch:
env:
  zenodo_deposition_id: <YOUR DEPSITION ID HERE>
  zenodo_server: 'https://sandbox.zenodo.org'
jobs:
  new_version:
    name: Create new version of zenodo deposition
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: kykrueger/zenodo-new-version@v1.0.1
        with:
          zenodo_deposition_id: ${{ env.zenodo_deposition_id }}
          zenodo_server: ${{ env.zenodo_server }}
          zenodo_token: ${{ secrets.ZENODO_SANDBOX_TOKEN }}
      - uses: kykrueger/zenodo-publish@v1
        with:
          zenodo_deposition_id: ${{ env.zenodo_deposition_id }}
          zenodo_server: ${{ env.zenodo_server }}
          zenodo_token: ${{ secrets.ZENODO_SANDBOX_TOKEN }}
      - uses: EndBug/add-and-commit@v7.1.1
        with:
          message: 'published to Zenodo'
```
