# ddrescue

Most of the time, the simple version of the command will do an excellent job.
Use the extended version of the command in case you need to have more control over the process.

## Simple command

```bash
ddrescue --verbose --idirect --retry 0 --sector-size 2048 /dev/sr0 output-name.iso output-name.log
```

## Extended command

```bash
ddrescue --verbose --no-scrape --no-trim --idirect --retry 0 --min-read-rate 3M --sector-size 2048 /dev/sr0 output-name.iso output-name.log
```
