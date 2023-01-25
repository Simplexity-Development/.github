# Plugin Ideas
These are some plugin ideas that we have that needed to be written down, and they were getting buried in the discord. 
If they inspire you, please go ahead and make them, as I will only be putting ideas here that we are okay with sharing


## Shut the cluck up
Plugin to silence mobs

- radius, mob type commands
- item that can be used to specifically silence individual mobs
- item that shows which mobs are currently silenced, when held
- command to silence individual mob with UUID
- command to highlight surrounding silenced mobs


## SimplePunishments
A fairly bare-bones plugin for bans/mutes/etc

### `/jail`
  - Blocks all but whitelisted commands
  - Puts player into adventure mode
  - Configuration for if they should be automatically muted
  - Teleports to set position, saves previous coords and world
  - configuration for message sent to jailed player, where <sender> and <reason> are placeholders
  - configuration for message sent to server, where <sender> and <reason> are placeholders
  - configuration for message sent to staff, where <sender> and <reason> are placeholders
  - configuration for message sent to discord, where <sender> and <reason> are placeholders
  Permissions: `sp.jail.set` `sp.jail.send` `sp.jail.send.self` `sp.jail.bypass` `sp.notif.jail.self` `sp.notif.jail.server` `sp.notif.jail.staff` 

### `/unjail`
  - Places player into default gamemode of the world they left from
  - teleports back to previous coords
  - unmutes if muted
  - configuration for message sent to unjailed player, where <sender> and <reason> are placeholders
  - configuration for message sent to server, where <sender> and <reason> are placeholders
  - configuration for message sent to staff, where <sender> and <reason> are placeholders
  - configuration for message sent to discord, where <sender> and <reason> are placeholders
  Permissions: `sp.jail.remove.own` `sp.jail.remove.others` `sp.jail.remove.self` `sp.notif.jail.self` `sp.notif.jail.server` `sp.notif.jail.staff` 
  
### `/kick`
  - disconnects the player from the server
  - configuration for if there should be a required reason
  - configuration for message sent to kicked player, where <sender> and <reason> are placeholders
  - configuration for message sent to server, where <sender> and <reason> are placeholders
  - configuration for message sent to staff, where <sender> and <reason> are placeholders
  - configuration for message sent to discord, where <sender> and <reason> are placeholders
  Permissions: `sp.kick` `sp.kick.self` `sp.kick.bypass` `sp.notif.kick.server` `sp.notif.kick.staff` 
  
### `/mute <player> <time>`
  - stops the player from sending messages to chat
  - configuration for if there should be a required reason
  - configuration for minimum time
  - configuration for maximum time
  - configuration for default time
  - configuration for message sent to muted player, where <sender> and <reason> are placeholders
  - configuration for message sent to server, where <sender> and <reason> are placeholders
  - configuration for message sent to staff, where <sender> and <reason> are placeholders
  - configuration for message sent to discord, where <sender> and <reason> are placeholders
  Permissions: `sp.mute` `sp.mute.self` `sp.mute.bypass` `sp.notif.mute.self` `sp.notif.mute.server` `sp.notif.mute.staff`
  
### `/unmute <player>`
  - allows the player to chat again
  - configuration for message sent to unmuted player, where <sender> is a placeholder
  - configuration for message sent to server, where <sender> is a placeholder
  - configuration for message sent to staff, where <sender> is a placeholder
  - configuration for message sent to discord, where <sender> is a placeholder
  Permissions: `sp.unmute.own` `sp.unmute.others` `sp.unmute.self` `sp.notif.unmute.server` `sp.notif.unmute.self` `sp.notif.unmute.staff` 

### `/tempban <player> <time>`
  - bans a player for a specified amount of time, and unbans them when that time has passed
  - configuration for if there should be a required reason
  - configuration for minimum time
  - configuration for maximum time
  - configuration for default time
  - overrides a normal ban, if one is already in place
  - configuration for kick message sent to tempbanned player, where <sender> and <reason> are placeholders
  - configuration for message sent to tempbanned player, where <sender> and <reason> are placeholders
  - configuration for message sent to server, where <sender> and <reason> are placeholders
  - configuration for message sent to staff, where <sender> and <reason> are placeholders
  - configuration for message sent to discord, where <sender> and <reason> are placeholders
  Permissions: `sp.tempban` `sp.tempban.own` `sp.tempban.others` `sp.tempban.self` `sp.tempban.bypass`  `sp.notif.tempban.server` `sp.notif.tempban.staff`
  
### `/ban <player> <reason>`
  - configuration for if there should be a required reason
  - configuration for kick message sent to banned player, where <sender> and <reason> are placeholders
  - configuration for message sent to banned player, where <sender> and <reason> are placeholders
  - configuration for message sent to server, where <sender> and <reason> are placeholders
  - configuration for message sent to staff, where <sender> and <reason> are placeholders
  - configuration for message sent to discord, where <sender> and <reason> are placeholders
  Permissons: `sp.ban` `sp.ban.self` `sp.ban.bypass`  `sp.notif.ban.server` `sp.notif.ban.staff`

### `/unban <player>`
  - allows the player to join the server again
  - configuration for message sent to server, where <sender> is a placeholder
  - configuration for message sent to staff, where <sender> is a placeholder
  - configuration for message sent to discord, where <sender> is a placeholder
  Permissions: `sp.unban.own` `sp.unban.others` `sp.notif.unban.server` `sp.notif.unban.staff` 
  
### `/history <player> <type>`
  - allows looking at history
  Permissions: `sp.history.own` `sp.history.others.<type>` `sp.history.modify`

## Grief Prevention Persistent Trust Addon
  People will be added in every new claim

  - lists for access, container, build, permission trust
  - Adding someone will give that level of trust to all your current claims
  - Removing someone will remove that level of trust from all your current claims (if possible- might only be possible to untrust completely)
  
## Chaotic Enchants
  Custom enchants but chaotic

  - Explosion pickaxe enchant that causes an actual explosion, that would damage the player (would have a similar effect to bed mining, excep would be point-blank)
  - Skeleton bow enchant that will occasionally cause creepers to drop a disc
  - Charged Creeper enchant that will cause an explosion and make a mob drop their skull (if they can drop a skull) or a player drop their skull, not sure on implementation yet
  - Frog enchant that makes magma slimes drop froglights
  
## Chat disabled Warning
  Warning when a player's chat is disabled due to a missing public key

  - Tell a player why their chat isn't working when their public key is missing
  - Explain possible solutions
  
## Custom text
   
  - Custom text files that can be sent to a player on command, similar to essentials's custom text file
  
# Better Crops
  Improvements on crop mechanics
  
  - Bonemeal sugarcane
  - Bonemeal Nether wart
  - Bonemeal cactus
  - Fortune gives more wheat/beets and not more seeds
  - configurable compost items
