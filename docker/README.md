# GEAR-SONIC training container

## Build and start

Run from the repository root on a machine with Docker and NVIDIA Container
Toolkit. Fetch Git LFS assets and place motion datasets in `sample_data/` first.

```bash
docker compose -f docker/compose.yml build
docker compose -f docker/compose.yml up -d
docker compose -f docker/compose.yml exec sonic bash
```

Source, logs, and motion datasets are mounted from the repository. Run the
training command below inside the container.

## Training

G1 with Newton and Warp managers:

```bash
python gear_sonic/train_agent_trl.py \
  +exp=manager/universal_token/all_modes/sonic_release \
  presets=newton manager_env.config.use_warp_env=true \
  headless=true use_wandb=false \
  manager_env.commands.motion.motion_lib_cfg.motion_file=sample_data/g1_motions \
  manager_env.commands.motion.motion_lib_cfg.smpl_motion_file=sample_data/g1_smpl_motions
```

Replace dataset paths with your actual data and supply any additional
SMPL/SOMA inputs required by the experiment. Set `presets=physx` and
`manager_env.config.use_warp_env=false` for the PhysX baseline, or disable only
`use_warp_env` for Newton with Torch managers.

## Stop

```bash
docker compose -f docker/compose.yml down
```
