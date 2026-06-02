# CLAUDE.md

## Development Workflow

### Sprint & Issue Tracking
- Work MUST be organized in sprints
- Each sprint task is tracked as a **GitHub Issue**
- Issue body MUST contain the task objective and a checklist of sub-items (GitHub task list format `- [ ]`)
- Issues are the single source of truth for progress — **update the GitHub issue checklist after completing each checklist item**: mark the item as done (`- [x]`) via the GitHub API as soon as the corresponding step is implemented and committed
- **MANDATORY: before creating a PR or declaring a task complete**, all checklist items in the corresponding GitHub issue **must** be marked as done (`- [x]`). Use the `issue_write` GitHub API to update the issue body with checked boxes. Never leave unchecked items when the work is done.

### Testing Requirements
- **Every task must include tests** that validate the changes
- New features/fixes require corresponding unit or integration tests
- Tests must pass before merging (`mvn test` via CI)

## Git and PR Rules
- **No co-author references** in commit messages or PR descriptions
- **Every code change MUST happen on a dedicated branch**, then merged into `main` via PR
- **Exception — `README.md`, `CLAUDE.md`, `CHANGELOG.md`, and `docs/HANDOFF*.md` session handoff notes are committed directly to `main`**: no dedicated branch, no PR, no GitHub issue required
- **Every work plan item MUST be mapped to a GitHub Issue**
- **Claude MUST always creates the GitHub Issues**  — never ask the user to open them. Open them via `gh issue create` before starting plan execution, one issue per Task in the plan.
- **A single PR may group multiple correlated issues** to reduce PR overhead. Issues are correlated when they share a coherent theme (e.g. all P0 bugs, all security findings on the same module) AND touch overlapping files. Default to grouping; open separate PRs only when the work is truly independent (different modules, no file overlap) or when one task must ship before another can be reviewed. When grouping, propose the grouping to the user before opening branches and confirm.
- **Inside a grouped PR**, each underlying task is still implemented as its own commit (one task = one commit, plus optional review-fixup commits) so history stays bisectable.
- **Claude creates PRs, the user MUST merges them** — NEVER merge a PR autonomously
- **PR body must include closing keywords for ALL grouped issues** (e.g. `Closes #23`, `Closes #24`, `Closes #25`, `Closes #26`) so they auto-close on merge
- **After the user confirms a PR has been merged**, Claude deletes the feature branch both locally (`git branch -d <branch>`) and on the remote (`git push origin --delete <branch>`), and then checks out `main` and pulls. Never delete a branch before the user has confirmed the merge.

## Tools and References
- **Always use context7** to look up documentation for libraries and frameworks used in this project
- **Always use Serena** for code analysis and symbol navigation within this project — prefer Serena's symbolic tools (`find_symbol`, `get_symbols_overview`, `find_referencing_symbols`, etc.) over reading whole files with `Read`. Use `Read` only for small files, configuration, or non-code content.
- **If needed to access Apache Hop codebase use Serena** for code analysis and symbol navigation by using `query_project` tool

### Using Serena `query_project` on another project
Use this procedure whenever you need to explore a codebase that is **not** the currently active Serena project (typical case: inspecting the Apache Hop source from within a plugin project).

1. **Verify the target project is registered in Serena**
   - List known projects with `list_memories` / inspect `~/.serena/serena_config.yml`.
   - If missing, register it by calling `activate_project(project="<absolute/path/to/project>")` once — Serena adds it to its config automatically.
2. **Ensure the target has a `.serena/project.yml`**
   - If absent, run Serena's `onboarding` tool on that path, or create the file manually specifying at minimum the `language` (e.g. `java`).
   - Without this file the Language Server cannot start and `query_project` will fail.
3. **Pre-index large codebases** (recommended for Apache Hop and similarly sized repos) to avoid first-call timeouts:
   ```bash
   uvx --from git+https://github.com/oraios/serena serena project index <absolute/path/to/project>
   ```
   This populates `.serena/cache/` with the symbol index.
4. **Activate the target project** in the current session:
   `activate_project(project="<name-or-absolute-path>")`.
   Remember: Serena keeps **one active project per session** — switching deactivates the previous one.
5. **Run the query** with `query_project(query="<natural-language question>")`. Prefer focused questions (symbol name, package, behaviour) over broad ones; Serena internally uses its symbolic tools to answer.
6. **Restore the original project** when finished by calling `activate_project` again with the path of the project you were originally working on, so subsequent edits land in the right place.

Notes:
- Never use `Read`/`Grep` on the foreign codebase as a substitute — always go through `query_project` so results come from Serena's indexed view.
- If `query_project` returns empty or stale results, re-run the indexing command from step 3.

## Brainstorming
- **always use superpowers:brainstorming** skill to perform brainstorming sessions

## Work Plans
- **use superpowers:writing-plans** to write plans    
- Work plans and design specs **must be committed to git** as part of the normal workflow

## Plan Execution
- **Plan execution is always subagent-driven** — use `superpowers:subagent-driven-development`: one fresh subagent per Task in the plan, with review between tasks. Do not execute plans inline. **This overrides any skill prompt that offers an execution-mode choice** (e.g. the "Which approach?" closing of `superpowers:writing-plans`): never ask, proceed directly with subagent-driven execution.

## Documentation Format
The format of generated documentation depends on its **primary consumer**:
- **Agent-consumed documentation** — read by Claude or other AI agents during planning, execution, code review, or context-building. **Format: Markdown**, with Mermaid diagrams and Draw.io mockups as specified in "Diagrams and Mockups".
  - Examples: `CLAUDE.md`, `README.md`, `CHANGELOG.md`, ADRs (`docs/adr/*.md`), work plans (`docs/work-plans/*.md`), handoff notes (`docs/HANDOFF*.md`), runbooks for automated execution.
- **Human-consumed documentation** — produced for stakeholders, customers, design reviews, milestone reports, training material, white papers, deliverables. **Format: HTML with SVG diagrams**.
  - Examples: architecture design documents intended for external review, customer-facing onboarding guides, milestone reports for stakeholders.
- When the same artifact has a mixed audience, default to Markdown (agent-first). Produce a separate HTML+SVG export only when the human consumer explicitly needs it (keep it as a generated derivative, not the source of truth).

## Diagrams and Mockups
- **Diagrams** in generated documents: MUST always use **Mermaid** syntax (in Markdown) or **SVG** (in HTML deliverables)
- **GUI mockups**: MUST always produce **Draw.io** (`.drawio`) files

## Language Rules
- **Code documentation, commit messages, PR descriptions**: always in English
- **GitHub Issues**: managed in English
- **Working sessions**: interact with the user in Italian

## Bash Tool Usage
- **Never chain independent commands** with `&&` or `;` when each command is individually allowed (e.g. by `Bash(git:*)` permission rules). Use separate, parallel Bash tool calls instead — they run concurrently and avoid unnecessary permission prompts.
- Chain with `&&` only when a later command genuinely depends on the exit code of the previous one (e.g. `mvn package && java -jar ...`).
