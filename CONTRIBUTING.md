# Contributing to UI5 App Skill

Thank you for your interest in contributing to the UI5 App Skill documentation! This project helps developers build better SAPUI5/Fiori applications by providing comprehensive standards and best practices.

## How Can I Contribute?

### Reporting Issues

If you find errors, inconsistencies, or areas for improvement in the documentation:

1. Check if the issue already exists in the [Issues](../../issues) section
2. If not, create a new issue with:
   - Clear title describing the problem
   - Detailed description of the issue
   - Location in the documentation (file name, section, line number if applicable)
   - Suggested improvement (if applicable)

### Suggesting Enhancements

We welcome suggestions for:
- New SAPUI5/Fiori patterns and best practices
- Additional code examples
- Clarifications of existing content
- New sections covering additional topics

To suggest an enhancement:
1. Open an issue with the label `enhancement`
2. Describe the proposed change and its benefits
3. Provide examples if possible

### Submitting Changes

#### Setting Up Your Environment

1. Fork the repository
2. Clone your fork:
   ```bash
   git clone https://github.com/YOUR_USERNAME/ui5-app-skill.git
   cd ui5-app-skill
   ```
3. Create a new branch for your changes:
   ```bash
   git checkout -b feature/your-feature-name
   ```

#### Making Changes

1. Edit the relevant files (`SKILL.md` for technical content, `README.md` for overview)
2. Follow the existing formatting and style conventions
3. Ensure code examples are syntactically correct
4. Test any new code examples in a real SAPUI5 environment if possible

#### Commit Guidelines

Use clear, descriptive commit messages:

```
Add section on OData v4 side effects

- Explain requestSideEffects method
- Provide code examples for after-action updates
- Include best practices
```

#### Submitting Pull Requests

1. Push your changes to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
2. Open a Pull Request from your fork to the main repository
3. Provide a clear description of the changes
4. Reference any related issues

## Style Guidelines

### Documentation Structure

- Use clear, concise language
- Follow the existing table of contents structure in SKILL.md
- Use code blocks with language specification for all code examples
- Include comments in code examples where helpful

### Code Examples

- Use realistic, production-ready code
- Follow the Hungarian notation conventions defined in the skill
- Use the example namespace `com.relacon.purchorders` or replace with generic examples
- Include error handling in critical examples
- Test code snippets before submitting

### Formatting

- Use ATX-style headers (`# Header`)
- Use fenced code blocks with language tags
- Use tables for structured data
- Use bullet points for lists
- Keep line length reasonable (wrap at 120 characters when possible)

## Areas of Focus

We especially welcome contributions in these areas:

- **Additional UI5 Libraries**: Guidelines for sap.ui.comp, sap.ui.table, etc.
- **Testing Patterns**: Unit testing with QUnit, OPA5 integration testing
- **TypeScript**: TypeScript support and patterns for UI5
- **CI/CD**: Build and deployment automation
- **Security**: Security best practices for UI5 applications
- **Performance**: Additional performance optimization techniques
- **Accessibility**: A11y guidelines and patterns

## Code of Conduct

### Our Standards

- Be respectful and inclusive in all interactions
- Accept constructive criticism gracefully
- Focus on what is best for the community and developers

### Unacceptable Behavior

- Trolling, insulting/derogatory comments, or personal attacks
- Public or private harassment
- Publishing others' private information without permission
- Other conduct which could reasonably be considered inappropriate

## Questions?

If you have questions about contributing:
- Open an issue with the label `question`
- Contact the maintainers

## Attribution

This contributing guide is adapted from standard open source contribution guidelines.

Thank you for helping make UI5 development better for everyone!
