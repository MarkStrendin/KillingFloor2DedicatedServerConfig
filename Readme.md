# Table of Contents
- [Resources](#resources)
- [Ports to forward to your server](#ports-to-forward-to-your-server)
- [Web admin](#web-admin)
- [Installing KF2 dedicated server](#installing-kf2-dedicated-server)
- [Updating KF2 dedicated server](#updating-kf2-dedicated-server)
- [Running the dedicated server](#running-the-dedicated-server)
- [Stopping the dedicated server](#stopping-the-dedicated-server)
- [Tips](#tips)
- [How to install custom maps from Steam Workshop](#how-to-install-custom-maps-from-steam-workshop)
  * [Adding custom maps to the map rotation using web admin](#adding-custom-maps-to-the-map-rotation-using-web-admin)
- [Preventing your server from being used by the Server Takeover feature](#preventing-your-server-from-being-used-by-the-server-takeover-feature)
- [Changing difficulty in Web Admin](#changing-difficulty-in-web-admin)
- [Enabling friendly fire](#Enabling-friendly-fire)

# Resources
The official wiki page for setting up a dedicated server is a great resource:
https://wiki.killingfloor2.com/index.php?title=Dedicated_Server_(Killing_Floor_2)

Article on ports that steam uses if you are having troubles
https://support.steampowered.com/kb_article.php?ref=8571-GLVN-8711

Killing Floor 2 dedicated server is steam app ID `232130`.

# Ports to forward to your server
If you run this behind a router, you need to forward some ports to your server in order for people outside your network to join.

| Port Number or range | Protocol | Notes                             |
|----------------------|----------|-----------------------------------|
| 7777                 | UDP      | Port the game actually listens on |
| 27015-27030          | UDP      | Needed for steam integration      |
| 20560                | UDP      | Needed for steam integration      |
| 3478-3480            | Both     | To allow steamworks map downloads |


You probably _shouldn't_ forward the web admin ports outside your network, for security reasons.

# Installing KF2 dedicated server
 1. Install SteamCMD on your server - https://developer.valvesoftware.com/wiki/SteamCMD
 2. Logon anonymously using the command `logon anonymous`
 3. Set an install directory appropriate for your server with the command `force_install_dir E:\killingfloor2\`
 4. Install the dedicated server using the command `app_update 232130 validate`. This will take a while, depending on your internet connection.
 5. Create a custom "server start" batch file which won't get overwritten every time the server updates. This can be as easy as copying "KF2Server.bat" and naming it something else. An example file is included in this repo.
 6. Launch the game once, so it creates the necesary config files. 
 7. Give it a minute or two and then close the game by pressing __CTRL+C__ on the server console.
 8. In the `/Config/KFWeb.ini` file, find the `bEnabled=false` line and change this to `bEnabled=true`.

On your server, make sure the above ports are allowed through your firewall, in addition to the web admin port `8080`. 
Make sure that it can communicate outbound on all of those ports as well, as well as port `123` for network time (for weekly outbreaks). Most systems allow outbound traffic by default, so you may not need to worry about this.

You _could_ disable your local firewall on your game server, but I don't recommend this for security reasons.

## First time configuration

### config/PCServer-KFGame.ini

Find the following section and change the server name so you can find it
```
[Engine.GameReplicationInfo]
ServerName=Killing Floor 2 Server
ShortName=KFServer
```

# Updating KF2 dedicated server
You update it the same way you installed it.
 1. Run SteamCMD on your server
 2. Logon anonymously using the command `logon anonymous`
 3. It won't remember where it's installed, so tell it where you've installed it using the command `force_install_dir E:\killingfloor2\`
 4. Update using the command `app_update 232130 validate`. This will take a while, depending on your internet connection.

# Web admin
Web Admin is enabled in the `Config/KFWeb.ini` file.
The username is `administrator`. The password is the admin password, either set in the `PCServer-KFGame.ini` file, or in your `KFServer.bat` file (the bat file overrides the ini file)
By default it is configured to run on port 8080. Access it via a web browser on your local network by going to your server's IP address, port 8080, like this: https://10.0.0.104:8080

# Running the dedicated server
Double click the batch file `KF2Server.bat`, or ideally make your own (as I've done in this repo) and run that instead. Mine is called `StartServer.bat`.

# Stopping the dedicated server
Click the server console and push __CTRL+C__, and it will shut it down.

# Tips
* When the server is updated, it will overwrite the `KF2Server.bat` file, and may overwrite your admin password. Make a seperate batch file to start the server with with either your own password or no password (it will default to the one in the config file `PCServer-KFGame.ini`).

# How to install custom maps from Steam Workshop

1. Find the map on steam workshop. Many people don't know how to tag their maps properly on Steam, so there may be maps that aren't tagged as "map".
2. Get the URL for the page. In steam, right click and select __Copy Page URL__.
3. Paste into notepad and extract the steam ID from the URL. For example, the ID for `https://steamcommunity.com/sharedfiles/filedetails/?id=2261700057&searchtext=` would be `2261700057`.
4. If your KF2 dedicated server program is running, you __must__ stop it or it will overwrite your changes when the game shuts down next.
5. If this is the first map you are adding, add a section to the bottom of `PCServer-KFEngine.ini` called `[OnlineSubsystemSteamworks.KFWorkshopSteamworks]`.
6. Subscribe the server to your map by adding a line underneath `[OnlineSubsystemSteamworks.KFWorkshopSteamworks]` that reads `ServerSubscribedWorkshopItems=2328985104 ; KF-WestSideWinter`, replacing the ID number with the one from your link from step 3. Optionally add a comment so you know which ID is which, in case you need to remove this map later.
7. Start your server, and allow it to download the map. The server console will tell you that it's checking subscriptions, but it won't tell you that it's finished downloading them.
8. Give your server a good 15-20 minutes to download all the maps, then __shut it down again__.

It will download the maps to `<kf2 server install location>/KFGame/Cache/<steam ID>/`.
Now, find the exact name of the actual map file (usually its a `.kfm` file), and copy the filename. The next steps are case sensitive.

Then, for each map you are adding, add a section like this to your `PCServer-KFGame.ini` file. 

```
[KF-WestSideWinter KFMapSummary]
MapName=KF-WestSideWinter
ScreenshotPathName=UI_MapPreview_TEX.UI_MapPreview_Placeholder
```

The name of the map (`KF-WestSideWinter` in the above example) must exactly match the name of the file.

Now, start your server up, and your maps should show up in the available maps in web admin. This _will not_ automatically add them to the map rotation.

## Adding custom maps to the map rotation using web admin

 1. Log into web admin in a web browser
 2. Under _Settings_ go to _Map Cycles_
 3. Move your custom maps to the right side (or just push `Add Missing`).
 
__NOTE:__ The map `KF-Default` is not an actual map - it's an internal part of the game that you should ignore. Do not add this map to your map cycle, it will probably crash any player trying to load it.

# Preventing your server from being used by the Server Takeover feature
The "Server Takeover" feature allows other random people on the internet to essentially take over your server with their own game/maps/password, if nobody is currently using it. If you would rather this not happen to your server, the option to configure this can be found in `PCServer-KFEngine.ini`, under the section `[Engine.GameEngine]`.  The setting is called `bUsedForTakeover=False`. If you do not have this setting, you can add it here. `True` enables the feature, and `False` disables it.

# Changing difficulty in Web Admin
The server should be sitting at a game lobby when you change this (where it's waiting for people to click ready). It will ignore your change if the server is getting ready to cycle to the next map (after a match, but before the next one has started). 

1. In web admin, go to __Settings__ -> __General__ and select the __Game__ tab. Set the difficulty as appropriate, and click __Save Settings__.
2. Go to __Change Map__ and change the map so the server reloads. You can change it to the same map as you're on right now. It should be the new difficulty setting when it reloads.

# Enabling friendly fire

__NOTE:__ Some guns will behave differently if friendly fire is enabled. Notably, the Medic's _Healthrower_ __will cause damage to teammates__ if friendly fire is enabled, rendering the gun useless on your server.

To enable friendly fire:
1. Edit `Config/PCServer-KFGame.ini`
2. Find the `[KFGame.KFGameInfo]` section
3. Edit `FriendlyFireScale=0.000000` to an appropriate ratio.

This value is a percentage, expressed as a decimal number. `1` is 100%, `0.5` is 50%, `0.05` is 5%, etc.


