# Development tooling

Goal of this page is to show all the external tooling we are using during development.

## Travis CI (https://travis-ci.org or https://travis-ci.com/)

Our CI environment. All of the automated tests and builds are done there.
Every publishing (NPM and others) or deployment (now.sh and others) shall happen in a Travis pipeline.

### How to set it up

- Add `.travis.yml` to your repository. Examples are all over our projects, pick the right one.
- Enable the repository CI process in travis settings.

### Useful notes

- If you need environment variables:
  - Either set them up in travis settings
  - Or use `travis encrypt` command like `travis encrypt NOW_TOKEN=1234 --add`
- If you need regular deployments:
  - You can use Travis CRON jobs
  - Watch for jobs and stages conditions, it might not work for CRON jobs (they are not filtered by tags, but by branch names)

## Coveralls (https://coveralls.io)

Code coverage tool. Every build from Travis should report to Coveralls and 
Coveralls should report to GitHub Pull Requests.

### How to set it up

- Setup repository to use [nyc](https://www.npmjs.com/package/nyc) and [coveralls](https://www.npmjs.com/package/coveralls) tools
- Setup Travis to report coverage after the tests are done
- Just enable the repository in coveralls settings

```json
{
  "scripts": {
    "test": "rimraf .nyc_output .nyc_coverage && nyc --clean=false --reporter=text mocha \"test/**/*.spec.js\" --timeout 20000",
    "coverage": "nyc report --reporter=text-lcov | coveralls"
  }
}
```

```yaml
jobs:
  include:
  - stage: test
    script:
    - npm run test
    - npm run coverage
```

## Greenkeeper (http://greenkeeper.io)

Dependency checker. It automatically keeps track of new releases of all of our
dependencies and automatically creates Pull Requests.

### How to set it up

- You need Travis enabled.
- Enable the Greenkeeper integration in GitHub Winding Tree's organization settings for given repository
or go [through here](https://github.com/apps/greenkeeper) and request the repo access you're not the owner of
via Configure button.
- After some time (they say 30 minutes but more is possible), a PR should appear.
- After the PR is properly merged, Greenkeeper starts working.

### Useful notes

- It works only against a default branch on github - no configuration option to change that. It might not be what you want,
so be careful when accepting PRs. You might manually change the target branch of the PR (which is prone to human errors and false positives from GK side)
or change the setup of the repository if it works for you.
- You can't use `npm ci` on Travis CI because it really does not work.  Use `install: case $TRAVIS_BRANCH in greenkeeper*) npm i;; *) npm ci;; esac;`
in `.travis.yml` to use `npm install` in greenkeeper builds and `npm ci` everywhere else.
- It does not work out of the box with `package-lock.json`. There's [a solution](https://github.com/greenkeeperio/greenkeeper-lockfile/) that requires
GitHub non-project-specific token to be able to commit back from Travis. That's not great. Make the lockfile up to date by hand until Travis supports
the improved GitHub per-project integration.
- Some packages that are used for example for builds might not get properly updated (for example Babel broke). Make sure that you test all
`npm scripts` and other stuff not executed by `npm test`.

## now.sh (http://now.sh)

Container based hosting. Majority of our client facing apps is running from there
either as NPM applications or Docker containers.

See [App deployment page](app-deployment.md) for details.
