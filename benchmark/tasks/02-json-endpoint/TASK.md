Add a `POST /api/subscribe` endpoint.

It takes a JSON body with an `email` and a `plan`. The email has to be a valid address, and
the plan has to be either `basic` or `pro`. Invalid input gets a 422 listing what was wrong.
A valid request returns 201.
