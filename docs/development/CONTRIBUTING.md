# Contributing to Agentic-Flow

Thank you for your interest in contributing to Agentic-Flow! This guide will help you understand our code quality standards and development workflow.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Code Quality Standards](#code-quality-standards)
- [Development Workflow](#development-workflow)
- [Commit Guidelines](#commit-guidelines)
- [Pull Request Process](#pull-request-process)
- [Testing Requirements](#testing-requirements)

## Code of Conduct

We are committed to providing a welcoming and inclusive environment. Please be respectful and professional in all interactions.

## Getting Started

### Prerequisites

- Node.js >= 18.0.0
- npm >= 8.0.0
- Git

### Setup

```bash
# Clone the repository
git clone https://github.com/ruvnet/agentic-flow.git
cd agentic-flow

# Install dependencies
npm install

# Setup Git hooks
npm run prepare

# Build the project
npm run build

# Run tests
npm test
```

## Code Quality Standards

We maintain high code quality standards to ensure reliability, maintainability, and security.

### TypeScript Standards

#### Strict Type Checking

All TypeScript code must pass strict type checking:

```typescript
// ✅ GOOD: Explicit types
function processUser(user: User): Promise<UserResult> {
  return userService.process(user);
}

// ❌ BAD: Implicit any
function processUser(user) {
  return userService.process(user);
}
```

#### No Explicit Any

Avoid using `any` type. Use proper types or `unknown` when necessary:

```typescript
// ✅ GOOD: Proper typing
function handleData(data: unknown): void {
  if (isValidUser(data)) {
    processUser(data);
  }
}

// ❌ BAD: Using any
function handleData(data: any): void {
  processUser(data);
}
```

#### Type Imports

Use type imports for better tree-shaking:

```typescript
// ✅ GOOD: Type imports
import type { User, Config } from './types';
import { processUser } from './services';

// ❌ BAD: Mixed imports
import { User, Config, processUser } from './services';
```

### Code Complexity

#### Function Complexity

Keep cyclomatic complexity under 15:

```typescript
// ✅ GOOD: Simple, focused function
function validateUser(user: User): ValidationResult {
  if (!user.email) {
    return { valid: false, error: 'Email required' };
  }
  if (!user.name) {
    return { valid: false, error: 'Name required' };
  }
  return { valid: true };
}

// ❌ BAD: Too complex
function processUser(user: User) {
  if (user) {
    if (user.email) {
      if (validateEmail(user.email)) {
        if (user.name) {
          if (user.age > 0) {
            // ... 10 more nested conditions
          }
        }
      }
    }
  }
}
```

#### Function Length

Keep functions under 150 lines:

```typescript
// ✅ GOOD: Extracted into smaller functions
function createUser(data: UserData): User {
  const validated = validateUserData(data);
  const normalized = normalizeUserData(validated);
  const user = buildUserObject(normalized);
  return user;
}

// ❌ BAD: 200-line function doing everything
```

#### Maximum Parameters

Limit functions to 5 parameters. Use options object for more:

```typescript
// ✅ GOOD: Options object
interface CreateUserOptions {
  name: string;
  email: string;
  age: number;
  role: string;
  permissions: string[];
  metadata?: Record<string, unknown>;
}

function createUser(options: CreateUserOptions): User {
  // Implementation
}

// ❌ BAD: Too many parameters
function createUser(
  name: string,
  email: string,
  age: number,
  role: string,
  permissions: string[],
  metadata: Record<string, unknown>
): User {
  // Implementation
}
```

### Async/Await Best Practices

#### Always Await Promises

```typescript
// ✅ GOOD: Proper error handling
async function fetchUser(id: string): Promise<User> {
  try {
    const user = await userService.findById(id);
    return user;
  } catch (error) {
    logger.error('Failed to fetch user', { id, error });
    throw new UserNotFoundError(id);
  }
}

// ❌ BAD: Floating promise
async function fetchUser(id: string): Promise<User> {
  userService.findById(id); // Not awaited!
  return null;
}
```

#### Return Await in Try-Catch

```typescript
// ✅ GOOD: Return await for proper error handling
async function getUser(id: string): Promise<User> {
  try {
    return await userService.findById(id);
  } catch (error) {
    throw new UserNotFoundError(id);
  }
}

// ❌ BAD: Not awaiting return
async function getUser(id: string): Promise<User> {
  try {
    return userService.findById(id); // May bypass catch
  } catch (error) {
    throw new UserNotFoundError(id);
  }
}
```

### Error Handling

#### Custom Error Classes

```typescript
// ✅ GOOD: Typed error classes
class UserNotFoundError extends Error {
  constructor(public userId: string) {
    super(`User not found: ${userId}`);
    this.name = 'UserNotFoundError';
  }
}

// ❌ BAD: Throwing strings
throw 'User not found';
```

#### Never Throw Non-Errors

```typescript
// ✅ GOOD: Throw Error objects
throw new ValidationError('Invalid input');

// ❌ BAD: Throw primitives
throw 'Invalid input';
throw { error: 'Invalid input' };
```

### Security Standards

#### No Eval or Dynamic Code Execution

```typescript
// ✅ GOOD: Safe code execution
const result = executeFunction(validatedFunction, args);

// ❌ BAD: Using eval
eval(userInput); // Never do this!
new Function(userInput)(); // Also dangerous
```

#### Input Validation

```typescript
// ✅ GOOD: Validated input
function processUserId(id: string): User {
  if (!/^[a-zA-Z0-9-]+$/.test(id)) {
    throw new ValidationError('Invalid user ID format');
  }
  return userService.findById(id);
}

// ❌ BAD: No validation
function processUserId(id: string): User {
  return userService.findById(id); // SQL injection risk!
}
```

## Formatting Standards

### Prettier Configuration

We use Prettier for consistent code formatting. Configuration is in `config/.prettierrc.js`:

- Single quotes
- 2 space indentation
- 100 character line length
- Trailing commas (ES5)
- Semicolons required

### Editor Setup

Add to your `.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.tsdk": "node_modules/typescript/lib"
}
```

## Development Workflow

### 1. Create a Branch

```bash
git checkout -b feature/your-feature-name
# or
git checkout -b fix/bug-description
```

### 2. Make Changes

Write code following our standards. The pre-commit hook will automatically:
- Run ESLint on staged TypeScript files
- Format code with Prettier
- Run type checking

### 3. Run Quality Checks

```bash
# Lint code
npm run lint

# Fix linting issues
npm run lint:fix

# Format code
npm run format

# Check formatting
npm run format:check

# Type checking
npm run typecheck

# Strict type checking
npm run typecheck:strict

# Run all quality checks
npm run quality:check
```

### 4. Run Tests

```bash
# Unit tests
npm test

# Tests with coverage
npm run test:coverage

# Watch mode
npm run test:watch
```

### 5. Build

```bash
npm run build
```

## Commit Guidelines

We follow Conventional Commits specification:

### Commit Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code style changes (formatting, semicolons, etc.)
- **refactor**: Code refactoring
- **perf**: Performance improvements
- **test**: Adding or updating tests
- **chore**: Maintenance tasks
- **ci**: CI/CD changes

### Examples

```bash
# Feature
git commit -m "feat(agent): Add Byzantine consensus coordination"

# Bug fix
git commit -m "fix(memory): Resolve memory leak in session manager"

# Documentation
git commit -m "docs(api): Update agent orchestration examples"

# Breaking change
git commit -m "feat(api)!: Change agent spawn interface

BREAKING CHANGE: Agent spawn now requires topology parameter"
```

## Pull Request Process

### 1. Pre-PR Checklist

- [ ] All tests pass
- [ ] Code coverage meets threshold (80%+)
- [ ] No ESLint errors
- [ ] Code is formatted with Prettier
- [ ] TypeScript strict mode passes
- [ ] Documentation is updated
- [ ] CHANGELOG.md is updated

### 2. Create Pull Request

1. Push your branch to GitHub
2. Create a Pull Request against `main` or `develop`
3. Fill in the PR template completely
4. Link related issues

### 3. PR Title Format

Use conventional commit format:

```
feat(scope): Add new feature
fix(scope): Fix critical bug
docs(scope): Update documentation
```

### 4. PR Description Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing performed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex code
- [ ] Documentation updated
- [ ] No new warnings generated
- [ ] Tests pass locally
- [ ] Changes are backwards compatible

## Related Issues
Closes #123
Related to #456
```

### 5. CI Checks

When you open a pull request, the following automated checks run automatically:

| Check | Workflow | What it validates |
|-------|----------|-------------------|
| Lint & Format | `ci.yml` | ESLint and Prettier |
| Build | `ci.yml` | Compiles on Node 20 & 22, Ubuntu / macOS / Windows |
| Test | `ci.yml` | Jest test suite with coverage |
| Security Audit | `ci.yml` | `npm audit --audit-level=high` |
| CodeQL | `security.yml` | Static security analysis |
| Secret Detection | `security.yml` | TruffleHog verified secret scan |
| Dependency Review | `security.yml` | Blocks high-severity or restricted-license packages |

All checks must be green before a PR can be merged. See [CI/CD Setup](CI-CD-SETUP.md) for details on each workflow and how to reproduce failures locally.

### 6. Code Review

All PRs require:
- At least one approval from a maintainer
- All CI checks passing
- No merge conflicts

### 7. After Approval

- Squash and merge or rebase as appropriate
- Delete branch after merge

## Testing Requirements

### Unit Tests

- Minimum 80% code coverage
- Test all public APIs
- Test error conditions
- Use descriptive test names

```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid data', async () => {
      // Arrange
      const userData = { name: 'Test', email: 'test@example.com' };

      // Act
      const user = await userService.createUser(userData);

      // Assert
      expect(user.id).toBeDefined();
      expect(user.name).toBe(userData.name);
    });

    it('should throw ValidationError for invalid email', async () => {
      // Arrange
      const userData = { name: 'Test', email: 'invalid' };

      // Act & Assert
      await expect(userService.createUser(userData))
        .rejects.toThrow(ValidationError);
    });
  });
});
```

### Integration Tests

- Test component interactions
- Test external dependencies with mocks
- Test happy paths and error scenarios

## Questions?

If you have questions, please:
1. Check existing documentation
2. Search closed issues
3. Create a new issue with the `question` tag

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
