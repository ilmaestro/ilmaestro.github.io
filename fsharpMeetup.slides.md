class: center, middle

Let's build a Unity3D game with F\#
-----------------------------------
Ryan Kilkenny

4/22/2015


---

# Agenda

1. Introduction
2. Setting up F\# with Unity
3. Do some stuff with Unity!
4. Asyncronous Gameflow
5. Lessons Learned
6. Game Demo

---

Introduction
------------
 * [Unity Logo]
 * [Visual Studio Log]
 * [F\# Logo]


---
Setting up F\# with Unity
-------------------------
 1. Download Unity
 2. Download Unity for Visual Studio
 3. Set up a project in Unity
 4. Set up the VS solution
 5. Add F\# project
 6. Add FSharp.core.dll to Unity
 7. Add Unity references to F\# project
 8. Test!


---
Simple F\# Interop Notes
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
Simple F\# Interop Notes
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
Simple F\# Interop Notes
------------------------

### Void functions
```F#
member this.Start() =
```

---
Simple F\# Interop Notes
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
