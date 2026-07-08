# Bug Detection Checklist

Use this as an internal reference during reviews. Not all items apply to every codebase — select based on language, domain, and change scope.

## Logic & Control Flow

- [ ] Off-by-one errors in loops, slices, array indexing
- [ ] Incorrect loop termination conditions (infinite loops, early exits)
- [ ] Missing or incorrect base cases in recursion
- [ ] Fall-through in switch/case without break/return
- [ ] Incorrect operator precedence causing wrong evaluation order
- [ ] Short-circuit evaluation assumptions that don't hold
- [ ] State machine transitions missing or allowing invalid states

## Null/Undefined Safety

- [ ] Dereferencing values that can be null/undefined/None
- [ ] Missing guards before method calls on optional objects
- [ ] Default values that are themselves problematic (empty string vs null)
- [ ] Optional chaining that silently swallows errors instead of surfacing them

## Error Handling

- [ ] Bare catch/except blocks that swallow all errors
- [ ] Exceptions caught but not re-raised or logged
- [ ] Return codes ignored from functions that can fail
- [ ] Partial failures in multi-step operations without rollback
- [ ] Retry logic without backoff causing thundering herds

## Concurrency & Race Conditions

- [ ] Shared mutable state accessed without synchronization
- [ ] Check-then-act patterns without atomicity
- [ ] Deadlock potential in lock ordering
- [ ] Goroutine/thread leaks from unjoined workers
- [ ] Channel/queue operations that block indefinitely
- [ ] Async operations without proper error propagation

## Resource Management

- [ ] File handles, DB connections, network sockets not closed
- [ ] Memory leaks from growing caches without eviction
- [ ] Unbounded data structures (lists, maps) in long-running processes
- [ ] Temporary files not cleaned up on error paths

## Data Integrity

- [ ] Integer overflow/underflow in arithmetic
- [ ] Floating-point comparison without epsilon tolerance
- [ ] String encoding mismatches (UTF-8 vs other encodings)
- [ ] Date/timezone handling errors (naive vs aware datetimes)
- [ ] Precision loss in numeric conversions
- [ ] Incorrect serialization/deserialization of complex types

## API Contract Violations

- [ ] Calling APIs with wrong parameter order or types
- [ ] Assuming response shape that can vary
- [ ] Ignoring pagination, rate limits, or timeouts
- [ ] Using deprecated APIs without migration path
- [ ] Not handling version mismatches in interfaces

## Boundary Conditions

- [ ] Empty input not handled (empty lists, empty strings, zero values)
- [ ] Maximum size inputs causing overflow or performance issues
- [ ] Single-element edge cases in algorithms expecting multiple items
- [ ] Whitespace/newline handling in string parsing

## Security-Relevant Logic Errors

- [ ] Authentication/authorization checks bypassed on error paths
- [ ] Input validation missing before use in queries, commands, or output
- [ ] Sensitive data logged or exposed in error messages
- [ ] Insecure defaults for security parameters
- [ ] Privilege escalation through parameter manipulation
