# Quick Usage Guide

## Basic Setup

Add this step to your workflow:

```yaml
- name: Setup Spoo.me Service
  uses: spoo-me/setup-action@v1
  id: spoo
```

## With Custom Configuration

```yaml
- name: Setup Spoo.me Service
  uses: spoo-me/setup-action@v1
  id: spoo
  with:
    python-version: '3.12'
    mongodb-version: '7.0'
    redis-version: '7.2'
    wait-timeout: '180'
```

## Access Service

After setup, use the outputs:

```yaml
- name: Test the service
  run: |
    echo "Service URL: ${{ steps.spoo.outputs.service-url }}"
    curl ${{ steps.spoo.outputs.service-url }}
```

## Complete Example

```yaml
name: Test with Spoo.me

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Spoo.me
        uses: spoo-me/setup-action@v1
        id: spoo
        
      - name: Run tests
        run: |
          # Your tests here
          curl ${{ steps.spoo.outputs.service-url }}
``` 