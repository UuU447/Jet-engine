# Jet-engine
Parametric Turbojet Engine in Fusion 360
A full 9-part axial-flow turbojet engine, modeled in Autodesk Fusion using a single Python script. Change a few numbers at the top of the file, hit Run, and the whole engine rebuilds itself.
I started this because I wanted to understand how a jet engine actually works, and "draw it yourself" turned out to be a much better teacher than watching videos. This repo is the result.
Overall size: 400 mm long, 126 mm max diameter, 19 solid bodies.
---
What a turbojet actually does
Four things, in order, along one straight line:
Suck — spinning compressor blades pull air in
Squeeze — each stage of blades packs that air tighter
Bang — fuel sprays in and ignites in the combustion chamber
Blow — the hot gas rushes out the back, spinning a turbine on the way that drives the compressor at the front
The clever part is step 4 powering step 1. The compressor and the turbine sit on the same shaft, so the engine feeds itself once it's running. Everything in this model exists to make that loop work.
---
The nine parts
Everything is laid out along the X axis. Inlet at X = 0, exhaust toward +X.
#	Part	X span (mm)	What it does
1	Central Rotor Shaft	15–310	The spine. Connects the compressor to the turbine so they spin as one piece
2	Compressor Rotor Blades (×5 stages)	19–142	Spinning blades that grab air and shove it backward
3	Compressor Stator Vanes (×5 stages)	34–153	Fixed vanes between rotors that straighten the swirl back out
4	Forward Compressor Casing	20–168	The tapered shell holding it all together
5a	Combustion Liner (inner)	172–250	The ring where the fire happens
5b	Cooling Jacket (outer)	170–255	An air gap around the fire so the engine doesn't melt
6	Fuel Injector Ring	160–184	12 nozzles spraying fuel into the flame zone
7	Turbine Rotor Blades (×2 stages)	251–302	Hot gas spins these, which spin the shaft, which spins the compressor
8	Exhaust Cone	312–372	Smooths the exhaust flow on its way out
9	Aft Casing / Exhaust Nozzle	168–400	Rear shell, narrowing at the exit to speed the jet up
---
Design decisions and why
Why the compressor gets smaller toward the back
Each stage squeezes the air, so the same air takes up less room. If the passage stayed the same size, the air would slow down and the later stages would do almost nothing. So the hub grows (R20 → R28) while the casing shrinks (R58 → R46), and the passage narrows.
```
       inlet                                    outlet
    ┌───────────────────────────────────────────┐
    │  ▓  ▒  ▓  ▒  ▓  ▒  ▓  ▒  ▓  ▒             │  casing wall (shrinking)
    │                                           │
    ├────────┐                                  │
    │ shaft  └──────────────────────────────────┤  hub (growing)
```
Why rotors and stators pitch opposite ways
A rotor doesn't just push air backward — it also flings it sideways, so the air leaves spinning. Spinning air is useless to the next rotor. The stator vanes are angled the other way to catch that spin and straighten it out before the next stage.
In the code this is literally just a sign flip:
```python
"comp_twist":   (22.0, 52.0),     # rotors:  positive
"stator_twist": (-20.0, -42.0),   # stators: negative
```
Why the blades are twisted
A blade tip travels much faster than its root, because it's further from the center of rotation. Same rotation, bigger circle. So the air hits the tip at a completely different angle than it hits the root.
To keep the whole blade working properly, the tip has to be twisted relative to the root. In this model, rotor blades go from 22° at the root to 52° at the tip — a 30° twist over the span. The script builds the blade by drawing one airfoil at the root, another at the tip, and lofting between them.
Why the combustion chamber has two walls
Burning fuel gets hotter than the melting point of the metal around it. The fix is an air gap: cool compressed air flows through the space between the liner and the jacket, keeping a cushion of cool air against the hot wall. That's what the 44 cooling holes are for — they bleed cool air along the inside surface.
This is why part 5 is two bodies instead of one.
Why the exit narrows
Squeeze a flow into a smaller opening and it has to speed up. Thrust comes from throwing mass backward fast, so the nozzle tapers from R60 down to R40 at the exit.
---
The airfoil math
The blades aren't guessed shapes — they use the NACA 4-digit formula, which describes an airfoil with three numbers: how much it curves (camber), where the curve peaks, and how thick it is.
The thickness distribution:
```
yt = 5·t·(0.2969·√x − 0.1260·x − 0.3516·x² + 0.2843·x³ − 0.1015·x⁴)
```
Compressor blades use 10% thickness and 6% camber — thin and efficient. Turbine blades use 16% thickness — much chunkier, because they're sitting in the hot gas stream and need the material.
The points get generated with cosine spacing rather than evenly. This packs more points near the leading and trailing edges where the curve changes fastest, so the resulting spline is smooth instead of faceted.
---
Running it yourself
Install Autodesk Fusion (free for personal use)
Create a new design
Utilities tab → ADD-INS → Scripts and Add-Ins
Scripts tab → + → create a script named `turbojet_generator`
Replace the generated file with `turbojet_generator.py` from this repo
Press Run
Takes about 30 seconds. You'll get 19 named bodies in the browser tree.
To actually see inside
The interesting geometry is hidden under the casing. Turn on Inspect → Section Analysis and pick the XY plane. That cuts the engine in half lengthways and shows the flowpath, the liner walls, and the injector nozzles.
To split into components
The script builds named bodies, not separate components, because Fusion's Part Design documents only allow one component. To split them: select all bodies in the browser, right-click → Create Components from Bodies.
---
Changing the design
Everything lives in the `CONFIG` dictionary at the top of the script. Some things worth trying:
```python
"comp_stages": 8,              # more compression stages
"comp_twist": (15.0, 60.0),    # more aggressive blade twist
"comp_blades": (24, 3),        # more blades per stage
"casing_r": (70.0, 50.0),      # a fatter engine
```
Change a number, re-run, watch what happens. That's the whole point of building it parametrically.
---
Honest limitations
This is a geometry study, not an engineering design. Things it does not do:
No aerodynamic analysis. Blade angles were chosen to look and behave plausibly, not calculated from velocity triangles
No stress or thermal analysis
No bearings, seals, mounts, fasteners, or accessory gearbox
No inlet cone or nozzle guide vanes ahead of the turbine
Tip clearances are a flat 1 mm, not derived from thermal growth
Do not attempt to build this as a working engine. Real turbomachinery involves spinning parts at enormous speeds and continuous combustion, both of which are genuinely dangerous without proper engineering, materials, and testing
If you know turbomachinery and something here is wrong, please open an issue. I'd like to know.
---
Roadmap
[ ] Nozzle guide vanes between combustor and turbine
[ ] Inlet nose cone and bearing housings
[ ] Blade angles derived from actual velocity triangles
[ ] Exported STLs for 3D printing a cutaway display model
[ ] Section-view renders in this README
---
What I learned
Modeling this taught me more than reading about it did. A few things that only clicked once I had to build them:
The compressor is the hard part. The turbine is only two stages; the compressor is five, and every one of them is a different size
"Stator" sounded like a boring detail until I understood it's fixing a problem the rotor creates
The dual-wall combustor isn't over-engineering — without it the chamber melts
Almost every part is a shape spun around one axis. Once I saw that, the modeling got much faster
---
License
MIT — do whatever you want with it.
