# Development Operating Agreement

This document describes our working agreement on all projects. It is authoritative and must only be edited by humans.

## Do this every session

- If starting a new project
    - Create a `README.md` if it hasn't been created already. If you have any context as to the purpose and goals of the project, you may add them now. If not, a simple title derived from the folder name will do for now. As we progress through the project, you are free to update this file autonomously, always reporting any changes. Always include a setup section which includes such things as shell scripts to be run, environment variables to be set, dependencies required, and anything else necessary to get the user up and running with a minimum of manual intervention. `README.md` is end-user facing and should not influence operational choices. It is simply a deliverable that must be kept up to date like any other.
    - Create a `project.md` in the `project` folder. This is where project level plans and checklists go, as well as the overall architecture, stack decisions, etc. It is our primary operational document. At this point, its contents will likely mirror the `README.md`.  Further updates to `project.md` **require** human authorization and it should otherwise be considered **write-locked**.
    - Create a `context.md` in a new `project` folder. The format of your first entry is defined below. The body can be as simple as 'Started [projectname] project.' This is our ongoing work log. Entries *must* never be edited after they have been entered. Consider it an auditable artifact.
    - Create context management scripts in `project/scripts/` if they don't exist (add-context, rotate-context). These scripts handle the mechanical parts of context entry creation.

- If resuming an existing project
    - Read `project/project.md`. This will only be updated after discussion and is considered *write-locked* unless given explicit human authorization. This is our primary operational document.
    - Read `project/context.md`. You do not need to read the whole thing. Just the last entry or two, enough to know where we left off in the previous session. Cap at 12K of text at the most. If you do not have enough context from that, ask for help. If no context exists because we are using these standards for the first time on an existing project, create the file if necessary, review the existing codebase, and add an entry that summarizes the current state as well as you can. This is our ongoing work log. Entries *must* never be edited after they have been entered. Consider it an auditable artifact.

## Context Management

Context entries should be added using the provided `project/scripts/add-context` helper:

- **Required parameters:**
  - `--agent <name>` - Agent name and version (e.g., "OpenCode", "Claude Desktop", "Claude.ai")
  - `--model <version>` - Model name and version (e.g., "claude-sonnet-4-5", "gpt-4")

- **Body input methods (in priority order):**
  - Direct argument: `add-context --agent "..." --model "..." "Entry text"`
  - From file: `add-context --agent "..." --model "..." --file body.txt`
  - From stdin: `echo "Entry" | add-context --agent "..." --model "..."`

- **Auto-generated fields:**
  - Date (ISO 8601 with timezone offset)
  - Hash (Base64-encoded SHA-256 of body text)
  - Git commit hash (if in repository)

- **Entry guidelines:**
  - The `EOF` marker at the end is a visual delimiter, not part of the body
  - The script handles formatting automatically
  - Never edit `context.md` directly - always append via script
  - Think: "What would I need to know if session crashed now?"

- **Context rotation:**
  - Use `project/scripts/rotate-context` when context.md exceeds ~100K
  - Archives old entries to `context-archive-YYYY-MM-DD.md`
  - Keeps recent entries in active `context.md`

## Phases of a feature

- Planning
    - Ask questions
    - Suggest options
    - This is your time to voice your opinions
    - Decisions, plans, and task checklists should go in `project.md`.
    - Tool, dependency, and stack decisions will be decided here and **must** be followed until discussion opens up again in the next phase.
    - You *should* press me for a decision if I have forgotten something or something is ambiguous. After all, you will be required to live by our choices until the next planning phase. This includes, but is not limited to:
        - Stack and language (i.e. Python/Flask, Node/Express/NextJS, etc.)
        - Explicit coding standards and linters (i.e. PEP, Google Go style guide, ESLint)
        - Testing framework (i.e. go built-in, Jest)
    - At some point I will decide that planning is complete for this step. We will come back to it later many times. At no time should you autonomously update `project.md`. Updates to `project.md` **require** human authorization and should otherwise be considered **write-locked**.
    - It is inevitable that, occassionally, we may get part way through the later steps and decide that we should revisit a decision made here. You may, if absolutely necessary, suggest it. However, **only a human can authorize changes to the plan**.
    - When a decision is superceded or retired, the original decision should remain in place in `project.md` but be marked as superceded with a reference to the replacement decision.
    - In the rare but eventually inevitable event that external factors force re-planning (i.e. dependency deprecation, high severity CVE), we both have the responsibility to call it out as soon as we become aware of it so that we can make a decision on our next steps. 

- Branching
    - All new work after a push to origin must begin on a fresh branch. If the new work refers to an itemized step in `project.md`, refer to that step in the branch name.
    - For now, we will attempt to work one feature at a time, obviating the need for rebasing or anything like that. If that changes, we will deal with it on a per-project implementation detail. If it becomes the norm, we may wish to codify our approach in this document.
 
- Coding style
    - Code in a way that is idiomatic for the stack/language/framework. Follow the style guides that were chosen in the planning stage. In particular, style precedence is as follows:

        - Existing project conventions
        - Chosen style guides
        - Community best practices

        If you are unsure, ask.

    - Suggest refactors where appropriate. For example, if a file gets too long or covers too many concerns.
    - Wherever appropriate, use pure functions. They are easier to test.

- Error handling
    - Prefer explicit, typed errors over silent failure. Avoid catching errors unless there is a clear recovery path. Log with enough context to reconstruct failure after the fact.

- Dependencies
    - If a new dependency becomes necessary that has not previously been **human-approved**, follow these rules:
        - New dependencies must be justified in commit messages
        - Prefer standard library over third-party
        - Avoid adding dependencies solely for syntactic convenience
        - For the most part, pinning to a major semver version will be appropriate. Where it isn't, we will deal with it on a case-by-case basis.

- Testing
    - Use the testing framework decided upon during the planning phase.
    - All tests must pass before each commit.
    - After each interaction which requires changes to code, consider whether there are appropriate tests that could be added or need to be changed. If so, do so and report those test changes.
    - There are no hard rules on coverage. We should, however, always keep testability top of mind as we build. Things like pure functions and dependency injection, even at the system level, are our friends.

- Linting
    - Use any tool(s) which we decided upon during the planning phase. Any errors or warnings must not be ignored. Strive to maintain a clean codebase at all times.

- Performance
    - Optimize for clarity first. Address performance only when there is evidence (measurements, scale assumptions, or explicit requirements) that it matters.

The coding, testing, and linting phases may be commingled as you see fit. In some cases you may wish to, for example, adopt a test-first pattern. You are free to make this decision autonomously. Where appropriate, you are encouraged to develop script(s) for human use so that I may operate without you. Where appropriate these scripts should be surfaced in documentation and places like `package.json` or a `Makefile`.

- Rollbacks
    - It is inevitable that these will happen once in a while. In most cases we will catch them quickly and a simple `git revert` will handle the issue. More complex scenarios will be dealt with on a case-by-case basis and we should reflect upon them together to determine if the error could have been avoided.

- Documentation
    - We should strive to keep user-facing documentation like `README.md`, API docs, etc. continuously up to date with the truth in the code. They are everyone's responsibility. As mentioned before, these are artifacts just like code and do not influence our operational choices.

## After **every** interaction

Complete this checklist before marking the interaction done:

1. **Security scan** (if any files changed)
   - Run security scan on changed files
   - Replace secrets with placeholders
   - Update secrets.env and secrets.env.example
   - Ensure secrets.env is in .gitignore
   - Report any substitutions
   - Security scan script encouraged with clear exit codes (0=pass, 1=fail, 2=error)

2. **Add context entry** (ALWAYS)
   - Use `project/scripts/add-context` with agent/model parameters
   - Summarize: what was done, what changed, what's next
   - Reference relevant steps in `project.md` where appropriate
   - Think: "What would I need to know if session crashed now?"

3. **Report progress** (ALWAYS)
   - Include diffs of changed files
   - For large diffs (>200 lines), provide summary + representative excerpts
   - For generated files, note their existence without full content
   - List any new dependencies or configuration changes

4. **Run tests** (if code changed)
   - All tests must pass
   - Report test additions/modifications
   - Note any skipped tests with reasons

5. **Commit** (recommended after each interaction)
   - Stage changed files
   - Write clear commit message (see Commit Messages section)
   - Do NOT push (only humans can push)

## Commit Messages

Use clear, descriptive commit messages that explain WHAT changed and WHY.

**Format:**
```
<component>: <summary>

[Optional detailed explanation]
[Reference to project.md steps if applicable]
```

**Examples:**
```
Phase 1: Foundation - Project structure, config, Gmail auth

Classifier: Add retry logic with exponential backoff
- 3 attempts with 2-10s exponential backoff
- Handles transient API failures
- Per project.md Phase 4 requirements

Tests: Add corpus-based classification suite

Fixes #42 - Handle edge case in email parsing
```

**Guidelines:**
- Keep summary line under 72 characters where practical
- Use imperative mood ("Add" not "Added")
- Include context in body for non-obvious changes
- Reference project.md steps or issue numbers when relevant
- What does not belong in commits:
    - Generated files
    - Lockfiles
    - Editor ephemera (.vscode, .un~, etc.)
    - Build artifacts
    - Binaries or vendored code (but make a note of them in the `README.md`)

## Feature build/test cycle complete

- You will have been reporting your progress after each interaction. When I declare the feature is complete, then and only then, can we move on to the next step which is...

## `Send 'er`

- The phrase `send 'er` is a command which has been chosen because it is unlikely to come up in normal discussion. It is a specific command comprised of the following steps:

1. Run a security scan on the entire project. If any substitutions were required, report to me.
2. If any file changes have occurred since the last test run, run all tests. All tests must pass. If not, report to me.
3. If any code changes have occurred since the last linter run, run the linter. It must not report any errors or warnings. If it does, report to me.
4. If a build is required, run it. If any errors or warnings are reported, report them to me, along with any insight or suggestions to fix them.
5. Show summary and confirm push:
   - Display: branch name, files changed, commits to push
   - Prompt: "Ready to push to origin? (y/n)"
   - Only proceed if confirmed
6. Push to origin.
7. Open a pull request for the change.

## Troubleshooting

**If a context entry was missed:**
- Create the entry immediately using `add-context`
- Note in the body: "Retroactive entry for interaction at [approximate time]"
- Include what was done and current state

**If a DOA step was skipped:**
- Acknowledge it in the next context entry
- Complete the missed step if still relevant
- If a pattern of skipping emerges, discuss process improvement

**If in doubt:**
- Ask the human before proceeding
- Better to clarify than to guess wrong
- Document the clarification for future reference

## Quick Reference

**Starting a new feature:**
1. Read `project/project.md` for decisions
2. Read last 1-2 entries in `project/context.md`
3. Create feature branch: `git checkout -b feature-name`
4. Implement following DOA phases
5. Add context after each interaction

**Completing an interaction:**
1. Run security scan if files changed
2. Add context: `add-context --agent "..." --model "..." "Summary"`
3. Report progress with diffs
4. Commit: `git add . && git commit -m "Component: Summary"`

**Finishing a feature:**
1. Human declares feature complete
2. Human says "send 'er"
3. Run full check sequence
4. Confirm and push
5. Open PR

## Overarching goals

If you haven't figured out yet, these rules are designed to help us work together most efficiently, with the fewest surprises, and the smallest blast radius when things inevitably go wrong. Just remember, if you think you know something I don't, tell me. If you think I know something you don't or you have a question **stop** and ask.
