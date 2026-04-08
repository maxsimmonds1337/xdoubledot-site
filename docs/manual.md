---
layout: xdoubledot
title: Technical Manual
---

# Technical Manual
## HET RL Control System — Developer Guide

Authors: Max Simmonds, Tiziano Fiore

Version: MVP v0.1 --- March 2026

ESA BIC Estonia Application

## 1. Introduction

This manual covers the ẍ xdoubledot Hall Effect Thruster (HET) Reinforcement Learning (RL) Control System. It is written for both technical and non-technical readers --- if you are new to HET propulsion, start here. If you are a developer jumping straight to the code, you can skim Section 1 and start at Section 3.

### 1.1 What is a Hall Effect Thruster?

A Hall Effect Thruster (HET) is a type of electric propulsion system used on satellites. Instead of burning chemical fuel like a rocket, it uses electricity to accelerate ions of gas (typically Xenon or Krypton) to very high speeds, producing thrust.

Think of it this way: a conventional rocket throws mass out the back quickly. A HET throws mass out the back much more efficiently --- the exhaust travels 10--25 km/s vs 3--4 km/s for a chemical rocket. This means a satellite can carry far less propellant and still complete long missions.

The key components inside a HET are:

-   Discharge channel: A ceramic ring (annulus) where the propellant is ionised

-   Anode: A positively charged electrode at the back of the channel, which also distributes propellant gas

-   Magnetic circuit: A set of electromagnetic coils (solenoids) wrapped around the channel. These create a radial magnetic field that traps electrons, enabling ionisation

-   Hollow cathode: An electron source mounted outside the thruster. Electrons flow from here into the discharge channel and also neutralise the ion beam downstream

-   Power Processing Unit (PPU): The power electronics that supply and regulate the high voltages and currents the thruster needs

The propellant (gas) enters through the anode, gets ionised by collisions with the trapped electrons, and the resulting ions are accelerated out the back of the thruster by the electric field --- producing thrust.


> **Why Krypton?**                                                                                                                                                                                                                                                                                                                                                                 
>
> Krypton is 3--5x cheaper than Xenon and available from a more resilient European supply chain. It has a slightly higher ionisation energy (14.0 eV vs 12.1 eV for Xenon), meaning it requires a slightly stronger magnetic field for efficient ionisation. The ẍ system is optimised for Krypton as the primary propellant, with Xenon comparison mode included for benchmarking.


### 1.2 The Cathode Erosion Problem

The hollow cathode is the primary lifespan-limiting component of a modern HET. Here is why:

Some of the high-energy ions in the thruster plume do not travel cleanly out the back --- they get redirected by the magnetic field topology and bombard the cathode surface. Over time, this erodes the cathode emitter material (typically barium oxide on a tungsten matrix). As the cathode erodes:

-   Its geometry changes, altering the electron emission characteristics

-   Plasma instabilities increase

-   Eventually the cathode fails entirely, ending the mission

Current industry-standard HETs last 10,000--15,000 hours before cathode failure. For Very Low Earth Orbit (VLEO) missions --- where a satellite at 250 km altitude needs near-continuous thrust to fight atmospheric drag --- this is not enough. A 5-year VLEO mission needs over 40,000 hours of thruster life.


> **The ẍ insight**                                                                                                                                                                                                                                                                                                                                                                       
>
> Most propulsion engineers treat cathode erosion as a materials problem --- use harder materials, add redundant cathodes. ẍ treats it as a control problem: if you can dynamically adjust the magnetic field to steer ions away from the cathode surface in real time, you eliminate the root cause rather than managing the symptom. This is the core idea behind the RL control system.


### 1.3 The Magnetic Shielding Concept

The magnetic field in a HET determines where ions go. The field topology --- the shape of the magnetic field lines --- determines whether ions are directed away from or toward the cathode.

NASA/JPL demonstrated in 2012 (Hofer et al.) that a carefully chosen static magnetic field geometry can substantially reduce cathode erosion. This is called magnetic shielding. It works by establishing a potential gradient that repels ions away from the cathode surface.

The ẍ approach goes further: instead of a fixed optimal geometry, we use three independently controllable electromagnetic coils (inner, outer, and trim) and adjust their currents in real time using a reinforcement learning controller. This is dynamic magnetic shielding.

The advantage of dynamic shielding is that it can:

-   Continuously re-optimise as operating conditions change (power level, propellant flow, thruster temperature)

-   Adapt to thruster ageing --- as the cathode geometry changes over time, the controller adjusts

-   Handle VLEO operating conditions, where drag varies significantly with solar activity, requiring the thruster to operate across a wide throttle range

> **2. System Overview**

The MVP system consists of three Python modules and a Jupyter notebook. Understanding how they connect is important before diving into the details of each.

### 2.1 Architecture

The system has three layers:

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Layer**            **File**        **What it does**
  -------------------- --------------- ---------------------------------------------------------------------------------------------------------------------------------
  Physics simulation   het_env.py      Simulates a 200W Krypton HET using analytical physics models. The RL agent lives inside this environment and interacts with it.

  RL agent             train.py        Trains a Soft Actor-Critic (SAC) agent to control the coil currents. Saves the trained model and a log of training progress.

  Evaluation           evaluate.py     Loads a trained model, runs it against the environment, and produces comparison plots vs a static (uncontrolled) baseline.
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------

The Jupyter notebook (xdoubledot_mvp.ipynb) wraps all three layers and is the recommended starting point for running the system on Google Colab.

### 2.2 Data Flow

At each timestep, the following sequence occurs:

1.  The RL agent observes the current state of the thruster (9 numbers --- described in Section 3).

2.  The agent outputs an action: how much to adjust each of the three coil currents (3 numbers).

3.  The physics simulation applies those adjustments, recomputes the magnetic field, and updates the plasma state.

4.  A reward is calculated from the new plasma state --- primarily how much the cathode ion flux has been reduced.

5.  The agent uses the reward to update its policy (during training) or simply acts on it (during evaluation).

This loop runs 500 times per episode. During training, thousands of episodes are run.

> **3. The HET Environment (het_env.py)**

This is the most important file in the system. It contains the physics model of the thruster and defines what the RL agent can observe, what actions it can take, and what reward it receives.

### 3.1 What is a Gymnasium Environment?

Gymnasium (formerly OpenAI Gym) is the standard interface for RL environments in Python. It defines a common API that any RL algorithm can use:

-   reset() --- start a new episode, return the initial observation

-   step(action) --- apply an action, return (observation, reward, terminated, truncated, info)

-   observation_space --- what the agent can see (defined as a Box of min/max values)

-   action_space --- what the agent can do (defined as a Box of min/max values)

The HETEnv class in het_env.py implements this interface for our thruster simulation.

### 3.2 Configuration: ThrusterConfig

Before creating the environment, you can customise the thruster parameters using the ThrusterConfig dataclass. All parameters have sensible defaults for a 200W VLEO-class Krypton HET.

  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Parameter**       **Default**   **Units**   **Description**
  ------------------- ------------- ----------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  discharge_voltage   250           V           The voltage between anode and cathode. Higher voltage = faster ions = more thrust per unit propellant, but also more cathode bombardment energy.

  anode_flow_sccm     2.5           sccm        Propellant flow rate at the anode (sccm = standard cubic centimetres per minute). Sets how much gas is available to ionise.

  nominal_power_W     200           W           Target electrical power. Used for context --- the actual power depends on discharge voltage and current.

  target_thrust_mN    12.0          mN          The thrust level the agent is rewarded for maintaining. For a 70 kg VLEO satellite at 250 km, \~12 mN covers moderate solar activity drag.

  propellant          \"Kr\"        ---         Propellant choice: \"Kr\" for Krypton or \"Xe\" for Xenon. Affects ionisation efficiency curve and mass flow calibration.

  I_inner_min/max     0.0 / 5.0     A           Current limits for the inner electromagnetic coil. The agent cannot command currents outside this range.

  I_outer_min/max     0.0 / 5.0     A           Current limits for the outer electromagnetic coil.

  I_trim_min/max      −2.0 / 2.0    A           Current limits for the trim coil. Trim is bidirectional (can add or subtract field) --- hence the negative lower limit.

  dI_max              0.25          A/step      Maximum coil current change per timestep. This is a rate limiter --- prevents the agent from making unrealistically fast changes that would not be achievable in hardware.

  I_inner_nom         3.0           A           Nominal inner coil current --- where the thruster starts each episode (with small random perturbation).

  I_outer_nom         2.5           A           Nominal outer coil current.

  I_trim_nom          0.0           A           Nominal trim coil current (zero = symmetric field at start).
  --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


> **Why sccm?**                                                                                                                                                                                                                                                                                                                                                             
>
> SCCM (Standard Cubic Centimetres per Minute) is the standard unit for small gas flow rates in propulsion and semiconductor equipment. It refers to the flow rate at standard temperature and pressure (0°C, 1 atm). The actual mass flow in kg/s depends on the gas density, which is why Krypton and Xenon have different conversion factors despite the same sccm value.


### 3.3 The Magnetic Circuit Model

The magnetic circuit model (MagneticCircuit class) maps the three coil currents to the magnetic field at two critical locations: the channel exit plane and the cathode plane.

The model uses a simplified reluctance network --- analogous to Ohm\'s law but for magnetic flux instead of electric current. Each coil contributes to the field at each location via a coupling coefficient:

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Coil**           **Turns**   **Primary effect**                                                                                                                                                                                                 **Coupling to cathode**
  ------------------ ----------- ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ -------------------------
  Inner (solenoid)   80          Dominates field at channel exit and ionisation zone. Increasing I_inner raises B_exit, raising the Hall parameter and ionisation efficiency.                                                                       Moderate (30%)

  Outer (solenoid)   60          Shapes the field profile across the channel width. Works with the inner coil to set the radial field uniformity.                                                                                                   Moderate (25%)

  Trim (shaping)     40          Fine-tunes the field gradient at the cathode plane. The primary shielding actuator --- small changes in I_trim have a large effect on dB/dz at the cathode, which directly controls the shielding effectiveness.   High (80%)
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Three quantities are computed from the coil currents:

-   B_exit: The magnetic field strength at the channel exit plane (in Tesla). Target: 150--250 Gauss (0.015--0.025 T) for a 200W class thruster.

-   B_cathode: The field strength at the cathode plane. Used in the shielding model.

-   dB/dz: The axial gradient of the magnetic field at the cathode --- the key shielding parameter. A steep gradient pointing away from the cathode repels ions.

### 3.4 The Plasma Physics Surrogate

The HETPlasmaSurrogate class maps the magnetic field (and fixed operating parameters) to the observable plasma state. This is the physics model that the RL agent is effectively learning to control.

#### 3.4.1 Hall Parameter and Ionisation Efficiency

The Hall parameter (Ωe) is the ratio of the electron cyclotron frequency to the effective electron collision frequency. It is the single most important dimensionless number in HET physics:

-   Low Ωe (\< 5): Electrons are not well trapped by the magnetic field --- they escape too easily, reducing ionisation efficiency

-   Optimal Ωe (15--20 for Krypton): Electrons are well trapped, maximising time spent ionising propellant --- peak efficiency

-   High Ωe (\> 30): Electrons are over-trapped --- they cannot reach the anode, reducing current and causing instabilities

Krypton requires a higher Ωe optimum than Xenon (18 vs 13) because of its higher ionisation energy --- you need more energetic electrons spending more time in the channel to ionise it efficiently.

In the surrogate, ionisation efficiency peaks at Ωe \~ 18 (Krypton) using a Gaussian model fit to published experimental trends (Boeuf 2017).

#### 3.4.2 Thrust

Thrust is computed from the Morozov scaling law:

T = ṁ × ηion × vexhaust × fdiv

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Term**   **Meaning**                        **What affects it**
  ---------- ---------------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------
  ṁ          Propellant mass flow rate (kg/s)   Fixed by anode_flow_sccm and propellant choice. Calibrated so nominal operating point gives target_thrust_mN.

  ηion       Ionisation efficiency (0--1)       Controlled by B_exit via the Hall parameter. Agent must keep B_exit near the optimal value.

  vexhaust   Ion exhaust velocity (m/s)         sqrt(2 × e × Vd / m_ion). Fixed by discharge voltage and propellant mass. \~24,000 m/s for Kr at 250V.

  fdiv       Plume divergence factor (0--1)     cos²(θ), where θ is plume half-angle. Lower B → wider plume → less axial thrust. Accounts for the fact that not all ion momentum is in the thrust direction.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

#### 3.4.3 Cathode Ion Flux (the erosion proxy)

This is the primary output the RL agent is rewarded for minimising. The model is based on the Mikellides et al. (2014) sheath approximation:

flux = exp(−k × \|dB/dz\| / B_cathode)

In plain English: the flux decays exponentially as the ratio of the field gradient to the field magnitude at the cathode increases. A steep shielding gradient (large \|dB/dz\|) relative to the background field at the cathode (B_cathode) produces strong shielding.

The output is normalised 0--1:

-   1.0 = fully unshielded --- maximum cathode bombardment, fastest erosion

-   0.0 = perfectly shielded --- zero ion flux at cathode (theoretical limit)

-   Typical static baseline: \~0.63

-   Target with RL control: \< 0.15 (\>75% reduction in ion flux)


> **Why is cathode flux an erosion proxy and not the actual erosion rate?**                                                                                                                                                                                                                                                                                                                                                                                                                                                                    
>
> Actual sputtering erosion depends on ion flux, ion energy, ion incidence angle, and cathode material properties. Modelling all of this accurately requires a full particle-in-cell simulation (WarpX) which takes hours per operating point. The flux proxy captures the dominant driver (ion bombardment rate) and is fast enough for RL training. The relationship between flux reduction and lifetime extension is approximately linear for the flux range we operate in --- a 60% flux reduction implies roughly 2.5x lifetime extension.


#### 3.4.4 Breathing Mode Oscillations

Breathing mode oscillations are a plasma instability in HETs --- a 10--20 kHz oscillation in the ionisation rate that causes periodic surges in discharge current. They are caused by a feedback loop between the neutral gas supply and the ionisation zone.

Why does this matter for the RL controller? Two reasons:

-   Strong oscillations reduce thrust efficiency --- the instantaneous thrust varies, averaging lower than the peak

-   Changing the magnetic field geometry during operation can either stabilise or destabilise the oscillation --- the controller must avoid configurations that trigger strong breathing modes

The surrogate models oscillation amplitude as a function of how far the current B-field is from the nominal (stable) configuration and how steep the field gradient is. The oscillation amplitude is penalised in the reward function.

### 3.5 State Space (What the Agent Observes)

The agent observes 9 numbers at each timestep --- the observation vector:

  ----------------------------------------------------------------------------------------------------------------------------------------------------
  **Index**   **Variable**   **Range**    **Units**   **Description**
  ----------- -------------- ------------ ----------- ------------------------------------------------------------------------------------------------
  0           I_inner        0 -- 5       A           Current inner coil current --- what the agent set last step.

  1           I_outer        0 -- 5       A           Current outer coil current.

  2           I_trim         −2 -- 2      A           Current trim coil current.

  3           B_exit         0 -- 0.030   T           Magnetic field at channel exit. Tells the agent how strongly electrons are being trapped.

  4           B_cathode      0 -- 0.015   T           Field at the cathode plane. Key input to the shielding model.

  5           eta_ion        0 -- 1       ---         Ionisation efficiency. The agent can use this to see if it is near the optimal Hall parameter.

  6           thrust_mN      0 -- 50      mN          Current thrust output. Agent is rewarded for keeping this near the target.

  7           cathode_flux   0 -- 1       ---         Normalised ion flux at cathode. The primary erosion proxy --- lower is better.

  8           osc_amp        0 -- 1       ---         Breathing mode oscillation amplitude. Should be kept low.
  ----------------------------------------------------------------------------------------------------------------------------------------------------

### 3.6 Action Space (What the Agent Controls)

The agent outputs 3 numbers at each timestep --- the delta (change) to apply to each coil current:

  --------------------------------------------------------------------------------------------------------------
  **Index**   **Variable**   **Range**        **Units**   **Description**
  ----------- -------------- ---------------- ----------- ------------------------------------------------------
  0           dI_inner       −0.25 to +0.25   A/step      Change to apply to the inner coil current this step.

  1           dI_outer       −0.25 to +0.25   A/step      Change to apply to the outer coil current.

  2           dI_trim        −0.25 to +0.25   A/step      Change to apply to the trim coil current.
  --------------------------------------------------------------------------------------------------------------

The delta formulation (rather than absolute current commands) is important for two reasons:

-   Hardware realism: In practice, coil currents cannot change instantaneously. The dI_max rate limit models the slew rate of the power electronics.

-   RL stability: Outputting deltas keeps the action space small and makes it easier for the agent to learn --- it only needs to learn how to nudge the field, not where to set it absolutely.

### 3.7 Reward Function

The reward at each timestep is a weighted sum of four components:

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Component**         **Weight**   **Formula**                          **Goal**
  --------------------- ------------ ------------------------------------ -------------------------------------------------------------------------------------------------------------------------
  Cathode shielding     0.50         1 − cathode_flux                     Primary objective. Maximise shielding by minimising flux. Range 0--1.

  Thrust maintenance    0.30         exp(−0.5 × ((thrust − target)/3)²)   Keep thrust within ±3 mN of target. Gaussian penalty for deviation --- agent is not harshly penalised for small errors.

  Oscillation penalty   −0.15        osc_amp                              Penalise strong breathing mode oscillations. Deducted from reward.

  Coil power cost       −0.05        ΣI² / (3 × Imax²)                    Small penalty for running coils at high current unnecessarily. Encourages efficient solutions.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The weights reflect the priority hierarchy: erosion reduction is the primary goal (50%), thrust is important but secondary (30%), stability is monitored but has some tolerance (15%), and efficiency is a minor consideration (5%).

These weights are configurable --- editing W_FLUX, W_THRUST, W_OSC, W_POWER in HETEnv allows you to explore different objective trade-offs.

> **4. Training the Agent (train.py)**

### 4.1 Why Soft Actor-Critic?

Several RL algorithms could in principle be used here. SAC was chosen for the following reasons:

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Property**               **Why it matters here**
  -------------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Continuous action space    Coil currents are real-valued --- discrete action algorithms (DQN, PPO with discrete actions) would require bucketing the current range, losing precision.

  Sample efficiency          Each training step involves running the physics surrogate, which is fast but not free. SAC\'s off-policy replay buffer means it learns from past experience, requiring fewer environment interactions than on-policy methods like PPO.

  Automatic entropy tuning   SAC automatically balances exploration (trying new coil configurations) and exploitation (repeating known good configurations). This is critical for finding non-obvious shielding geometries.

  Stability                  SAC is one of the most stable continuous-action RL algorithms. It is unlikely to catastrophically diverge, which matters when training time is limited.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### 4.2 Training Parameters

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Parameter**     **Default**    **What it controls**
  ----------------- -------------- ------------------------------------------------------------------------------------------------------------------------------------------------------------
  timesteps         200,000        Total number of environment steps to train for. More = better agent but longer training. 200K is sufficient for a clear demonstration on this environment.

  learning_rate     3e-4           How fast the neural network weights update. Too high = unstable training; too low = slow convergence. 3e-4 is the SAC default and generally safe.

  buffer_size       100,000        Size of the experience replay buffer. Larger = more diverse training data but more memory. 100K stores 200 complete episodes.

  learning_starts   1,000          Number of random steps before learning begins. Fills the replay buffer with diverse experience before the agent starts updating.

  batch_size        256            Number of experiences sampled from the replay buffer per training update. Larger batches = more stable gradients.

  net_arch          \[256, 256\]   Neural network architecture: 2 hidden layers of 256 neurons each. Sufficient for a 9-dimensional state space.

  eval_freq         10,000         How often to evaluate the current agent on a separate evaluation environment and save the best model.

  n_eval_episodes   5              Number of episodes used for each evaluation. Mean reward across these episodes determines if the model is the new best.
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### 4.3 Training Outputs

After training, the following files are saved in the outputs/\<run_name\>/ directory:

-   model_final.zip: The final trained SAC model. Can be loaded for evaluation or further training.

-   best_model.zip: The model with the highest mean evaluation reward seen during training. Usually better than model_final.

-   training_log.csv: Episode-by-episode log of reward, cathode flux, thrust, and oscillation amplitude. Use this to plot the learning curve.

-   config.json: The configuration used for this run --- propellant, target thrust, timesteps, etc.

-   tb_logs/: TensorBoard logs for detailed training metrics (optional, requires TensorBoard).

### 4.4 Expected Training Behaviour

A typical training run on Google Colab CPU takes 5--10 minutes for 200,000 timesteps. The expected progression:

  ------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Phase**            **Timesteps**        **What you will see**
  -------------------- -------------------- ------------------------------------------------------------------------------------------------------------------
  Random exploration   0 -- 1,000           Agent takes random actions. Reward is variable and low. Replay buffer is being filled.

  Early learning       1,000 -- 20,000      Reward begins trending upward. Agent discovers that increasing trim coil current reduces cathode flux.

  Refinement           20,000 -- 100,000    Agent learns the trade-off between shielding and thrust maintenance. Reward continues improving but more slowly.

  Convergence          100,000 -- 200,000   Reward plateaus. Agent has found a stable policy. Cathode flux should be 40--70% below the static baseline.
  ------------------------------------------------------------------------------------------------------------------------------------------------------------

> **5. Evaluation (evaluate.py)**

### 5.1 What Evaluation Does

The evaluation script loads a trained model and runs it for multiple episodes, then compares its performance against the static baseline: the same thruster starting from the same conditions but with the coil currents fixed at the nominal values (no agent control).

This comparison directly demonstrates the value of the RL approach --- how much cathode ion flux is reduced, whether thrust is maintained, and whether oscillations are controlled.

### 5.2 Output Plots

The main output is a 4-panel figure (evaluation_results.png):

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Panel**                 **Shows**                                                                               **What to look for**
  ------------------------- --------------------------------------------------------------------------------------- -------------------------------------------------------------------------------------------------------------------
  Cathode Ion Flux          Blue (RL) vs red (baseline) flux over the episode. Shaded area shows the improvement.   Blue line well below red. Large shaded area = strong shielding.

  Thrust                    RL and baseline thrust vs time, with target line.                                       Both lines should track the green target dashed line closely.

  Oscillation amplitude     RL and baseline oscillation proxy vs time.                                              RL line should be at or below the baseline --- agent should not introduce instability.

  Coil current trajectory   Inner, outer, and trim currents over the episode.                                       Shows what the agent is actually doing. Trim coil will typically move positive to enhance the shielding gradient.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### 5.3 Summary Statistics

The evaluate() function also prints a summary:

-   Flux reduction (%): The percentage decrease in mean cathode ion flux vs the static baseline. This is the headline metric. A well-trained agent should achieve 40--70% reduction.

-   Thrust error (mN): Mean absolute deviation from the target thrust. Should be \< 2 mN.

-   Oscillation reduction (%): How much the agent reduces breathing mode amplitude vs baseline.

> **6. Google Colab Quickstart**

### 6.1 Setup

6.  Go to colab.google.com and create a new notebook.

7.  Upload het_env.py, train.py, and evaluate.py to /content/ using the file browser on the left.

8.  Upload xdoubledot_mvp.ipynb, then open it (File → Open notebook → Upload).

9.  Run cell 1 to install dependencies (gymnasium, stable-baselines3, torch). This takes 1--2 minutes.

10. Run cells 2--7 in order. Full training and evaluation takes \~10 minutes on CPU.

### 6.2 GPU Acceleration

For faster training, switch to a GPU runtime (Runtime → Change runtime type → T4 GPU). Training time drops to \~2 minutes.

No code changes are needed --- stable-baselines3 automatically detects and uses the GPU.

### 6.3 Saving Results

After evaluation, the outputs/ directory contains all results. Download them using:

from google.colab import files \# then\...

files.download(\'outputs/vleo_kr_v1/evaluation_results.png\')

> **7. Scope, Limitations & Next Steps**

This section is important: the MVP is a proof of concept. It demonstrates the RL control concept with a physics-informed surrogate, but it is not a validated engineering model. Here is an honest account of what is and is not included.

### 7.1 What IS Included (MVP Scope)

-   3-coil magnetic circuit model (reluctance network) --- calibrated to reproduce correct B-field magnitudes (150--250 Gauss at channel exit)

-   Hall parameter calculation with anomalous transport correction --- Omega_e \~ 18 at nominal operating point, consistent with Boeuf (2017)

-   Ionisation efficiency model (Gaussian peak in Hall parameter) --- calibrated separately for Krypton and Xenon

-   Morozov thrust scaling with plume divergence correction --- calibrated to 12 mN target at 200W nominal point

-   Mikellides cathode ion flux model (shielding via field gradient ratio)

-   Breathing mode oscillation proxy (B-deviation and gradient steepness)

-   Full Gymnasium-compatible RL environment with correct action/observation spaces

-   SAC training with automatic entropy tuning, replay buffer, evaluation callbacks

-   Comparison plots: RL vs static baseline across all key metrics

-   Krypton and Xenon propellant modes

### 7.2 What is NOT Included (Scoped Out for MVP)

  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Scoped out**                   **Why**                                                                                                                          **How to add it (next steps)**
  -------------------------------- -------------------------------------------------------------------------------------------------------------------------------- ------------------------------------------------------------------------------------------------------
  2D/3D plasma simulation          PIC simulations (WarpX) take hours per operating point --- incompatible with RL training loops that need thousands of episodes   Use WarpX to validate the surrogate at key operating points; retrain surrogate if needed

  Actual cathode erosion rate      Requires sputtering yield models (Xe→BN, Kr→BN) and ion energy/angle distributions --- adds significant complexity               Integrate sputtering yield from Ranjan et al. (2018) or TRIM/SRIM database

  Thermal model                    Channel wall and cathode temperature affect plasma and erosion --- omitted for simplicity                                        Add thermal RC network; coupling to ionisation efficiency via Bohm current

  Propellant feed dynamics         Flow response, pressure transients, and feed system interactions omitted --- flow assumed steady                                 Add first-order lag model for flow controller response

  Multi-coil geometries beyond 3   Some HET designs use 4--5 independent coil circuits. MVP uses 3.                                                                 Extend action space; requires updated magnetic circuit coupling matrix

  PPU dynamics                     The power processing unit introduces its own dynamics (bandwidth limits, switching noise). Omitted.                              Add PPU transfer function; affects how quickly coil currents can actually change

  Hardware-in-loop testing         All physics is simulated --- no actual thruster hardware involved at MVP stage                                                   Deploy trained policy on FPGA/microcontroller; validate against bench discharge current measurements

  HallThruster.jl integration      Julia interop (PyJulia) adds setup complexity. Surrogate is physics-informed but not a validated 1D fluid model.                 Replace surrogate step() with PyJulia call to HallThruster.jl for higher fidelity

  In-orbit conditions              Vacuum facility effects, space plasma environment, radiation effects not modelled                                                Addressed during flight qualification programme
  ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### 7.3 Surrogate Model Accuracy

The surrogate model is physics-informed but simplified. Key calibration points that have been verified:

-   B_exit = 206 Gauss at nominal coil currents (within the 150--250 G target range for a 200W HET)

-   Hall parameter Ωe = 18.1 at nominal (target 15--20 for Krypton, per Boeuf 2017)

-   Ionisation efficiency η = 0.90 at nominal (target 0.75--0.90)

-   Thrust = 12.5 mN at nominal (target \~12 mN for VLEO drag compensation)

-   Mass flow = 0.63 mg/s at nominal (target \~0.6 mg/s for 200W Krypton)

What the surrogate does NOT capture accurately:

-   Absolute erosion rates (it models flux, not actual material removal)

-   Spatial distribution of ion flux across the cathode face

-   Coupling between thruster temperature and plasma state

-   Dynamic responses faster than the coil current slew rate

### 7.4 Recommended Next Steps

In priority order for the incubation period:

11. Run the MVP on Colab and confirm the agent learns to reduce cathode flux. Document the numerical results for the ESA BIC application.

12. Replace the ionisation efficiency surrogate with HallThruster.jl via PyJulia. This gives a validated 1D fluid model as the environment backend.

13. Run WarpX at 3--5 operating points to validate the surrogate\'s cathode flux predictions. Adjust k_shield in cathode_ion_flux() if needed.

14. Add the sputtering yield model to convert cathode flux to an actual erosion rate (mg/hr). This makes the reward signal directly interpretable as lifetime extension.

15. Publish the RL environment (het_env.py) as an open-source package on PyPI / GitHub. Target: IEPC 2027 paper on RL-based HET magnetic field control.

> **8. Physics Reference & Equations**

Quick reference for the key equations implemented in the surrogate.

### 8.1 Hall Parameter

Omega_e = omega_ce \* tau_eff where:

-   omega_ce = e × B / me (electron cyclotron frequency, rad/s)

-   tau_eff = 5×10⁻⁹ s (effective collision time, anomalous transport)

-   e = 1.602×10⁻¹⁹ C, me = 9.109×10⁻³¹ kg

-   Target: Omega_e = 15--20 for Krypton, 10--15 for Xenon

### 8.2 Thrust

T = mdot × eta_ion × sqrt(2 × e × Vd / m_ion) × cos²(theta)

-   mdot: propellant mass flow (kg/s), calibrated to target thrust at nominal

-   eta_ion: ionisation efficiency from Hall parameter model

-   Vd: discharge voltage (V)

-   m_ion: ion mass (Kr: 1.39×10⁻²⁵ kg, Xe: 2.18×10⁻²⁵ kg)

-   theta: plume divergence angle = arctan(k_div / B_exit), k_div = 0.006 T·rad

### 8.3 Cathode Ion Flux

flux = exp(−k_shield × \|dB/dz\| / B_cathode)

-   k_shield = 0.015 m (calibration constant)

-   \|dB/dz\|: axial B-field gradient at cathode plane (T/m)

-   B_cathode: field magnitude at cathode plane (T)

-   Range: 0.02 (strongly shielded) to 1.0 (unshielded)

### 8.4 Magnetic Circuit

B_exit = (K_exit · NI) / R_gap (dot product of coupling vector with ampere-turns vector)

-   K_exit = \[0.85, 0.70, 0.50\] (coupling to exit for inner, outer, trim coils)

-   K_cathode = \[0.30, 0.25, 0.80\] (coupling to cathode plane)

-   K_grad = \[0.60, −0.40, 0.90\] (coupling to axial gradient)

-   N = \[80, 60, 40\] turns (inner, outer, trim)

-   R_gap = 1.5×10⁴ A/Wb (magnetic circuit reluctance)

> **9. Glossary**

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **Term**                      **Definition**
  ----------------------------- ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Anode                         Positively charged electrode inside the discharge channel. Propellant is fed through it and it creates the accelerating electric field.

  Breathing mode                A 10--20 kHz plasma instability in HETs --- periodic oscillation of the ionisation zone that causes current spikes. Linked to reduced efficiency and lifespan.

  Cathode                       Electron source mounted outside the thruster channel. Provides electrons for ionisation and for neutralising the ion beam.

  Discharge channel             The ceramic annular ring where propellant is ionised and ions are accelerated. The primary erosion location (walls) and location of the hollow cathode.

  dB/dz                         Axial gradient of the magnetic field --- the rate of change of field strength along the thruster axis. Critical shielding parameter.

  Gymnasium                     Python library that defines the standard interface (reset, step, observation_space, action_space) for RL environments.

  Hall current                  The azimuthal (circular) drift of electrons trapped by the crossed electric and magnetic fields. This is the Hall effect --- hence Hall thruster.

  Hall parameter (Ωe)           Ratio of electron cyclotron frequency to effective collision frequency. Dimensionless. Controls ionisation efficiency --- optimal value is 15--20 for Krypton.

  HET                           Hall Effect Thruster. Electric propulsion device using crossed E and B fields to ionise and accelerate propellant ions.

  Isp (specific impulse)        Propulsion efficiency metric --- thrust per unit propellant weight flow rate. Higher = more efficient. HETs: 1,500--2,500 s. Chemical rockets: 300--450 s.

  ITAR                          International Traffic in Arms Regulations. US export control law that restricts export of defence/space technologies. ITAR-free means not subject to these restrictions.

  Krypton (Kr)                  Noble gas used as propellant. Cheaper and more available than Xenon. Higher ionisation energy (14.0 eV) requires stronger electron trapping.

  mN (millinewton)              Unit of thrust. 1 mN = 0.001 N. HET thrusters typically produce 1--500 mN. For comparison, the weight of a 100g object on Earth is about 1 N.

  PPU                           Power Processing Unit. The power electronics that supply and regulate the voltages and currents needed by the thruster. Analogous to the engine management computer in a car.

  PIC (Particle-In-Cell)        A high-fidelity plasma simulation method that tracks individual simulation particles. Very accurate but computationally expensive (hours per operating point).

  Plume                         The exhaust stream of ions exiting the thruster. The ions are neutralised by electrons from the cathode shortly after leaving the thruster.

  RL (Reinforcement Learning)   Machine learning paradigm where an agent learns by trial and error, receiving rewards for good actions. The agent in this system learns to adjust coil currents to minimise cathode erosion.

  SAC (Soft Actor-Critic)       A specific RL algorithm well-suited to continuous action spaces. Uses entropy regularisation to balance exploration and exploitation automatically.

  sccm                          Standard cubic centimetres per minute. Unit for small gas flow rates at standard conditions (0°C, 1 atm).

  Surrogate model               A fast approximating model that replaces an expensive physics simulation. In this system, the HETPlasmaSurrogate replaces a full PIC simulation for RL training.

  T (Tesla)                     SI unit of magnetic field strength. 1 T = 10,000 Gauss. Typical HET channel exit field: 150--250 Gauss = 0.015--0.025 T.

  VLEO                          Very Low Earth Orbit. Altitude 180--300 km. Much higher atmospheric drag than standard LEO, requiring near-continuous propulsion. 10x better imaging resolution than 550 km LEO.

  Xenon (Xe)                    Noble gas, industry-standard HET propellant. Lower ionisation energy (12.1 eV) than Krypton. More expensive and supply-chain constrained.
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

> **10. References**

All physics models in this system are grounded in the following peer-reviewed literature:

-   Morozov, A. & Savelyev, V. (2000). Fundamentals of stationary plasma thruster theory. Reviews of Plasma Physics, 21, 203--391.

-   Boeuf, J.P. (2017). Tutorial: Physics and modeling of Hall thrusters. Journal of Applied Physics, 121(1). doi:10.1063/1.4972269

-   Mikellides, I.G. et al. (2014). Magnetic shielding of walls from the unmagnetized ion beam in a Hall thruster. Journal of Applied Physics, 115, 043303.

-   Hofer, R. et al. (2012). Magnetically Shielded Hall Thruster. NASA/JPL Technical Report.

-   Bak, N.P. & Walker, M.L.R. (2020). Review of plasma-induced Hall thruster erosion. Applied Sciences, 10(11), 3775.

-   Ben Slimane, T. et al. (2024). Analysis and control of Hall effect thruster using optical emission spectroscopy and artificial neural network. Journal of Applied Physics, 136(15).

-   Marks, T. et al. (2025). Real-time machine learning control of a Hall thruster discharge plasma. IEPC-2025-515.

-   Marks, T., Schedler, P. & Jorns, B. (2023). HallThruster.jl: a Julia package for 1D Hall thruster discharge simulation. Journal of Open Source Software, 8(86), 4672.

-   Marks, T. et al. (2024). Hall thruster simulations in WarpX. IEPC-2024-409.

-   Marks, T. et al. (2025). GPU-accelerated kinetic Hall thruster simulations in WarpX. Journal of Electric Propulsion.

-   Eckels, J. et al. (2024). Hall thruster model improvement by multidisciplinary uncertainty quantification. Journal of Electric Propulsion, 3(19).

-   Lafleur, T. et al. (2016). Theory for the anomalous electron transport in Hall effect thrusters. Physics of Plasmas, 23(5).

-   Choueiri, E.Y. (2001). Breathing instability of Hall thrusters. AIAA Paper 2001-3542.

*End of manual --- v0.1 --- March 2026*
