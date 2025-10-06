# 🚢 Caronte

[![PyPI version](https://badge.fury.io/py/caronte.svg)](https://badge.fury.io/py/caronte)
[![Python Version](https://img.shields.io/pypi/pyversions/caronte.svg)](https://pypi.org/project/caronte/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Ruff](https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/v2.json)](https://github.com/astral-sh/ruff)

> **Caronte** - _The multi-cloud conductor that guides your code safely across AWS, Azure, and Google Cloud_

**Stop writing cloud-specific code.** **Caronte** is a unified multi-cloud abstraction library that lets you write once and run anywhere - AWS, Azure, or Google Cloud. Just like the mythological ferryman who safely guided souls across treacherous waters, Caronte guides your code safely across the complex landscape of cloud providers.

## 🎯 Why Caronte Exists

### The Problem: Cloud Lock-in and Code Duplication

```python
# Without Caronte: Different APIs for every provider
# AWS
s3_client.upload_file(bucket_name, object_key, file_path)

# Azure
blob_client.upload_blob(data, overwrite=True)

# GCP
bucket.blob(object_name).upload_from_filename(file_path)
```

### The Solution: One API, All Providers

```python
# With Caronte: Same code works everywhere
result = storage.upload_file(bucket, key, data)
if result.is_ok():
    print(f"✅ Uploaded to {result.unwrap()}")
else:
    print(f"❌ Failed: {result.unwrap_err()}")
```

## ✨ What Makes Caronte Special

### 🔒 **Development-Time Safety**

Never unwrap a `Result` without checking it first. Our `UncheckedResultError` catches mistakes during development:

```python
result = function.deploy()
url = result.unwrap()  # 💥 UncheckedResultError with exact location!

# Fix: Always check first
if result.is_ok():
    url = result.unwrap()  # ✅ Safe after checking
```

### 🧩 **Interface Segregation**

Use only what you need. No more 25+ method interfaces - access focused components:

```python
# Access only what you need
manager.core.exists()              # Basic operations only
manager.configuration.get_config() # Configuration only
manager.invocation.invoke_sync()   # Invocation only
```

### 🔄 **Zero Migration Cost**

Switch providers without changing your code:

```python
# Development with AWS
aws_manager = AWSFunctionServiceManager("my-function")

# Production with GCP - same interface!
gcp_manager = GCPFunctionServiceManager("my-function")

# Identical code for both providers
result = manager.core.exists()
```

## 🚀 Quick Start

### Installation

```bash
pip install caronte
```

### Your First Caronte Program

```python
from caronte.providers.aws import AWSFunctionServiceManager

# Create a function manager
manager = AWSFunctionServiceManager("my-function")

# Check if function exists (returns Result[bool, FunctionError])
result = manager.core.exists()

if result.is_ok():
    exists = result.unwrap()
    print(f"Function exists: {exists}")
else:
    print(f"Error: {result.unwrap_err()}")

# Chain operations safely
deployment_result = (
    manager.core.exists()
    .and_then(lambda _: manager.configuration.update_memory(512))
    .and_then(lambda _: manager.deployment.deploy())
)
```

## 🛠️ Real-World Examples

### Multi-Cloud Storage

```python
# Same code works with any provider
def upload_backup(storage_manager, backup_data: bytes):
    result = storage_manager.core.upload_file(
        "backups",
        f"backup-{datetime.now().isoformat()}.zip",
        backup_data
    )

    if result.is_ok():
        return f"Backup stored at: {result.unwrap()}"
    else:
        return f"Backup failed: {result.unwrap_err()}"

# Use with AWS S3
aws_storage = AWSStorageManager("my-bucket")
upload_backup(aws_storage, backup_data)

# Same function works with GCP Cloud Storage
gcp_storage = GCPStorageManager("my-bucket")
upload_backup(gcp_storage, backup_data)
```

### Safe Function Deployment

```python
async def deploy_function_safely(manager, config):
    """Deploy with automatic rollback on failure."""

    # Check if function exists
    exists_result = manager.core.exists()
    if not exists_result.is_ok():
        return exists_result  # Return error

    # Deploy new version
    deploy_result = manager.deployment.deploy(config)
    if not deploy_result.is_ok():
        return deploy_result  # Return error

    # Test the deployment
    test_result = manager.invocation.invoke_sync({"test": True})
    if not test_result.is_ok():
        # Rollback on test failure
        manager.deployment.rollback()
        return Err("Deployment test failed, rolled back")

    return Ok("Deployment successful and tested")
```

## 🏗️ Architecture Principles

### 1. Result Type Pattern

No more try/catch blocks. All operations return `Result[T, Error]`:

```python
# Before (exception-based)
try:
    url = upload_file(bucket, key, data)
    process(url)
except StorageError as e:
    handle_error(e)

# After (Result-based)
result = storage.upload_file(bucket, key, data)
if result.is_ok():
    process(result.unwrap())
else:
    handle_error(result.unwrap_err())
```

### 2. Interface Segregation

Small, focused interfaces instead of large monolithic ones:

```python
# Access only what you need
manager.core.exists()              # Basic operations
manager.configuration.get_config() # Configuration only
manager.invocation.invoke_sync()   # Invocation only
manager.monitoring.get_metrics()   # Monitoring only
```

### 3. Development-Time Safety

Caronte prevents Result misuse during development:

```python
result = deploy_function(config)
url = result.unwrap()  # 💥 UncheckedResultError with exact location!

# Fix: Always check first
if result.is_ok():
    url = result.unwrap()  # ✅ Safe after checking
```

## 🛠️ Supported Services

### Current (v0.1.0)

- 🔧 **Authentication** - Credential management across providers
- 📦 **Functions** - AWS Lambda, Azure Functions, Google Cloud Functions
- 💾 **Storage** - S3, Azure Blob, Google Cloud Storage
- 📨 **Queues** - SQS, Azure Service Bus, Google Pub/Sub

### Planned (v0.2.0+)

- 🗄️ **Databases** - RDS, CosmosDB, Cloud SQL
- 🌐 **Networking** - VPC, Virtual Networks, VPC Networks
- 🔐 **Security** - IAM, Active Directory, Cloud IAM

## 🎨 Design Philosophy

### "Gateway, Not the Whole House"

Caronte is designed to be your **interface to cloud providers**, not a complete cloud management platform:

```python
# ✅ Caronte's responsibility: Provider abstraction
result = storage.upload_file(bucket, key, data)

# ❌ NOT Caronte's responsibility: Business logic
# cache_manager.smart_invalidation()    # You implement
# retry_manager.exponential_backoff()   # You implement
# circuit_breaker.handle_failures()     # You implement
```

This keeps Caronte focused, flexible, and prevents feature creep.

## 🧪 Example: Complete Function Workflow

```python
from caronte.providers.aws import AWSFunctionServiceManager
from caronte.common.models import FunctionConfig, Runtime

async def deploy_and_test_function():
    manager = AWSFunctionServiceManager("my-lambda")

    # 1. Check if function exists
    exists_result = manager.core.exists()
    if not exists_result.is_ok():
        return exists_result

    # 2. Configure function
    config = FunctionConfig(
        runtime=Runtime.PYTHON_311,
        memory_mb=512,
        timeout_seconds=30
    )

    config_result = manager.configuration.update(config)
    if not config_result.is_ok():
        return config_result

    # 3. Deploy code
    deploy_result = manager.deployment.deploy_from_zip("function.zip")
    if not deploy_result.is_ok():
        return deploy_result

    # 4. Test deployment
    test_payload = {"test": "deployment"}
    test_result = manager.invocation.invoke_sync(test_payload)

    return test_result  # Result[dict, FunctionError]

# Usage
result = await deploy_and_test_function()
if result.is_ok():
    response = result.unwrap()
    print(f"Function test successful: {response}")
else:
    error = result.unwrap_err()
    print(f"Function workflow failed: {error}")
```

## 🔄 Migration Between Providers

Switch providers without changing your code:

```python
# Development: Use AWS
aws_factory = AWSFunctionComponentFactory(region="us-east-1")
manager = FunctionServiceManager(aws_factory, "my-function")

# Production: Switch to GCP
gcp_factory = GCPFunctionComponentFactory(project="my-project")
manager = FunctionServiceManager(gcp_factory, "my-function")

# Same code, different provider!
result = manager.core.exists()
```

## 🏁 Getting Started

### 1. Installation

```bash
pip install caronte

# Or with Poetry
poetry add caronte
```

### 2. Configure Credentials

```python
# AWS
export AWS_ACCESS_KEY_ID=your_key
export AWS_SECRET_ACCESS_KEY=your_secret

# Azure
export AZURE_CLIENT_ID=your_client_id
export AZURE_CLIENT_SECRET=your_secret

# GCP
export GOOGLE_APPLICATION_CREDENTIALS=path/to/service-account.json
```

### 3. Your First Caronte Program

```python
from caronte.providers.aws import AWSStorageManager

# Create storage manager
storage = AWSStorageManager("my-bucket")

# Upload file
result = storage.core.upload_file("hello.txt", b"Hello, Caronte!")

if result.is_ok():
    url = result.unwrap()
    print(f"File uploaded: {url}")
else:
    print(f"Upload failed: {result.unwrap_err()}")
```

## 🤝 Contributing

We welcome contributions! Please read our [Contributing Guide](CONTRIBUTING.md) first.

### Development Setup

```bash
git clone https://github.com/giovani-dev/caronte.git
cd caronte
poetry install
poetry run pre-commit install
```

### Running Tests

```bash
make test          # Run all tests
make test-unit     # Unit tests only
make test-integration  # Integration tests only
```

## 📊 Project Status

**Current Version**: v0.1.0a0 (Pre-Alpha)
**Development Status**: Active development, architecture-first phase
**Production Ready**: Not yet, skeleton implementation in progress

### Roadmap

- **Phase 1** (Q4 2025): Skeleton implementation with mocks
- **Phase 2** (Q1 2026): Authentication foundation
- **Phase 3** (Q2 2026): First concrete service (Functions)
- **Phase 4** (Q3 2026): Multi-provider expansion

## 💭 Philosophy

> _"Just as Caronte ferried souls safely across the treacherous waters of the Styx, our library ferries your code safely across the complex landscape of cloud providers."_

Caronte embodies:

- **Reliability**: Like the mythological ferryman, we get you where you need to go
- **Simplicity**: One boat, many destinations
- **Trust**: Consistent, predictable behavior across all providers
- **Guidance**: Clear path through cloud complexity

## 📄 License

This project is licensed under the [MIT License](LICENSE).

## 🙏 Acknowledgments

- **Author**: Giovani Liskoski Zanini
- **Email**: giovanilzanini@hotmail.com
- **GitHub**: [@giovani-dev](https://github.com/giovani-dev)

---

**Made with ❤️ and ☕ in Brazil** 🇧🇷
