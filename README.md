#mutt-vid
Inspired by [Virtual Identity](https://www.absorb.it/virtual-id) for the Thunderbird email client, mutt-vid is a similar utility for mutt. It is useful for managing multiple sender accounts, allowing users to automatically associate an arbitrary sender account with a recipient. It scans sent emails for the most recent "From" details associated with each recipient, saving them in a file for mutt to source. The next time you email this recipient, mutt will automatically invoke a `send-hook` with the same email address and real name that you used previously.

##Usage
###Create database with `mutt-vid`
Run mutt-vid with the location of the directories containing your sent mail (in maildir format) as parameters. For example, `mutt-vid ~/.mail/*/*Sent*/cur`. The first run may take a while. (It takes about 10 minutes for me to go through 25 000 emails on an SSD in an aging laptop.)

mutt-vid updates the database for emails that are newer than the one previously saved. Hence, you only need to run mutt-vid on directories that have been updated since the last run. This makes subsequent runs much quicker if you archive your sent email into separate folders (e.g. by year).

**Options**
* **Database location** (`-d PATH`): define the location of the mutt-vid database. By default, this is `~/.mutt/sources/mutt-vid.db`.
* **Backup database** (`-b`): backup the current database (if it exists) to `*.prev`.
* **New database** (`-n`): create a new database, instead of modifying entries in the current database. Either delete or backup the current database depending on `-b`.
* **Help** (`-h`): display a brief help, then exit.

###Source in `mutt`
Source the database in your `muttrc` (e.g. with `source ~/.mutt/sources/mutt-vid.db`).

###Multiple default accounts in mutt
You may use this database with multiple default accounts. This allows you to set a default sender account depending on the current mailbox. For example, if you were in your work account's mailbox, email to a new recipient would be sent from your work account; if you were in your personal account, it would use your personal address. You can achieve this by changing the default with a `send-hook`, then sourcing the mutt-vid database again. Re-sourcing a large database is near instantaneous in my experience. An example follows.

`~/.mutt/muttrc`:

    set my_mutt-vid-db = ~/.mutt/sources/mutt-vid.db
    folder-hook personal-folder source ~/.mutt/sources/account_personal
    folder-hook work-folder source ~/.mutt/sources/account_work

`~/.mutt/sources/account_personal`:

    set my_from = 'foo@home.com'
    source ~/.mutt/sources/post-account_switch

`~/.mutt/sources/account_work`:

    set my_from = 'foo@work.com'
    source ~/.mutt/sources/post-account_switch

`~/.mutt/sources/post-account_switch`:

    send-hook . "set from=$my_from"
    source ${my_mutt-vid-db}
