# CI Workflow Templates

Language-agnostic CI template for GitHub Actions:

```yaml
name: CI
on:
  pull_request:
    branches: [main, develop]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup
        uses: SETUP_ACTION
        with:
          VERSION_KEY: 'VERSION'
      - run: INSTALL_CMD
      - run: TEST_CMD
```

## Language Mappings

| Language | Setup Action                    | Version Key    | Default Version |
| -------- | ------------------------------- | -------------- | --------------- |
| python   | actions/setup-python@v5         | python-version | 3.12            |
| node     | actions/setup-node@v4           | node-version   | 20              |
| go       | actions/setup-go@v5             | go-version     | 1.22            |
| rust     | dtolnay/rust-toolchain@stable   | (none)         | stable          |
