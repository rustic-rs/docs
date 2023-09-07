# Removing snapshots according to a policy

Removing snapshots manually is tedious and error-prone, therefore rustic allows
specifying a policy (one or more `--keep-*` options) for which snapshots to
keep. You can for example define how many hourly, daily, weekly, monthly and
yearly snapshots to keep, and any other snapshots will be removed.

**Warning**: If you use an append-only repository with policy-based snapshot
removal, some security considerations are important. Please refer to the section
below for more information.

**Note**: You can always use the `--dry-run` option of the `forget` command,
which instructs rustic to not remove anything but instead just print what
actions would be performed.

The `forget` command accepts the following policy options:

- `--keep-last n` keep the `n` last (most recent) snapshots.
- `--keep-hourly n` for the last `n` hours which have one or more snapshots,
  keep only the most recent one for each hour.
- `--keep-daily n` for the last `n` days which have one or more snapshots, keep
  only the most recent one for each day.
- `--keep-weekly n` for the last `n` weeks which have one or more snapshots,
  keep only the most recent one for each week.
- `--keep-monthly n` for the last `n` months which have one or more snapshots,
  keep only the most recent one for each month.
- `--keep-yearly n` for the last `n` years which have one or more snapshots,
  keep only the most recent one for each year.
- `--keep-tag` keep all snapshots which have all tags specified by this option
  (can be specified multiple times).
- `--keep-within duration` keep all snapshots having a timestamp within the
  specified duration of the latest snapshot, where `duration` is a number of
  years, months, days, and hours. E.g. `2y5m7d3h` will keep all snapshots made
  in the two years, five months, seven days and three hours before the latest
  (most recent) snapshot.
- `--keep-within-hourly duration` keep all hourly snapshots made within the
  specified duration of the latest snapshot. The `duration` is specified in the
  same way as for `--keep-within` and the method for determining hourly
  snapshots is the same as for `--keep-hourly`.
- `--keep-within-daily duration` keep all daily snapshots made within the
  specified duration of the latest snapshot.
- `--keep-within-weekly duration` keep all weekly snapshots made within the
  specified duration of the latest snapshot.
- `--keep-within-monthly duration` keep all monthly snapshots made within the
  specified duration of the latest snapshot.
- `--keep-within-yearly duration` keep all yearly snapshots made within the
  specified duration of the latest snapshot.

**Note**: All calendar related options (`--keep-{hourly,daily,...}`) work on
natural time boundaries and *not* relative to when you run `forget`. Weeks are
Monday 00:00 to Sunday 23:59, days 00:00 to 23:59, hours :00 to :59, etc. They
also only count hours/days/weeks/etc which have one or more snapshots.

**Note**: All duration related options (`--keep-{within,-*}`) ignore snapshots
with a timestamp in the future (relative to when the `forget` command is run)
and these snapshots will hence not be removed.

**Note**: Specifying `--keep-tag ''` will match untagged snapshots only.

When `forget` is run with a policy, rustic first loads the list of all snapshots
and groups them by their host name and paths. The grouping options can be set
with `--group-by`, e.g. using `--group-by paths,tags` to instead group snapshots
by paths and tags. The policy is then applied to each group of snapshots
individually. This is a safety feature to prevent accidental removal of
unrelated backup sets. To disable grouping and apply the policy to all snapshots
regardless of their host, paths and tags, use `--group-by ''` (that is, an empty
value to `--group-by`).

Additionally, you can restrict the policy to only process snapshots which have a
particular hostname with the `--host` parameter, or tags with the `--tag`
option. When multiple tags are specified, only the snapshots which have all the
tags are considered. For example, the following command removes all but the
latest snapshot of all snapshots that have the tag `foo`:

```console
rustic forget --tag foo --keep-last 1
```

This command removes all but the last snapshot of all snapshots that have either
the `foo` or `bar` tag set:

```console
rustic forget --tag foo --tag bar --keep-last 1
```

To only keep the last snapshot of all snapshots with both the tag `foo` and
`bar` set use:

```console
rustic forget --tag foo,bar --keep-last 1
```

To ensure only untagged snapshots are considered, specify the empty string '' as
the tag.

```console
rustic forget --tag '' --keep-last 1
```

Let's look at a simple example: Suppose you have only made one backup every
Sunday for 12 weeks:

```console
rustic snapshots repository f00c6e2a opened successfully, password is correct ID Time Host Tags Paths

## 0a1f9759 2019-09-01 11:00:00 mopped /home/user/work 46cfe4d5 2019-09-08 11:00:00 mopped /home/user/work f6b1f037 2019-09-15 11:00:00 mopped /home/user/work eb430a5d 2019-09-22 11:00:00 mopped /home/user/work 8cf1cb9a 2019-09-29 11:00:00 mopped /home/user/work 5d33b116 2019-10-06 11:00:00 mopped /home/user/work b9553125 2019-10-13 11:00:00 mopped /home/user/work e1a7b58b 2019-10-20 11:00:00 mopped /home/user/work 8f8018c0 2019-10-27 11:00:00 mopped /home/user/work 59403279 2019-11-03 11:00:00 mopped /home/user/work dfee9fb4 2019-11-10 11:00:00 mopped /home/user/work e1ae2f40 2019-11-17 11:00:00 mopped /home/user/work

12 snapshots
```

Then `forget --keep-daily 4` will keep the last four snapshots, for the last
four Sundays, and remove the other snapshots:

```console
$ rustic forget --keep-daily 4 --dry-run repository f00c6e2a opened successfully, password is correct Applying Policy: keep the last 4 daily snapshots keep 4 snapshots: ID Time Host Tags Reasons Paths

8f8018c0 2019-10-27 11:00:00 mopped daily snapshot /home/user/work 59403279 2019-11-03 11:00:00 mopped daily snapshot /home/user/work dfee9fb4 2019-11-10 11:00:00 mopped daily snapshot /home/user/work e1ae2f40 2019-11-17 11:00:00 mopped daily snapshot /home/user/work

4 snapshots

remove 8 snapshots: ID Time Host Tags Paths

0a1f9759 2019-09-01 11:00:00 mopped /home/user/work 46cfe4d5 2019-09-08 11:00:00 mopped /home/user/work f6b1f037 2019-09-15 11:00:00 mopped /home/user/work eb430a5d 2019-09-22 11:00:00 mopped /home/user/work 8cf1cb9a 2019-09-29 11:00:00 mopped /home/user/work 5d33b116 2019-10-06 11:00:00 mopped /home/user/work b9553125 2019-10-13 11:00:00 mopped /home/user/work e1a7b58b 2019-10-20 11:00:00 mopped /home/user/work

8 snapshots
```

The processed snapshots are evaluated against all `--keep-*` options but a
snapshot only need to match a single option to be kept (the results are ORed).
This means that the most recent snapshot on a Sunday would match both hourly,
daily and weekly `--keep-*` options, and possibly more depending on calendar.

For example, suppose you make one backup every day for 100 years. Then
`forget --keep-daily 7 --keep-weekly 5 --keep-monthly 12 --keep-yearly 75` would
keep the most recent 7 daily snapshots and 4 last-day-of-the-week ones (since
the 7 dailies already include 1 weekly). Additionally, 12 or 11
last-day-of-the-month snapshots will be kept (depending on whether one of them
ends up being the same as a daily or weekly). And finally 75 or 74
last-day-of-the-year snapshots are kept, depending on whether one of them ends
up being the same as an already kept snapshot. All other snapshots are removed.

You might want to maintain the same policy as in the example above, but have
irregular backups. For example, the 7 snapshots specified with `--keep-daily 7`
might be spread over a longer period. If what you want is to keep daily
snapshots for the last week, weekly for the last month, monthly for the last
year and yearly for the last 75 years, you can instead specify
`forget --keep-within-daily 7d --keep-within-weekly 1m --keep-within-monthly 1y --keep-within-yearly 75y`
(note that `1w` is not a recognized duration, so you will have to specify `7d`
instead).

For safety reasons, rustic refuses to act on an "empty" policy. For example, if
one were to specify `--keep-last 0` to forget *all* snapshots in the repository,
rustic will respond that no snapshots will be removed. To delete all snapshots,
use `--keep-last 1` and then finally remove the last snapshot manually (by
passing the ID to `forget`).
