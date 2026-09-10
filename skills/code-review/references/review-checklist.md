# Code Review Checklist

## Correctness
- Boundary conditions
- Null/undefined
- Wrong branch
- Incorrect state transition
- Off-by-one
- Wrong unit / timezone / precision

## Security
- Authentication
- Authorization
- Injection
- Secret exposure
- Unsafe deserialization
- Path traversal
- Input validation

## Data
- Transaction boundaries
- Duplicate writes
- Lost updates
- Partial failure
- Idempotency

## API
- Breaking changes
- Invalid status codes
- Error contract
- Validation
- Pagination
- Retry behavior

## Performance
- N+1 queries
- Unbounded loops
- Excess allocations
- Blocking I/O
- Missing index assumptions

## Maintainability
- Tight coupling
- Hidden side effects
- Duplicate domain logic
- Incorrect abstraction boundary
