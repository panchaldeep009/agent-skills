# Presentation examples

These examples are fictional. Real review output must use the inspected project's
symbols, behavior, assertions, and source links. All comment-only bodies below are
reading sketches, not executable implementations.

## Existing change, with no caller yet

The new service saves a report locally, then sends it to the archive; a failed archive
request leaves the local report saved.

Scope: a fictional change adding `ReportPublisher`. No caller was found in the inspected
application. This illustrative consumer demonstrates the contract, not an existing route:

```ts
// Illustrative consumer; proposed wiring, not found in source.
const result = await publisher.publish(tenantId, reportId);
if (result.status === "archive-pending") {
  // Tell the user the report is saved locally but has not reached the archive.
}
```

In a real review, a source link to the service would go immediately before this block:

```ts
// Reading sketch: real declarations, bodies summarized.
type PublishResult = {
  status: "published" | "archive-pending";
  reportId: string;
};

class ReportPublisher {
  constructor(private reports: ReportStore, private archive: ArchiveClient) {}

  async publish(tenantId: string, reportId: string): Promise<PublishResult> {
    // Load by tenantId + reportId; throw NotFound if absent.
    // Save the finalized report locally before calling the archive.
    // Send the saved report to the archive.
    // On archive success, return "published".
    // On archive failure, retain the local save and return "archive-pending".
  }
}
```

`archive-pending` is only a returned status in this example; there is no durable retry
job. The name must not be presented as evidence that background recovery exists.

Assertions inspected; tests not run:

- A report belonging to another tenant is treated as absent; neither destination is written.
- An archive failure returns `archive-pending`. No assertion checks the retained local save.

The gap belongs beside the claimed behavior. Do not describe the second test as proving
partial-failure recovery merely because its title says so.

## Expanding a test

When asked for the steps of that second test, preserve its real title but describe only
what the body establishes:

```ts
// Reading sketch of the existing test; setup and assertions replaced with comments.
it("keeps the report safe when the archive fails", async () => {
  // Given: a report exists in the tenant; the archive client rejects its write.
  // When: publish(tenantId, reportId) runs.
  // Then: the returned status equals "archive-pending".
  // Not asserted: the locally saved report survives the failure.
});
```

This exposes the difference between the test's name and its evidence without showing
factories, mock setup, dependency injection, or matcher mechanics.

## A small implementation fix with an unchanged interface

An unchanged signature can hide the entire point of a change. Keep the decisive predicate:

```ts
// Before: another tenant's report can match.
findReport({ reportId });

// After: the query itself scopes the lookup to the tenant.
findReport({ tenantId, reportId });
```

Link the before and after excerpts to their respective revisions in a real review.

## Choosing architecture before implementation

Proposed: the caller submits a report once. The choice is whether the request waits for
the external archive or durably schedules delivery.

```ts
// Option A — proposed: the request waits for the archive.
publish(tenantId: string, reportId: string): Promise<PublishResult>;
// Save locally → await archive → return published or archive-pending.
// Simpler operation, but a request failure needs an explicit recovery path.

// Option B — proposed: the request waits only for a durable local commit.
requestPublish(tenantId: string, reportId: string): Promise<{ operationId: string }>;
// One database transaction: save report + delivery intent → return operationId.
// Worker sends to archive → records delivery; retries use the same operation key.
// Requires a worker and a way for the caller to observe delivery status.
```

Recommend B if delivery must continue after the browser closes and the archive supports
deduplication. Without archive deduplication, a lost acknowledgement can still produce
duplicate delivery; local intent alone does not solve that.

Proposed tests: reject cross-tenant access; atomically save the report and delivery intent;
resume pending delivery after restart; handle a lost archive acknowledgement.

The reviewable choice is the completion contract: does success mean **delivered** or
**durably accepted for delivery**? In a design-only request, leave this sketch for the user
to decide before implementing it.
