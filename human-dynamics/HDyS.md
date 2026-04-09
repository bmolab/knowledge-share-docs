# HDyS — High-Level Summary

Paper: [Homogeneous Dynamics Space for Heterogeneous Humans](https://openaccess.thecvf.com/content/CVPR2025/html/Liu_Homogeneous_Dynamics_Space_for_Heterogeneous_Humans_CVPR_2025_paper.html)

## Table of Contents
  - [What HDyS is](#what-hdys-is)
  - [Inputs](#inputs)
  - [Outputs](#outputs)
  - [What the model is](#what-the-model-is)
  - [Their latent-space idea](#their-latent-space-idea)


## What HDyS is

**HDyS (Homogeneous Dynamics Space for Heterogeneous Humans)** is a model that learns a **shared latent space** across different human-motion representations.

Its goal is not only to model body pose, but to connect **kinematics** and **dynamics** across heterogeneous data sources and body representations.

The main idea is that markers, keypoints, SMPL, joint angles, **torques**, muscle activations, and sEMG all describe the same underlying human motion from different views, so they should be mapped into a common latent space.

**many motion representations → shared latent space → recover dynamics or predict motion changes**



## Inputs

HDyS takes in multiple kinds of **kinematic representations**, including:

- markers
- skeletal keypoints
- joint angles
- SMPL parameters

These kinematic inputs may include **position, velocity, and acceleration** information.

It also works with **dynamics representations**, including:

- joint torques
- muscle activations
- sEMG signals

## Outputs

HDyS has two main directions.

### 1. Inverse dynamics path
It encodes kinematics into the latent space, then decodes dynamics such as:

- Rajagopal-model torques
- SMPL-space torques
- muscle activations
- sEMG

### 2. Forward dynamics path
It encodes available kinematics plus dynamics into the latent space, then decodes **accelerations**.

## What the model is

HDyS is an **aggregation of multiple autoencoders** built around the inverse–forward dynamics procedure.

### Architecture

HDyS contains two main blocks.

#### Inverse-Dynamics Auto-Encoder (IDAE)
- encodes kinematics into latent vectors
- uses **Transformers** for markers and keypoints
- uses **MLPs** for joint angles and SMPL
- uses a shared temporal Transformer plus separate MLP heads to decode dynamics

#### Forward-Dynamics Auto-Encoder (FDAE)
- encodes dynamics plus non-acceleration kinematics into latent vectors
- uses separate MLP encoders for torques, muscle activations, and sEMG
- concatenates these latent codes
- passes them through a shared MLP composer
- decodes accelerations


## Their latent-space idea

The latent space is designed to be a **homogeneous dynamics space**.

That means different input modalities from the **same moment of motion** should land close together in latent space, even if one input comes from keypoints, another from SMPL, and another from torques or sEMG.

The model trains this space using:

- **reconstruction loss** to recover dynamics or accelerations
- **alignment loss** to bring together latent vectors from the same frame and separate different frames, using InfoNCE

### Core idea
The latent space should capture the underlying **motor state** or **motor knowledge**, rather than the quirks of one specific representation.

