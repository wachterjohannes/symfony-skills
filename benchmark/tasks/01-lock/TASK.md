This application has a console command `app:send-reminders` that emails users about
upcoming deadlines. It runs every minute from cron.

Occasionally two runs overlap and users get the same reminder twice. Make that impossible.
