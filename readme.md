# OpenTTD in Docker
__An image brought to you by /r/openttd__

Built from OpenTTD source to provide the leanest, meanest image you'll come across for putting trainsets in containers.


## Using this Container
```
docker run -d -p 3979:3979/tcp -p 3979:3979/udp redditopenttd/openttd:latest
```

The container is set by default to start a fresh game every time you restart the container. You can, however, change this behaviour with the `loadgame` envvar:
```
-e "loadgame={false|last-autosave|exit|(savename)}"
```
where:
* false: standard behaviour, just start a new game
* last-autosave: load the last chronological autosave
* exit: try to load autosave/exit.sav, otherwise default to a new game
* (savename): full name of a save file in config/saves

You'll probably want stuff to be persistant between container rebuilds, so we've got the `/config` volume for exactly that purpose.

```
-v /home/{username}/.openttd:/config:rw
```
**Heads up:** If we can't find an `openttd.cfg` in `/config`, we'll attempt to ask OpenTTD to start a new configuration directory there. We strongly recommend that if you're starting fresh, you stop the container and configure openttd.cfg as per [the wiki](https://wiki.openttd.org/Openttd.cfg).

If you don't want the entire `.openttd` directory to be copied to your local FS statically, you may want to consider mounting files / directories directly like so:

```
-v /home/{username}/.openttd/openttd.cfg:/config/openttd.cfg:ro
-v /home/{username}/.openttd/save/:/config/save:rw
```
The easiest way to play with NewGRF's is to first download and configure them how you want on a local machine with a GUI. Then in the config/ directory copy the folder from local machine named content_downloaded to the server. Next update the openttd.cfg file from your local machine, this is to ensure that when you create a new server your NewGRF settings will be copied across.

## An example command to start a server
```
docker run -it -p 3979:3979/tcp -p 3979:3979/udp -v /home/{username}/.openttd:/config:rw -e "loadgame=game.sav" redditopenttd/openttd:latest
```
This will start a server with the console accessible due to ```-it``` in the command line, to run in the background use ```-d```.

## Tags
We'll automatically build a new tag every time a new beta or release candidate is released. If you'd like nightlies as well, please contact us, and I'll work it into our build scripts.

* `stable` tracks the latest stable release of OpenTTD.
* `rc` tracks the latest release candidate of OpenTTD, falling back to the latest stable if it's newer.
* `beta` tracks the latest beta release of OpenTTD, falling back to, you guessed it, the latest release candidate or stable if a newer one is available.
* `latest` currently tracks `stable`, but this may change in future to track nightly releases.

## Alpine, OpenTTD, and You

This repo contains a dockerfile for building against Alpine Linux. However, if you try to run a server with openttd compiled on alpine, you’ll get a segmentation fault in saveload as soon as a client tries to connect. From what I’m aware, this is due to a quirk in musl-libc, so at present we’re stuck with using Debian (attempts have been made to shoehorn glibc into alpine with no success). If you succeed at getting OpenTTD to run and serve players with an alpine-derived image, please let us know!
