## Lerna

- [Change detection](#change-detection)
- [Caveats](#caveats)


### Change Detection

`lerna` determines a change by checking if the latest commit `HEAD` is pointing at is ahead of last tagged commit. If so, it detects a change.

This will conclude a change by `lerna`. Note conventional commit message only helps with versioning decision but not change detection.
```
* a6b2ded - (HEAD -> main) fix(node): ver++ (9 seconds ago) <Lerna>
* df32b34 - (tag: node@0.5.1, origin/main) chore(release): Publish Images (3 minutes ago) <Lerna>
```

### Caveats

- NEVER EVER do version bump/tagging from feature branch unless it's targeting a prerelease which should be in a different release channel than official release.
- Make sure branch protection is not that strict otherwise `lerna` will fail to be able to push commited changes on version/changelog back on your behalf during CD.


---

## semantic-release

- It uses [env-ci](https://github.com/pvdlg/env-ci) to help gather metadata infor such as commit/branch from CI it's detected.
- Sometimes when the next release has been tagged but actual module is failed to be published, you simply cannot re-publish on that version. To fix it, you need to delete that tag from `origin` and then remove its local counterpart as well. Once done, try re-publishing. `commit-analyzer` **ONLY** looks at local tags when calculating new versions.
