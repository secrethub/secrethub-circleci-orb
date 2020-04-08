<p align="center">
  <img src="https://secrethub.io/img/integrations/circleci/github-banner.png?v1" alt="CircleCI + SecretHub" width="390">
</p>
<br/>

<p align="center">
  <a href="https://secrethub.io/integrations/circleci/"><img alt="Learn More" src="https://secrethub.io/img/buttons/github/learn-more.png?v1" height="28" /></a>
  <a href="https://secrethub.io/docs/guides/circleci/"><img alt="View Docs" src="https://secrethub.io/img/buttons/github/view-docs.png?v1" height="28" /></a>
</p>
<br/>

<h1>
  CircleCI Orb <img src="https://secrethub.io/img/integrations/circleci/partner-badge.png" alt="Partner badge" width="60" />
</h1>

[![CircleCI](https://circleci.com/gh/secrethub/secrethub-circleci-orb.svg?style=shield)](https://circleci.com/gh/secrethub/secrethub-circleci-orb)
[![Version]( https://img.shields.io/github/release/secrethub/secrethub-circleci-orb.svg)](https://github.com/secrethub/secrethub-circleci-orb/releases/latest)
[![Discord](https://img.shields.io/badge/chat-on%20discord-7289da.svg?logo=discord)](https://discord.gg/NWmxVeb)

No more copy-pasting sensitive values into a GUI. Securely load secrets into CircleCI and sync them automatically.

## Usage

To execute a command that needs secrets, replace your CircleCI `run` command with `secrethub/exec`.
You can make secrets available to your command as environment variables by referencing their SecretHub path, prefixed by `secrethub://`:

```yml
version: 2.1
orbs:
  secrethub: secrethub/cli@1.0.0

jobs:
  deploy:
    docker:
      - image: cimg/base:stable
    environment:
      AWS_ACCESS_KEY_ID: secrethub://company/app/aws/access_key_id
      AWS_SECRET_ACCESS_KEY: secrethub://company/app/aws/secret_access_key
    steps:
      - checkout
      - secrethub/exec:
          command: |
            echo "This value will be masked: $AWS_ACCESS_KEY_ID"
            echo "This value will be masked: $AWS_SECRET_ACCESS_KEY"
            ./deploy-my-app.sh
workflows:
  deploy:
    jobs:
      - deploy
```

Alternatively, you can set the `shell` of the native CircleCI `run` command:

```yml
version: 2.1
orbs:
  secrethub: secrethub/cli@1.0.0

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

You can either set the shell on the `run` command level, or you can set it on the `job` level to use it for every step in the job.
That way you can also load secrets into other orbs:

```yml
version: 2.1
orbs:
  aws-cli: circleci/aws-cli@0.1.20
  secrethub: secrethub/cli@1.0.0

jobs:
  deploy:
    executor: aws-cli/default
    shell: secrethub run -- /bin/bash
    environment:
      AWS_DEFAULT_REGION: us-east-1
      AWS_ACCESS_KEY_ID: secrethub://company/app/aws/access_key_id
      AWS_SECRET_ACCESS_KEY: secrethub://company/app/aws/secret_access_key
    steps:
      - secrethub/install
      - checkout
      - aws-cli/setup

workflows:
  deploy:
    jobs:
      - deploy
```

See the [src/examples](./src/examples/) directory for more examples.

## Masking

When using either the `secrethub/exec` orb command or the `secrethub run` shell wrapper, all secrets are automatically masked from the CI log output.
If secrets (accidentally) get logged, they will be replaced with:

```
<redacted by SecretHub>
```

## Authentication

For your CircleCI jobs to authenticate to SecretHub and decrypt the secrets they need, [create a SecretHub service account](https://secrethub.io/docs/reference/cli/service/), give it [read access](https://secrethub.io/docs/reference/cli/acl/) to the secrets it needs, and configure the credential as `SECRETHUB_CREDENTIAL` in your CircleCI project settings or Context environment variables.
