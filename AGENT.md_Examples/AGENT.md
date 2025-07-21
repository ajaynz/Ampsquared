## GitHub Workflow

### Issue & Pull Request Management
- Always use GitHub MCP (Managed Code Profiles) for interacting with the GitHub repository
- Use GitHub MCP for fetching issues, creating pull requests, commenting, and any other GitHub operations
- Auth tokens are already associated with the GitHub MCP functions - do not use separate auth tokens
- Never use curl commands for GitHub API operations
- For operations like fetching issues, commit changes, or creating pull requests, rely exclusively on the GitHub MCP tools

## Development Guidelines

### Core Philosophy

**TEST-DRIVEN DEVELOPMENT IS NON-NEGOTIABLE.** Every single line of production code must be written in response to a failing test. No exceptions. This is not a suggestion or a preference - it is the fundamental practice that enables all other principles.

I follow Test-Driven Development (TDD) with a strong emphasis on behavior-driven testing and functional programming principles. All work should be done in small, incremental changes that maintain a working state throughout development.

### Quick Reference

**Key Principles:**
- Write tests first (TDD)
- Test behavior, not implementation
- No `any` types or type assertions
- Immutable data only
- Small, pure functions
- TypeScript strict mode always
- Use real schemas/types in tests, never redefine them

**Preferred Tools:**
- **Language**: TypeScript (strict mode)
- **Testing**: Jest/Vitest + React Testing Library
- **State Management**: Prefer immutable patterns

### Testing Principles

#### Behavior-Driven Testing
- **No "unit tests"** - this term is not helpful. Tests should verify expected behavior, treating implementation as a black box
- Test through the public API exclusively - internals should be invisible to tests
- No 1:1 mapping between test files and implementation files
- Tests that examine internal implementation details are wasteful and should be avoided
- **Coverage targets**: 100% coverage should be expected at all times, but these tests must ALWAYS be based on business behaviour, not implementation details
- Tests must document expected business behaviour

#### Test Data Pattern
Use factory functions with optional overrides for test data:

```typescript
const getMockPaymentPostPaymentRequest = (
  overrides?: Partial<PostPaymentsRequestV3>
): PostPaymentsRequestV3 => {
  return {
    CardAccountId: "1234567890123456",
    Amount: 100,
    Source: "Web",
    AccountStatus: "Normal",
    LastName: "Doe",
    DateOfBirth: "1980-01-01",
    PayingCardDetails: {
      Cvv: "123",
      Token: "token",
    },
    AddressDetails: getMockAddressDetails(),
    Brand: "Visa",
    ...overrides,
  };
};
```

### TypeScript Guidelines

#### Strict Mode Requirements
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

- **No `any`** - ever. Use `unknown` if type is truly unknown
- **No type assertions** (`as SomeType`) unless absolutely necessary with clear justification
- **No `@ts-ignore`** or `@ts-expect-error` without explicit explanation
- These rules apply to test code as well as production code

### Development Workflow

#### TDD Process - THE FUNDAMENTAL PRACTICE
**CRITICAL**: TDD is not optional. Every feature, every bug fix, every change MUST follow this process:

1. **Red**: Write a failing test for the desired behavior. NO PRODUCTION CODE until you have a failing test.
2. **Green**: Write the MINIMUM code to make the test pass. Resist the urge to write more than needed.
3. **Refactor**: Assess the code for improvement opportunities. If refactoring would add value, clean up the code while keeping tests green.

**Common TDD Violations to Avoid:**
- Writing production code without a failing test first
- Writing multiple tests before making the first one pass
- Writing more production code than needed to pass the current test
- Skipping the refactor assessment step when code could be improved

**Remember**: If you're typing production code and there isn't a failing test demanding that code, you're not doing TDD.

### Code Quality Guidelines

#### No Comments in Code
Code should be self-documenting through clear naming and structure. Comments indicate that the code itself is not clear enough.

#### Prefer Options Objects
Use options objects for function parameters as the default pattern. Only use positional parameters when there's a clear, compelling reason.

```typescript
// Good: Options object with clear property names
type CreatePaymentOptions = {
  amount: number;
  currency: string;
  cardId: string;
  customerId: string;
  description?: string;
};

const createPayment = (options: CreatePaymentOptions): Payment => {
  // implementation
};
```

#### Understanding DRY - It's About Knowledge, Not Code
DRY (Don't Repeat Yourself) is about not duplicating **knowledge** in the system, not about eliminating all code that looks similar.

### Refactoring Guidelines

Refactoring must never break existing consumers of your code. Always:
1. Commit before refactoring
2. Look for useful abstractions based on semantic meaning
3. Maintain external APIs during refactoring
4. Verify and commit after refactoring

### Error Handling
Use Result types or early returns:

```typescript
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

const processPayment = (
  payment: Payment
): Result<ProcessedPayment, PaymentError> => {
  if (!isValidPayment(payment)) {
    return { success: false, error: new PaymentError("Invalid payment") };
  }
  
  return { success: true, data: executePayment(payment) };
};
```
