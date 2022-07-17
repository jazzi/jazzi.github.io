
*Jellyfin is an Open Source and free media server that you can play on any machines, compare to Flex or Kodi, it requires no subscription yet has all the power.*

To install it on Debian, type the following commands:

* `curl -fsSL https://repo.jellyfin.org/debian/jellyfin_team.gpg.key | gpg --dearmor -o /etc/apt/trusted.gpg.d/debian-jellyfin.gpg` to import the GPG signing key
* `echo "deb [arch=$( dpkg --print-architecture )] https://repo.jellyfin.org/debian $( lsb_release -c -s ) main" | sudo tee /etc/apt/sources.list.d/jellyfin.list` to add a repository
* `sudo apt update; sudo apt install jellyfin-ffmpeg jellyfin`

More detailed install instructions are on the offical [Jellyfin documentation](https://jellyfin.org/docs/general/administration/installing.html).

## Troubleshooting

1. Forget Admin Password: in short, edit table `Users` in sqlite3 database by `sqlite3 /var/lib/jellyfin/data/jellyfin.db`

Another trick is to disable the setup wizard and create the administrator account again,how to do that? Easy, change the value `true` to `false` by edit file `/etc/jellyfin/system.xml` then locate the 4th line `<IsStartupWizardCompleted>true</IsStartupWizardCompleted>`, after that open web browser with http://ip-address:8096/
