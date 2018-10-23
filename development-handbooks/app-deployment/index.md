# Application deployment

The goal of this document is to describe how we deploy applications
on the Winding Tree platform.

We are using multiple services for running our stuff:

- GitHub pages
- [now.sh](now.sh)
- [AWS](aws.sh)

You can see details about it in the respective documents. There
is no golden rule for what goes where, but in general, GitHub pages
work for statically built content, now.sh works well if you need to
get something out there pretty quickly, and AWS should be used for
everything that should be seemingly reliable.

The only rule we should follow is to never do manual building and
deployment. Everything that gets out there should be built automatically,
ideally by our CI, Travis.