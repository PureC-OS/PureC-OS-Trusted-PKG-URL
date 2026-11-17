# CLAUDE.md

Guidance for Claude Code in this repository. The full rules for all agents are
in [AGENTS.md](AGENTS.md); they apply here without exception.

@AGENTS.md

## Notes for Claude Code

- This repository is usually checked out as the `Trusted/` submodule of the
  PureC OS tree. Run the publishing tools from the parent tree
  (`python3 mk/pkg/publish.py --registry Trusted ...`), not from here.
- Before saying a package is published, walk the checklist in AGENTS.md
  ("Publishing a package") point by point and verify the signatures with the
  public key. Building the archive is only the first step.
- Never touch the signing key beyond passing its path to `publish.py`. Do not
  read it into the conversation.
- Do not edit `registry.conf` or `repository.conf` after they are signed unless
  you re-sign them in the same step.
- Commit and push only when the user asks. Work on a branch, not on `main`,
  unless told otherwise.
