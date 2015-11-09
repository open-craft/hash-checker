OpenCraft Hash Checker
======================

Objective
---------

Remind developers to update the hash in a requirements file after committing changes to the required
repository.

Design goals
------------

* Keep it simple.  We want to get this running quickly.  The feature set should be just the bare
  minimum to send reminders if we forget to update a hash.  We will do without a web interface for
  now and simply run an hourly cron job instead.

* Keep it generic.  Don't make the tool specific to edx-platform or OpenCraft.  It may come in handy
  in other contexts as well.

* Try hard to avoid false positives.  Reminder noise would result in genuine reminders being
  ignored.

Reminder Configuration
----------------------

Reminders are configured in a YAML configuration file with this format:

```yaml
# Map of github user names to reminder email address.
users:
  smarnach: sven@opencraft.com
  antoviaque: xavier@opencraft.com

# A list of email addresses all reminders are cc'ed to.
reminder_cc:
  - xavier+hash-checker@opencraft.com

# How often to send reminders for the same PR; interval in seconds.
reminder_interval: 86400   # 1 day

# The hashes to check.
requirements_files:
  - label: "edx/edx-platform - edx-private.txt"
    url: https://raw.githubusercontent.com/edx/edx-platform/master/requirements/edx/edx-private.txt
    requirements:
      - repo: open-craft/problem-builder
        branch: edx-release
  - label: "edx/edx-platform - github.txt"
    url: https://raw.githubusercontent.com/edx/edx-platform/master/requirements/edx/github.txt
    requirements:
      - repo: edx/xblock-utils
        branch: edx-release
      - repo: open-craft/xblock-poll
        branch: edx-release
```

Operation
---------

Pseudo-code of a single checker run:

    for all requirements files:
        download requirements file
        for all requirements:
            configured_hash = hash from requirements file
            latest_hash = latest hash on branch
            retrieve commit range configured_hash..latest_hash
            for all pull request merges in commit range:
                if PR creator or author of merge commit in users:
                   last_reminder = timestamp the last reminder was sent for this PR
                   if time since last_reminder longer than reminder_interval
                      send reminder to PR creator, author of merge commit  and reminder_cc addresses

The requirements file is downloaded from the configured URL (they will generally come from
raw.githubusercontent.com).  Hash ranges, commit information and PR information is retrieved via the
Github API.

Command-line parameters
-----------------------

* `--reminder-config` -- The YAML file in the configuration format given above.

* `--log-file`

* `--timestamp-db` -- sqlite database to store the last reminder time for each PR

Possible features for later versions
------------------------------------

* Include a link to the commit range between the currently configured hash and the latest hash on
  the branch in the email

* Include a link to silence reminders for a particular PR. (Requires to turn the tool into a web
  app.  The webhook could simply set the stored timestamp for the PR to a time in the future.)

* Include a link to a webhook that updates the hash.

* A full web interface to check the current state, silence reminders and update hashes.

* Load the reminder configuration from a URL instead of a local file.  This would facilitate to
  always use the latest configuration from a Github repository.
