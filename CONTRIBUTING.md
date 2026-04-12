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
   8. [Code Mechanics](#code-mechanics)
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

### Code Mechanics

This section covers low-level style decisions: how to declare variables, write conditions, use lambdas, and avoid patterns that obscure intent.

#### Variables

- Declare variables as close to their first use as possible.
- Extract values from method chains into named local variables before using them in a condition, or before using them more than once. This applies especially to config lookups, permission checks, and UUID resolution.
- Boolean flags that guard a block of logic should be extracted and named before the `if` — not inlined into the condition.

```java
// Prefer this: intent is visible at the call site
boolean bypassLength = sender.hasPermission(NickPermission.NICK_BYPASS_LENGTH.getPermission());
if (!bypassLength && normalizedNick.length() > ConfigHandler.getInstance().getMaxLength()) { ... }

// Over this: requires the reader to parse a permission string mid-condition
if (!sender.hasPermission(NickPermission.NICK_BYPASS_LENGTH.getPermission()) && ...) { ... }
```

- Use `final` on local variables that are never reassigned after declaration. It signals intent and prevents accidental re-use.

#### Magic Numbers and Strings

- Any literal number or string that carries semantic meaning must be a named constant. Bare literals in logic are not acceptable.
- Constants local to a class go in `private static final` fields at the top of the class. Constants meaningful across classes belong in an enum or dedicated utility.
- Numeric sentinel values (e.g. `-1` meaning "no limit") must be documented at the constant definition and at every method `@param` that accepts them.
- SQL query strings should be extracted to a named local variable at the top of the method. Never repeat or concatenate query strings inline.

```java
// Good: named constant, meaning is clear
private static final int SCHEMA_VERSION = 1;
private static final int COMMIT_THRESHOLD = 500;

if (batchCount >= COMMIT_THRESHOLD) { ... }

// Bad: reader must guess what 500 represents
if (batchCount >= 500) { ... }
```

#### Conditionals and Braces

- **Always use braces** on `if`, `else`, and loop bodies — no exceptions, including single-statement guard clauses.
- Use guard clauses (early returns or early throws) to reduce nesting. Do not wrap the main body of a method in an `if` when a guard at the top accomplishes the same thing.
- When a nested `if` and its outer `if` can be merged with `&&` without losing clarity, merge them. A double-nested `if` whose outer block contains only the inner `if` is always a candidate for flattening.

```java
// Prefer this: flat, readable
if (!bypassUsername && ConfigHandler.getInstance().isUsernameProtection() && isProtectedUsername(normalizedNick)) {
    throw Exceptions.ERROR_NICKNAME_IS_SOMEONES_USERNAME.create(normalizedNick);
}

// Over this: unnecessary nesting
if (!bypassUsername) {
    if (ConfigHandler.getInstance().isUsernameProtection() && isProtectedUsername(normalizedNick))
        throw Exceptions.ERROR_NICKNAME_IS_SOMEONES_USERNAME.create(normalizedNick);
}
```

- When multiple checks at the top of a method all lead to an early return or throw, invert each condition and return immediately rather than nesting them. Each guard clause should be its own `if` block — do not chain successful conditions into a nested success path.

```java
// Prefer this: each failing condition exits immediately, main logic stays flat
public boolean deleteSavedNickname(@NotNull UUID uuid, @NotNull String nickname) {
    boolean sqlDeleted = SqlHandler.getInstance().deleteNickname(uuid, nickname);
    if (!sqlDeleted) {
        return false;
    }
    if (!savedNicknames.containsKey(uuid)) {
        return false;
    }
    // main logic at the top level
    savedNicknames.get(uuid).removeIf(name -> name.getNickname().equals(nickname));
    return true;
}

// Over this: nesting successful conditions pushes the main logic further right
public boolean deleteSavedNickname(@NotNull UUID uuid, @NotNull String nickname) {
    boolean sqlDeleted = SqlHandler.getInstance().deleteNickname(uuid, nickname);
    if (sqlDeleted) {
        if (savedNicknames.containsKey(uuid)) {
            // main logic buried under two levels of nesting
            savedNicknames.get(uuid).removeIf(name -> name.getNickname().equals(nickname));
            return true;
        }
    }
    return false;
}
```

- Use ternary expressions for simple conditional assignments — one condition, one value on each branch, no nesting.

```java
// Acceptable: simple, symmetric
String upsert = isMySql
    ? "INSERT INTO ... ON DUPLICATE KEY UPDATE ..."
    : "INSERT INTO ... ON CONFLICT(...) DO UPDATE SET ...";
```

#### instanceof Pattern Matching

- Use Java's `instanceof` pattern matching everywhere a type check is immediately followed by a cast. Never cast manually after an `instanceof` check.
- For early-return guards, prefer the negated form to keep the happy path un-nested.

```java
// Prefer this: guard at the top, happy path stays flat
if (!(css.getSender() instanceof Player player)) return false;

// Over this: happy path is buried inside the block
if (css.getSender() instanceof Player player) {
    // ... rest of method indented one level unnecessarily
}

// Never do this: manual cast after instanceof check
if (sender instanceof Player) {
    UUID uuid = ((Player) sender).getUniqueId(); // redundant cast
}
```

#### Lambdas and Method References

- Prefer method references over lambdas when the body is a single method call with no transformation: `this::canExecute`, `argument::suggestOwnNicknames`.
- Use a block lambda (braces + explicit body) when the body has more than one statement or contains its own control flow.
- Do not nest lambdas more than one level deep. Extract inner logic to a named private method with a descriptive name.
- Never capture mutable state inside a lambda passed to an async scheduler. All captured variables must be effectively final.

```java
// Prefer method references when the body is a single delegating call
parent.then(Commands.literal("set")
        .requires(this::canExecute)
        .then(Commands.argument("nickname", argument)
                .suggests(argument::suggestOwnNicknames)
                .executes(this::execute)));

// Over an explicit lambda that just forwards to the same method
parent.then(Commands.literal("set")
        .requires(css -> canExecute(css))
        .then(Commands.argument("nickname", argument)
                .suggests((ctx, builder) -> argument.suggestOwnNicknames(ctx, builder))
                .executes(ctx -> execute(ctx))));
```

```java
// Single method call with no transformation → expression lambda (or method reference)
Bukkit.getScheduler().runTask(plugin, () -> NickUtils.refreshDisplayName(player.getUniqueId()));

// Multiple statements or control flow → block lambda
Bukkit.getScheduler().runTaskAsynchronously(plugin, () -> {
    boolean success = NicknameProcessor.getInstance().resetNickname(player);
    if (success) {
        Bukkit.getScheduler().runTask(plugin, () -> NickUtils.refreshDisplayName(player.getUniqueId()));
    } else {
        sendFeedback(player, LocaleMessage.ERROR_RESET_FAILURE, null);
    }
});
```

#### Collections and Map Access

- When iterating a `Map` and you need both the key and the value, iterate `entrySet()`. Do not iterate `keySet()` and then call `get()` inside the loop — that is two lookups where one suffices.
- Prefer `Map.getOrDefault()` over a `containsKey()` check followed by a separate `get()`.
- For `List` removal by predicate, use `removeIf()` rather than iterating manually and calling `remove()`.

```java
// Prefer this: single lookup
for (Map.Entry<UUID, Nickname> entry : activeNicknames.entrySet()) {
    if (entry.getValue().getNormalizedNickname().equalsIgnoreCase(normalizedNick)) { ... }
}

// Over this: double lookup
for (UUID playerUuid : activeNicknames.keySet()) {
    if (activeNicknames.get(playerUuid).getNormalizedNickname().equalsIgnoreCase(normalizedNick)) { ... }
}
```

#### Method Length and Scope

- If a method body spans more than roughly 30 lines, look for a natural decomposition into private helper methods with descriptive names. A long method is usually doing more than one thing.
- Extract logic that will be called from inside a lambda into a named private method rather than writing it inline. This keeps lambda bodies short and gives the extracted logic a searchable name.
- Scope `@SuppressWarnings` to the smallest element possible — prefer annotating a single method over an entire class. Always leave a brief comment explaining why the suppression is justified if it is not immediately obvious from context.

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
