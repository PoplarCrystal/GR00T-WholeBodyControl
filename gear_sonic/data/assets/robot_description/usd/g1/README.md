# Unitree G1 USD asset

Source: https://huggingface.co/datasets/unitreerobotics/unitree_model/tree/main/G1/29dof/usd

Entry point: `g1_29dof_rev_1_0.usd`.
Keep the `configuration/` directory beside this file; its layers contain the
geometry, physics, and sensor definitions. Geometry is embedded in the base layer.

The composed asset has 29 revolute joints, 30 rigid bodies, and 33 enabled
collision shapes: 15 meshes, 6 cylinders, 2 boxes, and 10 spheres.
