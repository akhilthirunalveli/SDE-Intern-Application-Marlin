# Part 3: Prompt Preparation

## Selected Pull Request
Repository: https://github.com/beetbox/beets  
Pull Request: https://github.com/beetbox/beets/pull/3214


## 3.1.1 Repository Context

Beets is a command-line tool designed to help users manage and organize large music libraries. It is commonly used by people who maintain long-running collections and want to automate tasks such as metadata tagging, file organization, and consistency checks. The tool emphasizes correctness and transparency over hidden automation, which is especially important when operating on large batches of files.

A central component of Beets is its import workflow. During an import, the system scans audio files, attempts to match them against external metadata providers, applies tags, and updates its internal library database. This process involves multiple subsystems, including file system operations, metadata resolution, database updates, and plugin hooks. Each of these steps can fail independently.

The repository is structured so that core import logic is separated from optional behavior. Plugins can extend or modify how imports behave, but the importer remains responsible for coordinating the overall workflow. This makes error handling particularly important, as failures may originate from plugins, external services, or I/O operations.

Given that imports often run in batches and may take a long time to complete, failures must be handled predictably. The system should fail clearly and safely rather than continue in a partially successful state. Reliable importer error handling is therefore essential to maintaining library consistency and user trust.

Relevant code areas include:
- Importer implementation: https://github.com/beetbox/beets/tree/master/beets/importer
- Import-related tests under the `test/` directory


## 3.1.2 Pull Request Description

This pull request focuses on improving how failures are handled during the import process in Beets. Before this change, certain importer errors were logged but did not always stop execution. In practice, this meant that an import could partially complete even though a critical error had occurred, leaving users with unclear results and potentially inconsistent library state.

The PR modifies the importer workflow so that critical failures reliably terminate the import process. Instead of allowing exceptions to fall through generic handlers or be logged without halting execution, the importer now treats these errors as hard failures. Logging has also been refined to provide clearer information about where the failure occurred within the import process.

Previously, users encountering these issues might only notice that an import behaved unexpectedly, without receiving a clear explanation. With the updated behavior, failures are surfaced immediately and in a more predictable way, allowing users to correct issues or retry imports with confidence.

The changes are internal to the importer subsystem and do not alter public commands or APIs. The goal is to strengthen guarantees around failure handling while preserving existing successful import behavior.


## 3.1.3 Acceptance Criteria

1. When an importer encounters a critical error, the import process should terminate immediately
2. The system should log a clear and descriptive error message indicating the failure stage
3. No partial or inconsistent library state should be committed after an import failure
4. Successful import workflows should continue to behave exactly as before
5. Import-related tests should pass and explicitly cover failure scenarios  


## 3.1.4 Edge Cases

1. An import failure occurring after metadata has been partially resolved but before files are moved  
2. Exceptions raised by plugins during the import workflow  
3. Batch imports where one item fails while others are queued or in progress  


## 3.1.5 Initial Prompt

You are working within the Beets codebase, a Python-based command-line tool for managing music libraries. Your task is to improve error handling in the importer subsystem so that import failures are handled clearly and safely.

Focus on scenarios where errors occur during the import workflow. Currently, some failures may be logged without reliably stopping execution, which can result in partially completed imports. Your goal is to ensure that when a critical importer error occurs, the import process terminates immediately and provides a clear, actionable error message.

Identify the parts of the importer where exceptions may be raised and ensure these exceptions are propagated in a consistent and predictable way. Logging should include enough context to indicate which stage of the import failed, without relying on vague or generic messages.

Your implementation must satisfy the following acceptance criteria:
- Import operations terminate immediately on critical errors  
- Error messages clearly describe what failed and where  
- No partial library state remains after a failed import  
- Existing successful import behavior remains unchanged  
- Relevant unit tests pass, and new tests are added where appropriate

Be sure to consider edge cases such as failures after partial processing, exceptions raised by plugins, and batch imports where one item fails mid-run. The solution should integrate cleanly with the existing importer architecture and avoid introducing breaking changes.
All changes should be supported by unit tests that validate both failure handling and normal import behavior.


Be sure to consider edge cases such as failures after partial processing, exceptions raised by plugins, and batch imports where one item fails mid-run. The solution should integrate cleanly with the existing importer architecture and avoid introducing breaking changes.

All changes should be supported by unit tests that validate both failure handling and normal import behavior.
