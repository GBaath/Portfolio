# *Dusty Deliveries*

<img src="Images\dustydeliveriesdemo.gif" width="100%"/>

[Itch.io page](https://yrgo-game-creator.itch.io/dusty-deliveries)  

[Repository Link](https://github.com/Llrac/ProjectBoat)  

## *Game description*

**Dusty Deliveries** is a CO-OP adventure game where you together walk around on a ship. Set the course and engage the thrusters and explore the desert landscape!

----

## My Contrbutions

*This project was made entirely using blueprints for Unrealscript examples check out a [newer repo](https://github.com/GBaath/Portfolio/tree/main/UnrealProductConfigurator#unreal-product-configurator)*


----

# *Dual Player Controller*

*Note: The project is made in Unreal 5.1 and this version has a bugged controller list, preventing the use of local co-op with both mouse and keyboard. Thus, this controller is approximately twice as big as it could have been, were it made in a later engine version*
[Dual Player Controller](https://blueprintue.com/blueprint/xo33kgx1/)

----

# *Ship Control Modules*
These are the interactable stations that control the movement of the ship, mostly controlling external variables of the ship's physics controller (not made by me).
They interact with the player controllers using an interface, along with getting possesed by the interacting player pawn, overwriting the regular movement input.

[Example Station (SpeedController)](https://blueprintue.com/blueprint/tjs49yt5/)


<img src="Images\ShipStations.PNG" width="50%"/>

----

# *Inventory & Pickup*

Super simple pickupsystem with data store in a widget blueprint which interact with drop off station to add score.

<img src="Images\CollectibleHUD.PNG" width="50%"/>

[demo pickup](https://blueprintue.com/blueprint/bo9wc_ht/) 
[HUD Widget](https://blueprintue.com/blueprint/6olfpms9/)

----

# *Sand Serpent*

*This blueprint is quite large, and I've definetly became a lot better at structuring blueprints since I wrote this one.*

The sand serpent roams in areas around the map triggering a chase sequence when the ship gets within a radius.
Since one of the game's objectives is collecting the meat of several serpents, the chase logic neede to be implemented in a way that allowed players to easily attack the serpent with the limited camera angles.
In summary, roaming when out of range, begin chase when entering range, following the side of the ship for x amount of time, and finally attacking and fleeing if not fended off.
The following gifs are a demo of entering and finishing the chase sequence.

<table>
  <tr>
    <td><img src="Images\serpentchasebeing_demo.gif"/></td>
    <td><img src="Images\serpentAttackSequence_demo.png"/></td>
  </tr>
</table>

[Blueprint](https://blueprintue.com/blueprint/yanp-al5/)

----

# *Adaptive Music System*

----

# *Particles & Materials* 
