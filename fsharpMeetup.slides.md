name: inverse
layout: true
class: center, middle, inverse
---
Let's build a Unity game with F\#
-----------------------------------
[F\# Meetup 4/22/2015 - Ryan Kilkenny]
.footnote[http://ilmaestro.github.io/fsharpMeetup.html]
---
layout: false
.left-column[
# Agenda
]
.right-column[
- Introduction

- Setting up F\# with Unity

- Do some stuff with Unity!

- Asyncronous Gameflow

- Lessons Learned

- Game Demo
]
---
template: inverse

![Default-aligned image](images/Unity_3D_logo.png)
---
.left-column[
    ## What is it?
]
.right-column[
- Unity is a multi-platform IDE for scripting games and working with 3D virtual worlds.

]
---

.left-column[
## What is it?
## How to get it?
]
.right-column[
- Unity is a multi-platform IDE for scripting games and working with 3D virtual worlds.

- Download the free personal edition from [http://unity3d.com]. Currently using version 5.0.1

]
---
.left-column[
## What is it?
## How to get it?
## Multi-Platform!
]
.right-column[
- Unity is a multi-platform IDE for scripting games and working with 3D virtual worlds.

- Download the free personal edition from [http://unity3d.com]. Currently using version 5.0.1

- Exports to many platforms, including:
    - Web / WebGL
    - Mac / PC
    - iOS
    - Android
    - Xbox
    - Ps3/Ps4
    - Windows Store
    - Windows Phone 8
    - ... and more!
]

---

template: inverse

## Unity Scripting in Action... Csharp style.
---
template: inverse

Setting up F\# with Unity
-------------------------

---
.left-column[
## Unity Setup
]
.right-column[
- Download Unity Tools for Visual Studio [http://unityvs.com]

- Start a new project in Unity

- Import Package -> Visual Studio 2013 Tools

- Add project files: Visual Studio Tools -> Generate Project Files

- Open solution: Visual Studio Tools -> Open in Visual Studio
]

---
.left-column[
## Unity Setup
## VS2013 Setup
]
.right-column[
1. Add F\# project to the solution: New Project -> Visual F# -> Library

2. Make sure FSharp.core.dll has "copy local" set to true

3. Under Project Settings -> Application:
    - Target Framework: Unity 3.5 full Base Class Libraries
    - Target F\# Runtime: F\# 3.0

4. Project Settings -> Build:
    - Output path: Path to "Assets\bin" Folder

5. Add Unity references to F\# project, make sure each has "copy local" set to false
    - Unity Engine: C:\Program Files\Unity\Editor\Data\Managed\UnityEngine.dll
    - Unity Editor: C:\Program Files\Unity\Editor\Data\Managed\UnityEditor.dll
    - Unity Editor UI: C:\Program Files\Unity\Editor\Data\UnityExtensions\Unity\GUISystem\UnityEngine.UI.dll
]

---
Hello Unity from F\#
--------------------
```F#
open UnityEngine

type RandomRotator() = 
    inherit MonoBehaviour() // MonoBehaviour is the Unity base class

    // This field will get serialized into the Unity Editor
    [<SerializeField>][<DefaultValue>] val mutable tumble : float32

    // Start is only called once when the Scene loads
    member this.Start() = 
        // Create a random velocity vector
        let rv = Random.insideUnitSphere * this.tumble
        // Set the rigidbody's angular velocity
        this.GetComponent<Rigidbody>().angularVelocity <- rv
```

---
Some F\# Interop Notes
------------------------

### Basic Class Definition
```F#
type CustomerName(firstName, middleInitial, lastName) = 
    member this.FirstName = firstName
    member this.MiddleInitial = middleInitial
    member this.LastName = lastName  
```

Compared with the C\# equivalent:

```C#
public class CustomerName
{
    public CustomerName(string firstName, 
       string middleInitial, string lastName)
    {
        this.FirstName = firstName;
        this.MiddleInitial = middleInitial;
        this.LastName = lastName;
    }

    public string FirstName { get; private set; }
    public string MiddleInitial { get; private set; }
    public string LastName { get; private set; }
}
```

---
Some F\# Interop Notes
------------------------

### Methods: Tupled form vs. Curried form

Any methods that are interoperating with Unity (or other non-fsharp .Net code), must be defined in tupled form. 

```F#
member this.OnTriggerExit((other : Collider), name) =
```

Any methods defined in curried form are only evaluated once at runtime, and most likely will not work properly.

```F#
member this.OnTriggerExit (other : Collider) name =
```

---
Some F\# Interop Notes
------------------------

### Void functions
```F#
member this.Start() =
```

---
Some F\# Interop Notes
------------------------

### Defining mutable, serializable fields without a default value
```F#
[<SerializeField>][<DefaultValue>] val mutable speed : float32
```

For more examples check out: [http://fsharpforfunandprofit.com/posts/classes]

---
Do some stuff with Unity!
-------------------------
Demo

---
An Asyncronous Gameflow
-----------------------
 * Games are both asynchronous and statefull
 * Synchronous workflows are simpler than most asynchronous tasks involving callbacks, but are not as responsive
 * F\# asynchronous workflows are simple and responsive

---
## Example
Let's say in our game we want to:
 1. Fire a "shot" when the Fire1 button is triggered
 2. Limit the rate of fire to specific times

### Create a Custom Event
```F#
let fireEvt  = Event<_>()
```

### Setup a Triggering Mechanism
```F#
member this.FireAsync() = Async.AwaitEvent fireEvt.Publish
```

---
## Example

### Trigger the event and pass arguments
```F#
if Input.GetButton("Fire1") then fireEvt.Trigger(Time.time)
```
Aside: async workflows do not execute on the main thread.  Statically scoped values, such as Time.time, are only available on the main thread and must therefore be passed as arguments.

---

### Setup the async workflow to handle the event and calculate the next Fire time
```F#
//Define the Firing workflow
let fireWF = async {
    
    // partially apply our firing controller
    let fireCtrlr = GameLogic.playerFire this.shot this.shotSpawn this.fireRate

    // set up an infinite loop with game state
    let rec fireloop(nextFire) = async {
        // WAIT for the fire button to be triggered
        let! time = this.FireAsync()

        // handle the fire and get our nextfiretime
        let nextfiretime = fireCtrlr time nextFire
        return! fireloop(nextfiretime)
    }
    return! fireloop(0.0f)
}
fireWF  |> Async.StartImmediate |> ignore
```

```F#
// define the fire controller
let playerFire (shot : GameObject) (shotSpawn: Transform) fireRate (time : float32) nextFire = 
    if time >= nextFire then 
        GameObject.Instantiate(shot, shotSpawn.position, Quaternion.identity) |> ignore
        time + fireRate
    else nextFire
```


---

Some Lessons Learned
--------------------
## FSharp Limitations
 * No folders
 * Dependencies must be loaded in order
 * Use ALT+UP/DOWN to control the order of the files






---

Code Organization
-----------------

## Typical FSharp Organization
 * files organized into layers: data first, then functions
 * cyclic dependencies?
 * first file in the ordering has ZERO dependencies
 * 1 module per file

## proposed f# unity organization
 * top-level file contains the game components and should have no dependencies
 * use modules to expand game functionality
