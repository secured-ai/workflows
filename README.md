# Shared workflows

Reusable GitHub Actions workflows for the SecuredAI product repositories:
[intent-envelope](https://github.com/secured-ai/intent-envelope),
[supervisor-check](https://github.com/secured-ai/supervisor-check) and
[red-team-harness](https://github.com/secured-ai/red-team-harness), plus the
agent-action-ledger, which is private.

## Pin these by commit SHA

A shared workflow is a supply chain concentration point. Anyone who can push here
changes CI in every repository that calls it, and `@main` means that change lands with no
review in the repository it affects. These repositories already pin every action by SHA
for exactly that reason, and a shared workflow deserves the same treatment or the pinning
elsewhere is decorative.

```yaml
uses: secured-ai/workflows/.github/workflows/node-suite.yml@<commit-sha>
```

Dependabot updates a pinned reusable workflow the same way it updates a pinned action, so
the cost of this is a PR rather than a stale copy.

## node-suite.yml

Install, typecheck and test a Node package across a version matrix.

```yaml
jobs:
  suite:
    uses: secured-ai/workflows/.github/workflows/node-suite.yml@<sha>
    with:
      node-versions: '["22.18","24"]'
```

Inputs: `node-versions` (JSON array, default `["22.18","24"]`), `working-directory`
(default `.`), `typecheck` (default true), `install` (default `npm ci`).

## claims.yml

Assert that specific tests exist and pass, by name.

This is the one worth explaining, because an ordinary green suite does not catch what it
catches. Each of these repositories carries a few tests that are not checking a feature,
they are checking a guarantee: the harness will not run without written authorisation,
the supervisor denies when a fact cannot be established, the envelope store has no write
path at all. Those properties are the difference between a control and a liability.

A normal suite stays green when one of them is deleted. Deleting a test is the cheapest
way to make a failing test stop failing, and it is silent. This names each guarantee and
fails with the specific one that went missing, so removing a safety test is as loud as
breaking the behaviour it protects.

```yaml
jobs:
  fails-closed:
    uses: secured-ai/workflows/.github/workflows/claims.yml@<sha>
    with:
      claims: |
        the agent does not get its own claim evaluated
        facts that cannot be established deny, they do not fall back to the agent
        a supervisor that throws still denies
```

Each line must appear as a passing TAP test, which is what `node --test` prints. A claim
is matched literally rather than as a pattern, so a bracket or a dot in a test title does
not quietly match something else. The three outcomes are distinguished: `ok` when it
passes, `FAILING` when the test exists and fails, `MISSING` when no test by that name
ran at all.

Inputs: `claims` (required, one per line), `node-version`, `working-directory`,
`install`, `test-command`.

## scorecard.yml

OpenSSF Scorecard, publishing the result so the badge in each README reads from it.
`publish_results` only works on a public repository, which these are.

```yaml
jobs:
  scorecard:
    uses: secured-ai/workflows/.github/workflows/scorecard.yml@<sha>
```

## License

Apache-2.0.
