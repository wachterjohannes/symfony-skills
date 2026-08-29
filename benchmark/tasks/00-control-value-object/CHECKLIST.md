- The amount is an integer in minor units. Not a float, not a string, and not a library
  type pulled in for the purpose.
- The type is immutable — adding or multiplying returns a new instance, and there is no
  setter and no assignment to an existing instance's state.
- Adding two different currencies throws. Not a return of `null`, `false` or an error
  object, and not a silent conversion.
- The currency mismatch is covered by a test that asserts the exception, not merely a test
  of the happy path.
