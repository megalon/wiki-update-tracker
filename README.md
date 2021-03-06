# Update Tracker for Translations

Don't bother with your localisation when you make a change in a localised file, let this Action automatically backtrack changes and instruct your translators through Github Issues or Github Projects.

Currently, the Update Tracker Action purpose is to serve only the [BSMG Wiki](https://github.com/beat-saber-modding-group/wiki), to track updated English pages and notifying in issues/projects.
As such, templates are specifically customized for a Wiki using vuepress i18n. It is currently being reviewed to work in more places and localisation structures.

The Action has 2 main features:
- checking within a local git repository the status of every translation pages based on modifications on their corresponding original pages.
- updating issues for every translation pages, to point translation instructions out.

Here's the target workflow:

[![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gICAgV3JpdGVyLT4-T3JpZ2luYWwgUGFnZTogYXBwbHkgY2hhbmdlc1xuICAgIFdpa2kgVXBkYXRlIFRyYWNrZXItLT5PcmlnaW5hbCBQYWdlOiBjaGFuZ2VzIGRldGVjdGVkXG4gICAgV2lraSBVcGRhdGUgVHJhY2tlci0-PkdpdGh1YiBJc3N1ZXM6IGNyZWF0ZSAvIHVwZGF0ZVxuICAgIFRyYW5zbGF0b3ItLT4-R2l0aHViIElzc3Vlczogc2VlIGluc3RydWN0aW9uc1xuICAgIFRyYW5zbGF0b3ItPj5UcmFuc2xhdGlvbiBQYWdlOiBhcHBseSBjaGFuZ2VzXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gICAgV3JpdGVyLT4-T3JpZ2luYWwgUGFnZTogYXBwbHkgY2hhbmdlc1xuICAgIFdpa2kgVXBkYXRlIFRyYWNrZXItLT5PcmlnaW5hbCBQYWdlOiBjaGFuZ2VzIGRldGVjdGVkXG4gICAgV2lraSBVcGRhdGUgVHJhY2tlci0-PkdpdGh1YiBJc3N1ZXM6IGNyZWF0ZSAvIHVwZGF0ZVxuICAgIFRyYW5zbGF0b3ItLT4-R2l0aHViIElzc3Vlczogc2VlIGluc3RydWN0aW9uc1xuICAgIFRyYW5zbGF0b3ItPj5UcmFuc2xhdGlvbiBQYWdlOiBhcHBseSBjaGFuZ2VzXG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

It requires a local repository (use [checkout](https://github.com/actions/checkout)) on the active branch you want to check and multiple inputs.

New in v1.7: Github Projects is supported. Issues can now be enabled and disabled. Script much more stable and adaptable.
WARNING: `translation-paths` was removed, because paths must now be given with corresponding language tags as described in new input `translations`.

New in v1.6: Update Tracker now automatically creates stub files when missing, with the correct header. Can be enabled or disabled.

New in v1.5: set the Frontmatter header `translation-done: false` in a markdown translation file to tell the script the page has not been initialized yet.

```yml
---
translation-done: false
---
```

## Inputs

**`repo-path`**

The path where the git repo is located within the filesystem.

Example: `${GITHUB_WORKSPACE}`

**`original-path`**

The original pages path.

It must be given relatively to the git repo root path.

Example: `wiki`

**`ignored-paths`**

A blacklist to mark unassociated directories and files within the original pages path.

These must be given relatively to the git repo root path.

Example: `wiki/LICENSE,wiki/.vuepress`

**`translations`**

The translation pages relative paths and their associated language tag. You may have different paths for translation, but each must follow the same structure as the original pages path.

These must be given relatively to the git repo root path. Language tags are defined in [RFC 5646](https://tools.ietf.org/html/rfc5646).

Example: `fr:wiki/fr,zh:wiki/zh`

**`repository`**

The GitHub repository in the form `Author/repo`, used to update issues. Defaults to `${{ github.repository }}`.

**`file-suffix`**

The file suffix (or extension) used to identify Wiki page. Defaults to `.md`.

**`bot-label`**

The Github label used to track managed issues. Issues with this label will be seen by the script, other will be out of scope.

Required when `update-issues` is enabled.

[More info](https://help.github.com/en/github/managing-your-work-on-github/about-labels).

**`token`**

The job's access token, given with `${{ secrets.GITHUB_TOKEN }}` or other means to restrict to least-privilege permissions (recommended).

[More info](https://help.github.com/en/actions/configuring-and-managing-workflows/authenticating-with-the-github_token).

**`log-level`**

The Python script's log level. It can be `DEBUG`, `INFO`, `WARNING` or `CRITICAL`. Defaults to `INFO`.

[More info](https://docs.python.org/3/library/logging.html).

**`auto-create`**

Allow automatic creation of files that need to be created, with a preset body containing the header `translation-done: false` meaning the page has not been initialized.
It can be `true`, `1`, `yes` to enable auto create or `false`, `0`, `no` to disable auto create.

The Action will commit through git and push to Github repo on the same branch.

**`update-issues`**

Enable Github Issues update.
It can be `true`, `1`, `yes` to enable issues update or `false`, `0`, `no` to disable issues update.

**`update-projects`**

Enable Github Projects update.
It can be `true`, `1`, `yes` to enable projects update or `false`, `0`, `no` to disable projects update.

## Outputs

**`translation-status`**

Comma-separated list containing the Wiki pages used for translation and their status, in the form `path:status`. The path is relative to the git repo.

The status can be:
- `UTD` if the page is up-to-date
- `UPDATE` if the page requires an update
- `TBC` if the page needs to be created and initialized
- `TBI` if the page needs to be initialized

Example: `wiki/fr/README.md:UPDATE,wiki/fr/beginners-guide.md:UTD`

**`open-issues`**

Comma-separated list containing open issue numbers (when a file requires updating, creation or initialization).

Example: `65,67,70`


## How it works

Quickly resumed, the script:
- checks blobs, trees, commits and diffs to tell either a translation file should be updated or not, using [GitPython](https://github.com/gitpython-developers/GitPython) on a local repository.
- updates issues accordingly, giving tools to help the translator, using [Github API](https://developer.github.com/v3/issues/) through the [repo's GitHub App](https://help.github.com/en/actions/configuring-and-managing-workflows/authenticating-with-the-github_token).


