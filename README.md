# Exploring Safety-Critical Control over Gaussian Splatting Maps

## About
This project extends SaferSplat with a simplified and efficient Control Barrier Function (CBF) framework for safe real-time robot navigation in Gaussian Splat (GSplat) mapped environments. We implement a single integrator formulation for CBF dynamics, significantly reducing complexity while maintaining strong safety guarantees. The approach enables fast and memory-efficient safety filtering over scenes with hundreds of thousands of Gaussian primitives, operating at 15 Hz during online GSplat training with minimal GPU usage. The safety layer is minimally invasive, correcting robot actions only when necessary to prevent collisions. We validate our system in both simulation and real-world hardware experiments.

## Dependencies
For this project we have simplified the installation dependencies with containarized installation. We have seperate containers for training the GSplat and for filtering safe trajectories. Follow the intstructions below to setup. 

