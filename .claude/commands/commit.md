# Commit Command

Goal: Execute a safe, strictly attributed git commit for **George Khananaev**.

## 1. Identity & Attribution
- **Author**: Must be **George Khananaev**.
- **Check**: Verify `git config user.name` matches if running in a new environment.
- **Credit**: No co-authors or AI attribution in the commit message unless explicitly requested.

## 2. Safety Protocols
**Forbidden Files (Never Commit):**
- Secrets: `.env`, `.env.*`, `*.pem`, `*.key`, `id_rsa`, `secrets.*`, `credentials.*`
- Logs/Debug: `*.log`, `npm-debug.log`, `yarn-error.log`
- System/Build: `.DS_Store`, `node_modules/`, `dist/`, `build/`, `__pycache__/`, `.venv/`

**Critical Flags:**
- ⛔ `no --no-verify`: Hooks must pass.
- ⛔ `no --no-gpg-sign`: Commits should be signed if GPG is configured.
- ⛔ `no --amend`: Create new commits for history preservation unless fixing the immediate previous commit.

## 3. Version Control Strategy (Interactive)
**Prompt**: "Do you want to update the project version?"

**If YES:**
1. **Detect Project Type**:
   - Node.js: `package.json`
   - Python: `pyproject.toml`, `setup.py`
   - Rust: `Cargo.toml`
   - General: `VERSION` file
2. **Determine Bump Type**:
   - **Major** (BREAKING CHANGE): +1.0.0
   - **Minor** (feat): +0.1.0
   - **Patch** (fix): +0.0.1
3. **Execute**: Update file(s) -> Stage version file -> Continue to commit.

## 4. Execution Flow
1. **Status**: `git status` - Ensure clean working tree state.
2. **Diff**: `git diff` - Review changes. **Stop** if secrets or debug code found.
3. **Version**: Run *Version Control Strategy* (above).
4. **Stage**: 
   - Use `git add <file>` for precision.
   - Use `git add -p` if files contain unrelated changes.
   - **Avoid** `git add .` if untracked junk files exist.
5. **Message Construction**: Apply *Conventional Commits* (see below).
6. **Commit**: `git commit -m "..."`
7. **Verify**: `git status` - Confirm success.

## 5. Commit Message Standard (Conventional Commits)
**Format**: `<type>(<scope>): <description>`

**Types**:
- `feat`: A new feature
- `fix`: A bug fix
- `docs`: Documentation only changes
- `style`: Changes that do not affect the meaning of the code (white-space, formatting, etc)
- `refactor`: A code change that neither fixes a bug nor adds a feature
- `perf`: A code change that improves performance
- `test`: Adding missing tests or correcting existing tests
- `build`: Changes that affect the build system or external dependencies
- `ci`: Changes to our CI configuration files and scripts
- `chore`: Other changes that don't modify src or test files
- `revert`: Reverts a previous commit

**Rules**:
- Use imperative mood ("add" not "added").
- No period at the end.
- Max 72 chars for the subject line.
- **Scope** is optional but recommended (e.g., `feat(auth): ...`).

## 6. Grouping Strategy
- **Atomic Commits**: If `git status` shows unrelated changes (e.g., a bug fix in `api/` and a typo in `README`), split them into two separate commits.
- **Ask User**: "I see changes in X and Y. Should these be separate commits?"

## 7. Changelog Integration

After successful commit, update `CHANGELOG.md` using the `project-change-log` skill:

1. **Check**: Does `CHANGELOG.md` exist in project root?
   - If no: Create from template (see skill)
   - If yes: Read current content

2. **Parse Commit**: Extract type, scope, and description from commit message

3. **Map to Category**:
   | Commit Type | Changelog Category |
   |-------------|-------------------|
   | `feat` | Added |
   | `fix` | Fixed |
   | `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore` | Changed |
   | `security` | Security |

4. **Add Entry**: Insert under `[Unreleased]` in appropriate category:
   ```markdown
   ### Added
   - **scope**: Description from commit
   ```

5. **Stage**: `git add CHANGELOG.md`

6. **Amend or New Commit**:
   - If changelog update is minor: Amend the commit with `--amend --no-edit`
   - If user prefers: Create separate `docs(changelog): update changelog` commit

**Note**: Reference `.claude/skills/project-change-log/SKILL.md` for full changelog format and conventions.