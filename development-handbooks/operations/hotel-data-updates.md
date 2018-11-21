# Regular hotel data updates

## What?

Hotels have time-sensitive data:

- `cancellationPolicies`
- `ratePlans`
- `availability`

So it expires from time to time. Luckily, we have a script that
can randomly regenerate all of that data in
[`wt-fixtures`](https://github.com/windingtree/wt-fixtures/blob/master/scripts/regenerate.js).
But it's not really feasible to run it manually every so often,
particularly for `availability` that gets partly stale every day.

## Where?

We can deploy an [AWS Lambda](https://aws.amazon.com/lambda/) for
every single hotel that can be triggered at any given time with
[rate or cron event](https://docs.aws.amazon.com/lambda/latest/dg/tutorial-scheduled-events-schedule-expressions.html).

We have a [simple wrapper](https://github.com/windingtree/wt-fixtures-refresh-lambda)
over `wt-fixtures` that contains a version of `wt-fixtures`'s
command that by default regenerates availability and updates
the hotel data via [wt-write-api](https://github.com/windingtree/wt-write-api).

This is currently setup for the following hotels (data should be
regenerated on the 21st of each month):

- playground - 0xE11fe7Ed3f127422173d1f4fCB48C215E7E90C3e
- playground - 0x10B8EAF63f594C24892fa35C9F0C8A3A69Db6022
- playground - 0xf5BD6c846c14CD6E6c45934cf6e7D2072184a86b
- playground - 0x9d226A90C5a4c711c92e23cD246e9eb77d2D21f9
- playground - 0x0925CF078a218b75853A714dF8F742A1962996B7

## How?

Every AWS Lambda is basically a directory with code with a single
entry point, in our case [`update-time-data.handler`](https://github.com/windingtree/wt-fixtures-refresh-lambda/blob/master/update-time-data.js).

So for every hotel we want to get updated, we have a separate Lambda
function that is configured for that hotel. The great thing about
this is, that the code is *isolated*. So if you need something special
for your hotel, you can *directly edit the code* that the Lambda will run.
Including the data that will be updated, which might come in handy when you
for example need to set a single value.

All of the lambdas are configured to run once a month, but you can run
them whenever you need in a **Test** mode, which just tells the Lambda
to run. We are not using any data from the incoming event, so you can
probably choose a random event and the Lambda will just run.

## Culprits

- AWS Lambda supports only Node 8.
- `config` of wt-fixtures will get re-written on `npm install` in
`wt-fixtures-refresh-lambda`. But you can edit it in any way you
want on directly AWS Lambda.
- If you change the Lambda code directly, it will get overwritten if
you upload new archive from `wt-fixtures-refresh-lambda`.
- Lambdas are mercilessly killed after timeout. You might need to
fiddle with that settings.

## Add similar

To add a new lambda for hotel, you have to follow [this guide](https://github.com/windingtree/wt-fixtures-refresh-lambda#setup).

## Change existing

Go to AWS Lambda and choose your function. You can directly edit
the code in the browser or you can change the entire function
with uploading a new archive.