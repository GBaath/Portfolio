# *Myrmidon: Phantom Protocol*
 ## Description
 This page contains work I have contributed to Myrmidon for an Unreal 5 startup.
Myrmidon is a top-down, isometric, real-time tactics stealth game set in a brutalist cyberpunk future. 
The player takes control of up to 5 characters with different abilities and strengths to overcome the enemy in both open-solution strategy sections,
as well as more streamlined, one-solution puzzle sections.

<table>
  <tr>
    <td><img src="Images\scene-1.png"/></td>
    <td><img src="Images\scene-2.png"/></td>
  </tr>
</table>

<table>
  <tr>
    <td width = "33%"><img src="Images\scene-3.png"/></td>
    <td width = "33%"><img src="Images\scene-4.png"/></td>
    <td width = "33%"><img src="Images\scene-5.png"/></td>
  </tr>
</table>

<table>
  <tr>
    <td width = "33%"><img src="Images\takedown_alarm.gif"/></td>
    <td width = "33%"><img src="Images\takedown_warehouse.gif"/></td>
    <td width = "33%"><img src="Images\viewcones_large_2.gif"/></td>
  </tr>
</table>

---

## *Current Contributions*

---

## Camera System
The camera in this game is largely based on my [*camera plugin*](https://github.com/GBaath/UnrealPlugins/tree/main/Plugins/GB_OrbitCamera) (more info on the development of this can be found [*here*](https://github.com/GBaath/Portfolio/tree/main/UnrealProductConfigurator#--nicer-camera-controls)) with a few modifications and extentions, with the main one being better panning and smooth level bounds.

### - Hooking the plugin into the exsiting project
When i joined this startup, they had a temporary camera system in place, but we wanted to switch it out fr a more flexible, self contained version, i.e. my camera plugin. This meant replacing the default plugin input and communication with the existing component and interface based system, as well as incorporating some new types of controls to limit the camera smoothly within the levelbounds.

*[W.I.P] (TODO: Images, Event Calls, Panning, Smooth Bounds Clamping)*

---
## Hacking System

<table>
  <tr>
    <td><img src="Images\hack_and_shortcircuit.gif"/></td>
    <td><img src="Images\myrmidon_4.gif" /></td>
  </tr>
</table>

---
## Level Design
<img src="Images\levellayout.PNG" width="50%"/>

---
## Music Programming

