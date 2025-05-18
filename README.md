# DevOps Challenges

![DevOps](https://img.shields.io/badge/DevOps-Challenges-blue?style=for-the-badge&logo=amazon-aws)

> A collection of DevOps-related challenges, study materials, and certification preparation resources.

## Repository Contents

This repository contains resources for various DevOps challenges and certification preparation materials:

### AWS Certification Materials

- [AWS Developer Associate Study Notes](aws-developer-associate.md) - Comprehensive study notes for the AWS Certified Developer - Associate exam (DVA-C02)

## Purpose

This repository serves as a centralized location for:

- DevOps learning materials and challenges
- Cloud certification preparation resources
- Best practices and reference architectures
- Code examples for various cloud services

## How to Use This Repository

1. Browse the available resources in the repository
2. Use the study materials to prepare for certifications
3. Explore code examples to understand implementation details
4. Contribute your own challenges or improvements

## Contributing

Contributions are welcome! If you'd like to add new challenges, improve existing materials, or fix errors:

1. Fork the repository
2. Create a new branch for your changes
3. Submit a pull request with a clear description of your improvements

## Branch Strategy

We follow a simplified Git Flow strategy for this repository:

- `main` - Production-ready code. Protected branch that only accepts merge requests.
- `develop` - Integration branch for features. This is where most pull requests should target.
- `feature/*` - New features or improvements (e.g., `feature/add-eks-notes`).
- `bugfix/*` - Bug fixes (e.g., `bugfix/fix-lambda-example`).
- `docs/*` - Documentation updates (e.g., `docs/update-s3-section`).
- `release/*` - Release preparation branches (e.g., `release/v1.2.0`).
- `hotfix/*` - Urgent fixes for production (e.g., `hotfix/fix-critical-error`).

### Branch Workflow

1. Create a branch from `develop` (or `main` for hotfixes)
2. Work on your changes
3. Submit a pull request back to `develop` (or `main` for hotfixes)
4. After review and approval, your changes will be merged

## Conventional Commits

We use conventional commits to make our commit messages more meaningful and to automate versioning and changelog generation.

### Format

```
<type>: <description>

[optional body]

[optional footer(s)]
```

### Types

- `feat`: A new feature
- `fix`: A bug fix
- `docs`: Documentation changes
- `style`: Changes that don't affect code functionality (formatting, etc.)
- `refactor`: Code changes that neither fix bugs nor add features
- `perf`: Performance improvements
- `test`: Adding or correcting tests
- `chore`: Changes to build process or auxiliary tools

### Examples

```
feat: add new code examples for Lambda functions

fix: correct capacity calculation example

docs: update repository description and add badges
```

## Conventional Pull Request Names

Pull request titles should follow the same convention as commit messages:

```
<type>: <description>
```

### Examples

```
feat: add encryption examples and best practices

fix: correct CORS configuration example

docs: improve alarm configuration documentation
```

## License

MIT License

---

> 💡 **Note**: This repository is continuously updated with new challenges and materials. Check back regularly for updates!
