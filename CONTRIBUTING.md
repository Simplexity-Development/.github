# Contributing to Simplexity Development Projects

Thanks for contributing, or considering it! Contributions go beyond writing code:

- Writing code / submitting a pull request
- Fixing documentation, wikis, or guides
- Writing or reporting issues
- Suggesting features and enhancements
- Using and testing our software
- Performing code review / quality assurance

Before contributing, please read the [Code of Conduct](https://github.com/Simplexity-Development/.github/blob/main/CODE_OF_CONDUCT.md). All contributions are expected to follow it.

> [!Note]
> This document was written by Peashooter101 and reformatted by Claude Code.

---

## Table of Contents

1. [AI and LLMs](#ai-and-llms)
2. [Branches](#branches)
3. [Commits](#commits)
4. [Pull Requests](#pull-requests)
5. [Code Style](#code-style)
   1. [Naming](#naming)
   2. [Nullability](#nullability)
   3. [Documentation](#documentation)
   4. [Structure and Organization](#structure-and-organization)
   5. [Error Handling](#error-handling)
   6. [SQL](#sql)
   7. [Minecraft-Specific](#minecraft-specific)
6. [Code Review](#code-review)
7. [Testing](#testing)

---

## AI and LLMs

Using AI or LLMs to assist with development or documentation is fine. The expectation is the same as grabbing code from StackOverflow: **you must understand what the code is doing**. Submitting code you cannot explain leads to poorly maintained or high-risk contributions and will be reflected in review.

---

## Branches

- Never name a branch `main` or `master` (reserved for the default branch).
- Use the following prefixes:
  - `feature/` — new features (e.g. `feature/tablist-support`)
  - `fix/` — bug fixes (e.g. `fix/null-nickname-crash`)
  - `ci/` — CI/workflow changes (e.g. `ci/update-workflow`)
  - `doc/` — documentation only (e.g. `doc/update-readme`)
- Each branch should address one issue or one feature where possible.
- Rebase onto the target branch regularly to avoid merge conflicts.

---

## Commits

Each commit should:

- Leave the project in a **compilable state**.
- Have a **meaningful commit message** that describes *what* changed and ideally *why*.
- Include a short description body if the change is non-trivial.
- Be scoped to a single logical change — avoid bundling unrelated modifications.

---

## Pull Requests

Each PR should:

- Accomplish one issue or one feature where possible.
- **Rebase** onto the target branch before merge — do not merge-commit from target into your branch.
- Have a **minimal diff** — avoid bulk whitespace or formatting changes unless the PR's explicit purpose is reformatting.
- Pass all CI/CD pipelines, or have an open issue explaining a known failure.
- Squash commits into a single merge commit with a meaningful message when appropriate. The default `Merge branch X into Y` message is not acceptable.

### Handling Issues Found During Development

- If it is **small and in a file you are already touching**, fix it as part of your current work.
- If it requires significant investigation or a larger code change, **open a GitHub issue** or a dedicated fix branch rather than folding it into unrelated work.

---

## Code Style

These guidelines apply to all Java code in Simplexity projects. The goal is consistency and readability, not pedantry — use judgment when edge cases arise.

> [!Note]
> This Code Style section was written by Claude Code based on the SimpleNicks codebase.
> 
> This section has been reviewed and edited by Peashooter101 before publishing.

### Naming

| Element | Convention | Example |
|---|---|---|
| Classes and interfaces | `PascalCase` | `NicknameProcessor`, `SubCommand` |
| Methods | `camelCase`, verb-first | `getActiveNickname()`, `refreshDisplayName()` |
| Variables | `camelCase` | `normalizedNick`, `playerUuid` |
| Constants (`static final`) | `SCREAMING_SNAKE_CASE` | `SCHEMA_VERSION`, `MILLI_PER_DAY` |
| Packages | `lowercase`, dot-separated | `simplexity.simplenicks.saving` |
| Enum values | `SCREAMING_SNAKE_CASE` | `NICK_ADMIN_SET`, `ERROR_LENGTH` |

**Additional rules:**

- Names should be descriptive. Avoid single-letter variables outside of short loop indices.
- Boolean methods should read as yes/no questions: `isDebugMode()`, `playerSaveExists()`, `canExecute()`.
- Avoid abbreviations unless they are universally understood in context (e.g. `uuid`, `sql`).
- Do not prefix interfaces with `I` or abstract classes with `Abstract` unless there is a genuine ambiguity to resolve.

### Nullability

- Annotate all method parameters and return types with `@NotNull` or `@Nullable` from `org.jetbrains.annotations`.
- A method that returns `@Nullable` signals to the caller that a null check is required. Methods that should never return null must be annotated `@NotNull` and must uphold that contract.
- Prefer returning an empty collection over returning `null` for collection return types.
- Prefer returning a `boolean` success/failure over returning `null` as a sentinel for operation results.

```java
// Prefer this
@NotNull
public List<Nickname> getSavedNicknames(@NotNull UUID uuid) {
    ...
    return new ArrayList<>(); // never null
}

// Over this
@Nullable
public List<Nickname> getSavedNicknames(@NotNull UUID uuid) {
    ...
    return null; // forces callers to null-check a list
}
```

### Documentation

- Write Javadoc on all `public` methods in non-trivial classes (handlers, caches, utilities, interfaces).
- Include `@param`, `@return`, and `@throws` tags where relevant.
- Use `{@code null}`, `{@link ClassName}`, and `{@see}` tags to link related types.
- Inline comments should explain *why*, not *what* — the code itself should be clear enough that restating what it does is unnecessary.

```java
// Bad: restates the code
// Set the nickname
player.setNickname(nick);

// Good: explains the why
// Display name must be set on the main thread — schedule back from async context
Bukkit.getScheduler().runTask(plugin, () -> player.displayName(parsed));
```

- Do not leave `TODO` or `FIXME` comments in merged code without an associated open issue.

### Structure and Organization

- **Singletons** use lazy initialization via a `getInstance()` static method. Do not use static state outside of singletons.

```java
public static MyHandler getInstance() {
    if (instance == null) instance = new MyHandler();
    return instance;
}
```

- **Enums** are preferred over constants classes for related sets of values — especially permissions, message keys, and tag definitions.
- **Interfaces** are preferred over abstract classes for defining shared behavior across unrelated implementations (e.g. the `SubCommand` interface).
- Keep classes focused. If a class is handling both business logic and persistence, split it.
- Package by feature/layer, not by type. Related classes should live close together.
- Utility methods that have no meaningful state belong in a `static`-only utility class or a package-private helper, not scattered across unrelated classes.

### Error Handling

- Do not silently swallow exceptions. At minimum, log a warning with enough context to diagnose the problem.
- Log the exception object itself alongside the message — do not only log `e.getMessage()`, as that omits the stack trace.
- Use specific exception types where possible; avoid catching `Exception` broadly unless you are at a boundary where all errors must be handled the same way.
- For operations with recoverable failure (e.g. DB writes), return `boolean` from the method rather than throwing, and log the failure internally.
- Use `@SuppressWarnings` sparingly and only when the warning is a known false positive. Always leave a brief comment explaining why it is suppressed if it is not obvious.

```java
} catch (SQLException e) {
    logger.warn("Failed to save nickname '{}' for UUID '{}'", nickname, uuid, e);
    return false;
}
```

### SQL

- Always use `PreparedStatement`. Never concatenate user-supplied or runtime values into a query string.
- SQLite and MySQL require different syntax in several places (upserts, ignore-on-conflict). Always branch on `ConfigHandler.getInstance().isMySql()` and provide both variants.
- Manage schema changes with a `schema_version` table. Migrations run forward only; never modify a migration that has already shipped.
- Use try-with-resources for all `Connection`, `PreparedStatement`, and `ResultSet` usage to guarantee cleanup.
- Batch inserts should be committed in chunks (e.g. every 500 rows) to avoid holding a transaction open too long.

```java
// Correct: branch on backend
String upsert = isMySql
    ? "INSERT INTO ... ON DUPLICATE KEY UPDATE ..."
    : "INSERT INTO ... ON CONFLICT(...) DO UPDATE SET ...";
```

### Minecraft-Specific

- **Threading:** All database and file I/O must run **asynchronously** (`runTaskAsynchronously`). Any follow-up that touches the Bukkit API — display names, inventories, players, worlds — must be scheduled back to the **main thread** (`runTask`).
- **Text formatting:** Use MiniMessage (`net.kyori.adventure.text.minimessage`) for all player-facing text. Do not use legacy `§` codes or `ChatColor` unless necessary in cases like the Tab List.
- **Commands:** Use the Paper Brigadier lifecycle API (`LifecycleEvents.COMMANDS`). Do not use `onCommand` / `plugin.yml` command handlers for new code.
- **Configuration:** Cache all config values in a handler class. Never call `plugin.getConfig()` in feature code.
- **Messages:** All player-facing strings must live in `locale.yml` and be accessed through a `LocaleMessage` enum or equivalent. No hardcoded strings.
- **Permissions:** Define permissions in an enum and register them programmatically in `onEnable`. Treat the enum as the source of truth; keep `plugin.yml` in sync.
- **Third-party hooks:** Check availability before registering (`Bukkit.getPluginManager().isPluginEnabled(...)`). All third-party integrations are `softdepend`, never `depend`.

---

## Code Review

All significant code changes require **at least one round of code review** before merge.

**As a reviewer:**
- Enforce naming conventions, coding style, and functional correctness.
- Raise anything that feels off — gut feeling is a valid review comment.
- Ask questions freely: *Why was this implemented this way? How does this work?*
- Assume good intent; feedback is about improving the software.

**As an author:**
- Receive feedback constructively.
- Every significant change since the previous review must be re-reviewed. Reviewers do not need to be the same people each round.

---

## Testing

> [!Important]
> ### Untested code will not be merged.

- You are responsible for testing your own changes. You will be asked to demonstrate it.
- Document your testing process in the PR — steps taken, inputs used, edge cases covered.
- Perform **regression testing**: test surrounding areas you touched, not just the core change.
- For Minecraft plugins specifically, there is no standard automated test harness — testing is done against a live Paper server.
