# Spoo.me Setup Action

A reusable GitHub Action that automatically sets up the [spoo.me URL shortener service](https://github.com/spoo-me/url-shortener) locally within any GitHub Actions workflow. This action handles all the complexity of setting up MongoDB, Redis, Python dependencies, and the spoo.me service itself.

## 🚀 Features

- **🔄 Reusable**: Use in any repository with `uses: spoo-me/setup-action@v1`
- **⚡ Complete Environment Setup**: Automatically installs and configures Python, MongoDB, and Redis
- **🔧 Service Management**: Clones the spoo.me repository and starts the service in the background
- **🔍 Health Monitoring**: Verifies all services are running properly before proceeding
- **⚙️ Configurable**: Supports custom versions for Python, MongoDB, and Redis
- **⚡ Fast Startup**: Optimized MongoDB setup (single instance, not replica set) for faster initialization
- **📊 Comprehensive Logging**: Provides detailed logs for debugging and monitoring
- **🎯 Clean Integration**: Easy to integrate into existing workflows as a single step

## 📋 Quick Start

### Basic Usage

```yaml
name: Test with Spoo.me Service

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Setup Spoo.me Service
        uses: spoo-me/setup-action@v1
        id: spoo-setup
        
      - name: Run tests against Spoo.me
        run: |
          echo "Service running at: ${{ steps.spoo-setup.outputs.service-url }}"
          curl -s ${{ steps.spoo-setup.outputs.service-url }}
```

### Advanced Usage with Custom Configuration

```yaml
name: Integration Tests

on: [push]

jobs:
  integration-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup Spoo.me Service
        uses: spoo-me/setup-action@v1
        id: spoo-setup
        with:
          python-version: '3.12'
          mongodb-version: '7.0'
          redis-version: '7.2'
          spoo-directory: 'my-spoo-instance'
          wait-timeout: '180'
          
      - name: Test URL shortening API
        run: |
          # Your integration tests here
          echo "Testing against: ${{ steps.spoo-setup.outputs.service-url }}"
          echo "MongoDB: ${{ steps.spoo-setup.outputs.mongodb-uri }}"
          echo "Redis: ${{ steps.spoo-setup.outputs.redis-uri }}"
```

## ⚙️ Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `python-version` | Python version to install | No | `3.11` |
| `mongodb-version` | MongoDB version to use | No | `7.0` |
| `redis-version` | Redis version to use | No | `7.2` |
| `spoo-directory` | Directory to clone spoo.me repository | No | `spoo-me` |
| `wait-timeout` | Timeout in seconds to wait for services | No | `120` |

## 📤 Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `service-url` | URL where the spoo.me service is running | `http://127.0.0.1:8000` |
| `mongodb-uri` | MongoDB connection URI | `mongodb://localhost:27017/url-shortener` |
| `redis-uri` | Redis connection URI | `redis://localhost:6379` |

## 🔧 Environment Configuration

The action automatically configures the following environment variables for the spoo.me service:

```bash
# MongoDB connection details (optimized single instance)
MONGODB_URI=mongodb://localhost:27017/url-shortener
MONGODB_URI_DEV=mongodb://localhost:27017/url-shortener
MONGO_DB_NAME=url-shortener

# Redis connection details
REDIS_URI=redis://localhost:6379
REDIS_URI_DEV=redis://localhost:6379
REDIS_TTL_SECONDS=3600

# Flask configs
SECRET_KEY=github-action-secret-key-for-testing
HOST_URI=127.0.0.1:8000
SHORTEN_API_RATE_LIMIT_PER_HOUR=100

# Webhook configs (empty for local testing)
CONTACT_WEBHOOK=
URL_REPORT_WEBHOOK=
HCAPTCHA_SECRET=
```

## 📊 What the Action Does

1. **🐍 Python Setup**: Installs the specified Python version
2. **🍃 MongoDB Setup**: Starts optimized single-instance MongoDB (faster than replica sets)
3. **🔴 Redis Setup**: Starts Redis service on the default port
4. **✅ Service Verification**: Verifies both databases are accessible
5. **📦 Repository Cloning**: Clones the spoo.me URL shortener repository
6. **🚀 UV Installation**: Installs `uv` for fast Python package management
7. **⚙️ Environment Configuration**: Sets up all required environment variables
8. **📦 Dependency Installation**: Installs Python dependencies using `uv sync`
9. **🗄️ Database Initialization**: Creates necessary database collections
10. **🌐 Service Startup**: Starts the spoo.me service in the background
11. **🔍 Health Checks**: Verifies the service is running and accessible

## 🗂️ Repository Structure

This action repository contains:

```
setup-action/
├── action.yml                     # Main action definition
├── README.md                      # This documentation
├── USAGE.md                       # Quick usage guide
├── LICENSE                        # MIT License
└── .github/
    └── workflows/
        ├── test.yml              # Comprehensive test workflow
        └── release.yml           # Automated release workflow
```

## 🐛 Troubleshooting

### Service Not Starting

If the service fails to start, check the logs:

```yaml
- name: Check service logs
  if: failure()
  run: |
    cd spoo-me  # or your custom directory
    cat spoo-service.log
```

### Database Connection Issues

The action includes built-in retry logic for database connections. If issues persist:

1. Increase the `wait-timeout` value to `180` or higher
2. Check if there are port conflicts
3. Verify the GitHub Actions runner has sufficient resources

### Common Issues

- **Timeout errors**: Increase `wait-timeout` to `180` or higher
- **Port conflicts**: The action uses standard ports (27017 for MongoDB, 6379 for Redis, 8000 for the service)
- **Python version compatibility**: Use Python 3.11 or higher for best compatibility
- **MongoDB slow startup**: We've optimized this! Now uses single instance (not replica set) for 5x faster startup

### 🚀 Performance Optimizations

Our action includes several optimizations for faster startup:

- **MongoDB Single Instance**: No replica set setup (reduces startup from ~60s to ~15s)
- **Smart Health Checks**: Uses `netcat` for port checking before database commands
- **Optimized Connection Strings**: Direct database connection URLs
- **Performance Indexes**: Automatically creates database indexes for faster queries

## 📚 Usage Examples

### Integration Testing

```yaml
name: Integration Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Spoo.me
        uses: spoo-me/setup-action@v1
        id: spoo
        
      - name: Run integration tests
        run: |
          # Test URL shortening
          response=$(curl -s -X POST \
            -H "Content-Type: application/json" \
            -d '{"url": "https://example.com"}' \
            ${{ steps.spoo.outputs.service-url }}/api/shorten)
          echo "API Response: $response"
```

### Multi-Version Testing

```yaml
name: Multi-Version Test

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.11', '3.12']
        mongodb-version: ['6.0', '7.0']
        
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Spoo.me
        uses: spoo-me/setup-action@v1
        with:
          python-version: ${{ matrix.python-version }}
          mongodb-version: ${{ matrix.mongodb-version }}
```

### Load Testing

```yaml
name: Load Testing

on: workflow_dispatch

jobs:
  load-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Spoo.me
        uses: spoo-me/setup-action@v1
        with:
          wait-timeout: '300'  # Longer timeout for load testing
        id: spoo
        
      - name: Install Apache Bench
        run: sudo apt-get update && sudo apt-get install -y apache2-utils
        
      - name: Run load test
        run: |
          echo "Running load test against ${{ steps.spoo.outputs.service-url }}"
          ab -n 1000 -c 10 ${{ steps.spoo.outputs.service-url }}/
```

## 🔄 Versioning

This action follows semantic versioning. Available versions:

- `@v1` - Latest stable v1.x release (recommended)
- `@v1.0.0` - Specific version
- `@main` - Latest development version (not recommended for production)

<!-- ## 🚀 Publishing to GitHub Marketplace

To publish this action to GitHub Marketplace:

1. **Create a new repository** on GitHub with the name `setup-action`
2. **Push this code** to the repository
3. **Create a release** with tag `v1.0.0`
4. **Add marketplace info** in the release description
5. **Publish to marketplace** via GitHub interface

### Release Workflow

Create `.github/workflows/release.yml`:

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          draft: false
          prerelease: false
``` -->

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🙏 Acknowledgments

- [Spoo.me URL Shortener](https://github.com/spoo-me/url-shortener) - The amazing URL shortening service
- [Supercharge MongoDB Action](https://github.com/supercharge/mongodb-github-action) - MongoDB setup
- [Supercharge Redis Action](https://github.com/supercharge/redis-github-action) - Redis setup
- [UV](https://github.com/astral-sh/uv) - Fast Python package management


---

<h6 align="center">
<img src="https://spoo.me/static/images/favicon.png" height=30 title="Spoo.me Copyright">
<br>
© spoo.me . 2025

All Rights Reserved</h6>

<p align="center">
	<a href="https://github.com/spoo-me/setup-action/blob/master/LICENSE.txt"><img src="https://img.shields.io/static/v1.svg?style=for-the-badge&label=License&message=APACHE-2.0&logoColor=d9e0ee&colorA=363a4f&colorB=b7bdf8"/></a>
</p>