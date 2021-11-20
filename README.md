# throttle
An extremely simple bash script that allows a limit on concurrent subshell daemon execution.

This is ideal for cases such as curling a thousand urls concurrently, but without hitting all of them at once.  Or limiting the number of records inserted into a database from an off-hand script.  If there's a simpler linux command to do so, I've missed it.  So I made this!

```bash
MAXCONCURRENT=10
TOTALCOUNT=100
cat infinite_stream | throttle key $MAXCONCURRENT $TOTALCOUNT | while read line; do
  (
    # Using unthrottle on EXIT lets it catch no matter the subshell's result.
    trap 'unthrottle key' EXIT
    # Run task of arbitrary length...
    curl -O some.site@or.whatever/thing
  ) &
done
```

One may also leave out the total count.  The option was only added to allow a hard limit as a sanity check, anyway.  This way, it'll continue until it hits the end of the stream, when the `read` command fails.

```bash
MAXCONCURRENT=10
cat other_stream | throttle other_key $MAXCONCURRENT | ...
```

## Execution

Additionally, the source is arranged to _be_ sourced.  It has its own throttle and unthrottle functions declared.  So it can be used as an executable script in the source path, with a symlink renamed to 'unthrottle', or sourced directly into another shell script.

```bash
cp throttle /usr/local/bin/
ln -s /usr/local/bin/throttle /usr/local/bin/unthrottle
```

or...

```bash
(
  . /path/to/throttle
  cat thing | throttle key 5 | while read line; do
    ( somesuch ; unthrottle key ) &
  done
)
```
