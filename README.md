# Local AI Inference Recipes

This repository contains recommended recipes for running models on NVIDIA local hardware. Each recipe contains the exact model checkpoint, inference backend and recommended settings.

⚠️ Note this is a work in progress and is actively being developed.

Files are Apache-2.0 licensed (see `LICENSE`) unless otherwise specified
in the file — for example, the `.yaml` and `.html` files are NVIDIA
Proprietary.

## Run a recipe

1. Select your model and exact hardware configuration.
2. Review the prerequisites, limitations, and safety guidance.
3. Set any required environment variables, such as a scoped Hugging Face token.
4. Copy and run the setup command, when required.
5. Copy and run the launch command.
6. Use the provided client example to test the endpoint.

## Recommended configurations

`recommended_catalog.json` lists a curated set of standout recipes, grouped by backend and operating system.

## Feedback

This repository does not accept external contributions (no pull requests).

If a recipe appears incorrect or outdated, or you would like to request support for another model or hardware configuration, please open an issue.
