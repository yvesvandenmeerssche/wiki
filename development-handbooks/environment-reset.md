# Environment reset checklist

This is an extension of
[technical steps](https://github.com/windingtree/wiki/blob/master/developer-guides/preparing-environment.md#resetting-an-existing-environment)
and is relevant only to environments managed by Winding Tree. As
we don't have a production-like environment yet, this all applies
to every environment considered as
[*testing*](https://github.com/windingtree/wiki/blob/master/developer-resources.md#publicly-available-wt-deployments).

Our platform is evolving at a rapid pace so inevitably, we are
introducing breaking changes every so often. Unfortunately, these
changes also sometimes render existing data incompatible with
the new platform version.

Goal of this checklist is to help anybody who wants to deploy a new
version of the environment that is not backwards compatible with the
existing data. Sometimes it's just easier to start over than to fix
what we already have.

Even if we specifically declare that both **playground** and **demo**
can change at any time, we want to keep the surprises to a minimum
and hopefully keep every interested party in the loop.

## Checklist

- [ ] Decide a date of the reset (deployment of a breaking change).
- [ ] Inform community about the planned change (Google Group thread,
more dev channels in the future). The announcement should contain
information and links to what has changed and how anyone can update
their integration to adapt (and how can they get help if needed).
This should go out a few days before the reset.
- [ ] Create and deploy new iteration of WTIndex. Don't forget to set up
Lif token address (migration in wt-contracts does that for you).
- [ ] Document the new WTIndex address in
[deployments](https://github.com/windingtree/wiki/blob/master/deployments.md)
page. We might want to keep the old index addresses there as well.
- [ ] Deploy updated wt-fixtures to the new WTIndex.
- [ ] Wait for the announced date.
- [ ] Deploy all up-to-date tooling in said envirnoment (wt-read-api,
wt-write-api, ...).
- [ ] Update all Winding Tree *satellite* operations (CRON based hotels etc.).

## Notes

- This might evolve into an automated recurring reset (for example once a month)
- This might evolve into a release guide for Winding Tree platform as a whole
- This might not be necessary if we can transparently describe properties
of our environments
- This will probably not work for production-like environment
- If data format has not changed, you can freely reuse your off chain data. Nothing
is actually deleted.
