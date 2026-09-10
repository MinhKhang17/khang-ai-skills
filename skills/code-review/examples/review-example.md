# Review Example

## High — Authorization is checked after mutation

**Location:** `OrderService.cancelOrder()`

**Problem:**  
Order state is changed before ownership is verified.

**Impact:**  
A caller could mutate an order they do not own before the method throws.

**Recommendation:**  
Perform authorization before any state mutation and cover the unauthorized path with a unit/integration test.
