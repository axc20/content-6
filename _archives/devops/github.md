# CI/CD and automation on GitHub

- GitHub Actions: a CI/CD tool that runs along side your code in GitHub
- pre-built CI/CD workflows and automations can be found in the GitHub Marketplace
- can also write your own workflows in YAML files
- A GitHub Actions workflow can be designed to respond to any webhook event on GitHub. That means you can turn any webhook on GitHub into a trigger for an automation within your CI/CD pipeline.

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Using the Checkout action from Marketplace
        uses: actions/checkout@v2
      - name: Run a one-line script
        run: echo Hello, world!
      - name: Run a multi-line script
        run: |
          echo Add other actions to build,
          echo test, and deploy your project
      - name: My own action in the same repo
        uses: ./
```

- the above workflow is composed of:
  - **Runners**: a GitHub Actions server, hosted by GitHub or self-hosted on a localized server
  - **Events**: defined triggers that start a workflow
  - **Jobs**: sets of steps that execute on the same runner
  - **Steps**: individual tasks that run commands in a job; can be an action or a shell command
  - **Actions**: command that is executed on a runner as well as the core element of GitHub Actions
