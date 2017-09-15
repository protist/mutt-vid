# mutt-vid
Inspired by [Virtual Identity](https://www.absorb.it/virtual-id) for the Thunderbird email client, mutt-vid is a similar utility for mutt. It is useful for managing multiple sender accounts, allowing users to automatically associate an arbitrary sender account with a recipient. It scans sent emails for the most recent "From" details associated with each recipient, saving them in a file for mutt to source. The next time you email this recipient, mutt will automatically invoke a `send-hook` with the same email address and real name that you used previously.

## Usage
### Create database with `mutt-vid`
Run mutt-vid with the location of the directories containing your sent mail (in maildir format) as parameters. For example, `mutt-vid ~/.mail/*/*Sent*/cur`. The first run may take a while. (It takes about 15 minutes for me to go through 26 000 emails on an SSD in an aging laptop.)

mutt-vid updates the database for emails that are newer than the one previously saved. Hence, you only need to run mutt-vid on directories that have been updated since the last run. This makes subsequent runs much quicker if you archive your sent email into separate folders (e.g. by year).

**Options**
* **Database location** (`-d PATH`): define the location of the mutt-vid database. By default, this is `~/.mutt/sources/mutt-vid.db`.
* **Backup database** (`-b`): backup the current database (if it exists) to `*.prev`.
* **New database** (`-n`): create a new database, instead of modifying entries in the current database. Either delete or backup the current database depending on `-b`.
* **Help** (`-h`): display a brief help, then exit.

### Source in mutt
Source the database in your `muttrc` (e.g. with `source ~/.mutt/sources/mutt-vid.db`). In my experience, sourcing even a 2000-line file has a near-zero effect on mutt's performance.

To use a default sender account for new recipients, include the following line before sourcing the `mutt-vid` database, replacing `foo@bar.com` with your email address.

    send-hook . "set from=foo@bar.com"

### Usage in mutt
Mutt's `send-hook`s act *before* editing the email in the editor. Hence, when composing a new email, you should immediately enter the recipient *within* mutt. When mutt launches your editor, the `From:` field should be pre-filled with the associated sender address.

## Limitations and workarounds
* Mutt doesn't explicitly deal with conflicting sender accounts when composing to multiple recipients.
* Hence, mutt-vid only scrapes details for recipients that are explicitly in the `To:` field. It ignores recipients that are in the `Cc:` or `Bcc:` fields.
* There is currently no way of ignoring or overriding details for a particular recipient. If a rule is manually added to the database, with a date in the far future as a comment (see previous entries for format), then it will never be replaced.
