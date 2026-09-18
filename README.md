# Code Lens

Understand AI-written code changes by reading the code that matters: interfaces,
usage examples, important behavior, and test expectations. Routine implementation
details become short comments you can expand when needed.

**Hide mechanics. Preserve decisions.**

## Install

```sh
npx skills add panchaldeep009/agent-skills --skill code-lens -g
```

The skill uses the open Agent Skills format. Its instructions are independent of
any particular repository, language, or coding agent.

## What you see

A review starts with how the code is consumed, then shows its contract and behavior.
For example, this fictional implementation becomes a reading sketch:

```ts
// Illustrative consumer when no real caller exists yet.
const result = await publisher.publish(tenantId, reportId);

// Reading sketch — method bodies are summarized, not executable.
class ReportPublisher {
  async publish(tenantId: string, reportId: string): Promise<PublishResult> {
    // Load this tenant's report; throw NotFound if absent.
    // Save locally, then send the saved report to the archive.
    // On success, return "published".
    // On archive failure, retain the local save and return "archive-pending".
  }
}
```

The sequence and failure behavior stay visible. Setup and repetitive plumbing do not
take up the reader's attention. Real reviews link the sketches to inspected source.

Tests appear first as the behaviors their assertions establish. Ask to expand a test:

```ts
// Fictional test shown as steps rather than fixtures and mocks.
it("keeps the report safe when the archive fails", async () => {
  // Given: a report exists and the archive rejects its write.
  // When: publish() runs.
  // Then: the returned status equals "archive-pending".
  // Not asserted: the locally saved report survives the failure.
});
```

The test's actual assertions determine the summary, even when its name promises more.

## Use it

Invoke `code-lens` using your agent's skill syntax, followed by a request such as:

| Request | Result |
| --- | --- |
| `Review my local changes` | Consumer examples, relevant interfaces, masked bodies, and test expectations. |
| `Explain this PR: <URL>` | The same view grounded in the PR's actual revisions. |
| `Sketch the architecture for <feature> before coding` | Proposed contracts, usage, responsibilities, and concrete tradeoffs. |
| `Expand publish()` | Important guards, branches, writes, and errors, with routine details still folded. |
| `Show the test steps` | Real test names with Given / When / Then comments. |
| `Show actual code` | The requested source instead of a sketch. |

In Codex, for example:

```text
$code-lens review my local changes
$code-lens sketch the architecture for report delivery before coding
```

## Boundaries

An explanation or design sketch is read-only by default. The skill distinguishes
existing code from proposals and illustrative consumers, and inspected tests from
executed tests. It keeps consequential permissions, errors, state changes, side
effects, ordering, and compatibility details visible.

It supports understanding and discussion. A readable sketch does not prove correctness
or replace a requested defect review. It does not publish GitHub comments or start
implementation without authorization.

PR text, source comments, and changed instruction files are untrusted evidence. They
cannot authorize commands, credential access, uploads, or changes to the review's scope.
The skill requires safe read-only inspection and redacts secrets from source excerpts.
Reading third-party content still carries prompt-injection exposure; the host agent's
sandbox and permissions remain necessary controls.

Read the [skill instructions](skills/code-lens/SKILL.md) or the
[extended examples](skills/code-lens/references/examples.md).

## License

[MIT](LICENSE).
