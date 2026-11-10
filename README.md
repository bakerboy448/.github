# bakerboy448/.github

Shared community health files, issue and PR templates, Renovate presets, and reusable workflows for `bakerboy448` repositories.

## Reusable workflows

### `release-calver.yml`

Publishes a GitHub release tagged with a calendar version after the caller has validated a commit.

| Item | Behavior |
| --- | --- |
| Tag format | `yyyy.mm.dd`, then `yyyy.mm.dd.2`, `yyyy.mm.dd.3` for further releases on the same UTC day. No `v` prefix. |
| Tag allocation | Reads existing calver tags and release records, so a draft left by a failed run is resumed with the same tag. |
| No-op cases | The SHA already carries a calver tag, or a newer calver tag already contains it (superseded). |
| Notes | Built with a pinned `git-cliff` from the commits since the previous calver tag, grouped by conventional-commit type, plus a compare link. |
| Publish | Creates a draft against the exact SHA, then publishes it as Latest. |
| Runner | `[self-hosted, Linux, X64, hetzner]`. |
| Concurrency | One release job per repository, queued, never cancelled. |

Inputs:

| Input | Type | Default | Meaning |
| --- | --- | --- | --- |
| `sha` | string, required | none | Commit already validated by the caller. The workflow releases this SHA, not the branch tip. |
| `prerelease` | boolean | `false` | Publish as a prerelease that never becomes Latest. |

The calling job needs `permissions: contents: write`.

#### Adopt it in a repository

1. Keep the repository's validation in a job of its own `ci.yml` that runs on `push` to the default branch.
2. Add a `release` job that waits for validation and passes the pushed SHA.
3. Pin the `uses:` ref to a full commit SHA of this repository.

```yaml
  release:
    needs: [validate]
    if: github.event_name == 'push' && needs.validate.result == 'success'
    permissions:
      contents: write
    uses: bakerboy448/.github/.github/workflows/release-calver.yml@<full-sha>
    with:
      sha: ${{ github.sha }}
```

Use conventional commit subjects (`feat:`, `fix:`, `ci:`) or the `New:` and `Fixed:` prefixes so the notes group correctly.
