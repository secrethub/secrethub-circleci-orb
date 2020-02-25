# SecretHub Orb
Use this orb to install the SecretHub CLI and provision secrets to your CI/CD pipeline.

## Usage

```
version: 2.1

orbs:
  secrethub: secrethub/cli

jobs:
  build:
    docker:
      - image: your-image
    shell: secrethub run -- /bin/bash
    steps:
      - secrethub/install
      - checkout
      - run: ./your-app

workflows:
  your-workflow:
    jobs:
      - build
```

More example use-cases are provided in the `src/examples` directory.

## Resources

[CircleCI Orb Docs](https://circleci.com/docs/2.0/orb-intro/#section=configuration) - Docs for using and creating CircleCI Orbs.
