# calvin-scripts

* `visualize_dataset.py`: script from calvin developers. Try `pip install opencv-python` to make `import cv2` work.

* `calvin_extract_numbers.py,jl`: extracts the numeric fields of calvin data files in current directory and prints them to stdout in tab separated format. The fields are given below. Only file-id(1), actions(7), rel_actions(7), robot_obs(15), scene_obs(24) are printed (54 columns). https://github.com/mees/calvin/blob/main/dataset/README.md has more details about the splits and the fields. We also provide description and statistics in the next section.

* `debug-training.tsv`, `debug-validation.tsv`, etc.: https://github.com/denizyuret/calvin-scripts/releases/tag/v0.0.1 The release page has the output of calvin_extract_numbers scripts.

* `calvin_scene_info.py`: Read and print info in the scene_info.npy, ep_lens.npy, ep_start_end_ids.npy files in each calvin data directory.

* `calvin_extract_language.py`: Print all the unique task_ids and language annotations in the dataset. There are 34 unique task ids and 389 unique annotations.

* `zcat D-validation.tsv.gz | python calvin_diffs.py`: prints out the differences with the previous line for all but the first line.

* `zcat D-validation.tsv.gz | python calvin_episodes.py`: Tries to guess the episode boundaries if xyz of successive frames differ by more than 8.5 std.

* `zcat D-validation.tsv.gz | python calvin_intervals.py`: Prints out intervals based on idnum discontinuities.

* `zcat D-validation.tsv.gz | python calvin_summarize_numbers.py`: Print statistics for each column.

* `python visualize_calvin_npz.py /datasets/calvin/D/validation/episode_0000000.npz`: Visualize a single frame from a given npz file.

* `python calvin_visualize.py`: Visualize a series of frames in the current directory, a more detailed version of the original visualize_dataset.py.

* `calvin_controller_coordinates.py` and `calvin_controller_regression.py`: Scripts I tried to discover the controller (button etc) coordinates with.


# calvin-files

* episode_XXXXXXX.npz: Each frame is represented in a file named episode_idnum.npz, consecutive idnums indicate consecutive frames (with the exception of episode transitions I guess). Other files indicating the contents: 
* scene_info.npy indicates the first and last frame numbers for a particular directory as well as which scene (A,B,C,D) they come from (although there seems to be some confusion about this). It only exists in */training but describes the union of training and validation.
* ep_start_end_ids.npy indicates the start and end idnums of segments in that particular directory.
* ep_lens.npy indicates the lengths of segments given by ep_start_end_ids.npy.
* statistics.yaml gives basic stats for numeric variables.

| directory | scene_info.npy |
| --------- | -------------- |
| debug/training | {'calvin_scene_D': [358482, 361252]} |
| debug/validation | {'calvin_scene_D': [553567, 555241]} |
| D/training | {'calvin_scene_D': [0, 611098]} |
| D/validation | . |
| ABC/training | {'calvin_scene_B': [0, 598909], 'calvin_scene_C': [598910, 1191338], 'calvin_scene_A': [1191339, 1795044]} |
| ABC/validation | . |
| ABCD/training | {'calvin_scene_D': [0, 611098], 'calvin_scene_B': [611099, 1210008], 'calvin_scene_C': [1210009, 1802437], 'calvin_scene_A': [1802438, 2406143]} |
| ABCD/validation | . |

The validation directories for ABCD, ABC, and D have the same content: calvin_scene_D:
0:53818, 219635:244284, 399946:420498. (99022 frames).

The training directory for D has the rest of the scene D data: calvin_scene_D: 53819:219634,
244285:399945, 420499:611098. (512077 frames).  (scene_info.npy says scene_A but should be
scene_D).

The training directory for ABCD starts with the same content as D/training (except for
several off-by-one errors) followed by B (598910), C (592429), and A (603706) sections which
are identical to ABC/training but renumbered.

The scenes differ by desk color and drawer positioning. The objects look the same.


## calvin-data

A summary of all data (including images) in episode_XXXXXXX.npz files:

```
# julia> for f in data[:files]; println(f, "\t", summary(get(data, f))); end
# actions	7-element Vector{Float64}
# rel_actions	7-element Vector{Float64}
# robot_obs	15-element Vector{Float64}
# scene_obs	24-element Vector{Float64}
# rgb_static	200×200×3 Array{UInt8, 3}
# rgb_gripper	84×84×3 Array{UInt8, 3}
# rgb_tactile	160×120×6 Array{UInt8, 3}
# depth_static	200×200 Matrix{Float32}
# depth_gripper	84×84 Matrix{Float32}
# depth_tactile	160×120×2 Array{Float32, 3}
```

## calvin-fields

The fields in the output of calvin_extract_numbers.py is as follows:

00. idnum
01. actions/x (tcp (tool center point) position (3): x,y,z in absolute world coordinates)
02. actions/y
03. actions/z
04. actions/a (tcp orientation (3): euler angles a,b,c in absolute world coordinates)
05. actions/b
06. actions/c
07. actions/g (gripper_action (1): binary close=-1, open=1)
08. rel_actions/x (tcp position (3): x,y,z in relative world coordinates normalized and clipped to (-1, 1) with scaling factor 50)
09. rel_actions/y
10. rel_actions/z
11. rel_actions/a (tcp orientation (3): euler angles a,b,c in relative world coordinates normalized and clipped to (-1, 1) with scaling factor 20)
12. rel_actions/b
13. rel_actions/c
14. rel_actions/g (gripper_action (1): binary close=-1, open=1)
15. robot_obs/x (tcp position (3): x,y,z in world coordinates)
16. robot_obs/y
17. robot_obs/z
18. robot_obs/a (tcp orientation (3): euler angles a,b,c in world coordinates)
19. robot_obs/b
20. robot_obs/c
21. robot_obs/w (gripper opening width (1): in meters)
22. robot_obs/j1 (arm_joint_states (7): in rad)
23. robot_obs/j2
24. robot_obs/j3
25. robot_obs/j4
26. robot_obs/j5
27. robot_obs/j6
28. robot_obs/j7
29. robot_obs/g (gripper_action (1): binary close = -1, open = 1)
30. scene_obs/sliding_door (1): joint state: range=[-0.002359:0.306696]
31. scene_obs/drawer (1): joint state: range=[-0.002028:0.221432]
32. scene_obs/button (1): joint state: range=[-0.000935:0.033721]
33. scene_obs/switch (1): joint state: range=[-0.004783:0.091777]
34. scene_obs/lightbulb (1): on=1, off=0
35. scene_obs/green light (1): on=1, off=0
36. scene_obs/redx (red block (6): (x, y, z, euler_x, euler_y, euler_z)
37. scene_obs/redy
38. scene_obs/redz
39. scene_obs/reda
40. scene_obs/redb
41. scene_obs/redc
42. scene_obs/bluex (blue block (6): (x, y, z, euler_x, euler_y, euler_z)
43. scene_obs/bluey
44. scene_obs/bluez
45. scene_obs/bluea
46. scene_obs/blueb
47. scene_obs/bluec
48. scene_obs/pinkx (pink block (6): (x, y, z, euler_x, euler_y, euler_z)
49. scene_obs/pinky
50. scene_obs/pinkz
51. scene_obs/pinka
52. scene_obs/pinkb
53. scene_obs/pinkc

## calvin-controller-coordinates

Here is a mapping from controller coordinates to tcp coordinates:

### door: 

Notes: A and D have the door at the same location (to the left of the desk), B and C are
slightly off. All scenes have door at y=0, z=0.53. The tcp.x and door.x have reverse
directions. The range for door.x is 0.00:30.00. TODO: Check the B-C difference!

```
door in A:         range=[-0.002359:0.283729]
tcpx = 0.04-doorx  range=[-0.290013:0.111992]
tcpy = 0.00        range=[-0.083748:0.063747]
tcpz = 0.53        range=[ 0.463807:0.607193]

door in B:         range=[-0.001999:0.306696]
tcpx = 0.23-doorx  range=[-0.182045:0.327057]
tcpy = 0.00        range=[-0.066444:0.069018]
tcpz = 0.53        range=[ 0.493990:0.660118]

door in C:         range=[-0.002693:0.242157]
tcpx = 0.20-doorx  range=[-0.075764:0.284265]
tcpy = 0.00        range=[-0.054552:0.047173]
tcpz = 0.53        range=[ 0.495090:0.606924]

door in D:         range=[-0.002078:0.281810]
tcpx = 0.04-doorx  range=[-0.312252:0.110304]
tcpy = 0.00        range=[-0.062834:0.041952]
tcpz = 0.53        range=[ 0.485743:0.611444]
```

### drawer:

Notes: All scenes have drawer at z=0.36. A and C have it at x=0.10, B and D have it at
x=0.18. Drawer operates on the y axis with the door coordinate the opposite of the y
coordinate. The range of door movement is 0.00:0.22.

```
drawer in A:        range=[-0.000782:0.220825]
tcpx = 0.10         range=[-0.009829:0.172383]
tcpy = -0.20-drawer range=[-0.455591:-0.188756]
tcpz = 0.36         range=[ 0.300350:0.543916]

drawer in B:        range=[-0.000748:0.220765]
tcpx = 0.18         range=[ 0.091167:0.293705]
tcpy = -0.20-drawer range=[-0.476947:-0.192490]
tcpz = 0.36         range=[ 0.304156:0.523035]

drawer in C:        range=[-0.000493:0.220602]
tcpx = 0.10         range=[ 0.034033:0.160883]
tcpy = -0.20-drawer range=[-0.451620:-0.191535]
tcpz = 0.36         range=[ 0.293439:0.518048]

drawer in D:        range=[-0.000900:0.220821]
tcpx = 0.18         range=[ 0.095772:0.253271]
tcpy = -0.20-drawer range=[-0.463522:-0.193341]
tcpz = 0.36         range=[ 0.296918:0.513985]
```


### switch:

Notes: The switch moves in the Y-Z plane. Its x position is around 0.27 in A&D, around -0.32
in B&C (probably should pool their data). Its movement range is 0.00:0.09. However the y-z
coefficients are all different: either the tip of the tool does not move with the slanted
switch or it is installed at a different angle at every scene (not the case). Also the
gripper has to be under or over the switch to push it up or down, which adds an offset
because of the thickness. Maybe can do better with access to simulator code. We also need to
know which point tcp refers to exactly.

```
switch in A:           range=[-0.001835:0.088369]
tcpx = 0.26            range=[ 0.243142:0.306039]
tcpy = 0.0296+0.4168s  range=[-0.008334:0.084772]
tcpz = 0.5412+0.8837s  range=[ 0.478306:0.639290]

switch in B:           range=[-0.001319: 0.090159]
tcpx = -0.31           range=[-0.350396:-0.275247]
tcpy = 0.0186+0.4374s  range=[-0.009503: 0.087056]
tcpz = 0.5412+0.6140s  range=[ 0.484341: 0.637629]

switch in C:           range=[-0.002909: 0.088975]
tcpx = -0.32           range=[-0.337730:-0.294614]
tcpy = 0.0186+0.4705s  range=[-0.006038: 0.080751]
tcpz = 0.5507+0.6574s  range=[ 0.486650: 0.638358]

switch in D:           range=[-0.002888: 0.088266]
tcpx = 0.28            range=[ 0.250364: 0.317681]
tcpy = 0.0238+0.4091s  range=[-0.007851: 0.074466]
tcpz = 0.5467+0.8044s  range=[ 0.496080: 0.638987]
```


### button:

Notes: There is some variation in the X-Y coordinates because the button is big and the tcp
is not always in the same position. However the mean should give us an idea about the center
of the button. It seems to be fixed at Y=-0.12 and A.X=-0.27 B.X=+0.28 C.X~=D.X=-0.12. The
button moves on the Z axis, its range is small [0.00-0.03]. The button coordinate has the
opposite direction from the Z coordinate. Interestingly the coefficient does not seem to be
-1.0, so the button state is not measured in meters?

```
button in A:           range=[ 0.000060: 0.032333]
tcpx = -0.27           range=[-0.306333:-0.163803]
tcpy = -0.11           range=[-0.159289:-0.070262]
tcpz = 0.5163-1.74b    range=[ 0.467474: 0.574068]

button in B:           range=[-0.000009: 0.025924]
tcpx =  0.28           range=[ 0.152613: 0.303529]
tcpy = -0.12           range=[-0.189507:-0.069197]
tcpz = 0.5169-1.87b    range=[ 0.473718: 0.592380]

button in C:           range=[ 0.000064: 0.030551]
tcpx = -0.11           range=[-0.171218:-0.032428]
tcpy = -0.12           range=[-0.157899:-0.074686]
tcpz = 0.5148-1.70b    range=[ 0.469201: 0.590159]

button in D:           range=[ 0.000064: 0.033721]
tcpx = -0.12           range=[-0.162780:-0.041127]
tcpy = -0.12           range=[-0.155131:-0.082582]
tcpz = 0.5158-1.76b    range=[ 0.465889: 0.567390]
```
