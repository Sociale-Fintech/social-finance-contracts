# Documentation

Welcome to Sociale Finance smart contracts documentation.

---

## Quick Links

- **[v1 Architecture](./v1/ARCHITECTURE.md)** - Current version design and specifications

---

## What We're Building

Smart contract validators that enable community-based savings groups to operate on-chain with transparency, automation, and security.

### Supported Entity Types
- **ROSCAs** - Rotating Savings and Credit Associations
- **VSLAs** - Village Savings and Loan Associations  
- **SACCOs** - Savings and Credit Cooperatives
- **ASCAs** - Accumulating Savings and Credit Associations

---

## Phased Development

We build incrementally, with each version adding focused capabilities:

### v1.0 - Core ROSCA (Current)
- Single validator proving concept
- Basic rotation and payouts
- Minimal complexity

### v1.1 - Architecture (Planned)
- Split into Treasury + Cycle validators
- Scalable multi-validator design

### v1.2 - Decentralization (Planned)
- Add Governance + Membership
- Community control

### v1.3 - Lending (Planned)
- Add Loans + Credit Tracking
- Internal lending capabilities

---

## Documentation Structure

```
architecture/
├── DOCS.md              # This file - documentation hub
└── v1/                  # v1.0 specific documentation
    ├── ARCHITECTURE.md  # Complete v1.0 design
    └── diagrams/        # Architecture diagrams
```

More version-specific folders (v1_1/, v1_2/, etc.) will be added as we build them.

---

## Getting Started

1. Read [v1 Architecture](./v1/ARCHITECTURE.md) to understand the current design
2. Review the [main README](../README.md) for development setup
3. Check the validators in `/validators/v1/` for implementation

---

## Contributing

We welcome contributions! 

- **Bug reports**: Open an issue
- **Feature ideas**: Start a discussion
- **Code contributions**: Submit a PR
- **Documentation**: Improvements always welcome

---

## Philosophy

### Why Incremental?

We build in small, focused phases because:
- **Lower Risk** - Each version is simple and testable
- **Faster Feedback** - Get real-world usage early
- **Better Learning** - Understand problems before adding complexity
- **Continuous Value** - Each version is production-ready

### Design Principles

1. **Simplicity First** - Start with minimal viable implementation
2. **Progressive Complexity** - Add features only when proven necessary
3. **Transparency** - All state and logic on-chain
4. **Security** - Auditable, tested, and conservative design
5. **Reusability** - Build components that work for multiple use cases

---

## Questions?

- Check [v1 Architecture](./v1/ARCHITECTURE.md) for detailed technical design
- Review code in `/validators/v1/`
- Open an issue for questions

---

**Note**: The `docs/` folder is reserved for Aiken's auto-generated documentation. Our design docs are in `architecture/`.

---

**Current Version**: v1.0 (Development)  
**License**: Apache 2.0

