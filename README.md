# Pull missing responses into Mastodon

This GitHub repository provides a GitHub action that runs every 10 mins, doing the following:

1. It can [pull remote replies into your instance](https://blog.thms.uk/2023/03/pull-missing-responses-into-mastodon?utm_source=github), using the Mastodon API. That part itself has two parts:
   1. It gets remote replies to posts that users on your instance have already replied to during the last `REPLY_INTERVAL_IN_HOURS` hours, and adds them to your own server.
   2. It gets remote replies to the last `HOME_TIMELINE_LENGTH` posts from your home timeline, and adds them to your own server.
2. It can also [backfill posts](https://blog.thms.uk/2023/03/backfill-recently-followed-accounts?utm_source=github) from the last `MAX_FOLLOWINGS` users that you have followed.
3. In the same way, it can also backfill posts form the last `MAX_FOLLOWERS` users that have followed you.

Each part can be disabled completely, and all of the values are configurable.

**Be aware, that this script may run for a long time, if these values are too high.** Experiment a bit with what works for you, by starting with fairly small numbers (maybe `HOME_TIMELINE_LENGTH = 200`, `REPLY_INTERVAL_IN_HOURS = 12`) and increase the numbers as you see fit.

For full context and discussion on why this is needed, read the following two blog posts: 

- The original announcement post: [Pull missing responses into Mastodon](https://blog.thms.uk/2023/03/pull-missing-responses-into-mastodon?utm_source=github)
- The announcement for v3.0.0: [Pull missing posts from recently followed accounts into Mastodon](https://blog.thms.uk/2023/03/backfill-recently-followed-accounts?utm_source=github)

## Setup

### 1) Get the required access token:

1. In Mastodon go to Preferences > Development > New Application
   1. give it a nice name
   2. enable `read:search`, `read:statuses` and `admin:read:accounts `
   3. Save
   4. Copy the value of `Your access token`

### 2) Configure and run the GitHub action

1. Fork this repository
2. Add your access token:
   1.  Go to Settings > Secrets and Variables > Actions
   2.  Click New Repository Secret
   3.  Supply the Name `ACCESS_TOKEN` and provide the Token generated above as Secret
3. Provide the required environment variables, to configure your Action:
   1. Go to Settings > Environments
   2. Click New Environment
   3. Provide the name `Mastodon`
   4. Add the following Environment Variables:
      1. For all parts of the script:
         - `MASTODON_SERVER` (required): The domain only of your mastodon server (without `https://` prefix) e.g. `mstdn.thms.uk`. 
      2. To pull in remote replies:
         - `HOME_TIMELINE_LENGTH` (optional): Look for replies to posts in the API-Key owner's home timeline, up to this many posts. (An integer number, e.g. `200`)
         - `REPLY_INTERVAL_IN_HOURS`: (optional)  Only look at posts that have received replies in this period. (An integer number, e.g. `24`)
      3. To backfill posts from your last followings (new in v3.0.0):
         - `MAX_FOLLOWINGS` (optional): How many of your last followings you want to backfill. (An integer number, e.g. `80`. Ensure you also provide `USER`).
         - `USER` (optional): The username of the user whose followings you want to pull in (e.g. `michael` for the user `@michael@thms.uk`).
      4. To backfill posts from your last followers (new in v3.0.1):
         - `MAX_FOLLOWERS` (optional):  How many of your last followers you want to backfill. (An integer number, e.g. `80`. Ensure you also provide `USER`).
         - `USER` (optional): The username of the user whose followers you want to pull in (e.g. `michael` for the user `@michael@thms.uk`).
4. Finally go to the Actions tab and enable the action. The action should now automatically run approximately once every 10 min. 

### 3) Run this script locally as a cron job

If you want to, you can of course also run this script locally as a cron job:

1. To get started, clone this repository. (If you'd rather not clone the full repository, you can simply download the `find_posts.py` file, but don't forget to create a directory called `artifacts` in the same directory: The script expects this directory to be present, and stores information about posts it has already pushed into your instance in that directory, to avoid pushing the same posts over and over again.)
2. Then simply run this script like so: `python find_posts.py --access-token=<TOKEN> --server=<SERVER>` etc. (run `python find_posts.py -h` to get a list of all options)

When setting up your cronjob, do make sure you are setting the interval long enough that two runs of the script don't overlap though! Running this script with overlapping will have unpleasant results ...

If you are running this script locally, my recommendation is to run it manually once, before turning on the cron job: The first run will be significantly slower than subsequent runs, and that will help you prevent overlapping during that first run.

## Acknowledgments

This script is mostly taken from [Abhinav Sarkar](https://notes.abhinavsarkar.net/2023/mastodon-context), with just some additions and alterations. Thank you Abhinav!
