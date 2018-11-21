# Hotel bookings

## What?

We want to show that bookings of hotels can be done. For that
we need a running instance of [wt-booking-api](https://github.com/windingtree/wt-booking-api).
And that in turn needs a running instance of [wt-write-api](https://github.com/windingtree/wt-write-api)
because it's using a pre-configured static `wt-write-api` account.
That's whywe cannot use exinst `wt-write-api` deployments that
are wiped every 24 hours.

## Where?

We have a private instance of `wt-write-api` and a public
instance of [`wt-booking-api`](https://mazurka-booking.windingtree.com/)
running alongside everything else in a single docker cluster on AWS ECS.

`wt-write-api` service is using the same task as the public playground
deployment is. If there is a significant version bump, you need to update
it manually (see below).

- playground - 0x9d226A90C5a4c711c92e23cD246e9eb77d2D21f9

## How?

The `wt-write-api` instance is not exposeed through Elastic Load Balancer,
but rather through [Service Discovery](https://aws.amazon.com/blogs/aws/amazon-ecs-service-discovery/)
which in short assigns a DNS name to ECS service. That is then used in
`wt-booking-api` configuration which is [deployed as a standard app](../aws.md)
except it's not deployed through Travis CI. Both of these services are
handled manually to ensure a fitting configuration. Because if this
`wt-write-api` is turned off, we need to create a new account there and
re-configure the `wt-booking-api`.

In order to do that, you need to:

- Enable SSH on the cluster instance where `wt-booking-api` service
is running. That's done in EC2 -> Security Groups. Pick the one that your
EC2 instance handling the cluster is in.
- Go to *Route 53*, and take a look into `.local` zone. You should see
A DNS record containing current service ID of `wt-write-api`. Take a note.
- In Route 53, there should also be a SRV record. It contains the A DNS path
and a few numbers. One of them (the third one) is port. Take a note.
- SSH where the `wt-booking-api` is. You need a keypair associated with
that EC2 instance. Ask around in WT dev team.
- From there, run the command to create `wt-write-api` account against the noted
DNS path and port - something like `http://fe0a6816-1452-4bc7-89b2-5486065cf7de.mazurka-write-api.local:38552`
- Take the returned secret and along with the `wt-write-api` address, set it as 
eenvironment variables in the `wt-booking-api` task along with other configuration
variables.


## Culprits

- If you set up service discovery on a new service, make sure that the Route 53
records are actually visible from the VPC that your EC2 instances are in
(Check in Route 53 settings).
- If you update the private `wt-write-api` service, you **need** to re-configure
the `wt-booking-api`. It's probable that the `wt-write-api` service address will
change.

## Add similar

To add a similar setup for another hotel, you don't need to setup another
instance of `wt-write-api`. A single instance per environment (one for demo, one
for playground) should be enough.

But you need to create a hotel-specific task for `wt-booking-api` with appropriate
configuration (including a fresh `wt-write-api` account) and start a fresh service
of it. All of the networking stuff is done exactly the same as in
[this handbook](../aws.md).

## Change existing

Just update the `wt-booking-api` task