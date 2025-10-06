# 🤝 Contributing to Caronte

### _Building the Future of Multi-Cloud Development Together_

Thank you for your interest in contributing to **Caronte**! Whether you're reporting bugs, suggesting features, or contributing code, your help is essential to making Caronte the best multi-cloud abstraction library.

This guide will help you get started and ensure your contributions align with our project standards and architectural decisions.

## 🌟 Ways to Contribute

We welcome contributions in many forms:

- 🐛 **Report Bugs** - Help us identify and fix issues
- 💡 **Suggest Features** - Share ideas for new cloud abstractions
- 🔧 **Submit Code** - Implement features, fix bugs, improve documentation
- 📖 **Improve Documentation** - Help other developers understand Caronte
- 🧪 **Write Tests** - Strengthen our safety guarantees
- 🎨 **Design & UX** - Improve developer experience
- 🌍 **Community** - Help other contributors and users

### � Good First Issues

New to Caronte? Look for issues labeled:

- `good-first-issue` - Perfect for newcomers
- `help-wanted` - We'd love community help on these
- `documentation` - Improve our docs
- `testing` - Add more test coverage

## 🏗️ Development Environment Setup

### Prerequisites

Before you begin, ensure you have the following installed:

- **Python 3.10+** (we recommend using [pyenv](https://github.com/pyenv/pyenv) for version management)
- **Poetry** for dependency management ([install here](https://python-poetry.org/docs/#installation))
- **Git** for version control
- **A cloud account** (AWS, Azure, or GCP) for integration testing (optional)

### Quick Setup

Get your development environment ready in under 5 minutes:

```bash
# 1. Fork and clone the repository
git clone https://github.com/your-username/caronte.git
cd caronte

# 2. Set up development environment (this does everything!)
make dev-install

# This command will:
# ✅ Install all dependencies via Poetry
# ✅ Set up pre-commit hooks for code quality
# ✅ Configure development tools (Black, Ruff, MyPy)
# ✅ Prepare your environment for contributing

# 3. Verify everything works
make ci-check

# 🎉 You're ready to contribute!
```

### Manual Setup (Alternative)

If you prefer to set up manually:

```bash
# Install dependencies
poetry install --with dev --with test

# Set up pre-commit hooks
poetry run pre-commit install

# Run initial checks
poetry run pytest
poetry run ruff check caronte tests
poetry run black --check caronte tests
```

## 🔧 Development Workflow

### Understanding Our Architecture

Before contributing, familiarize yourself with Caronte's core architectural principles:

1. **Result Type Pattern** - All operations return `Result[T, Error]` instead of raising exceptions
2. **Interface Segregation** - Small, focused interfaces rather than monolithic ones
3. **Development-Time Safety** - `UncheckedResultError` prevents unsafe Result usage
4. **Multi-Cloud Abstraction** - Same API works across AWS, Azure, and GCP

### Branch Strategy

We follow a structured Git flow documented in [GIT_WORKFLOW.md](/.github/GIT_WORKFLOW.md).

#### Creating Your Branch

```bash
# Features - new functionality
git checkout -b feature/aws-lambda-cold-start-optimization
git checkout -b feature/azure-blob-lifecycle-management

# Bug fixes - fixing existing issues
git checkout -b fix/gcp-timeout-handling
git checkout -b fix/result-type-unwrap-error

# Documentation improvements
git checkout -b docs/api-reference-examples
git checkout -b docs/multi-cloud-migration-guide

# Testing enhancements
git checkout -b test/aws-s3-integration-coverage
git checkout -b test/result-pattern-edge-cases
```

### Development Commands

Use our Makefile for all development tasks:

```bash
# 🧪 Testing
make test                    # Run all tests
make test-cov               # Run tests with coverage
make test-aws               # Test only AWS components
make test-azure             # Test only Azure components
make test-gcp               # Test only GCP components
make test-fast              # Skip slow integration tests

# 🔍 Code Quality
make lint                   # Check code with Ruff
make lint-fix              # Auto-fix linting issues
make format                # Format with Black + Ruff
make format-check          # Check formatting without changes
make type-check            # Run MyPy type checking
make security              # Security scan with Bandit

# 🔧 All-in-One Commands
make ci-check              # Run all CI checks locally
make pre-commit            # Run pre-commit hooks manually
make clean                 # Clean cache and temp files
```

### Commit Convention

We use [Conventional Commits](https://www.conventionalcommits.org/) to maintain a clean, semantic commit history.

#### Format

```
type(scope): description

[optional body]

[optional footer(s)]
```

#### Types & Examples

```bash
# ✅ Features
feat(aws): add S3 glacier storage tier support
feat(azure): implement blob storage lifecycle policies
feat(gcp): add cloud storage signed URL generation
feat(storage): add multi-part upload interface

# ✅ Bug Fixes
fix(aws): resolve lambda timeout configuration issue
fix(gcp): fix authentication credential refresh
fix(result): prevent unwrap() without safety check
fix(ci): resolve failing integration tests

# ✅ Documentation
docs(api): add storage service examples
docs(readme): update quick start guide
docs(contributing): improve development setup instructions

# ✅ Tests
test(aws): add comprehensive S3 error handling tests
test(integration): add multi-cloud storage tests
test(unit): improve result pattern test coverage

# ✅ Chores & Maintenance
chore(deps): update boto3 to v1.34.0
chore(ci): optimize GitHub Actions workflow
chore(lint): fix formatting issues
```

#### Scopes

Use these scopes to categorize your changes:

- **Providers**: `aws`, `azure`, `gcp`
- **Services**: `storage`, `functions`, `queues`, `auth`
- **Core**: `result`, `interfaces`, `config`, `errors`
- **Tooling**: `ci`, `deps`, `lint`, `test`, `docs`

### Pre-commit Hooks

Our pre-commit hooks ensure code quality and consistency:

```bash
# Automatically run on every commit:
# ✅ Format code with Black and Ruff
# ✅ Lint code with Ruff
# ✅ Type check with MyPy (when enabled)
# ✅ Security scan with Bandit
# ✅ Validate YAML/JSON/TOML syntax
# ✅ Check for merge conflicts
# ✅ Verify commit message format

# Run manually:
make pre-commit

# Skip hooks (emergency only):
git commit --no-verify -m "emergency fix"
```

## 📝 Code Standards & Architecture

### Caronte's Architectural Principles

Follow these core principles when contributing:

#### 1. Result Type Pattern (Mandatory)

All operations that can fail must return `Result[T, Error]`:

```python
# ✅ CORRECT: Return Result type
def upload_file(bucket: str, key: str, data: bytes) -> Result[str, StorageError]:
    try:
        url = s3_client.upload(bucket, key, data)
        return Result(value=url)
    except ClientError as e:
        return Result(error=StorageError(f"Upload failed: {e}"))

# ❌ WRONG: Don't raise exceptions from public APIs
def upload_file(bucket: str, key: str, data: bytes) -> str:
    return s3_client.upload(bucket, key, data)  # Can raise!
```

#### 2. Interface Segregation (Mandatory)

Create small, focused interfaces instead of large monolithic ones:

```python
# ✅ CORRECT: Focused interfaces
class StorageCore(Protocol):
    def upload_file(self, key: str, data: bytes) -> Result[str, StorageError]: ...
    def download_file(self, key: str) -> Result[bytes, StorageError]: ...
    def delete_file(self, key: str) -> Result[None, StorageError]: ...

class StorageMetadata(Protocol):
    def get_metadata(self, key: str) -> Result[dict, StorageError]: ...
    def set_metadata(self, key: str, metadata: dict) -> Result[None, StorageError]: ...

# ❌ WRONG: Monolithic interface
class StorageEverything(Protocol):
    def upload_file(self, ...): ...
    def download_file(self, ...): ...
    def delete_file(self, ...): ...
    def get_metadata(self, ...): ...
    def set_metadata(self, ...): ...
    def list_files(self, ...): ...
    def copy_file(self, ...): ...
    # ... 20+ more methods
```

#### 3. Development-Time Safety (Encouraged)

Include safety checks that help developers during development:

```python
# Example: Check Result usage patterns
def deploy_function(config: FunctionConfig) -> Result[str, FunctionError]:
    # Implementation here
    pass

# Usage - this will catch unsafe patterns:
result = deploy_function(config)
# url = result.unwrap()  # 💥 UncheckedResultError in development!

# Safe usage:
if result.is_ok():
    url = result.unwrap()  # ✅ Safe after checking
```

### Code Style Guidelines

#### Formatting & Linting

- **Line Length**: 88 characters (Black default)
- **Import Sorting**: Automatic via Ruff
- **Quote Style**: Double quotes for strings
- **Trailing Commas**: Required in multi-line structures

```python
# ✅ CORRECT formatting
from typing import Dict, List, Optional

class StorageManager:
    """Storage manager with proper formatting."""

    def __init__(
        self,
        bucket_name: str,
        region: str = "us-east-1",
        timeout: int = 30,
    ) -> None:
        self.bucket_name = bucket_name
        self.region = region
        self.timeout = timeout

    def get_config(self) -> Dict[str, Any]:
        """Return configuration dictionary."""
        return {
            "bucket": self.bucket_name,
            "region": self.region,
            "timeout": self.timeout,
        }
```

#### Type Annotations (Required)

All public APIs must have complete type annotations:

```python
# ✅ CORRECT: Full type annotations
from typing import Optional, Dict, Any, List
from caronte.common.result import Result
from caronte.common.errors import StorageError

def process_files(
    files: List[str],
    config: Optional[Dict[str, Any]] = None,
    batch_size: int = 10,
) -> Result[List[str], StorageError]:
    """Process multiple files with proper typing."""
    # Implementation
    pass

# ❌ WRONG: Missing type annotations
def process_files(files, config=None, batch_size=10):
    # Implementation
    pass
```

#### Documentation (Google Style)

Use Google-style docstrings for all public APIs:

```python
def upload_with_retry(
    self,
    key: str,
    data: bytes,
    max_retries: int = 3,
    backoff_factor: float = 1.5,
) -> Result[str, StorageError]:
    """Upload file with automatic retry on failure.

    Uploads a file to cloud storage with exponential backoff retry
    mechanism. This method is safe to call multiple times.

    Args:
        key: Object key/path for the uploaded file.
        data: File content as bytes.
        max_retries: Maximum number of retry attempts (default: 3).
        backoff_factor: Multiplier for exponential backoff (default: 1.5).

    Returns:
        Result containing the uploaded file URL on success, or StorageError
        on failure after all retry attempts are exhausted.

    Example:
        >>> storage = StorageManager("my-bucket")
        >>> result = storage.upload_with_retry("path/file.txt", b"content")
        >>> if result.is_ok():
        ...     print(f"Uploaded to: {result.unwrap()}")
        ... else:
        ...     print(f"Upload failed: {result.unwrap_err()}")
    """
    # Implementation
    pass
```

## 🧪 Testing Guidelines

### Test Structure

Our tests are organized by type and purpose:

```
tests/
├── unit/                      # Fast, isolated unit tests
│   ├── common/                # Core functionality tests
│   │   ├── test_result.py     # Result pattern tests
│   │   └── test_config.py     # Configuration tests
│   ├── providers/             # Provider-specific tests
│   │   ├── aws/               # AWS implementation tests
│   │   ├── azure/             # Azure implementation tests
│   │   └── gcp/               # GCP implementation tests
│   └── interfaces/            # Interface compliance tests
└── integration/               # Cloud provider integration tests
    ├── aws/                   # Real AWS API tests
    ├── azure/                 # Real Azure API tests
    └── gcp/                   # Real GCP API tests
```

### Writing Tests

#### Test Result Types

Always test both success and error cases for Result types:

```python
import pytest
from unittest.mock import Mock, patch
from caronte.providers.aws import AWSS3Storage
from caronte.common.errors import StorageError

class TestAWSS3Storage:
    """Tests for AWS S3 storage implementation."""

    @pytest.fixture
    def storage_manager(self):
        """Create storage manager for testing."""
        return AWSS3Storage("test-bucket")

    def test_upload_file_success(self, storage_manager):
        """Test successful file upload returns Result[str, StorageError]."""
        with patch('boto3.client') as mock_client:
            mock_client.return_value.upload_file.return_value = None

            result = storage_manager.upload_file("test.txt", b"content")

            assert result.is_ok()
            assert "test.txt" in result.unwrap()

    def test_upload_file_failure(self, storage_manager):
        """Test failed upload returns Result[str, StorageError]."""
        with patch('boto3.client') as mock_client:
            mock_client.return_value.upload_file.side_effect = Exception("S3 Error")

            result = storage_manager.upload_file("test.txt", b"content")

            assert result.is_err()
            assert isinstance(result.unwrap_err(), StorageError)
            assert "S3 Error" in str(result.unwrap_err())

    def test_upload_file_requires_safety_check(self, storage_manager):
        """Test that unwrap() without checking raises UncheckedResultError."""
        with patch('boto3.client') as mock_client:
            mock_client.return_value.upload_file.return_value = None

            result = storage_manager.upload_file("test.txt", b"content")

            # This should raise UncheckedResultError in development
            with pytest.raises(UncheckedResultError):
                result.unwrap()  # ❌ Called without is_ok() check
```

#### Test Interface Compliance

Ensure implementations properly follow interface contracts:

```python
def test_storage_interface_compliance():
    """Test that AWS storage implements the storage interface correctly."""
    from caronte.interfaces.storage import StorageCore

    storage = AWSS3Storage("test-bucket")

    # Verify interface compliance
    assert isinstance(storage, StorageCore)

    # Test interface methods exist and return correct types
    methods = ["upload_file", "download_file", "delete_file"]
    for method in methods:
        assert hasattr(storage, method)
        assert callable(getattr(storage, method))
```

#### Multi-Cloud Testing

Write tests that verify the same interface works across providers:

```python
@pytest.mark.parametrize("storage_class,provider", [
    (AWSS3Storage, "aws"),
    (GCPCloudStorage, "gcp"),
    # (AzureBlobStorage, "azure"),  # When implemented
])
def test_upload_interface_consistency(storage_class, provider):
    """Test upload interface works consistently across providers."""
    storage = storage_class("test-bucket")

    with patch(f'{storage_class.__module__}.{storage_class.__name__}.upload_file') as mock_upload:
        mock_upload.return_value = Result(value=f"https://{provider}.example.com/file.txt")

        result = storage.upload_file("test.txt", b"content")

        assert result.is_ok()
        assert provider in result.unwrap()
```

### Running Tests

```bash
# Run all tests
make test

# Run with coverage
make test-cov

# Run specific provider tests
make test-aws      # AWS only
make test-azure    # Azure only
make test-gcp      # GCP only

# Run fast tests (skip integration)
make test-fast

# Run on multiple Python versions
make test-all
```

### Integration Testing

For integration tests that use real cloud APIs:

```python
import os
import pytest

@pytest.mark.integration
@pytest.mark.skipif(
    not os.getenv("AWS_ACCESS_KEY_ID"),
    reason="AWS credentials not available"
)
def test_real_s3_upload():
    """Integration test with real S3 (only when credentials available)."""
    storage = AWSS3Storage("caronte-test-bucket")

    # Use real S3 API
    result = storage.upload_file("integration-test.txt", b"test content")

    if result.is_ok():
        # Clean up
        delete_result = storage.delete_file("integration-test.txt")
        assert delete_result.is_ok()
    else:
        pytest.fail(f"Integration test failed: {result.unwrap_err()}")
```

## 📋 Pull Request Process

### Before Submitting Your PR

Run through this checklist to ensure your contribution meets our standards:

#### ✅ Code Quality Checklist

```bash
# Run all checks locally
make ci-check

# This runs:
# ✅ Linting (Ruff)
# ✅ Formatting check (Black + Ruff)
# ✅ Type checking (MyPy) - when enabled
# ✅ Security scanning (Bandit) - when enabled
# ✅ Test suite with coverage
```

#### ✅ Manual Checklist

- [ ] **All tests pass** (`make test`)
- [ ] **Test coverage** is adequate (>80% for new code)
- [ ] **Code is formatted** (`make format`)
- [ ] **No linting errors** (`make lint`)
- [ ] **Type hints** are present on all public APIs
- [ ] **Documentation** is updated for new features
- [ ] **Result pattern** is used for all fallible operations
- [ ] **Interface segregation** is followed for new interfaces
- [ ] **Commit messages** follow conventional commit format
- [ ] **Branch is up to date** with master/dev

#### ✅ Architecture Compliance

- [ ] **Result types** are used instead of exceptions
- [ ] **Interfaces are focused** and follow single responsibility principle
- [ ] **Development-time safety** is preserved (no unsafe unwrap patterns)
- [ ] **Multi-cloud abstraction** is maintained (no provider-specific APIs leaked)
- [ ] **Error handling** is explicit and well-documented

### Creating Your Pull Request

#### PR Title Format

Use conventional commit format for PR titles:

```
feat(aws): add S3 lifecycle management support
fix(gcp): resolve authentication timeout issue
docs(api): improve storage service examples
test(azure): add blob storage integration tests
```

#### PR Description Template

Our GitHub template will help you provide the right information:

```markdown
## Description

Brief description of what this PR does and why.

## Type of Change

- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update

## Testing

- [ ] Unit tests added/updated
- [ ] Integration tests added/updated (if applicable)
- [ ] Manual testing performed

## Architecture Compliance

- [ ] Uses Result types for error handling
- [ ] Follows interface segregation principles
- [ ] Maintains development-time safety
- [ ] Preserves multi-cloud abstraction

## Related Issues

Fixes #(issue number)
```

### Review Process

#### What Reviewers Look For

1. **Architectural Alignment**

   - Proper use of Result types
   - Interface segregation compliance
   - Multi-cloud abstraction preservation

2. **Code Quality**

   - Clear, readable code
   - Comprehensive test coverage
   - Proper error handling

3. **Documentation**
   - Updated API documentation
   - Clear commit messages
   - Helpful code comments

#### Review Timeline

- **Initial Review**: Within 2-3 business days
- **Follow-up Reviews**: Within 1-2 business days
- **Final Approval**: After all feedback is addressed

#### Getting Your PR Merged

1. **All Checks Pass**: GitHub Actions CI must be green
2. **Approval Required**: At least one maintainer approval
3. **No Merge Conflicts**: Keep your branch up to date
4. **Documentation**: Include docs for user-facing changes

## 🌟 Community & Support

### Getting Help

We're here to help you contribute successfully:

#### � Communication Channels

- **GitHub Issues** - Bug reports, feature requests, questions
- **GitHub Discussions** - Community discussions, architecture questions
- **Pull Request Comments** - Code-specific discussions
- **Email** - Direct contact with maintainers: [giovanilzanini@hotmail.com](mailto:giovanilzanini@hotmail.com)

#### 🆘 When You're Stuck

- **Check existing issues** - Someone might have had the same problem
- **Review our documentation** - Architecture decisions are documented
- **Ask questions** - We're friendly and want to help!
- **Start small** - Look for `good-first-issue` labels

### Contributing Opportunities

#### 🚀 High Impact Areas

- **Authentication System** - Help build the foundation (Phase 2 roadmap)
- **AWS Lambda Functions** - First concrete service implementation
- **Multi-Cloud Testing** - Ensure consistency across providers
- **Documentation** - Help developers understand Caronte's power

#### 🎯 Skill-Based Contributions

**Cloud Expertise**

- AWS, Azure, or GCP service implementations
- Multi-cloud migration strategies
- Infrastructure automation

**Python Development**

- Type system improvements
- Performance optimizations
- Testing framework enhancements

**Developer Experience**

- Documentation and examples
- Error message improvements
- Development tooling

**Architecture & Design**

- Interface design
- Error handling patterns
- Performance optimization

### Recognition

Contributors are recognized in multiple ways:

- **Changelog Credits** - All contributors listed in release notes
- **GitHub Contributors Graph** - Visible contribution history
- **Special Recognition** - Major contributors highlighted in README
- **Maintainer Opportunities** - Active contributors invited to join core team

## 📚 Learning Resources

### Understanding Caronte's Architecture

- **[Copilot Instructions](/.github/instructions/copilot.instructions.md)** - Comprehensive architectural guide
- **[Executive Summary](/docs/executive-summary.md)** - High-level architectural decisions
- **[Git Workflow](/.github/GIT_WORKFLOW.md)** - Our development process

### External Resources

- **[Conventional Commits](https://www.conventionalcommits.org/)** - Commit message format
- **[Google Style Guide](https://google.github.io/styleguide/pyguide.html)** - Python documentation style
- **[Result Type Pattern](https://doc.rust-lang.org/std/result/)** - Rust's Result type (inspiration)
- **[Interface Segregation](https://en.wikipedia.org/wiki/Interface_segregation_principle)** - SOLID principles

### Cloud Provider Documentation

- **[AWS SDK for Python (Boto3)](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)**
- **[Azure SDK for Python](https://docs.microsoft.com/en-us/azure/developer/python/)**
- **[Google Cloud Client Libraries](https://cloud.google.com/python/docs/reference)**

## 🎉 Thank You!

Your contributions make Caronte better for everyone. Whether you're:

- 🐛 Reporting your first bug
- 📖 Improving documentation
- 🔧 Implementing a new feature
- 🧪 Adding test coverage
- 💡 Suggesting improvements

**Every contribution matters and is appreciated!**

---

## 📞 Contact Information

### Maintainers

**Giovani Liskoski Zanini** - _Project Creator & Lead Maintainer_

- **GitHub**: [@giovani-dev](https://github.com/giovani-dev)
- **Email**: [giovanilzanini@hotmail.com](mailto:giovanilzanini@hotmail.com)

### Project Links

- **Repository**: [https://github.com/giovani-dev/caronte](https://github.com/giovani-dev/caronte)
- **Issues**: [https://github.com/giovani-dev/caronte/issues](https://github.com/giovani-dev/caronte/issues)
- **Discussions**: [https://github.com/giovani-dev/caronte/discussions](https://github.com/giovani-dev/caronte/discussions)
- **Documentation**: [Project README](README.md)

---

<div align="center">

### Ready to Contribute?

[![Open Issues](https://img.shields.io/github/issues/giovani-dev/caronte?style=for-the-badge)](https://github.com/giovani-dev/caronte/issues)
[![Good First Issues](https://img.shields.io/github/issues/giovani-dev/caronte/good%20first%20issue?style=for-the-badge&color=green)](https://github.com/giovani-dev/caronte/labels/good%20first%20issue)
[![Help Wanted](https://img.shields.io/github/issues/giovani-dev/caronte/help%20wanted?style=for-the-badge&color=blue)](https://github.com/giovani-dev/caronte/labels/help%20wanted)

**Welcome to the Caronte community! Let's build the future of multi-cloud development together! 🚢**

</div>
