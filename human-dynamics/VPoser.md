# VPoser — High-Level Summary

Original repository: [nghorbani/human_body_prior](https://github.com/nghorbani/human_body_prior/tree/main)

## Table of Contents
- [What this repo is](#what-this-repo-is)
- [Main components](#main-components)
- [Inputs](#inputs)
- [Outputs](#outputs)
- [What the model is](#what-the-model-is)
- [Training loss](#training-loss)
- [Important Note](#important-note)

## What this repo is

This repo is a **pose-prior + fitting toolbox**.

Its core model, **VPoser**, is a variational autoencoder (VAE) that learns a compact latent space of realistic human body poses. Around that model, the repo also provides **Inverse Kinematics (IK)** tools that use the learned pose prior to fit **SMPL/SMPL-X** body parameters to target joints, markers, or other pose observations.

The repo is like a system that helps produce **plausible human body poses**. Instead of letting optimization search over any possible pose, it uses a learned prior so the final pose stays closer to realistic human motion.

## Main components

### 1. VPoser
A learned **pose prior** over valid human body poses.

### 2. IK / fitting engine
An optimization layer that uses the pose prior to fit body parameters to observed targets such as joints or markers.


## Inputs

Two common ways to use this repo.

### Direct VPoser input
A human **body pose** for **21 joints**, represented in **axis-angle** form.

### IK / fitting input
Target observations plus a body model.  
These targets can include:
- 3D joints
- 3D motion-capture markers
- 2D keypoints

The IK engine then optimizes body parameters to match those targets.


## Outputs

### VPoser outputs
When using VPoser directly, the decoder can return:

- `pose_body`: decoded body pose in **axis-angle**
- `pose_body_matrot`: decoded body pose as **rotation matrices**

It can also expose latent statistics such as:

- `poZ_body_mean`
- `poZ_body_std`

and can sample new poses from the latent space.

### IK engine outputs
When using the fitting engine, the output is a set of optimized body parameters, such as:

- `poZ_body`: latent pose code
- `trans`: body translation
- `root_orient`: global/root orientation
- `betas`: body shape coefficients


## What the model is

VPoser is a **variational autoencoder (VAE)** trained as a prior over valid human body poses. It's a relatively small MLP-style VAE.

At a high level:

- the **encoder** takes a body pose and maps it into a latent Gaussian distribution
- a latent code is sampled from that distribution
- the **decoder** reconstructs the pose from that latent code

The model is trained on **AMASS**.

### Encoder
- flatten input pose
- apply batch normalization
- pass through linear layers with LeakyReLU and dropout
- output a Gaussian latent distribution

### Decoder
- take a latent vector
- pass through linear layers
- predict **21x6** values
- convert them into valid rotation matrices
- optionally convert them to axis-angle representation

## Training loss
 
The main loss(in main paper) contains:

1. **KL divergence**  
   pushes the latent code toward a standard normal distribution

2. **Reconstruction loss**  
   makes the decoded pose match the input pose

3. **Rotation-validity terms**  
   encourage the decoded outputs to represent valid joint rotations

4. **Regularization**  
   helps reduce overfitting

In simple form:

total loss = KL loss + reconstruction loss + rotation-validity penalties + regularization

### Current repo implementation
In the public repo code, the loss is more concrete. It includes:

- **KL loss** between `q(z|x)` and a standard normal prior
- **mesh reconstruction loss**  
  L1 distance between reconstructed and original SMPL-X mesh vertices
- **geodesic rotation loss**  
  compares reconstructed and target joint rotations
- **joint-position L1 loss**  
  compares reconstructed and target joints

## Important Note
- In the current trainer, the rotation and joint-position auxiliary losses are only kept for the early part of training. The provided config keeps them until **epoch 15**, with default weights.
- VPoser is a **single-frame** pose prior, so using it for video or live motion capture usually requires additional temporal modeling to enforce smooth, consistent motion across frames.
