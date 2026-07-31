# Contributing to OBTest

Thank you for your interest in contributing to **OBTest**! We welcome contributions from the community to help make automated Java black-box testing faster, safer, and more comprehensive.

---

## Code of Conduct

By participating in this project, you agree to abide by our [Code of Conduct](CODE_OF_CONDUCT.md).

---

## How Can I Contribute?

### 1. Reporting Bugs
Before creating a bug report, please check existing issues to see if the problem has already been reported.

When creating an issue, please include:
* **JDK Version**: (e.g. Java 11, Java 17, Java 21)
* **OBTest Version**: (e.g. 1.0.0)
* **Minimal Reproducible Example**: A code snippet demonstrating the issue or unexpected behavior.
* **Expected vs Actual Behavior**.

### 2. Suggesting Enhancements
Feature requests are tracked as GitHub issues. Please provide:
* A clear and detailed explanation of the proposed feature.
* Use cases illustrating why this feature would be valuable.

### 3. Submitting Pull Requests
1. **Fork the repository** and create a feature branch off `main`.
2. **Ensure zero runtime dependencies**: OBTest must remain 100% free of external runtime dependencies.
3. **Write Unit Tests**: Every new feature, bug fix, or edge case handler must be accompanied by comprehensive JUnit tests in `src/test/java`.
4. **Run Build & Verification**:
   ```bash
   mvn clean test
   ```
5. **Commit Message Standards**: Use concise, descriptive commit messages (e.g. `feat: add custom date generator`, `fix: primitive array handling`).
6. **Submit PR**: Open a Pull Request targeting the `main` branch.

---

## Development Setup

### Prerequisites
* **JDK 8 or higher** (JDK 17 or 21 recommended for development)
* **Apache Maven 3.6+**

### Building Locally
```bash
git clone https://github.com/odebitzltd/OBTest.git
cd OBTest
mvn clean package
```

Thank you for helping improve OBTest!
