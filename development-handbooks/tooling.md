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
- Some packages that are used for example for builds might not get properly updated (for example Babel broke). Make sure that you test all
`npm scripts` and other stuff not executed by `npm test`.

## now.sh (https://now.sh)

Container based hosting. Some of our client facing apps are running from there
either as NPM applications or Docker containers.

See [now.sh deployment page](app-deployment/now.md) for details.

## AWS (https://aws.amazon.com/)

If we need something special, we use AWS. We try to run as much as possible
from [ECS](https://aws.amazon.com/ecs/) via Docker containers.

See [AWS deployment page](app-deployment/aws.md) for details.

## Infura (https://infura.io//)

For full Ethereum nodes, we are using excellent services of Infura.
We are currently not running our own Ethereum nodes.

## Swarm gateways

For accessing swarm we are using [our own instance](https://github.com/windingtree/ethereum-swarm-node)
running on https://swarm.windingtree.com that is connected to the same network
as https://swarm-gateways.net/ is.

Alternatively, it might be possible to use https://swarm.blockscan.com/
but we don't have a big history with them.

## Cloudflare (https://www.cloudflare.com/)

Proxy for almost every service we are running. Also, our DNS is there.