# Development and release handbook

The goal of this document is to describe how we develop and release parts of
the Winding Tree platform.

## Repositories

We have currently many repositories on our
[GitHub](https://github.com/windingtree/), but the following stand out:

### Smart contracts

- [LífToken](https://github.com/windingtree/LifToken) - Contains smart
contracts and tests for our cryptotoken, ICO and TGE
- [wt-contracts](https://github.com/windingtree/wt-contracts) - Contains smart
contracts and tests with business logic related to hotels, airlines etc.

### Javascript libraries

- [wt-js-libs](https://github.com/windingtree/wt-js-libs) - A wrapper around
wt-contracts which uses [web3.js](https://github.com/ethereum/web3.js/)

### End-user applications

- [wt-read-api](https://github.com/windingtree/wt-read-api) - REST-like
API for reading the hotel data. Useful if you're building an OTA.
- [wt-write-api](https://github.com/windingtree/wt-write-api) - REST-like
API for writing the hotel data. Useful if you're building a PMS.
- [wt-demo-app](https://github.com/windingtree/wt-demo-app) - Client-facing
example browser app for accessing Winding Tree assets. This will become obsolete
very soon.

## Development workflow

We are using a common
[git development workflow](http://nvie.com/posts/a-successful-git-branching-model/)
which in short means that:

- `master` is always production ready and should be merged to only when making
a release.
- `develop` holds all new code.
- `feature/*` or `feat/*` branches hold new features, when merged to develop, there should
always be a merge commit.
- `hotfix/*` or `fix/*` are the only branches that can be merged directly into master.
They usually have to be merged into develop as well.
- `proposal/*` are branches with some long-term and big update that we would
eventually like to promote to develop.
- `release/*` branches are used for automatic deployment via
Travis.

Other naming conventions for branches are possible, but should at least closely
follow the general
[commitizen](https://www.npmjs.com/package/commitizen#using-the-command-line-tool)
advice as we are going to adopt some form of commitizen in the future.

If you want to contribute, follow the contributing guidelines in each repository.
In general, if you find a problem or have an idea, start a GitHub issue. If you
have some code ready, feel free to create a Pull request, usually into a `develop`
branch. Thank you!

Every pull request has to be approved by one of our core members and only after
an approval can be merged.

For now we don't enforce any coding style, but that is probably going to change
at some point. Just keep your formatting sane or at least similar to the
surrounding code. If there's something unclear, it will probably come up during
Pull request approval process.


# Releases

We are aiming to use both [npm](https://npmjs.com) and
[EthPM](https://www.ethpm.com/) for our package distribution. The initial idea
was to publish only the compiled smart contracts into npm and smart contracts
source to EthPM, we shall see if that's a feasible approach. For now, you can
grab packages from [our npm organization](https://www.npmjs.com/org/windingtree).

The javascript libriaries will be published solely to npm.

The end-user applications (API, demo app) will be released as GitHub releases
artifacts. See [application deployment](https://github.com/windingtree/wiki/blob/master/app-deployment.md)
for more information.

Our releases are done by [Travis CI](https://travis-ci.org/). For now we are
manually creating a release by tagging a version commit in master. This is
subject to change as we would like to use
[semantic-release](https://semantic-release.gitbooks.io/semantic-release/#highlights)
in the future along with commitizen.

It is important to keep the version number in `git tag` and in `package.json`
in sync. That can be achieved by tagging a release with
[`npm version`](https://docs.npmjs.com/cli/version) command.

We are still in the prototyping phase, so we are using the major version of `0.x.x`.

In practice, the release process is as follows:

- Development is going on with PRs that are merged to `develop` branch.
- Once we are happy with the state of the `develop` branch, we generate current
documentation if necessary and we raise the version in `package.json` and
`package-lock.json`. That can be done with `npm version`.
- *Do not push the created tag yet!*
- Create a PR from `develop` to `master` marked as **chore: Release x.y.z**
- Merge this PR with a merge commit.
- Push the tag created by `npm version` (experimental) or manually tag the
resulting merge commit and push it to github.
- In case of libraries, Travis makes a release from a tagged commit.
- In case of services, you need to merge master to a dedicated
`release/*` branch to deploy the latest version.
