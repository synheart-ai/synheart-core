# Tools

Development tools for the Synheart Core SDK.

## Available Tools

As the Core SDK develops, tools may be added for:

- **HSV validation**: Verify Human State Vector calculations
- **HSI schema validation**: Ensure HSI 1.0 export format compliance
- **Capability testing**: Test capability system enforcement
- **Consent validation**: Verify consent model compliance
- **Performance benchmarks**: Measure HSV computation latency and resource usage
- **Integration testing**: Test coordination between modules (Wear, Phone, Behavior, etc.)
- **Cross-platform testing**: Test consistency across SDKs (Dart, Kotlin, Swift)

## Contributing

When adding new tools:

1. Create a dedicated directory for complex tools
2. Add a README.md explaining the tool's purpose and usage
3. Update this main README with tool information
4. Ensure tools work across all supported platforms
5. Add CI/CD integration where appropriate

See [CONTRIBUTING.md](../CONTRIBUTING.md) for more details.
