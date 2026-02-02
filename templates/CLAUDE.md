# CLAUDE.md - Project Constitution for Claude Code

## 🚨 CRITICAL RULES (NON-NEGOTIABLE)

These rules are **mandatory** and must be followed without exception:

1. **No Circular Dependencies**: Never import a module into itself or create circular references between services. Use `forwardRef()` only as a last resort.

2. **Strict Typing**: `noImplicitAny` is TRUE. Do not use `any` type. Create DTOs (Data Transfer Objects) for all inputs.

3. **One File, One Class**: Adhere strictly to the Single Responsibility Principle. Each file should contain exactly one class.

4. **Test-Driven Development**: You must not write implementation code without first writing a failing test (Red-Green-Refactor).

5. **Dependency Injection**: Never instantiate classes with `new`. Always use constructor injection.

6. **No Test Modification**: Developer agents CANNOT modify test files (`*.spec.ts`, `test/*`). Only QA agents can write/modify tests.

7. **Shared Task Notes**: Always read and update `SHARED_TASK_NOTES.md` to maintain context across iterations.

8. **Validation Gates**: Every implementation phase must pass defined validation gates before proceeding.

9. **LSP for Refactoring**: Always use TypeScript Language Server (LSP) via MCP for refactoring. Never use text search (grep/find) for renaming or moving code.

10. **Mutation Testing**: All code must achieve 80%+ mutation score (not just code coverage). Use StrykerJS to verify test quality.

---

## Code Navigation and Refactoring

### Use LSP (Language Server Protocol) for Intelligent Navigation

**Always use LSP via MCP for:**
- Finding all references to a class/function: `typescript_find_references("AuthService")`
- Understanding types: `typescript_get_type_definition("user.id")`
- Safe renaming: `typescript_rename_symbol("oldName", "newName")`
- Finding implementations: `typescript_find_implementations("IRepository")`
- Checking for errors: `typescript_get_diagnostics()`

**Never use text search (grep) for refactoring:**
- ❌ `grep -r "AuthService"` - Finds in comments, strings, variable names
- ✅ `typescript_find_references("AuthService")` - Only actual code references

**LSP is slower (1-2 seconds) but smart. Text search is fast but dumb.**

---

## Project Overview

**Project Name:** [Your Project Name]

**Description:** [Brief description of your project]

**Tech Stack:**
- NestJS (Node.js framework)
- TypeORM (ORM)
- PostgreSQL (Database)
- Jest (Testing)
- Swagger/OpenAPI (API Documentation)

## Architecture Principles

This project follows **Clean Architecture** and **SOLID principles**:

1. **Single Responsibility Principle (SRP)**: Each class has one reason to change.
2. **Open/Closed Principle (OCP)**: Open for extension, closed for modification.
3. **Liskov Substitution Principle (LSP)**: Derived classes are substitutable for base classes.
4. **Interface Segregation Principle (ISP)**: Clients should not depend on methods they don't use.
5. **Dependency Inversion Principle (DIP)**: Depend on abstractions, not concretions.

## Project Structure

```
src/
├── auth/                 # Authentication and authorization
├── users/                # User management
├── [feature]/            # Feature modules
│   ├── [feature].module.ts
│   ├── [feature].controller.ts
│   ├── [feature].service.ts
│   ├── entities/
│   │   └── [feature].entity.ts
│   ├── dto/
│   │   ├── create-[feature].dto.ts
│   │   └── update-[feature].dto.ts
│   ├── interfaces/
│   │   └── [feature]-service.interface.ts
│   ├── repositories/
│   │   └── [feature].repository.ts
│   └── guards/
│       └── [feature]-ownership.guard.ts
├── common/               # Shared utilities, guards, interceptors
├── config/               # Configuration files
└── database/             # Database migrations and seeds
```

## Coding Standards

### General Guidelines

- Use **TypeScript** for all code.
- Follow **ESLint** and **Prettier** configurations.
- Write **self-documenting code** with clear variable and function names.
- Add **JSDoc comments** for public APIs.
- Use **async/await** instead of callbacks or raw promises.

### NestJS Conventions

- Use **dependency injection** for all dependencies.
- Create **DTOs** for all request and response payloads.
- Use **class-validator** decorators for validation.
- Implement **interfaces** for services to enable loose coupling.
- Use **guards** for authentication and authorization.
- Use **interceptors** for cross-cutting concerns (logging, transformation).
- Use **pipes** for validation and transformation.

### Naming Conventions

- **Files**: kebab-case (e.g., `user.service.ts`)
- **Classes**: PascalCase (e.g., `UserService`)
- **Interfaces**: PascalCase with "I" prefix (e.g., `IUserService`)
- **Variables/Functions**: camelCase (e.g., `findUser`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_RETRY_ATTEMPTS`)

### Error Handling

- Use **NestJS built-in exceptions** (e.g., `NotFoundException`, `BadRequestException`).
- Create **custom exceptions** for domain-specific errors.
- Never expose sensitive information in error messages.
- Log errors server-side with proper context.

## Testing Standards

### Test Coverage

- **Minimum coverage:** 80%
- **Target coverage:** 85%+

### Test Types

- **Unit Tests (70%)**: Test individual components in isolation.
- **Integration Tests (20%)**: Test interactions between components.
- **E2E Tests (10%)**: Test complete user flows.

### Testing Guidelines

#### NestJS Unit Testing with Dependency Injection Mocking

**CRITICAL PATTERN:** When writing unit tests for NestJS services, **NEVER** import the real module. Always use `Test.createTestingModule` and mock all dependencies.

**✅ CORRECT PATTERN (ALWAYS USE THIS):**

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { getRepositoryToken } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { User } from './entities/user.entity';

describe('UsersService', () => {
  let service: UsersService;
  let mockRepository: any;

  beforeEach(async () => {
    // Mock the repository
    mockRepository = {
      create: jest.fn(),
      save: jest.fn(),
      findOne: jest.fn(),
      find: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    };

    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
  });

  it('should create a user', async () => {
    const createUserDto = { username: 'test', email: 'test@example.com' };
    const expectedUser = { id: 1, ...createUserDto };
    
    mockRepository.create.mockReturnValue(expectedUser);
    mockRepository.save.mockResolvedValue(expectedUser);

    const result = await service.create(createUserDto);
    
    expect(mockRepository.create).toHaveBeenCalledWith(createUserDto);
    expect(mockRepository.save).toHaveBeenCalledWith(expectedUser);
    expect(result).toEqual(expectedUser);
  });
});
```

**❌ WRONG PATTERN (NEVER DO THIS):**

```typescript
// ❌ Imports real module with all dependencies
import { UsersModule } from './users.module';

// ❌ Uses real database connection
const module = await Test.createTestingModule({
  imports: [UsersModule],
}).compile();
```

**Why:** Unit tests must be isolated and fast. Real modules bring in database connections, external APIs, and other dependencies that make tests slow and flaky.

---

## Advanced Backend Patterns

### Optimistic Locking for High-Concurrency Scenarios

**When to use:** High-contention entities (e.g., inventory, account balances)

**Pattern:**

```typescript
@Entity()
export class InventoryItem {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  quantity: number;

  @VersionColumn() // Critical: TypeORM auto-increments on every update
  version: number;
}
```

**Error Handling:**

```typescript
async updateInventory(id: number, quantity: number, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const item = await this.repository.findOne({ where: { id } });
      item.quantity = quantity;
      return await this.repository.save(item);
    } catch (error) {
      if (error instanceof OptimisticLockVersionMismatchError) {
        // Transient concurrency conflict - retry
        if (attempt === maxRetries - 1) throw error;
        continue;
      }
      // Persistent error - don't retry
      throw error;
    }
  }
}
```

**Decision Logic:**
- High read/write ratio → Optimistic locking
- Strict consistency required → Pessimistic locking

### Distributed Locking with Redlock (Redis)

**When to use:** Preventing double-processing across microservices (e.g., payments)

**Pattern:**

```typescript
// Custom decorator for distributed locking
export function Redlock(resourceKey: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function (...args: any[]) {
      const lock = await redlockClient.acquire([resourceKey], 5000); // 5s TTL
      try {
        return await originalMethod.apply(this, args);
      } finally {
        await lock.release(); // Always release in finally block
      }
    };
  };
}

// Usage
@Redlock('payment:process')
async processPayment(orderId: string) {
  // Critical section - only one instance can execute at a time
}
```

**Safety Features:**
- Aggressive TTL (5-10 seconds) to prevent deadlocks
- Fencing tokens for stale process detection
- Idempotency keys (Redis/Postgres) for retry safety

### Transaction Isolation Levels

**Repeatable Read:** Prevent phantom reads in multi-query transactions

```typescript
await this.dataSource.transaction('REPEATABLE READ', async (manager) => {
  const user = await manager.findOne(User, { where: { id } });
  const orders = await manager.find(Order, { where: { userId: id } });
  // Data is consistent - no phantom reads
});
```

**Pessimistic Locking:** For financial ledger updates

```typescript
await manager.findOne(Account, id, { 
  lock: { mode: 'pessimistic_write' } 
});
// Row is locked at database level - serialized access
```

### Mutation Testing for Test Quality

**Problem:** Code coverage metrics are misleading. AI can generate tests that execute code but assert nothing meaningful.

**Solution:** Mutation Testing with StrykerJS

**Installation:**

```bash
npm install --save-dev @stryker-mutator/core @stryker-mutator/jest-runner
```

**Configuration (`stryker.conf.json`):**

```json
{
  "testRunner": "jest",
  "coverageAnalysis": "perTest",
  "mutate": ["src/**/*.ts", "!src/**/*.spec.ts"],
  "thresholds": { "high": 80, "low": 60, "break": 50 }
}
```

**Workflow:**

1. **Baseline:** Ensure tests are green
2. **Mutation:** Run `npx stryker run`
3. **Analysis:** Parse `mutation-report.json`
4. **Test Generation:** For each surviving mutant, write test that kills it
5. **Verification:** Confirm new test fails on mutant, passes on original

**Metric:** Mutation Score (% of mutants killed) - Target: 80%+

### Observability-Driven Development (ODD)

**Structured Logging with Pino:**

```typescript
import { Logger } from '@nestjs/common';

export class PaymentService {
  private readonly logger = new Logger(PaymentService.name);

  async processPayment(orderId: string, userId: string) {
    this.logger.log({
      transactionId: generateId(),
      userId,
      orderId,
      action: 'payment:start',
    });
    
    try {
      // Business logic
      this.logger.log({ orderId, action: 'payment:success' });
    } catch (error) {
      this.logger.error({
        orderId,
        action: 'payment:failure',
        error: error.message,
        stack: error.stack,
      });
      throw error;
    }
  }
}
```

**Benefits:**
- Machine-readable JSON logs
- Easy correlation across microservices
- Agent can parse and analyze failures

---

### Testing Guidelines (continued)

- Follow the **AAA pattern** (Arrange, Act, Assert).
- Use **descriptive test names** (e.g., `should return user when found`).
- **Mock external dependencies** in unit tests.
- Use **real database** for integration and E2E tests.
- Ensure **test independence** (no shared state).

## Security Best Practices

- **Never commit secrets** to version control.
- Use **environment variables** for configuration.
- **Hash passwords** with bcrypt (10+ salt rounds).
- Implement **JWT authentication** with reasonable expiration times.
- Use **guards** for route protection.
- Implement **rate limiting** to prevent abuse.
- Validate and sanitize **all user inputs**.
- Use **parameterized queries** to prevent SQL injection.

## Development Workflow

### Starting a New Feature

1. **Invoke the Architect Agent**: Describe the feature you want to build.
2. **Review the Plan**: The Architect Agent will decompose the task and create a plan.
3. **Agent Execution**: Worker agents will execute their tasks in parallel.
4. **Post-Generation Hook**: Automated quality checks will run.
5. **Validation**: The Validation Agent will perform a final audit.
6. **Review and Customize**: Review the generated code and make any necessary adjustments.

### Using the CRUD Generator

For simple CRUD operations, use the `crud-generator` skill:

```bash
@crud-generator generate [EntityName] --auth --ownership --pagination --soft-delete
```

### Using the Design Pattern Implementer

For implementing design patterns, use the `design-pattern-implementer` skill:

```bash
@design-pattern implement [pattern-name] [context]
```

### Post-Generation Workflow

After generating any code, the following steps are automatically executed:

1. **Format code** with Prettier
2. **Run ESLint** and auto-fix issues
3. **Run TypeScript compiler**
4. **Run unit tests**
5. **Check test coverage** (minimum 80%)
6. **Validate architecture** (check for circular dependencies)
7. **Generate quality report**
8. **Invoke Validation Agent** for final review

If any check fails, fix the issues before marking the task as complete.

### Committing Code

Before committing, the pre-commit hook will automatically:

1. Run linting
2. Run formatting checks
3. Run tests for changed files
4. Check for console.log statements
5. Check for hardcoded secrets
6. Check for debugger statements

## Agent Responsibilities

### Architect Agent (Opus)
- Task decomposition and planning
- Agent orchestration
- Code integration
- Final sign-off

### Module Design Agent (Sonnet)
- Create module structure
- Define module boundaries
- Set up imports and exports

### Controller Agent (Sonnet)
- Create RESTful controllers
- Define DTOs with validation
- Generate Swagger documentation

### Service Agent (Sonnet)
- Implement business logic
- Follow SOLID principles
- Use design patterns
- Implement interfaces

### Repository Agent (Sonnet)
- Create TypeORM entities
- Implement custom repositories
- Write efficient queries

### Testing Agent (Opus)
- Write unit tests
- Write integration tests
- Achieve 80%+ coverage

### E2E Test Agent (Opus)
- Write E2E tests for critical flows
- Test against real database
- Validate complete workflows

### Security Agent (Opus)
- Implement authentication
- Implement authorization
- Create security guards
- Follow security best practices

### Validation Agent (Sonnet)
- Run linting and formatting checks
- Verify test coverage
- Check for code smells
- Provide final quality audit

## Common Patterns

### Strategy Pattern

Use for interchangeable algorithms (e.g., payment methods, notification channels):

```typescript
interface IPaymentStrategy {
  processPayment(amount: number, details: any): Promise<PaymentResult>;
}

@Injectable()
class CreditCardPaymentStrategy implements IPaymentStrategy {
  async processPayment(amount: number, details: any): Promise<PaymentResult> {
    // Implementation
  }
}
```

### Factory Pattern

Use for creating objects without specifying exact classes:

```typescript
@Injectable()
class ReportFactory {
  createReport(type: string): IReport {
    switch (type) {
      case 'pdf': return new PdfReport();
      case 'excel': return new ExcelReport();
      default: throw new Error('Unsupported report type');
    }
  }
}
```

### Repository Pattern

Use for data access abstraction:

```typescript
@Injectable()
class UsersRepository {
  constructor(
    @InjectRepository(User)
    private readonly repository: Repository<User>,
  ) {}

  async findByEmail(email: string): Promise<User | null> {
    return this.repository.findOne({ where: { email } });
  }
}
```

## Environment Variables

Required environment variables:

```env
# Database
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USER=user
DATABASE_PASSWORD=password
DATABASE_NAME=dbname

# JWT
JWT_SECRET=your-secret-key
JWT_REFRESH_SECRET=your-refresh-secret

# CORS
ALLOWED_ORIGINS=http://localhost:3000
```

## Git Workflow

1. Create a feature branch: `git checkout -b feature/your-feature`
2. Make changes and commit frequently
3. Run tests before pushing: `npm run test`
4. Push to remote: `git push origin feature/your-feature`
5. Create a pull request
6. Wait for CI/CD checks to pass
7. Request code review
8. Merge to main after approval

## CI/CD Pipeline

The CI/CD pipeline runs the following checks:

1. Install dependencies
2. Run linting
3. Run type checking
4. Run unit tests
5. Run integration tests
6. Run E2E tests
7. Check test coverage
8. Build the application
9. Deploy to staging (on merge to main)
10. Deploy to production (on release tag)

## Documentation

- **API Documentation**: Auto-generated with Swagger at `/api/docs`
- **Code Documentation**: JSDoc comments for public APIs
- **Architecture Documentation**: See `docs/architecture.md`
- **Deployment Documentation**: See `docs/deployment.md`

## Troubleshooting

### Common Issues

1. **Tests failing**: Run `npm run test:cov` to see coverage and identify missing tests.
2. **Linting errors**: Run `npm run lint -- --fix` to auto-fix issues.
3. **Type errors**: Run `npm run build` to see TypeScript errors.
4. **Database connection issues**: Check `.env` file and ensure database is running.

### Getting Help

- Check the [examples](./examples) directory for workflow examples.
- Review the [agents](./agents) directory for agent documentation.
- Consult the [NestJS documentation](https://docs.nestjs.com/).

## Project-Specific Notes

[Add any project-specific guidelines, conventions, or notes here]

---

**Last Updated:** [Date]
**Maintained By:** [Your Name/Team]