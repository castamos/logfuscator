# logfuscator

Replace sensitive data in text files (logs, traces, etc.) with anonymized placeholders, optionally allowing you to revert the changes.

## Overview

logfuscator is a tool designed to sanitize log files and other text-based outputs by replacing sensitive information with consistent, meaningless tokens. This allows you to:

- Share log files with external parties without exposing sensitive data
- Debug production issues using sanitized logs
- Create test data from real logs
- Maintain mappings to reverse the obfuscation when needed

## Features

**Planned capabilities:**

- **Automatic Detection**: Identifies common sensitive patterns (emails, IP addresses, hostnames, API keys, etc.)
- **Consistent Replacement**: Same input always maps to the same placeholder (e.g., `user@example.com` → `EMAIL_001`)
- **Reversible**: Optional mapping file allows restoration of original values
- **Configurable**: Custom patterns and replacement strategies
- **Preserves Structure**: Maintains log format and readability

## Status

⚠️ **This project is currently in early development.** No implementation exists yet.

## Installation

*Coming soon*

```bash
# Planned installation method
cpanm logfuscator
```

## Usage

*Planned usage:*

```bash
# Fuscate a log file
logfuscator input.log -o sanitized.log -m mapping.json

# Revert changes using mapping
logfuscator sanitized.log -r -m mapping.json -o restored.log
```

## Example

**Original log:**
```
2024-01-15 10:23:45 User john.doe@company.com logged in from 192.168.1.100
2024-01-15 10:24:12 API key abc123def456 used by john.doe@company.com
```

**Fuscated log:**
```
2024-01-15 10:23:45 User EMAIL_001 logged in from IP_001
2024-01-15 10:24:12 API key APIKEY_001 used by EMAIL_001
```

**Mapping file (mapping.json):**
```json
{
  "EMAIL_001": "john.doe@company.com",
  "IP_001": "192.168.1.100",
  "APIKEY_001": "abc123def456"
}
```

## Detected Patterns

*Planned detection for:*

- Email addresses
- IP addresses (IPv4 and IPv6)
- Hostnames and domains
- API keys and tokens
- Credit card numbers
- Social Security Numbers
- Phone numbers
- Custom patterns via regex

## Configuration

*Configuration options will be available via:*

- Command-line arguments
- Configuration file (`.logfuscator.conf`)
- Environment variables

## Requirements

- Perl 5.10 or higher
- Additional CPAN modules (TBD)

## Development

### Building from Source

```bash
git clone https://github.com/castamos/logfuscator.git
cd logfuscator
perl Makefile.PL
make
make test
make install
```

### Running Tests

```bash
make test
```

### Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -am 'Add new feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.

## Roadmap

- [ ] Core pattern detection engine
- [ ] Mapping file generation and management
- [ ] Revert functionality
- [ ] Command-line interface
- [ ] Configuration file support
- [ ] Custom pattern definitions
- [ ] Comprehensive test suite
- [ ] Documentation and examples

## Author

Created by castamos

## Acknowledgments

Inspired by the need to safely share debug logs without compromising sensitive information.
