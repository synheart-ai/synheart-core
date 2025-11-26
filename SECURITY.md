# Security Policy

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| 1.0.x   | :white_check_mark: |
| < 1.0   | :x:                |

## Reporting a Vulnerability

We take security vulnerabilities seriously. If you discover a security vulnerability, please report it responsibly.

### How to Report

**Email**: security@synheart.ai

**Please include**:
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

### Security Response Time

- **Initial Response**: Within 48 hours
- **Patch Timeline**: Based on severity
  - Critical: 7 days
  - High: 30 days
  - Medium: 90 days
  - Low: Next release cycle

## Security Best Practices

### For Developers

1. **Never commit secrets**: API keys, tokens, credentials
2. **Use environment variables**: For sensitive configuration
3. **Validate inputs**: All user inputs and external data
4. **Keep dependencies updated**: Regularly update dependencies
5. **Use secure defaults**: Encryption, authentication, authorization

### For Users

1. **Keep SDKs updated**: Use the latest versions
2. **Review permissions**: Only grant necessary permissions
3. **Secure storage**: Encrypt sensitive data at rest
4. **Network security**: Use HTTPS for all API calls
5. **Consent management**: Respect user privacy preferences

## Privacy Considerations

Synheart Focus is designed with privacy-first principles:

- **No Raw Data Storage**: Only derived focus signals are stored
- **On-Device Processing**: All inference happens locally
- **Consent-Gated**: Behavioral data requires explicit consent
- **No Content Extraction**: No text, URLs, or screen content
- **Encrypted Transmission**: All cloud uploads are encrypted

## Known Security Considerations

### Data Privacy
- Focus scores are derived signals, not raw biometrics
- No personal identifiers in focus data
- User consent required for cloud uploads

### Platform Permissions
- **iOS**: Requires appropriate Info.plist permissions
- **Android**: Requires appropriate manifest permissions
- **Web**: Requires user consent for sensor access

## Security Updates

Security updates will be:
- Released as patch versions (e.g., 1.0.1)
- Documented in CHANGELOG.md
- Announced via GitHub releases
- Prioritized in release cycles

## Disclosure Policy

We follow responsible disclosure:
1. Report vulnerability privately
2. Allow time for fix development
3. Coordinate public disclosure
4. Credit researchers (with permission)

---

**Last Updated**: 2025-11-25  
**Security Contact**: security@synheart.ai

