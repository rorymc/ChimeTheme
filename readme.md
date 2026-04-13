

# The official unofficial guide to restoring sounds and creating a new theme.
## Fixing sounds on the Met.
For whatever reason, some packages seem to be missing. The Met team and the community are looking into why this is happening, but our friends on the Met Discord found a workaround that allows us to _install_ those packages and restore sounds.

> [!WARNING]
> The following mini guide requires you to SSH into your Meticulous and install libraries on it. If you're not comfortable doing this, I suggest waiting for an update with a fix.

This guide will not be covering how to SSH into the machine — it's not gatekeeping, it's just making sure there's at least some technical knowledge.

> [!CAUTION]
> Disclaimer: proceed at your own risk. This is not a fix endorsed by the Meticulous team, and while I'm sharing this information, I'm in no way liable if this bricks your machine.

Ok, with all the warnings and disclaimers out of the way, here we go. Windwalker et al. tinkered a bit and dug around in the logs and discovered that two packages were missing. We still don't know why — users report having sound and then one day not having it anymore, so those packages must've been there in the beginning — but that's neither here nor there.

The quickest fix is to just SSH into your Met and `apt-get install` them:
```bash
apt-get install gstreamer1.0-alsa gstreamer1.0-plugins-base-apps
```
The Met will go and fetch those packages from the repository and install them. Once successful, you should get your default chimes back.

## I want my own custom chimes
Ok, the easiest way to upload a theme is to just download this folder, drop your sound files in it and rename them to the following:
- heating_end.mp3 // for the chime that lets you know the Met is at temp
- event_start.mp3 // for the sound that lets you know the brew is starting
- event_finished.mp3 // for the sound that lets you know the Met has finished brewing

Once that's done, we're gonna need to upload it to the Met. It's a very simple command from your terminal — sorry, this is the only way I know how to do it.

> [!CAUTION]
> Never just run commands on the terminal from sources you don't trust or if you don't understand them.

First we'll zip up the folder. We do this from the command line since OS's sometimes tend to put hidden files in zip files. Think of this as putting everything into a neat little package before sending it over.
```bash
zip -r ChimeTheme.zip ChimeTheme/
```
This will create a zip file in the same directory you're in.

Now we push that to the Met with a quick curl command. This will upload the package we just made.
```bash
curl -X POST http://your-machine-api/api/v1/sounds/theme/upload -F "file=@ChimeTheme.zip"
```
You should get back a response that says: `Zip file uploaded and unpacked successfully.`

Almost there! Now we just need to tell the Met to actually use the theme we uploaded — think of this as telling it to go open the package and put it to use.
```bash
curl -X POST http://your-machine-api/api/v1/sounds/theme/set/ChimeTheme
```

## The sound pack
> [!IMPORTANT]
> One final note: these sounds are royalty free. If you choose to create your own theme, make sure to observe your local licensing and copyright laws when selecting your sounds.

All sounds came from [Pixabay](https://pixabay.com/sound-effects/search/notification/):
- [New Notification 09 from Universfield](https://pixabay.com/sound-effects/film-special-effects-new-notification-09-352705/)
- [New Notification 051 from Universfield](https://pixabay.com/sound-effects/technology-new-notification-051-494246/)
- [New Notification 036 from Universfield](https://pixabay.com/sound-effects/technology-new-notification-036-485897/)

# Appendix

## How it works

The machine starts with one sound theme called `default`, which is located in `/opt/meticulous-backend/sounds/default/` on disk.

If another custom sound theme is uploaded and selected as active, the Met will layer the active custom theme sound hooks, on top of the default theme.

Sounds can be .wav or .mp3 files.

## Details of uploading

If you want to upload a new sound theme, the name of the folder, under which all the config.json and sound files are located, will be the theme name.
* themename/
  * config.json  (which point to the sound files)
  * (each of the sound files)

So in the ChimeTheme.zip example above, "ChimeTheme" is the new sound theme name, and it was created by zipping up a folder called ChimeTheme/ which had the config.json and sound files in it.

## Calling the APIs

After you upload the ChimeTheme.zip file but before you call the "set" API:
* the meticulous will unpack the files into `/meticulous-user/sounds/ChimeTheme/...` -- before you upload your first custom theme, the `sounds` folder in that path will not exist.
* you can call http://your-machine-ip-or-name/api/v1/sounds/theme/get , and it will return back `default`

After you call the "set" API:
* this reloads the sound theme, and the "get" API will return back "ChimeTheme"

The writers of this guide are unsure what else the APIs do when uploading a sound theme, in terms of processing the sound theme and caching an internal config.  Hence, unless you can confirm via a code crawl that a direct scp is similar to the POST upload, it may be advisable to stick to the API for uploading instead of direct scp.

## Additional Details for Sound Hooks

The way themes work, they overlay on top of the 'default' theme.  You can check the config.json to see what is configured, and http://your-machine-ip-or-name/api/v1/sounds/list to understand the full list of hooks which can be configured for sounds.  At the writing of this document, these are the hooks:
* startup
* heating\_start
* heating\_end
* brewing\_start
* brewing\_end
* abort
* idle
* notification

If you want to test playing a specific sound hook for the active sound theme, you can call it like this:
* http://your-machine-ip-or-name/api/v1/sounds/play/heating_end

## Sound Level

Finally, if you want to adjust the level of the sounds down, it is currently set to 70% manually in /opt/meticulous-backend/sounds.py -- you can edit that manually and set it to a lower number, if you so desire.

By default, nano is installed as an editor.  You can also `apt install vim`, but it will take up about 41 MB.
