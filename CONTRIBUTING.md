# Contributing to Synheart Focus

We welcome contributions to Synheart Focus! This is a multi-platform project containing SDKs for Python, Flutter/Dart, Android/Kotlin, and iOS/Swift. This document provides guidelines for contributing to the project.

## 🚀 Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork** locally:
   ```bash
   git clone https://github.com/your-username/synheart-focus.git
   cd synheart-focus
   ```
3. **Install dependencies** based on which SDK you're working on (see below)

## 📂 Repository Structure

This is a monorepo containing multiple SDKs:

```
synheart-focus/
├── docs/             # Documentation (Architecture, API Reference)
├── models/           # Model definitions and assets (if applicable)
├── examples/         # Example applications
├── scripts/          # Build and deployment scripts
└── [SDK repos]       # Platform-specific SDK repositories
```

**Platform-specific SDK repositories** (maintained separately):
- `synheart-focus-python` - Python SDK
- `synheart-focus-dart` - Flutter/Dart SDK
- `synheart-focus-kotlin` - Android/Kotlin SDK
- `synheart-focus-swift` - iOS/Swift SDK

## 🧪 Development Setup by SDK

### Python SDK

```bash
cd synheart-focus-python

# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt

# Run tests
pytest

# Run tests with coverage
pytest --cov=src/synheart_focus --cov-report=html
```

### Flutter/Dart SDK

```bash
cd synheart-focus-dart

# Install dependencies
flutter pub get

# Run tests
flutter test

# Run tests with coverage
flutter test --coverage
```

### Kotlin/Android SDK

```bash
cd synheart-focus-kotlin

# Build
./gradlew build

# Run tests
./gradlew test
```

### Swift/iOS SDK

```bash
cd synheart-focus-swift

# Build
swift build

# Run tests
swift test
```

## 📝 Development Workflow

1. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes**:
   - Follow the code style for each platform
   - Add tests for new functionality
   - Update documentation as needed

3. **Run tests**:
   - Ensure all tests pass
   - Check code coverage

4. **Commit your changes**:
   ```bash
   git commit -m "Add: description of your changes"
   ```

5. **Push to your fork**:
   ```bash
   git push origin feature/your-feature-name
   ```

6. **Create a Pull Request**:
   - Reference any related issues
   - Provide a clear description of changes
   - Include test results

## 🎨 Code Style

### Python
- Follow PEP 8
- Use type hints
- Maximum line length: 100 characters
- Use `black` for formatting
- Use `isort` for import sorting

### Dart
- Follow [Effective Dart](https://dart.dev/guides/language/effective-dart)
- Use `dart format` for formatting
- Follow Flutter conventions

### Kotlin
- Follow [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html)
- Use `ktlint` for formatting

### Swift
- Follow [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)
- Use `swiftformat` for formatting

## 🧪 Testing

- Write tests for all new functionality
- Maintain or improve code coverage
- Test on multiple platforms when applicable
- Include edge cases and error conditions

## 📚 Documentation

- Update README.md if adding new features
- Add API documentation for public APIs
- Include usage examples
- Update CHANGELOG.md for user-facing changes

## 🐛 Reporting Bugs

Use the [GitHub issue tracker](https://github.com/synheart-ai/synheart-focus/issues) and include:

- Description of the bug
- Steps to reproduce
- Expected behavior
- Actual behavior
- Platform and version information
- Relevant logs or error messages

## 💡 Feature Requests

We welcome feature requests! Please:

- Check if the feature already exists
- Describe the use case
- Explain the expected behavior
- Consider implementation complexity

## 📄 License

By contributing, you agree that your contributions will be licensed under the Apache 2.0 License.

## 🙏 Thank You!

Thank you for contributing to Synheart Focus! Your efforts help make cognitive concentration inference accessible to everyone.

---

**Author**: Israel Goytom  
**Organization**: Synheart Research & Engineering

