class: center, middle

Let's build a Unity3D game with F\#
-----------------------------------
Ryan Kilkenny

4/22/2015


---

# Agenda

1. Introduction
2. Setting up F\# with Unity - Visual Studio
3. An Asyncronous Gameflow
4. Lessons Learned
5. 

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

An Asyncronous Gameflow
-----------------------
```F#
        //start the Firing workflow
        let fireWF = async {
            do! this.StartAsync()
            let fireCtrlr = GameLogic.playerFire this.shot this.shotSpawn this.fireRate
            let rec fireloop(nextFire) = async {
                let! time = this.FireAsync()
                let nextfiretime = fireCtrlr time nextFire
                return! fireloop(nextfiretime)
            }
            return! fireloop(0.0f)
        }
        fireWF  |> Async.StartImmediate |> ignore
```

---
