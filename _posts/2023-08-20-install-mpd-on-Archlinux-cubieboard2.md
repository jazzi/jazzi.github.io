---
layout: post
---

MPD music player daemon is kind of simple to install on ArchLinux, just type:

`sudo pacman -Syu mpd mympd`

MyMPD is a web-based client, pretty simple and elegant, that's my tea now.

MyMPD has a [recommmended MPD configuration](https://jcorporation.github.io/myMPD/additional-topics/recommended-mpd-configuration), I copy it to /etc/mpd.conf and do a bit adjustment, so the final configuration is as following:

```
[alarm@cubieboard2]$ cat /etc/mpd.conf 

# See: /usr/share/doc/mpd/mpdconf.example

music_directory "/mnt/hdd/Music"
#pid_file "/run/mpd/mpd.pid" 
#db_file "/var/lib/mpd/mpd.db"
state_file "/var/lib/mpd/mpdstate"
playlist_directory "/var/lib/mpd/playlists"

# Enable stickers - myMPD uses stickers for play statistics
sticker_file            "/var/lib/mpd/sticker.sql"

# Enable metadata. If set to none, you can only browse the filesystem
metadata_to_use         "AlbumArtist,Artist,Album,Title,Track,Disc,Genre,Name"
# Enable also the musicbrainz_* tags if you want integration with MusicBrainz and ListenBrainz
# musicbrainz_albumid is the fallback for musicbrainz_releasegroupid (MPD 0.24)
#metadata_to_use         "AlbumArtist,Artist,Album,Title,Track,Disc,Genre,Name,musicbrainz_artistid, musicbrainz_albumid, musicbrainz_albumartistid, musicbrainz_trackid, musicbrainz_releasetrackid"

# bind mpd to a unix socket
# Only socket connection to mpd enables some myMPD auto configuration features
bind_to_address         "/run/mpd/socket"

# Mounting is only possible with the simple database plugin and a cache_directory
database {
    plugin          "simple"
    path            "/var/lib/mpd/tag_cache"
    cache_directory "/var/lib/mpd/cache"
}

# Enable neighbor plugins
#neighbors {
#    plugin          "udisks"
#}
#neighbors {
#    plugin          "upnp"
#}

# Output for http stream - myMPD local playback
audio_output {
    type        "httpd"
    name        "HTTP Stream"
    encoder     "lame" #to support safari on ios
    port        "8000"
    bitrate     "128"
    format      "44100:16:1"
    always_on   "yes"
    tags        "yes"
}

# ALSA output configuration, check command 'aplay -L' and 'aplay -l' 
# for the right paremeters of "device"
#
audio_output {
	type	"alsa"
	name	"sun4icodec"
	device	"hw:0,0"
	mixer_device	"default"
	mixer_control	"FM" # could be "PCM", check command 'alsamixer'
	mixer_index	"0"
}

```

## No Sound and Volume Mixer Control greyed

In the beginning, I didn't set the **mixer_control**, thus impossible to get Volume Mixer Control work. Normally the value is "PCM" but Cubieboard2 is kind of special, we need to check it out by applying command `alsamixer` to see the right name.

If a DAC used, the value of "device" need to be changed.

## MyMPD Port

MPD client MyMPD has default http port 80, this could get trouble of warning as modern browsers all encourage https, so it's better to change this port "80" to else like "8080".

All the MyMPD configuration files locates in /var/lib/mpd/config/, check it by `sudo ls /var/lib/mpd/config/`, then change somewhere as followings:

```
http_port: 8080
ssl: false # no need of https for local net
```

Lastly open a browser and type http://192.168.1.10:8080
