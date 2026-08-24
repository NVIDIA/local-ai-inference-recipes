<!--
SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: CC-BY-4.0 AND Apache-2.0
-->

# Local AI Inference Recipes

Tested, ready-to-run configurations for running local AI models on NVIDIA hardware.
Each recipe includes the exact model checkpoint, hardware configuration, inference backend, precision, prerequisites, and copyable commands needed to launch and test the model. In addition, the recipes contain sample expected performance on selected hardware.

## How to use

1. Browse the recipe catalog.
2. Filter by model, NVIDIA hardware, backend, and precision.
3. Select the recipe matching your exact hardware configuration.
4. Review the prerequisites and limitations.
5. Copy and run the setup, launch, and client commands.

## Run a recipe

1. Select your model and exact NVIDIA hardware configuration.
2. Review the prerequisites, limitations, and safety guidance.
3. Set any required environment variables, such as a scoped Hugging Face token.
4. Copy and run the setup command, when required.
5. Copy and run the launch command.
6. Use the provided client example to test the endpoint.

## Recommended configurations

`recommended_catalog.json` lists a curated set of standout recipes, grouped
by backend and operating system.

## Feedback
If a recipe appears incorrect or outdated, or you would like to request support for another model or hardware configuration, please open an issue.

## License

See [LICENSE](LICENSE) and [THIRD_PARTY_LICENSES](THIRD_PARTY_LICENSES).
