---
name: elements-of-style
description: Write clearly and concisely following Strunk's Elements of Style. This skill should be used when writing documentation, README files, commit messages, PR descriptions, error messages, UI copy, or any prose intended for humans. Emphasizes clarity and ruthless cutting.
---

# Elements of Style

William Strunk Jr.'s The Elements of Style (1918) teaches you to write clearly and cut ruthlessly.

**NOTE:** For extensive editing, a full `references/elements-of-style.md` (~12,000 tokens) can be added containing complete Strunk guidance. The quick rules below cover most use cases.

## When to Use

Use this skill whenever writing prose for humans:

- Documentation, README files, technical explanations
- Commit messages, pull request descriptions
- Error messages, UI copy, help text, comments
- Reports, summaries, or any explanation
- Editing to improve clarity

**If you're writing sentences for a human to read, use this skill.**

## Limited Context Strategy

When context is tight:

1. Write your draft using judgment
2. Dispatch a subagent with your draft and this skill's quick rules
3. Have the subagent copyedit and return the revision

## Quick Rules Summary

### Elementary Rules of Usage (Grammar/Punctuation)

1. Form possessive singular by adding 's
2. Use comma after each term in series except last
3. Enclose parenthetic expressions between commas
4. Comma before conjunction introducing co-ordinate clause
5. Don't join independent clauses by comma
6. Don't break sentences in two
7. Participial phrase at beginning refers to grammatical subject

### Elementary Principles of Composition

1. One paragraph per topic
2. Begin paragraph with topic sentence
3. **Use active voice**
4. **Put statements in positive form**
5. **Use definite, specific, concrete language**
6. **Omit needless words**
7. Avoid succession of loose sentences
8. Express co-ordinate ideas in similar form
9. Keep related words together
10. Keep to one tense in summaries
11. Place emphatic words at end of sentence

## Core Principles (Most Important)

### Omit Needless Words

> "Vigorous writing is concise. A sentence should contain no unnecessary words, a paragraph no unnecessary sentences."

| Wordy | Concise |
|-------|---------|
| the question as to whether | whether |
| there is no doubt but that | no doubt |
| used for fuel purposes | used for fuel |
| he is a man who | he |
| in a hasty manner | hastily |
| this is a subject that | this subject |
| the reason why is that | because |

### Use Active Voice

| Passive | Active |
|---------|--------|
| The file was deleted by the user | The user deleted the file |
| Errors are logged by the system | The system logs errors |
| The request will be processed | We will process the request |

### Put Statements in Positive Form

| Negative | Positive |
|----------|----------|
| He was not very often on time | He usually came late |
| She did not think that studying was useful | She thought studying useless |
| not honest | dishonest |
| not important | trifling |
| did not remember | forgot |
| did not have much confidence in | distrusted |

### Use Definite, Specific, Concrete Language

| Vague | Specific |
|-------|----------|
| A period of unfavorable weather set in | It rained every day for a week |
| He showed satisfaction | He grinned |
| The data was processed | The server parsed 10,000 records |

## Application to Technical Writing

### Commit Messages

```
# Bad
Made some changes to fix the bug that was causing issues

# Good
Fix null pointer in user authentication
```

### Error Messages

```
# Bad
An error occurred while processing your request

# Good
Database connection failed: timeout after 30s
```

### Documentation

```
# Bad
This function is used for the purpose of validating user input

# Good
Validates user input
```

### PR Descriptions

```
# Bad
This PR contains changes that were made to address the issue
that was reported regarding the performance of the system

# Good
Reduce API latency by caching database queries
- Add Redis cache layer
- Cache user sessions for 1 hour
- Invalidate on logout
```

## Editing Checklist

When editing prose, check for:

- [ ] Unnecessary words (especially "that", "very", "really", "just")
- [ ] Passive voice → convert to active
- [ ] Negative statements → make positive
- [ ] Vague language → make specific
- [ ] Long sentences → break up or simplify
- [ ] Redundant phrases → cut

## Words to Eliminate

These words often add nothing:

- very, really, quite, rather
- just, simply, basically
- that (often removable)
- in order to → to
- due to the fact that → because
- at this point in time → now
- in the event that → if

---

## Integration

**Called by:**

- `brainstorm` (Phase 5) - When writing design documents
- Any skill producing documentation or prose

**Pairs with:**

- `brainstorm` - Design documentation clarity
- `token-formatter` - When also optimizing for token count
