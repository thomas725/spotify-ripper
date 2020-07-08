# spotify-ripper

`spotify-ripper` is a small ripper program for Spotify that rips Spotify URIs to audio files with metadata and cover art. By default `spotify-ripper` will encode to MP3 files, but includes the ability to rip to WAV, FLAC, Ogg Vorbis, Opus, AAC, and MP4/M4A.

**Note that stream ripping violates Spotify's ToS**

## Spotify Web API

To generate your Spotify Client ID and Secret, access the [developer dashboard page](https://developer.spotify.com/dashboard/applications) and create a new application. As a redirect URI, add localhost and a port, so that [Spotipy](https://spotipy.readthedocs.io/) can parse it automatically.

Make sure you export the web api credentials and redirect URI in your unix shell:

```
export SPOTIPY_CLIENT_ID='77aa1aa93dc0416397f22a7a9b4a815b'
export SPOTIPY_CLIENT_SECRET='0d79181c57ee412aaa770af257edf07a'
export SPOTIPY_REDIRECT_URI='http://localhost:8000/'
```

At the first attempt to use the ripper against a playlist, a browser window with a Spotify login will open. Login, and the ripper parses automatically the returning OAuth redirect.

A couple notes about Spotify's WebAPI token authentication:
- The token is stored in a file named `.profile-<username>`.
- The authentication token is stored where the script is executed from, so if you're in your home directory and execute a script thats in `/usr/bin` it will be stored in your home directory.
- If you are running this in a script or other form of automation, you'll have to manually authenticate once but after that as long as you always execute it from the same location you won't have to authenticate again.

### Deprecation of `libspotify`

As of May 2015 `libspotify` is officially deprecated by Spotify and is no longer actively maintained. As of Jan 2016 Spotify does no longer issue developer keys.

Spotify has published newer libraries intended for Android and iOS development, as well as web APIs to access track metadata and manage playlists. Though, for making apps with Spotify playback capabilities, on any other platform than Android and iOS, there is currently no alternative to libspotify.

## Features

-  Real-time VBR or CBR ripping from Spotify PCM stream
-  Writes ID3v2/metadata tags (including album covers)
-  Rips files into the following directory structure: ``artist/album/artist - song.mp3`` by default or optionally into a user-specified structure (see `Format String`_ section below)
-  Option to skip or overwrite existing files
-  Accepts tracks, playlists, albums, and artist URIs
-  Search for tracks using Spotify queries
-  Options for interactive login (no password in shell history) and to relogin using previous credentials
-  Option to remove tracks from playlist after successful ripping
-  Globally installs ripper script using pip
-  Use a config file to specify common command-line options
-  Helpful progress bar to gauge the time remaining until completion
-  Keep local files in sync with a Spotify playlist, m3u and wpl playlist file
-  Option to rip to ALAC, a loseless codec, instead of MP3 (requires extra ``avconv`` dependency)
-  Option to rip to FLAC, a loseless codec, instead of MP3 (requires extra ``flac`` dependency)
-  Option to rip to AIFF, a loseless codec, instead of MP3 (requires extra ``sox`` dependency)
-  Option to rip to Ogg Vorbis instead of MP3 (requires extra ``vorbis-tools`` dependency)
-  Option to rip to Opus instead of MP3 (requires extra ``opus-tools`` dependency)
-  Option to rip to AAC instead of MP3 (requires extra ``faac`` dependency)
-  Option to rip to MP4/M4A instead of MP3 (requires extra ``fdk-aac`` dependency)
-  Option to replace output filenames
-  Option to normalize output filenames to NFKD (see http://unicode.org/faq/normalization.html)

**Please note: Spotify’s highest quality setting is 320 kbps, so the benefit of ripping to a lossless format is to not double encode the audio data. It’s not possible to rip in true lossless quality.**

## Usage

Takes many command-line options:

```
usage: spotify-ripper [-h] [-S SETTINGS] [-a] [--aac] [--aiff] [--alac] [--all-artists] [--artist-album-type ARTIST_ALBUM_TYPE] [--artist-album-market ARTIST_ALBUM_MARKET] [-A]
                      [-b BITRATE] [-c] [--comp COMP] [--comment COMMENT] [--cover-file COVER_FILE] [--cover-file-and-embed COVER_FILE] [-d DIRECTORY] [--fail-log FAIL_LOG] [--flac]
                      [-f FORMAT] [--format-case {upper,lower,capitalize}] [--flat] [--flat-with-index] [-g {artist,album}] [--grouping GROUPING] [--id3-v23] [-k KEY] [-u USER]
                      [-p PASSWORD] [--large-cover-art] [-l] [-L LOG] [--pcm] [--mp4] [--normalize] [-na] [-o] [--opus] [--partial-check {none,weak,strict}]
                      [--play-token-resume RESUME_AFTER] [--playlist-m3u] [--playlist-wpl] [--playlist-sync] [--plus-pcm] [--plus-wav] [-q VBR] [-Q {160,320,96}] [--remove-offline-cache]
                      [--resume-after RESUME_AFTER] [-R REPLACE [REPLACE ...]] [-s] [--stereo-mode {j,s,f,d,m,l,r}] [--stop-after STOP_AFTER] [--timeout TIMEOUT] [-V] [--wav]
                      [--windows-safe] [--vorbis] [-r]
                      uri [uri ...]

Rips Spotify URIs to media files with tags and album covers

positional arguments:
  uri                   One or more Spotify URI(s) (either URI, a file of URIs or a search query)

optional arguments:
  -h, --help            show this help message and exit
  -S SETTINGS, --settings SETTINGS
                        Path to settings, config and temp files directory [Default=~/.spotify-ripper]
  -a, --ascii           Convert the file name and the metadata tags to ASCII encoding [Default=utf-8]
  --aac                 Rip songs to AAC format with FreeAAC instead of MP3
  --aiff                Rip songs to lossless AIFF encoding instead of MP3
  --alac                Rip songs to Apple Lossless format instead of MP3
  --all-artists         Store all artists, rather than just the main artist, in the track's metadata tag
  --artist-album-type ARTIST_ALBUM_TYPE
                        Only load albums of specified types when passing a Spotify artist URI [Default=album,single,ep,compilation,appears_on]
  --artist-album-market ARTIST_ALBUM_MARKET
                        Only load albums with the specified ISO2 country code when passing a Spotify artist URI. You may get duplicate albums if not set. [Default=any]
  -A, --ascii-path-only
                        Convert the file name (but not the metadata tags) to ASCII encoding [Default=utf-8]
  -b BITRATE, --bitrate BITRATE
                        CBR bitrate [Default=320]
  -c, --cbr             CBR encoding [Default=VBR]
  --comp COMP           compression complexity for FLAC and Opus [Default=Max]
  --comment COMMENT     Set comment metadata tag to all songs. Can include same tags as --format.
  --cover-file COVER_FILE
                        Save album cover image to file name (e.g "cover.jpg") [Default=embed]
  --cover-file-and-embed COVER_FILE
                        Same as --cover-file but embeds the cover image too
  -d DIRECTORY, --directory DIRECTORY
                        Base directory where ripped MP3s are saved [Default=cwd]
  --fail-log FAIL_LOG   Logs the list of track URIs that failed to rip
  --flac                Rip songs to lossless FLAC encoding instead of MP3
  -f FORMAT, --format FORMAT
                        Save songs using this path and filename structure (see README)
  --format-case {upper,lower,capitalize}
                        Convert all words of the file name to upper-case, lower-case, or capitalized
  --flat                Save all songs to a single directory (overrides --format option)
  --flat-with-index     Similar to --flat [-f] but includes the playlist index at the start of the song file
  -g {artist,album}, --genres {artist,album}
                        Attempt to retrieve genre information from Spotify's Web API [Default=skip]
  --grouping GROUPING   Set grouping metadata tag to all songs. Can include same tags as --format.
  --id3-v23             Store ID3 tags using version v2.3 [Default=v2.4]
  -k KEY, --key KEY     Path to Spotify application key file [Default=Settings Directory]
  -u USER, --user USER  Spotify username
  -p PASSWORD, --password PASSWORD
                        Spotify password [Default=ask interactively]
  --large-cover-art     Attempt to retrieve 640x640 cover art from Spotify's Web API [Default=300x300]
  -l, --last            Use last login credentials
  -L LOG, --log LOG     Log in a log-friendly format to a file (use - to log to stdout)
  --pcm                 Saves a .pcm file with the raw PCM data instead of MP3
  --mp4                 Rip songs to MP4/M4A format with Fraunhofer FDK AAC codec instead of MP3
  --normalize           Normalize volume levels of tracks
  -na, --normalized-ascii
                        Convert the file name to normalized ASCII with unicodedata.normalize (NFKD)
  -o, --overwrite       Overwrite existing MP3 files [Default=skip]
  --opus                Rip songs to Opus encoding instead of MP3
  --partial-check {none,weak,strict}
                        Check for and overwrite partially ripped files. "weak" will err on the side of not re-ripping the file if it is unsure, whereas "strict" will re-rip the file [Default=weak]
  --play-token-resume RESUME_AFTER
                        If the 'play token' is lost to a different device using the same Spotify account, the script will wait a speficied amount of time before restarting. This argument takes the same values as --resume-after [Default=abort]
  --playlist-m3u        create a m3u file when ripping a playlist
  --playlist-wpl        create a wpl file when ripping a playlist
  --playlist-sync       Sync playlist songs (rename and remove old songs)
  --plus-pcm            Saves a .pcm file in addition to the encoded file (e.g. mp3)
  --plus-wav            Saves a .wav file in addition to the encoded file (e.g. mp3)
  -q VBR, --vbr VBR     VBR quality setting or target bitrate for Opus [Default=0]
  -Q {160,320,96}, --quality {160,320,96}
                        Spotify stream bitrate preference [Default=320]
  --remove-offline-cache
                        Remove libspotify's offline cache directory after the ripis complete to save disk space
  --resume-after RESUME_AFTER
                        Resumes script after a certain amount of time has passed after stopping (e.g. 1h30m). Alternatively, accepts a specific time in 24hr format to start after (e.g 03:30, 16:15). Requires --stop-after option to be set
  -R REPLACE [REPLACE ...], --replace REPLACE [REPLACE ...]
                        pattern to replace the output filename separated by "/". The following example replaces all spaces with "_" and all "-" with ".":    spotify-ripper --replace " /_" "\-/." uri
  -s, --strip-colors    Strip coloring from output [Default=colors]
  --stereo-mode {j,s,f,d,m,l,r}
                        Advanced stereo settings for Lame MP3 encoder only
  --stop-after STOP_AFTER
                        Stops script after a certain amount of time has passed (e.g. 1h30m). Alternatively, accepts a specific time in 24hr format to stop after (e.g 03:30, 16:15)
  --timeout TIMEOUT     Override the PySpotify timeout value in seconds (Default=10 seconds)
  -V, --version         show program's version number and exit
  --wav                 Rip songs to uncompressed WAV file instead of MP3
  --windows-safe        Make filename safe for Windows file system (truncate filename to 255 characters)
  --vorbis              Rip songs to Ogg Vorbis encoding instead of MP3
  -r, --remove-from-playlist
                        Delete tracks from playlist after successful ripping [Default=no]

Example usage:
    rip a single file: spotify-ripper -u <username> spotify:track:52xaypL0Kjzk0ngwv3oBPR
    rip entire playlist: spotify-ripper -u <username> spotify:user:<username>:playlist:4vkGNcsS8lRXj4q945NIA4
    rip a list of URIs: spotify-ripper -u <username> -p <password> list_of_uris.txt
    rip a list of URIs using last login: spotify-ripper -l list_of_uris.txt
    rip tracks from Spotify's charts: spotify-ripper -l spotify:charts:regional:global:weekly:latest
    search for tracks to rip: spotify-ripper -l -Q 160 -o "album:Rumours track:'the chain'"
```

### Facebook Login

`spotify-ripper` will work with your regular Facebook login/password if you setup your Spotify account to login using your Facebook credentials. Otherwise, use your Spotify login/password.

### Config File

For options that you want set on every run, you can use a config file named `config.ini` in the settings folder (defaults to `~/.spotify-ripper`).  The options in the config file use the same name as the command line options with the exception that dashes are translated to `snake_case`.  Any option specified in the command line will overwrite any setting in the config file.  Please put all options under a `[main]` section.

Here is an example config file:

```
[main]
ascii = True
format = {album_artist}/{album}/{artist} - {track_name}.{ext}
quality = 160
vorbis = True
last = True
```

### Format String

The format string dictates how `spotify-ripper` will organize your ripped files. This is controlled through the `-f | --format` option. The string should include the format of the file name and optionally a directory structure. If you do not include a format string, the default format will be used: `{album_artist}/{album}/{artist} - {track_name}.{ext}`.

The `--flat` option is shorthand for using the format string: `{artist} - {track_name}.{ext}`, and the `--flat-with-index` option is shorthand for using the format string: `{idx:3} - {artist} - {track_name}.{ext}`.  The use of these shorthand options will override any `--format` string option given.

Your format string can include the following variables names, which are case-sensitive and wrapped in curly braces, if you want your file/path name to be overwritten with Spotify metadata.

### Format String Variables

Names and Aliases | Description |
----------------- | ----------- |
`{track_artist}`, `{artist}` | The track's artist |
`{track_artists}`, `{artists}`  | Similar to `{track_artist}` but will join multiple artists with a comma (e.g. "artist 1, artist 2") |
`{album_artist}` | When passing an album, the album's artist (e.g. "Various Artists"). If no album artist exists, the track artist is used instead |
`{album_artists_web}` | Similar to `{album_artist}` but retrieves artist information from Spotify's Web API. Unlike ``{album_artist}``, multiple album artists can be retrieved and will be joined with a comma (e.g. "artist 1, artist 2") |
`{album}` | Album name |
`{track_name}`, `{track}` | Track name |
`{year}` | Release year of the album |
`{ext}`, `{extension}` | Filename extension (i.e. "mp3", "ogg", "flac", ...) |
`{idx}`, `{index}` | Playlist index |
`{track_num}`, `{track_idx}`, `{track_index}` | The track number of the disc |
`{disc_num}`, `{disc_idx}`, `{disc_index}` | The disc number of the album |
`{smart_track_num}`, `{smart_track_idx}` | For a multi-disc album, `{smart_track_num}` will return a number combining the disc and `{smart_track_index}` the track number. e.g. for disc 2, track 4 it will return "204". For a single disc album, it will return the track num. |
`{playlist}`, `{playlist_name}` | Name of playlist if passed a playlist uri, otherwise "No Playlist" |
`{playlist_owner}`, `{playlist_user}`, `{playlist_username}` | User name of playlist's owner if passed a playlist uri, otherwise "No Playlist Owner" |
`{playlist_track_add_time}`, `{track_add_time}` | When the track was added to the playlist |
`{playlist_track_add_user}`, `{track_add_user}` | The user that added the track to the playlist |
`{user}`, `{username}` | Spotify username of logged-in user |
`{feat_artists}`, `{featuring_artists}` | Featuring artists join by commas (see Prefix String section below) |
`{copyright}` | Album copyright message |
`{label}`, `{copyright_holder}` | Album copyright message with the year removed at the start of the string if it exists |
`{track_uri}`, `{uri}` | Spotify track uri |

Any substring in the format string that does not match a variable above will be passed through to the file/path name unchanged.

### Zero-Filled Padding

Format variables that represent an index can be padded with zeros to a user-specified length.  For example, `{idx:3}` will produce the following output: 001, 002, 003, etc.  If no number is provided, no zero-filled padding will occur (e.g. 8, 9, 10, 11, ...). The variables that accept this option include `{idx}`, `{track_num}`, `{disc_num}`, `{smart_track_num}` and their aliases.

### Prefix String

Format variable `feat_artists` takes a prefix string to be prepended before the output. For example, `{feat_artists:featuring}` will produce the follow output `featuing Bruno Mars`.  If there are no featuring artists, the prefix string (and any preceding spaces) will not be included.

### Playlist Sync Option

By default, other than checking for an overwrite, `spotify-ripper` will not keep track of local files once they are ripped from Spotify. However, if you use the `--playlist-sync` option when passing a playlist URI, `spotify-ripper` will store a json file in your settings directory that keeps track of location of your ripped files for that playlist.

If at a later time, the playlist is changed on Spotify (i.e. songs reordered, removed or added), `spotify-ripper` will try to keep your local files "in sync" the playlist if you rerun the same command.  For example, if your format string is `{index} {artist} - {track_name}.{ext}`, it will rename is existing files so the index is correct. Note that with option set, `spotify-ripper` will delete a song that was previously on the playlist, but was removed but still exists on your local machine.  It does not affect files outside of the playlist and has no effect on non-playlist URIs.

If you want to redownload a playlist (for example with improved quality), you either need to remove the song files from your local or use the `--overwrite` option.

## Installation

### Prerequisites

-  [libspotify](https://developer.spotify.com/technologies/libspotify)
-  [pyspotify](https://github.com/mopidy/pyspotify)
-  [spotipy](https://github.com/plamere/spotipy)
-  A Spotify binary app key [spotify\_appkey.key](https://devaccount.spotify.com/my-account/keys/)
-  [lame](http://lame.sourceforge.net)
-  [mutagen](https://mutagen.readthedocs.org/en/latest/)
-  [colorama](https://pypi.python.org/pypi/colorama)
-  (optional) [flac](https://xiph.org/flac/index.html)
-  (optional) [opus-tools](http://www.opus-codec.org/downloads/)
-  (optional) [vorbis-tools](http://downloads.xiph.org/releases/vorbis/)
-  (optional) [faac](http://www.audiocoding.com/downloads.html)
-  (optional) [fdkaac](https://github.com/nu774/fdkaac)
-  (optional) [sox](http://sox.sourceforge.net)

### Mac OS X

Recommend approach uses [homebrew](http://brew.sh/) and [pyenv](https://github.com/yyuu/pyenv).

To install `pyenv` using homebrew:

```
$ brew update
$ brew install pyenv
$ eval "$(pyenv init -)"
$ echo 'if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi' >> ~/.bash_profile
$ pyenv install 3.8.3
$ pyenv global 3.8.3
$ python -V
```

To install `spotify-ripper` once `pyenv` is setup:

```
$ brew install homebrew/binary/libspotify
$ sudo ln -s /usr/local/opt/libspotify/lib/libspotify.12.1.51.dylib /usr/local/opt/libspotify/lib/libspotify
$ brew install lame
$ pip install git+https://github.com/scaronni/spotify-ripper
$ pyenv rehash
```

Download an application key file `spotify_appkey.key` from `https://devaccount.spotify.com/my-account/keys/` (requires a Spotify Premium Account) and move the file to the `~/.spotify-ripper` directory (or use the `-k | --key` option).

### Ubuntu/Debian

Recommend approach uses the system Python.

```
$ sudo apt-get install lame build-essential libffi-dev
$ wget https://developer.spotify.com/download/libspotify/libspotify-12.1.51-Linux-x86_64-release.tar.gz
$ tar xvf libspotify-12.1.51-Linux-x86_64-release.tar.gz
$ cd libspotify-12.1.51-Linux-x86_64-release/
$ sudo make install prefix=/usr/local
$ pip3 install --user --upgrade git+https://github.com/scaronni/spotify-ripper
```

Download an application key file `spotify_appkey.key` from `https://devaccount.spotify.com/my-account/keys/` (requires a Spotify Premium Account) and move the file to the `~/.spotify-ripper` directory (or use the `-k | --key` option).

## Optional Encoding Formats

In addition to MP3 encoding, `spotify-ripper` supports encoding to FLAC, AAC, MP4/M4A, Ogg Vorbis and Opus. However, additional encoding tools need to be installed for each codec you wish to use.

### Mac OS X

```
$ brew install flac
$ brew install libav
$ brew install faac
$ brew install fdk-aac-encoder
$ brew install vorbis-tools
$ brew install opus-tools
$ brew install sox
```

### Ubuntu/Debian

```
$ sudo apt-get install flac
$ sudo apt-get install libav-tools
$ sudo apt-get install faac
$ sudo apt-get install libfdk-aac-dev automake autoconf
$ wget https://github.com/nu774/fdkaac/archive/v0.6.2.tar.gz
$ tar xvf v0.6.2.tar.gz
$ cd fdkaac-0.6.2
$ autoreconf -i
$ ./configure
$ sudo make install
$ sudo apt-get install vorbis-tools
$ sudo apt-get install opus-tools
$ sudo apt-get install install sox
```

## Upgrade

Use `pip3` to upgrade to the latest version.

```
$ pip3 install --user --upgrade git+https://github.com/scaronni/spotify-ripper
```

## License

[MIT License](http://en.wikipedia.org/wiki/MIT_License)
