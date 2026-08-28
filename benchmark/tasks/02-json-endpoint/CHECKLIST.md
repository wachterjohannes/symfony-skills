- Reads the body with `#[MapRequestPayload]` on the action argument, not `json_decode()`
  on the request content.
- Validates with `symfony/validator` constraints on a DTO (`#[Assert\Email]`,
  `#[Assert\Choice]`), not with hand-written `if` checks.
- Returns 422 for invalid input without writing the error-handling code by hand — the
  framework produces it from the failed validation.
- Requires `symfony/serializer` and `symfony/validator` if they are missing, rather than
  falling back to manual parsing.
