Users need reminding about upcoming deadlines. Add a console command that sends those
reminders, meant to be run every minute from cron.

Two runs must never overlap — if one is still going when the next minute comes round, the
second has to do nothing rather than send everything twice.
