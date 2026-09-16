# A Simple Guide on Changelog and Release Note Conventions

A changelog is a public record of what changed in a project, when it changed, and why it matters. It sits at the intersection of two very different audiences: developers who need precise, scannable technical detail, and search engines that need structured, keyword-rich text to index and surface your project when someone searches for a bug fix, a migration guide, or a new feature.

You should treat changelogs as a contract with your users. If you ship a breaking change and bury it in a vague commit message, you've broken that contract. This is because you, as a maintainer, owe your users a clear record of what to expect when they upgrade. 

A well-maintained changelog reduces support tickets, builds trust, and gives search engines a reason to rank your repository higher for troubleshooting queries.

---

## Core Principles of Changelogs

Every good changelog follows a small set of principles, regardless of project size or language.

1. **Changelogs are for humans, not machines:** Even if you generate them automatically from commit messages, the final output should read like something a person wrote for another person.

2. **Every version gets an entry:** No skipping releases, even patch releases. If a version shipped, it needs a paragraph or bullet list.

3. **Group changes by type:** Don't mix a security fix with a documentation typo in the same undifferentiated list.

4. **The latest release goes at the top:** Reverse chronological order is the universal convention.

5. **Dates use a consistent, unambiguous format:** ISO 8601 (YYYY-MM-DD) avoids the "is 03/04 March or April" confusion.

6. **Link to the diff or the tag:** Give readers a way to verify or dig deeper.

---

## Choosing a Format: Keep a Changelog

The most widely adopted convention is [Keep a Changelog](https://keepachangelog.com), and I recommend it as your default unless your organization has a strong reason to deviate. It defines six standard categories:

- **Added** for new features
- **Changed** for changes in existing functionality
- **Deprecated** for soon to be removed features
- **Removed** for now removed features
- **Fixed** for any bug fixes
- **Security** for vulnerability patches

Here's what a compliant entry looks like:

```markdown
## [1.4.0] - 2026-09-10

### Added
- Support for streaming responses in the CLI client.
- New `--dry-run` flag for the migration tool.

### Changed
- Default timeout increased from 5s to 15s to reduce false-positive failures on slow networks.

### Deprecated
- The `legacyAuth()` method will be removed in v2.0.0. Use `authenticate()` instead.

### Fixed
- Corrected a race condition in the connection pool that caused intermittent 502 errors under high load.

### Security
- Patched a path traversal vulnerability in the file upload handler (CVE-2026-31421).
```

This structure benefits both audiences at once. A developer scanning the page can jump straight to "Fixed" if they only care about bug fixes. A search engine crawling the page sees clear semantic headings (`### Fixed`, `### Security`) paired with specific terms like CVE numbers, error codes, and method names, all of which are exactly what people type into search bars when something breaks.

---

## Versioning: Pair Your Changelog With SemVer

A changelog without a versioning scheme is just a diary. Pair yours with [Semantic Versioning](https://semver.org) so that the version number itself communicates the scope of change before anyone reads a word.

- **MAJOR** version when you make incompatible API changes.
- **MINOR** version when you add functionality in a backward compatible manner.
- **PATCH** version when you make backward compatible bug fixes.

---

## Writing Style Guidelines

### Use Consistent Tense and Voice

Pick past tense, present tense, or imperative mood and stick with it across the entire project. Most teams use imperative present tense because it mirrors Git commit conventions ("Fix crash on startup" rather than "Fixed a crash on startup"). Whichever you choose, consistency matters more than the specific choice.

### Write for Skimming

Developers rarely read a changelog top to bottom. They scan for their version, their feature, or their error message. Use bullet points, bold the important nouns, and keep each line to a single change. Avoid paragraphs longer than two sentences inside a changelog entry.

### Be Specific, Not Vague

Compare these two entries:

Vague: "Fixed some bugs."

Specific: "Fixed a bug where the export button would silently fail if the filename contained a Unicode character."

The second entry is longer, but it's the one that shows up in search results when someone types "export button silently fails unicode." Specificity is not just good practice, it's the mechanism by which your changelog becomes discoverable.

### Explain the "Why" for Non-Obvious Changes

If you increased a timeout, changed a default, or removed a feature, a single clause explaining the reasoning saves your users from filing an issue asking "why did you do this?" You don't need a full paragraph. One sentence is usually enough.

---

## SEO Considerations for Changelogs

Search engines index changelogs the same way they index any other page: through headings, structured text, and keyword density. A few practical steps make a real difference.

1. **Use descriptive headings, not just version numbers:** Instead of only `## [1.4.0] - 2026-09-10`, consider adding a short summary line underneath, like "Streaming CLI responses and faster timeouts." This gives crawlers and readers a human-readable hook.

2. **Include exact error messages and codes:** If a bug produced a specific stack trace, error code, or HTTP status, quote it. 

People search for the literal error text far more often than they search for your internal bug ID.

3. **Link internally and externall:** Link to the relevant pull request, issue, or documentation page. 

Internal links help search engines understand your site structure; external links to authoritative sources (like a CVE database) build trust signals.

4. **Keep a canonical, stable URL for your changelog:** Whether it's `CHANGELOG.md` at the repo root or a `/releases` page on your docs site, don't move it. 

Broken or shifting URLs destroy accumulated search ranking.

5. **Avoid keyword stuffing:** Write naturally. A changelog stuffed with repeated phrases reads badly to humans and can be penalized by search algorithms designed to detect low-quality content.

---

## Automating Changelogs With Conventional Commits

Manually writing changelog entries doesn't scale once a project has multiple contributors. [Conventional Commits](https://www.conventionalcommits.org) gives you a commit message format that tools like `standard-version`, `semantic-release`, and `git-cliff` can parse automatically to generate changelog entries and even determine the next version number.

The format looks like this:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Common types map directly to Keep a Changelog categories:

```
| Commit Type | Changelog Category |
|---|---|
| `feat` | Added |
| `fix` | Fixed |
| `perf` | Changed |
| `deprecate` | Deprecated |
| `remove` | Removed |
| `security` | Security |
```
Marcus is a solo maintainer with a full time job. He doesn't have time to hand write release notes every Friday. By adopting Conventional Commits and wiring up `semantic-release` in his CI pipeline, his changelog now writes itself every time he merges to `main`. 

He still reviews the generated text before tagging a release, because automation gets you eighty percent of the way there, but a human pass catches the awkward phrasing a script can't.

A breaking change in Conventional Commits is signaled with a `!` after the type or scope, or a `BREAKING CHANGE:` footer:

```
feat(auth)!: remove support for legacy token format

BREAKING CHANGE: The `legacyToken` field is no longer accepted.
Clients must migrate to the `bearerToken` field before upgrading.
```

This single commit message, if you automate correctly, produces a changelog entry, triggers a major version bump, and even generates a migration note, all without you touching the `CHANGELOG.md` file by hand.

---

## Handling Breaking Changes and Migrations

Breaking changes deserve more real estate than a one line bullet. When I ship one, I include a short "Migration Guide" subsection directly in the release notes, not buried in separate documentation.

```markdown
## [4.0.0] - 2026-09-15

### Changed
- **Breaking:** Renamed `client.send()` to `client.dispatch()` for consistency with the new event system.

### Migration Guide
If you called `client.send(payload)`, replace it with `client.dispatch(payload)`.
No other arguments changed. A codemod is available at `scripts/rename-send.js`.
```

You want your future self, and everyone downstream of you, to be able to upgrade without opening a support ticket.

---

## Common Mistakes to Avoid

- **Skipping patch releases in the changelog:** Even a one line dependency bump deserves an entry if it shipped as a tagged release.

- **Using relative dates like "last week":** They become meaningless the moment someone reads the changelog six months later.

- **Burying breaking changes under a generic "Changed" heading with no bold warning:** Make breaking changes visually distinct.
- **Writing changelog entries as raw commit message dumps:** A commit history is not a changelog. Curate it.
- **Forgetting to update the "Unreleased" section as you merge PRs:** Waiting until release day to write the whole changelog from scratch leads to missed changes and vague summaries because you no longer remember the details.
- **Inconsistent category naming across versions:** If you call it "Fixed" in one release and "Bug Fixes" in the next, both your readers and your search indexing suffer from the inconsistency.

---

## A Ready to Use Template

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
-

### Changed
-

### Deprecated
-

### Removed
-

### Fixed
-

### Security
-

## [1.0.0] - 2026-01-01

### Added
- Initial public release.
```

Keep the `[Unreleased]` section at the top at all times. As you merge pull requests, add a line under the relevant category immediately, not on release day. 

When you're ready to cut a release, rename `[Unreleased]` to the new version number and date, then add a fresh empty `[Unreleased]` section above it.

---

## Final Checklist Before You Tag a Release

- [ ] Every category with entries has at least one specific, non-vague bullet point.
- [ ] Breaking changes are bolded or in their own subsection with migration steps.
- [ ] The version number follows SemVer rules based on the actual scope of change.
- [ ] The date is in ISO 8601 format.
- [ ] Links to relevant PRs, issues, or CVEs are included where useful.
- [ ] The entry has been read once by a human, even if it was generated automatically.
- [ ] The `[Unreleased]` section is empty and ready for the next cycle.

---

## Conclusion 

A changelog is one of the cheapest forms of technical marketing and user support you can maintain. It costs a few minutes per pull request, but it compounds into a searchable, trustworthy history of your project's evolution. 

Whether you're a solo developer writing your own release notes at midnight or part of a team automating them through CI, the same rule holds: write for the human who will read it under stress, at 2 a.m., trying to figure out why their build broke after an upgrade. If that person can find their answer in your changelog, you've done the job right.

