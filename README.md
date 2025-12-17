# Sociale Finance Smart Contracts

Smart contract validators for social finance entities on Cardano.

---

## Overview

This repository contains on-chain validators that power savings groups (ROSCAs, VSLAs, SACCOs). Built with [Aiken](https://aiken-lang.org) for Cardano blockchain.

---

## Documentation

📖 See [architecture/DOCS.md](./architecture/DOCS.md) for complete documentation.

---

## Development

### Prerequisites
- [Aiken](https://aiken-lang.org) - Latest version
- Cardano node access or [Blockfrost](https://blockfrost.io) API key

### Build
```bash
aiken build
```

### Test
```bash
aiken check
```

### Format
```bash
aiken fmt
```

---

## Repository Structure

```
social-finance-contracts/
├── validators/        # Smart contract validators
├── lib/              # Shared libraries and utilities
├── tests/            # Test suites
└── architecture/     # Documentation and design
```

---

## License

[Apache 2.0](./LICENSE) - Open source with patent and trademark protection.

---

## Contributing

Contributions welcome! Please open an issue or PR.

---

## Contact

For questions or support, reach out to the Sociale Finance team.
