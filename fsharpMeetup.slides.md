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
- Unity Introduction

- Setting up F\# with Unity

- Do some stuff with Unity!

- Asynchronous Gameflow?

- Lessons Learned.. so far

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

## Unity in Action...
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

- Assets -> Import Package -> Visual Studio 2013 Tools

- Visual Studio Tools -> Generate Project Files (adds the UnityVS folder)

- Open solution: Visual Studio Tools -> Open in Visual Studio
]

---
.left-column[
## Unity Setup
## VS2013 Setup
]
.right-column[
1. Add F\# project to the solution: New Project -> Visual F# -> Library
    * Locate it inside the Unity project folder, not in the Assets folder

2. Make sure FSharp.core.dll has "copy local" set to true

3. Project Properties -> Application:
    - Target Framework: Unity 3.5 full Base Class Libraries
    - Target F\# Runtime: F\# 3.0

4. Project Properties -> Build:
    - Output path: Path to "Assets\bin" Folder

5. Add Unity references to F\# project, make sure each has "copy local" set to false
- C:\Program Files\Unity\Editor\Data\Managed: UnityEngine.dll, UnityEditor.dll
- C:\Program Files\Unity\Editor\Data\UnityExtensions\Unity\GUISystem: UnityEngine.UI.dll
]

---
Hello Unity from F\#
--------------------
Create a class that inherits from MonoBehaviour

```F#
open UnityEngine

type RandomRotator() = 
    inherit MonoBehaviour() // MonoBehaviour is the Unity base class
```
---

Hello Unity from F\#
--------------------
Create a public field that we can see from the Unity Editor
```F#
open UnityEngine

type RandomRotator() = 
    inherit MonoBehaviour() // MonoBehaviour is the Unity base class

    // This field will get serialized into the Unity Editor
    [<SerializeField>][<DefaultValue>] val mutable tumble : float32
```

---
Hello Unity from F\#
--------------------
Add our code to the Start method, which sets the angular velocity to a random velocity vector
```F#
open UnityEngine

type RandomRotator() = 
    inherit MonoBehaviour() // MonoBehaviour is the Unity base class
    
    // This field will get serialized into the Unity Editor
    [<SerializeField>][<DefaultValue>] val mutable tumble : float32

    // Start is only called once when the Scene loads
    member this.Start() = 
        let rb = this.GetComponent<Rigidbody>()
        rb.angularVelocity <- Random.insideUnitSphere * this.tumble
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

### Defining mutable, serializable fields without a default value
```F#
[<SerializeField>][<DefaultValue>] val mutable speed : float32
```

For more examples check out: [http://fsharpforfunandprofit.com/posts/classes]

---
Do some F\# stuff with Unity!
-----------------------------
Demo

---
An Asyncronous Gameflow
-----------------------
 * Games are both asynchronous and statefull (lots of user input, collisions, and sequences that must be managed)
 * Synchronous workflows are simpler than most asynchronous tasks involving callbacks, but are not as responsive
 * F\# asynchronous workflows are simple and responsive

---
.left-column[
## Example
]
.right-column[
Let's say in our game we want to:
 1. Fire a "shot" when the Fire1 button is triggered
 2. Limit the rate of fire to specific times
]

---

.left-column[
## Example
]
.right-column[
In C\#, this is done by keeping track of the "next fire time" in a class variable.
```C#
public class FireHandler : MonoBehaviour {

    public float fireRate = 1.0f;

    float nextFireTime;
    
    void Start () 
    {
        nextFireTime = Time.time;
    }

    void Update()
    {
        if (Input.GetButton("Fire1") && Time.time >= nextFireTime)
        {
            //fire something..
            nextFireTime = Time.time + fireRate;
        }
    }
}
```
]

---

.left-column[
## Example
]
.right-column[
In F\#, we can set up an asynchronous workflow that waits for each Fire event, but passes the Time as an immutable value
### Create a Custom Event
```F#
let fireEvt  = Event<_>()
```

### Setup a Triggering Mechanism
```F#
member this.FireAsync() = Async.AwaitEvent fireEvt.Publish
```
]
---
.left-column[
## Example
]
.right-column[
### Trigger the event and pass arguments
```F#
if Input.GetButton("Fire1") then fireEvt.Trigger(Time.time)
```
Aside: async workflows do not execute on the main thread.  Statically scoped values, such as Time.time, are only available on the main thread and must therefore be passed as arguments.
]

---
.left-column[
## Example
]
.right-column[
### Setup the async workflow
- recursive asynchronous loop
- handles each event and calculates the next Fire time
- does it without mutating any values!

```F#
//Define the Firing workflow
let fireWF = async {
    // set up an infinite loop with game state
    let rec fireloop(nextFire) = async {
        // WAIT for the fire button to be triggered
        let! time = this.FireAsync()

        // handle the fire and get our nextfiretime
        let nextfiretime = fireCtrlr fireRate time nextFire
        return! fireloop(nextfiretime)
    }
    return! fireloop(0.0f)
}
fireWF  |> Async.StartImmediate |> ignore
```

```F#
// define the fire controller
let fireCtrlr fireRate (time : float32) nextFire = 
    if time >= nextFire then 
        // fire something!
        time + fireRate
    else nextFire
```
]

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
 * first file in the ordering has ZERO dependencies
 * 1 module per file

## proposed f# unity organization
 * top-level file contains the game components and should have no dependencies
 * use modules to expand game functionality

---

An F\# Unity Game
-----------------
- Taken from the Unity3d Space Shooter Tutorial, http://unity3d.com/learn/tutorials/projects/space-shooter
- Rewritten in F\#
- Feel free to fork on Github: https://github.com/ilmaestro/SpaceShooter-FSharp

---
template: inverse
## Thank you!
