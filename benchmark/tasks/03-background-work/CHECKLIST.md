- Uses Messenger — a message class plus a handler marked `#[AsMessageHandler]`, dispatched
  through `MessageBusInterface`.
- Does not build a queue by hand — no jobs table polled by a command, no `exec()` or
  `proc_open()` of a background process, no `fastcgi_finish_request()`.
- Configures an asynchronous transport so the message is not handled synchronously.
- Adds the component with `composer require symfony/messenger`.
