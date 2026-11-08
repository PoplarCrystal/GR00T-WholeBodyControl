# Isaac Lab 3.0 training backends

GEAR-SONIC supports G1 and H2 on Isaac Lab 3.0.
See the [container instructions](../docker/README.md) for Docker usage.

G1 loads `gear_sonic/data/assets/robot_description/usd/g1/g1_29dof_rev_1_0.usd`.
The asset and its configuration layers are stored together in the repository.
The original G1 URDF configuration remains available; H2 uses its URDF asset.

## Backend selection

Use the same experiment and motion files with these overrides:

| Physics | Environment managers | Overrides |
| --- | --- | --- |
| PhysX | Torch (default) | `presets=physx manager_env.config.use_warp_env=false` |
| Newton | Torch | `presets=newton manager_env.config.use_warp_env=false` |
| Newton | Warp | `presets=newton manager_env.config.use_warp_env=true` |

`manager_env.config.manager_call_config.default=1` selects eager Warp execution.
Set it to `2` to request CUDA graph capture after validating the eager path.
Managers containing non-capturable Torch adapters remain eager. Newton's
physics solver and the experimental Warp environment managers are separate
choices; selecting Newton alone does not enable Warp managers.

G1 uses `+exp=manager/universal_token/all_modes/sonic_release`; H2 uses
`+exp=manager/universal_token/all_modes/sonic_h2`. Supply the matching robot
motion files through `manager_env.commands.motion.motion_lib_cfg.motion_file`
and the experiment's SMPL/SOMA inputs when applicable.

G1 and H2 define separate `physx` and `newton` joint/body mappings, selected
through `presets`. Extra hand joints are excluded from motion-library permutations. The
simulator/MDP boundary uses XYZW quaternions; legacy WXYZ SMPL, SOMA, and object
data are converted at their boundaries. Existing checkpoints need validation
of observation/action ordering before reuse with a different physics backend.
For offline reconstruction, select the matching `presets` when constructing
`G1Converter` or `H2Converter`.

## Warp adapters

`mdp_warp/env_profile.py` adapts the instantiated Hydra configuration. Native
Warp terms handle supported observations, tracking rewards and terminations;
other stateless terms use the Torch bridge. The bridge exposes Torch views of
Warp outputs and copies results into persistent manager buffers. This does
not make arbitrary Torch computation CUDA graph capturable.

Per-term history buffers preserve chronological flattening and refill from the
first post-reset frame. Tokenizer terms recover their original multi-future
trailing shapes in `ManagerEnvWrapper`. Uniform noise has the same distribution
as the Torch path but uses different random draws for native Warp history.
Warp history currently applies noise from the first frame, whereas the stable
manager gates the first step; bitwise observation equivalence is not promised.

The native tracking command currently rejects height-map configurations and
reference-root-as-anchor configurations. Stateful reward/termination classes
are rejected by the profile; use Torch managers for those configurations.
Complex object-interaction/contact-filtering tasks require separate backend
validation beyond basic G1/H2 motion tracking.
