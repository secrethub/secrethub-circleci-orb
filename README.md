<p align="center">
  <a href="https://circleci.com">
    <img src="https://upload.wikimedia.org/wikipedia/commons/8/82/Circleci-icon-logo.svg" alt="CircleCI" width="110">
  </a>
  <img width="50px"/>
  <a href="https://secrethub.io">
    <img src="https://secrethub.io/img/secrethub-logo-shield.svg" alt="SecretHub" width="96">
  </a>
</p>
<h1 align="center">
  <i>CircleCI Orb</i>
</h1>

[![CircleCI](https://circleci.com/gh/secrethub/secrethub-circleci-orb.svg?style=shield)](https://circleci.com/gh/secrethub/secrethub-circleci-orb)
[![Version]( https://img.shields.io/github/release/secrethub/secrethub-circleci-orb.svg)](https://github.com/secrethub/secrethub-circleci-orb/releases/latest)
[![Discord](https://img.shields.io/badge/chat-on%20discord-7289da.svg?logo=discord)](https://discord.gg/NWmxVeb)

> [SecretHub](https://secrethub.io) is an end-to-end encrypted secret management service that helps developers keep database passwords, API keys, and other secrets out of source code.

Use this orb to load secrets from SecretHub into your CircleCI jobs.

## Usage

To execute a command with secrets loaded from SecretHub, install the CLI and set it as the `shell` of the `run` command:

```yml
version: 2.1
orbs:
  secrethub: secrethub/cli@v0.1.0

jobs:
  deploy:
    docker:
      - image: cimg/base:stable
    steps:
      - secrethub/install
      - checkout
      - run:
          shell: secrethub run -- /bin/bash
          environment:
            AWS_ACCESS_KEY_ID: secrethub://company/app/aws/access_key_id
            AWS_SECRET_ACCESS_KEY: secrethub://company/app/aws/secret_access_key
          command: |
            echo "This value will be masked: $AWS_ACCESS_KEY_ID"
            echo "This value will be masked: $AWS_SECRET_ACCESS_KEY"
            ./deploy-my-app.sh
workflows:
  deploy:
    jobs:
      - deploy
```

To load secrets from SecretHub into other orbs, install the CLI and set it as the `shell` of the `job`:

```yml
version: 2.1
orbs:
  aws-cli: circleci/aws-cli@0.1.20
  secrethub: secrethub/cli@v0.1.0

jobs:
  deploy:
    executor: aws-cli/default
    shell: secrethub run -- /bin/bash
    steps:
      - secrethub/install
      - checkout
      - aws-cli/setup:
          environment:
            AWS_DEFAULT_REGION: us-east-1
            AWS_ACCESS_KEY_ID: secrethub://company/app/aws/access_key_id
            AWS_SECRET_ACCESS_KEY: secrethub://company/app/aws/secret_access_key

workflows:
  deploy:
    jobs:
      - deploy
```

See the [src/examples](./src/examples/) directory for more examples.
