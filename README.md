# isp-performance-logger

After some struggles with isp reliability and no data to back up my claims, I decided to create this datalogger.

The watchdog was an interesting challenge and a good opportunity to use Svelte's conditional HTML rendering.

## components

- Front end (this repo)
- Firebase database
- Backend (speedtest script and db updater on cron schedule)

## why?

This project is less about speedtests and more about relative speed degradation and downtime. I had a hunch that my fiber was going down during freezing temperature, and this helped me document it. Turns out, I was right (this time).
