# mpv-coverart

This script is for automatically loading external cover art files into mpv media player as additional video tracks.

## Valid Cover Art
The script will automatically search the various scan locations for valid cover art. By default this will be all image files with the specified file names; names and image extensions can be modified through the config file. By setting the list of names to an empty string any image file in the scan locations will be loaded, by setting the list of image extensions to an empty string any file with the right name will be loaded. Naturally, it's a bad idea to disable both filters.


## Scan Locations
By default the script will search the directory of the currently played file for valid cover art. However, if the file system is unnavailable (when playing a network file), the script will then also search the current playlist, but still require that the file has the same directory. Both file system scanning and playlist scanning can be disabled in the config file.

Through config options scanning of the parent folder for the current file can also be enabled. The requirements that the file system must be unavailable for the playlist to be scanned, and the subsequent directory requirements, can also be disabled.


## Scan Conditions
By default the script only searches for cover art if the current file is detected to be an audio file. An audio file is a file with audio streams and no video streams above 1 fps. The script can be changed to always scan for cover art in the options. 

The script will detect when a file in the same directory is loaded and will reuse the same cover art without rescanning the folder. Additionally, if `--audio-display=no` is set then this script will not search for cover art at all. All options that can affect when files get scanned are documented in coverart.conf.

You can also force the script to search for and load cover art using `script-message load-coverart`. This will bypass most scanning restrictions, including all of those mentioned above.


## Misc Behaviour
If `--vid=no` is set then this
script will load external cover art, but will not select them by default. Any other value for vid will only
properly select internal cover art, though you might get lucky and have it select the right track by chance.


## Preload Mode
This is an experimental change to the way the script loads coverart, it loads cover art synchronously by hooking into the player's preloading phase
(after file is loaded, but before playback start or track selection). This creates more seamless coverart loading, and has better compatibility
with mpv's default track-loading behaviour. However, because the player has to wait for this script to finish processing before starting playback,
it could potentially lead to the player freezing if something slows down the script. For this reason the setting is currently disabled by default
while I wait and see if anyone reports any bugs, but I encourage everyone to try it out.

What this means in practice is the following:
* mpv player will not start playback until all cover art is loaded
* this means that on slow file/network systems playback may be delayed
* `track added` messages are not printed to the console
* the `--vid=n` property is supported since mpv doesn't attempt to select `n` until after covers are loaded
* watch-later saves will remember which cover the video was on
* no more awkward switch from the black no-video screen to cover art at the start of every file (force-window=yes)
* the window doesn't close and reopen in-between files when playing from the terminal (force-window=no)
* external cover art will be loaded by default instead of embedded images (mpv behaviour, can be changed with an option)
* may provide better compatibility with some other scripts and options

Preload mode can be enabled by setting the script opt: `preload=yes`


## Configuration
Look at [coverart.conf](coverart.conf) for the full list of options. This config file can be placed into the script-opts folder inside the mpv config directory to be loaded automatically.


## Other Functionality
I've thrown a few other small features into the script that may be useful.

None of these are enabled by default.

### Placeholder
This script supports loading a placeholder image when cover art is not found. This can be useful for people who don't like their music running in the terminal, but also dislike the default black box that mpv player shows.

### Skip Coverart
When this is enabled the script will automatically skip any valid cover art files in the playlist, this can be very useful when loading a whole folder directly.

### Icy Directory
If this option is set to a valid directory then when an icecast stream is played this script will search for cover art inside that directory. Only files with names matching the `icy-name` metadata of the stream will be loaded, the usual names list is ignored. Due to how icy metadata is handled these covers cannot be loaded during preload, they will automatically be loaded once mpv detects that `icy-name` has changed.

### Decode URLs
When this is enabled mpv will attempt to undo URL percent encoding for the specified protocols. This is probably only useful for the ftp protocol, for which ffmpeg does not support percent encoding.
