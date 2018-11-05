# Booking

## High-level overview

Booking is done by means of sending a POST request to
the specified URL. Each hotel on the index can optionally specify
a `bookingUri` that should point to a server implementing the
[wt-booking-api](https://github.com/windingtree/wt-booking-api) interface.

What exactly happens on the backend side is more or less up to
the system that handles it. Generally, it should take care at
least of the following things:

- Validate the incoming booking request (in terms of room
availability, price, cancellation fees).
- Update the availability information both on the Winding Tree
platform and in internal hotel systems, if such systems exist.
- Relay the booking information to hotel manager(s) in some way.
- Possibly confirm the booking directly to the customer based on
customer contact information.

## Example

```sh
$ curl -X POST localhost:8935/booking -H 'Content-Type: application/json' --data '
{
  "hotelId": "0xe92a8f9a7264695f4aed8d1f397dbc687ba40299",
  "customer": {
    "name": "Sherlock",
    "surname": "Holmes",
    "address": {
      "line1": "221B Baker Street",
      "city": "London",
      "country": "GB"
    },
    "email": "sherlock.holmes@houndofthebaskervilles.net"
  },
  "pricing": {
    "currency": "GBP",
    "total": 221,
    "cancellationFees": [
      { "from": "2018-12-01", "to": "2019-01-01", "amount": 50 }
    ]
  },
  "booking": {
    "arrival": "2019-01-01",
    "departure": "2019-01-03",
    "rooms": [
      {
        "id": "single-room",
        "guestInfoIds": ["1"]
      },
      {
        "id": "single-room",
        "guestInfoIds": ["2"]
      }
    ],
    "guestInfo": [
      {
        "id": "1",
        "name": "Sherlock",
        "surname": "Holmes"
      },
      {
        "id": "2",
        "name": "John",
        "surname": "Watson"
      }
    ]
  }
}'
```
