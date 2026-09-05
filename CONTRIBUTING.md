# Contributing

Bug reports, documentation fixes, runnable examples, and patches are welcome.
Open an issue in this public repository, or fork it and open a pull request here.
You do not need access to Pushary's private repository to contribute.

## Start small

Pick an unassigned `good first issue`, explain the change you plan, and include
steps that another developer can use to verify it. For a bug, include the package
and framework versions, your OS, expected behavior, and a minimal reproduction.
Never include API keys, enrollment links, customer data, or private transcripts.

For a JavaScript adapter, run `npm install`, `npm run typecheck`, `npm test`, and
`npm run build` in your clone. Follow the README for Python or plugin-specific
setup. Documentation changes should have working links and commands you tried.

## How your patch ships

This repository is a public mirror of a directory in Pushary's private monorepo.
We review your PR here, then apply accepted changes upstream before publishing
this mirror. A direct merge into the mirror could be overwritten by the next sync.

The maintainer handling your PR will:

1. Review the patch and discuss requested changes in the public PR.
2. Apply accepted changes upstream, retaining author attribution in that commit.
3. Run the relevant checks, release a package if needed, and sync this repository.
4. Link the public sync commit and released version (when applicable) back to your
   PR, credit your contribution publicly, then close it as shipped.

Public sync commits squash private history, so upstream author attribution does
not automatically appear in this mirror's GitHub contributor graph. The public
PR and shipping comment preserve visible credit. An upstream-only patch is not
considered shipped. If we cannot accept a change, we explain why on the PR.

## Security

Report vulnerabilities privately to aadil@pushary.com instead of opening a public
issue. If this repository has a SECURITY.md, follow its disclosure guidance.
