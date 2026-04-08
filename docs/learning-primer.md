---
layout: xdoubledot
title: Learning Primer
---

# Learning Primer
## Everything you need to understand this project, from scratch

*For: technically literate people (comfortable with maths, coding, engineering) with no prior knowledge of plasma physics or RL.*

---

## How to use this

This is a curriculum, not a reading list. Each section tells you:
- What the topic is and why it matters here
- The minimum you need to understand (don't go deeper until you need to)
- The best resources — specific chapters, not just "read this book"
- How it connects to actual lines of code in this repo

Work through them roughly in order — each one builds on the last. You don't need to master everything before moving forward; get 80% comfortable and keep going.

---

## Topic 1 — Spacecraft Propulsion Basics

**What it is.** How rockets and thrusters work. The fundamental trade-offs between thrust (force) and efficiency (how much propellant you burn per unit of impulse).

**Why it matters here.** The RL agent's reward function balances two objectives that are in tension — high shielding (cathode protection) and correct thrust (12 mN). You need to understand what 12 mN means, why Isp matters, and why we care about Krypton vs Xenon before you can interpret any result.

**Key concepts:**
- **Specific impulse (Isp)** — how efficiently a thruster uses propellant. Higher = better. HETs: 1000–3000 s. Chemical rockets: 300–450 s. Isp = exhaust velocity / g.
- **Thrust** — force produced. T = ṁ × v_exhaust. Low thrust but high Isp = good for long missions.
- **The rocket equation** — Δv = Isp × g × ln(m_initial / m_final). Tells you how much velocity change you can achieve with a given propellant load.
- **VLEO drag** — at 150–350 km, atmospheric drag is constant. Thruster must run continuously just to maintain orbit. This is why 40,000+ hour lifetime matters.
- **Propellant budget** — for a VLEO satellite, most of the mass budget is propellant. Krypton is 3–5× cheaper than Xenon and lighter (same volume of stored gas = more propellant mass).

**Best resource:**
- *Rocket Propulsion Elements* — Sutton & Biblarz. Chapter 2 (the rocket equation) and Chapter 19 (electric propulsion). You need maybe 2 hours on these two chapters.
- Alternatively: the Wikipedia article on "Specific impulse" is actually very good and will take 20 minutes.

**Connection to code:** `ThrusterConfig.target_thrust_mN = 12.0` and the thrust reward term `exp(-0.5 × (thrust_error / 3.0)²)` in `het_env.py:481`.

---

## Topic 2 — Electromagnetism (E and B fields)

**What it is.** The physics of electric and magnetic fields. How charged particles move in them.

**Why it matters here.** A Hall thruster is fundamentally an electromagnetic device. The magnetic field traps electrons. The electric field accelerates ions. Without understanding this you cannot understand why the coils matter.

**Key concepts:**
- **Electric field E** — force per unit charge. A positive charge in an electric field accelerates in the direction of E.
- **Magnetic field B** — doesn't do work on a charge but changes its direction. The force is F = q(v × B) — the Lorentz force. This is always perpendicular to velocity.
- **Cyclotron motion** — a charged particle in a uniform B field moves in a circle (perpendicular to B). The radius r = mv/(qB). The frequency ω_c = qB/m is the cyclotron frequency.
- **Solenoids** — a coil of wire carrying current produces a magnetic field along its axis. B = μ₀ × n × I where n is turns per metre. This is exactly what our three coils are.
- **Reluctance circuits** — magnetic circuits work like electrical circuits. Magnetic flux = MMF / Reluctance, analogous to Current = Voltage / Resistance. This is the model in `MagneticCircuit` in `het_env.py`.

**Best resource:**
- *Introduction to Electrodynamics* — Griffiths. Chapters 5 (magnetostatics) and 6 (magnetic fields in matter). If you know E&M well already, just read sections 5.1–5.4.
- For magnetic circuits specifically: any power electronics textbook. Or search "magnetic reluctance circuit tutorial" — there are good YouTube videos (e.g. the ones by Professor Leonhard from MIT OCW).

**Connection to code:** `MagneticCircuit.compute()` in `het_env.py:138`. The dot product `B_exit = dot(K_EXIT, N*I) / R_GAP` is the reluctance network in one line.

---

## Topic 3 — Plasma Physics Fundamentals

**What it is.** The behaviour of ionised gas — a mix of free electrons and ions. Plasma is the fourth state of matter. It's what's inside the thruster channel.

**Why it matters here.** The ionisation efficiency, thrust, and cathode erosion all emerge from plasma behaviour. The surrogate model encodes these plasma effects in simplified analytical form.

**Key concepts:**
- **Ionisation** — an electron collides with a neutral atom and knocks off one of its own electrons, creating an ion (positively charged) and two free electrons. Requires energy above the ionisation threshold (14.0 eV for Kr, 12.1 eV for Xe).
- **Quasi-neutrality** — in bulk plasma, the density of electrons ≈ density of ions. The net charge is nearly zero everywhere except near walls (the sheath).
- **Plasma sheath** — near a wall, ions and electrons are lost at different rates (electrons are faster). A thin negative potential layer forms — the sheath. Ions are accelerated through the sheath and hit the wall with energy ≈ sheath potential × charge. This is why ion energy at the cathode matters for sputtering.
- **Electron temperature** — electrons in a plasma have a range of energies, described by a temperature T_e in eV. Higher T_e = more energetic electrons = more ionisation, but also more energetic ions hitting walls.
- **Debye length** — the characteristic scale over which charge imbalances are screened out. λ_D = sqrt(ε₀kT_e / ne²). Typical HET: λ_D ~ 0.1 mm. Sheaths are a few Debye lengths thick.

**Best resource:**
- *Introduction to Plasma Physics* — Goldston & Rutherford. Chapter 1 (what is plasma), Chapter 3 (particle motion). This is graduate-level; don't get stuck. Read for concepts, not for derivations.
- Boeuf (2017) *Tutorial: Physics and modeling of Hall thrusters* — this is THE reference for this project. Download the PDF. Read sections I and II first. It's a tutorial paper, well-written. ~2 hours for the key sections.

**Connection to code:** `HETPlasmaSurrogate.hall_parameter()` and `ionisation_efficiency()` in `het_env.py:205–240`.

---

## Topic 4 — The Hall Effect and Electron Drift

**What it is.** What actually makes a Hall thruster work — the specific motion of electrons in crossed electric and magnetic fields.

**Why it matters here.** The "Hall parameter" Ωe is the central quantity in the ionisation model. Everything about why the magnetic field topology matters comes from here.

**Key concepts:**
- **E × B drift** — when both E and B fields are present and perpendicular to each other, charged particles drift in the direction E × B, perpendicular to both. In a Hall thruster, electrons drift azimuthally (around the ring), creating a "Hall current."
- **The Hall parameter Ωe = ω_ce × τ** — the number of cyclotron orbits an electron completes between collisions. If Ωe >> 1, electrons are "magnetised" — they follow field lines. If Ωe << 1, they collide too often to orbit. Optimal ionisation happens at Ωe ~ 13–20 for typical HETs.
- **Why B controls ionisation** — more B → higher ω_ce → higher Ωe → better electron trapping → more ionisation. But if B is too high, Ωe is too large, electrons never escape to ionise anything, and efficiency drops. The Gaussian peak in `ionisation_efficiency()` captures this.
- **Kr vs Xe** — Krypton needs Ωe ~ 18 for optimal ionisation; Xenon only needs ~13. Because Kr's ionisation energy is higher (14 eV vs 12.1 eV), you need more energetic, better-trapped electrons. This means Kr needs a stronger B field at the same power level.

**Best resource:**
- Boeuf (2017), Section III (Hall parameter) and Section IV (ionisation efficiency). This is exactly what the code models.
- Chen (2015) *Introduction to Plasma Physics*, Chapter 2 has a clear derivation of E×B drift.

**Connection to code:** `HETPlasmaSurrogate.hall_parameter()` at `het_env.py:205`. The constants `tau_eff = 5e-9` and `Omega_opt = 18.0` (Kr) are physically motivated here.

---

## Topic 5 — Hall Thruster Operation

**What it is.** How all the above physics combines into the actual thruster operating cycle.

**Why it matters here.** You need to understand the full system before you can understand what the RL agent is controlling and why.

**Key concepts:**
- **The operating cycle:** propellant (Kr or Xe) flows through the anode → electrons from the cathode drift into the channel → electrons ionise neutrals → E field (anode–cathode voltage, ~250V) accelerates ions out → cathode neutralises the beam → thrust.
- **The discharge channel** is an annular ring with ceramic (boron nitride, BN) walls. The plasma is confined radially by B but the ions can escape axially.
- **Breathing mode oscillation** — a 10–30 kHz ionisation instability where the ionisation zone periodically depletes neutrals, recovers, and ionises again. Like a relaxation oscillator. It affects efficiency and causes mechanical stress. Penalised in the reward.
- **Magnetic field topology** — the field is mainly radial at the channel exit (perpendicular to the ion flow direction). This is what traps electrons. The axial gradient (dB/dz) determines whether ions are deflected toward or away from walls.
- **The three coils** — inner solenoid controls field near the inner wall, outer solenoid near the outer wall, trim coil shapes the field gradient near the cathode/exit. They are ALL rings (axisymmetric). Not X/Y/Z axes.

**Best resource:**
- Boeuf (2017) — all of it, but prioritise sections I–IV.
- Goebel & Katz (2008) *Fundamentals of Electric Propulsion* — JPL-published textbook, free PDF online. Chapter 7 on Hall thrusters.

**Connection to code:** `HETEnv` class in `het_env.py:330`. The 9-dimensional state vector: coil currents, B fields, ionisation efficiency, thrust, cathode flux, oscillation amplitude.

---

## Topic 6 — Magnetic Circuit Design

**What it is.** How to design the iron-core magnetic structure that channels flux from the solenoids into the right parts of the thruster.

**Why it matters here.** The `MagneticCircuit` class models how current in each coil maps to B-field at the channel exit and cathode plane. The coupling coefficients K_EXIT, K_CATH, K_GRAD are the key numbers — understanding where they come from tells you why the trim coil is special.

**Key concepts:**
- **Reluctance** — resistance to magnetic flux. R = l / (μ × A) where l is path length, A is cross-section area, μ is permeability. Iron has very high permeability (μ_r ~ 1000–5000) so flux prefers to flow through iron.
- **Ampere-turns (NI)** — the "driving force" (MMF) for magnetic flux. N turns of wire carrying I amps generates NI ampere-turns of MMF.
- **Pole pieces** — the iron shapes that guide flux to the channel exit. The geometry determines where B is strongest and what direction it points.
- **Why K_CATH_trim = 0.80** — the trim coil is positioned in the back yoke, between the inner and outer circuits. This position gives it disproportionate leverage over the cathode-plane field topology, specifically the axial field gradient dB/dz. The inner and outer solenoids mostly set the radial B at the exit; the trim coil bends the field lines near the exit.

**Best resource:**
- Any power electronics or electrical machines textbook on magnetic circuits. *Power Electronics* by Mohan, Undeland & Robbins has a good Chapter 3.
- For HET-specific magnetic design: Hofer et al. (2012) "Magnetically shielded Hall thrusters" — the JPL paper that defines the H6MS design.

**Connection to code:** `MagneticCircuit` class, `het_env.py:99`. The `K_CATH = [0.30, 0.25, 0.80]` array encodes the coupling from each coil to the cathode plane.

---

## Topic 7 — Sputtering and Erosion

**What it is.** The physical process by which high-energy ions knock atoms off a surface — the mechanism that kills the thruster.

**Why it matters here.** Cathode flux (what the RL agent minimises) is a proxy for sputtering erosion rate. Understanding the physics explains why flux reduction → lifetime extension, and why the relationship might be super-linear (the 4× vs 100× question).

**Key concepts:**
- **Sputtering** — an incident ion transfers momentum to surface atoms. If the energy transferred exceeds the surface binding energy, atoms are ejected. The process is statistical.
- **Sputtering yield Y(E)** — atoms ejected per incident ion. Strongly depends on ion energy E:
  - Below threshold (~25–30 eV for BN): Y = 0 — no sputtering at all.
  - Above threshold: Y rises steeply, roughly as (E - E_threshold)².
  - At typical HET sheath energies (~50–200 eV): Y ~ 0.01–0.5 atoms/ion for common materials.
- **Erosion rate** = ion flux × Y(E). To minimise erosion: reduce flux, reduce energy, or both.
- **Why magnetic shielding gives such large erosion reduction** — it simultaneously reduces both flux (fewer ions reach the cathode) AND ion energy (near-cathode plasma potential is raised, reducing sheath drop, reducing ion energy). Near threshold, even a small energy reduction can drop Y to zero. This is why the H6MS achieved 1000× erosion reduction from a ~2× flux reduction.
- **Why our 76% flux reduction → 4× lifetime is conservative** — our model only counts the flux reduction, not the energy reduction. If energy effects are included, the real improvement could be much larger.

**Best resource:**
- Bak & Walker (2020) "Review of Plasma-Induced Hall Thruster Erosion" in *Applied Sciences* — this is the best current review, and one of our key references.
- Mikellides et al. (2014) — Section IV explains the shielding mechanism and why both flux and energy matter.
- For sputtering physics: *Handbook of Thin Film Technology* — Maissel & Glang. Chapter 1 on sputtering.

**Connection to code:** `cathode_ion_flux()` in `het_env.py:264`. The `flux = exp(-k_shield × |dBdz| / B_cathode)` formula is the Mikellides approximation. The sputtering yield model is NOT yet in the code — this is an open next step.

---

## Topic 8 — Classical Control Theory

**What it is.** The engineering discipline of designing systems that automatically regulate themselves toward a desired state.

**Why it matters here.** You need to know what PID and MPC controllers are to understand why RL is a better fit for this problem, and to have conversations with the electric propulsion community (who mostly use classical control).

**Key concepts:**
- **PID controller** — the workhorse of industrial control. Proportional (error), Integral (accumulated error), Derivative (rate of change). Simple, robust, but requires a single well-defined setpoint and doesn't handle multi-variable coupled systems well. Could theoretically control coil currents but would need separate PIDs for each coil, with no coupling between them.
- **State space** — representing a dynamic system as ẋ = Ax + Bu, y = Cx + Du. Allows multi-variable (MIMO) control design.
- **Model Predictive Control (MPC)** — at each timestep, solve an optimisation problem over a finite horizon to find the best control inputs. Very powerful but requires an explicit model and scales poorly with state dimension. The IEPC-2025 Georgia Tech paper uses MPC.
- **Why RL is better here** — PID can't handle the 3×3 coupled coil problem without careful MIMO design. MPC requires explicit model access and real-time optimisation (expensive in embedded control). RL learns a policy offline (during training) and executes it cheaply at inference — O(n) neural network forward pass.

**Best resource:**
- *Control Systems Engineering* — Nise. Chapters 1–4 for PID basics.
- For MPC: search "Model Predictive Control introduction" on YouTube. There are excellent 20-minute explainers.

**Connection to code:** Not directly in the code, but it's the answer to "why not just use PID?" when talking to propulsion engineers.

---

## Topic 9 — Machine Learning Foundations

**What it is.** Neural networks and how they learn. The foundation that RL builds on.

**Why it matters here.** The SAC policy and value functions are neural networks. Understanding forward passes, backpropagation, and gradient descent lets you understand what "training" actually does, why it takes 200k steps, and what "actor loss" and "critic loss" in the training output mean.

**Key concepts:**
- **Neural network** — a function approximator. Stacks of linear transformations (weights × input + bias) alternating with nonlinearities (ReLU, etc.). Can approximate any continuous function given enough parameters.
- **Forward pass** — given an input, compute the output. In our case: state vector (9 dims) → policy network → action distribution (3 dims, mean + std).
- **Backpropagation** — compute gradients of a loss function with respect to network weights. This is how the network learns.
- **Gradient descent** — update weights in the direction that reduces the loss: w ← w - lr × ∇L. Learning rate (lr = 3e-4 in our code) controls step size.
- **Loss function** — what we're minimising. In supervised learning: prediction error. In RL: it's more complex (see next section).
- **Overfitting / underfitting** — too few parameters or too little training → underfitting (bad performance). Too many parameters + too little data → overfitting (memorises training data, generalises poorly).

**Best resource:**
- **3Blue1Brown "Neural Networks" series on YouTube** — 4 videos, ~1 hour total. The best visual explanation that exists. Watch these before anything else.
- *Deep Learning* — Goodfellow, Bengio & Courville. Chapters 6 (deep feedforward networks) and 8 (optimisation). The book is free online at deeplearningbook.org.

**Connection to code:** The SAC actor and critic are `MlpPolicy` with `net_arch=[256, 256]` — two hidden layers of 256 neurons each. Defined in `train.py:122`.

---

## Topic 10 — Reinforcement Learning

**What it is.** Learning to make decisions by trial and error, guided by a reward signal. The framework we use to train the HET controller.

**Why it matters here.** This is the core technical contribution. Understanding RL lets you understand why the reward function is designed the way it is, what the training curves mean, and how to improve the system.

**Key concepts:**
- **Markov Decision Process (MDP)** — the formal framework: states S, actions A, transition dynamics P(s'|s,a), reward R(s,a). The agent cannot control the dynamics; it only controls its policy π(a|s).
- **Policy π** — the function the agent learns. Maps states to actions (or action distributions for stochastic policies). After training, π is a neural network.
- **Value function V(s)** — expected cumulative reward starting from state s, following policy π. The critic in actor-critic methods learns V (or the related Q-function).
- **Q-function Q(s,a)** — expected cumulative reward from taking action a in state s, then following π. SAC uses this.
- **Bellman equation** — the recursive relationship: Q(s,a) = R(s,a) + γ × E[Q(s',π(s'))]. The critic is trained to satisfy this. Critic loss in training = how well the Bellman equation is satisfied.
- **Policy gradient** — update the policy to increase the probability of actions that led to high Q values. Actor loss = -E[Q(s, π(s))]. The negative sign because we're doing gradient ascent on reward.
- **Entropy regularisation** — SAC adds an entropy bonus to the reward: r_aug = r + α × H(π(·|s)). This keeps the policy stochastic (exploratory). α (ent_coef) is tuned automatically. When it collapses to near zero (~0.0006 by end of training), the policy has converged.
- **Replay buffer** — SAC stores past (s, a, r, s') tuples and samples randomly from them for training. This breaks temporal correlations and allows reuse of experience (sample efficiency). Buffer size = 100,000 in our code.
- **On-policy vs off-policy** — on-policy algorithms (PPO, REINFORCE) can only learn from the current policy's experience. Off-policy (SAC, DDPG) can reuse old experience. For our slow surrogate, off-policy is better.

**Best resource:**
- **Spinning Up in Deep RL** (OpenAI) — spinningup.openai.com. Read the "Introduction to RL" and "Key Concepts" sections. Then read the SAC algorithm page. This is the best practical RL reference.
- *Reinforcement Learning: An Introduction* — Sutton & Barto. Free PDF at incompleteideas.net. Chapters 1–6 (fundamentals), then skip to Chapter 13 (policy gradient).
- **David Silver's RL course** — UCL, available on YouTube. Lectures 1–7 cover everything you need.

**Connection to code:** `train.py` — the SAC model definition (`train.py:112`), the `TrainingLogger` callback (records episode reward), and the reward function in `het_env.py:476`. The training curves: ep_rew_mean going from 202 → 351 is the key learning signal.

---

## Topic 11 — Soft Actor-Critic (SAC) Specifically

**What it is.** The specific RL algorithm we use. Worth understanding in detail since this is directly what's running.

**Key concepts:**
- **Actor-Critic architecture** — two networks trained jointly: actor (policy π) and critic (Q-function). The actor proposes actions; the critic evaluates them.
- **Soft** — the "soft" in SAC means the policy maximises reward + entropy (rather than just reward). This prevents premature convergence.
- **Automatic entropy tuning** — SAC automatically adjusts the temperature α to target a desired entropy level (set implicitly by `ent_coef="auto"`). This is why ent_coef starts at 0.74 and drops to 0.0006 — the policy found a good deterministic strategy.
- **Twin critics** — SAC uses two Q-networks and takes the minimum of their predictions. This prevents overestimation bias (a known failure mode in value-based RL).
- **Actor loss ≈ -70** — in training output, actor loss is -E[Q(s, π(s))]. The negative sign means the actor is learning to reach states with higher Q values. The large negative value (~-70) means the critic believes the policy will accumulate a lot of reward from most states — which is consistent with episode reward ~350.
- **Critic loss ≈ 0.003** — the Bellman equation residual. Small and decreasing means the critic accurately predicts future rewards.

**Best resource:**
- The original SAC paper: Haarnoja et al. (2018) "Soft Actor-Critic: Off-Policy Maximum Entropy Deep Reinforcement Learning with a Stochastic Actor." arXiv:1801.01290. Read the algorithm box (Figure 1) — it's 6 lines.
- Stable-Baselines3 SAC documentation — explains the implementation details.

**Connection to code:** `stable_baselines3.SAC` in `train.py:112`. All hyperparameters (lr, buffer_size, batch_size, tau, gamma) are set here.

---

## Topic 12 — Surrogate Modelling and Validation

**What it is.** Building fast approximate models of expensive physical systems, and proving they're accurate enough to be trusted.

**Why it matters here.** The surrogate IS the environment the RL agent trains on. If it's wrong, the trained policy is wrong. Validation methodology is what separates this from a toy project.

**Key concepts:**
- **Surrogate vs physics model** — a surrogate approximates the input-output behaviour without simulating the underlying physics in full detail. Our surrogate captures the key causal relationships (B → Ωe → η_ion → thrust) in closed-form equations.
- **Physics-informed** — unlike a pure data-driven surrogate (a neural network fitted to measurements), a physics-informed surrogate uses known governing equations as its structure. This means it generalises correctly to untested conditions.
- **Calibration** — fitting model parameters (coupling coefficients K, shielding constant k_shield, etc.) to match known data points. This is NOT validation.
- **Validation** — testing the calibrated model against data NOT used for calibration. Our Level 1 and Level 2 test suites are validation.
- **Levels of fidelity** — different simulation tools have different accuracy vs. cost trade-offs:
  - Our surrogate: microseconds, approximate
  - HallThruster.jl: seconds, 1D fluid simulation, reference-quality
  - WarpX PIC: hours–days, particle-in-cell, near-exact physics
  - Real hardware: expensive, slow, but ground truth
- **Sim-to-real transfer** — the gap between surrogate performance and real-hardware performance. Reducing this gap is the entire point of the validation programme.

**Best resource:**
- *VALIDATION.md* in this repo — our specific validation plan.
- *Verification and Validation of Simulation Models* — Sargent (2013). Available on ResearchGate. Short and very clear.
- For physics-informed ML more broadly: Raissi et al. (2019) "Physics-informed neural networks" in Journal of Computational Physics.

**Connection to code:** `tests/test_level1_literature.py` and `tests/test_level2_hallthrusterjl.py`. These are the validation suites. Run them with the `.venv` Python to see all 52 tests pass.

---

## Topic 13 — Why Krypton Over Xenon

**What it is.** The propellant choice is a deliberate business and technical decision. Understanding why matters for ESA conversations.

**Key points:**
- **Physics:** Kr has a higher ionisation energy (14.0 vs 12.1 eV for Xe) and lower atomic mass. This means slightly lower thrust per watt, but higher Isp (exhaust velocity ~ 1/√m_ion). With optimal B-field tuning (which RL provides), the efficiency gap closes significantly.
- **Cost:** Xe is ~€1,500–2,000/kg; Kr is ~€300–500/kg. For a satellite with 10–50 kg of propellant, this is €10,000–100,000 in mission budget.
- **Supply chain:** Xenon comes primarily from Ukraine (electrolytic air separation from steel plants). Kr has a more distributed European supply. ESA cares about this.
- **RL advantage:** The optimal B-field for Kr ionisation (Ωe ~ 18) differs from Xe (Ωe ~ 13). Statically designed thrusters are tuned for one propellant. Our RL agent adapts automatically — retrain with `--propellant Xe` and you get a Xe-optimised policy. This is a genuine differentiator.

**Best resource:**
- Su & Jorns (2021) "Performance comparison of a 9-kW magnetically shielded Hall thruster operating on xenon and krypton." JAP, 130, 163306. The definitive modern Xe/Kr comparison.

---

---

## Topic 14 — Aegis Simulation & Visualisation Tools

**What they are.** Three standalone Python scripts in `src/` that let you see the physics — particle motion, 3D geometry, and magnetic field topology — without reading equations.

**Why they matter here.** The RL agent acts on an abstract state vector (9 numbers). These tools make that state concrete. Before tuning reward functions or interpreting training curves, run these first and build a visual mental model of what the thruster is actually doing.

---

### `src/animate_het.py` — Particle Physics Animation

Animates the particle dynamics inside a running Hall thruster: neutral propellant atoms drifting in, electrons spiralling on cyclotron orbits, beam ions accelerating out, and back-streaming ions (the cathode erosion source) travelling backward through the plume.

**Run it:**
```
python src/animate_het.py
```
Saves `outputs/het_animation.mp4` (or `.gif` if ffmpeg is unavailable). Runs in ~10–20 seconds.

**What to look for:**
- **Grey particles** — neutral Kr atoms flowing from left (anode) toward the channel exit. They're slow and straight.
- **Cyan spirals** — electrons undergoing cyclotron motion in the radial B field. The tight spirals show they are magnetised (Hall parameter Ω_e >> 1). They drift azimuthally (into/out of the screen) via E×B drift.
- **Blue arrows** — beam ions accelerated axially by the anode–cathode voltage (~250 V). They travel in straight lines because they're heavy (unmagnetised — Ω_ion << 1).
- **Red particles** — back-streaming ions travelling backward through the plume toward the cathode. These are the erosion source that the RL agent is trying to redirect. Watch how they concentrate near the channel exit region.

The animation shows *why* magnetic topology matters: electrons are trapped by B and drive ionisation, while ions pass straight through. Redirecting back-streamers requires changing the field gradient at the cathode location — exactly what the trim coil does.

---

### `src/model_het_3d.py` — Interactive 3D Thruster Model

Builds a quarter-cutaway interactive 3D model of the thruster showing all structural components, coil positions, magnetic field lines, and ion beam trajectories.

**Run it:**
```
python src/model_het_3d.py
```
Saves `outputs/het_3d_model.html`. Open in any browser. Use mouse to rotate/zoom.

**What to look for:**
- **Blue-grey annular structure** — the iron magnetic circuit: back yoke (the flat disc at the rear), inner pole piece (central cylinder), outer pole piece (outer ring). Iron's high permeability (μ_r ~ 2000) guides flux from the coils to the channel gap.
- **White surfaces** — boron nitride (BN) ceramic walls forming the discharge channel. BN is the material whose sputtering threshold (E_th ≈ 28 eV) determines the erosion rate.
- **Orange/yellow rings** — the three electromagnetic coils: inner solenoid (innermost), outer solenoid (outermost), trim coil (small ring in the back yoke). Note all three are axisymmetric rings — there's no X/Y/Z asymmetry.
- **Green field lines** — magnetic flux arching from inner pole tip to outer pole tip across the channel gap. These are the physical field lines the electrons follow.
- **Blue cones** — ion beam trajectories leaving the exit plane. The plume divergence angle (θ_plume ~ 25°) appears here.
- **Red curves** — back-streaming ion trajectories curving back toward the cathode. In the `external cathode` configuration, the goal is to bend these away.

---

### `src/solve_het_bfield.py` — Finite-Difference Magnetostatic Solver

Solves the 2D axisymmetric magnetostatic PDE for the HET's actual B-field topology. Not a schematic — a real numerical solution of Maxwell's equations with the iron magnetic circuit modelled explicitly.

**Run it:**
```
python src/solve_het_bfield.py
```
Saves two outputs in ~5 seconds:
- `outputs/het_bfield.png` — 4-panel static figure
- `outputs/het_bfield_interactive.html` — zoomable interactive version

**What it solves.** The governing equation is:

$$\frac{\partial}{\partial r}\!\left(\nu \frac{\partial A}{\partial r}\right) + \frac{\partial}{\partial z}\!\left(\nu \frac{\partial A}{\partial z}\right) + \frac{\nu}{r}\frac{\partial A}{\partial r} - \frac{\nu A}{r^2} = -J_\phi$$

where $A_\phi(r,z)$ is the azimuthal magnetic vector potential, $\nu = 1/(\mu_r \mu_0)$ is the reluctivity (low in iron, high in air), and $J_\phi$ is the coil current density. From $A_\phi$ you get: $B_r = -\partial A/\partial z$, $B_z = A/r + \partial A/\partial r$. Field lines are contours of $\psi = r A_\phi$.

**Reading the 4-panel output:**

| Panel | Coils active | What it shows |
|---|---|---|
| Top-left | All three (nominal) | The operating field: bright exit-plane barrier, arching field lines |
| Top-right | Inner solenoid only | Field concentrated near the axis and inner pole |
| Bottom-left | Outer solenoid only | Fills the outer gap; note the opposite winding direction reinforces the radial barrier |
| Bottom-right | Trim coil only | Much weaker but concentrated exactly at the exit plane — this is why the RL agent uses it for fine-tuning |

**Key features to find:**
- **Bright yellow band at z ≈ 7** — the radial B-field barrier at the channel exit. This is what the Ω_e equation cares about. Optimal Ω_e ~ 18 for Kr occurs here.
- **Field lines arching inner→outer pole** — the working flux path. Lines that are perpendicular to the channel axis at the gap = maximum electron trapping.
- **Dark region inside the iron** — the iron almost disappears in the log scale because it guides flux with very little leakage. This is the reluctance circuit working.
- **Trim coil panel** — notice the field is weakest of all three but peaks right at the exit/cathode plane. High leverage on the shielding gradient with low impact on the main ionisation zone.

**Connection to Aegis RL:** The `MagneticCircuit` class in `het_env.py` uses linear coupling coefficients $K_\text{exit}$, $K_\text{cath}$, $K_\text{grad}$ to map coil currents to B field. The FD solver *validates* those coefficients — if the field line topology from the solver matches what the linear model predicts, the surrogate is trustworthy.

---

## Suggested Learning Order

If you have **2 weeks** to get up to speed:

| Week | Topics | Hours/day |
|---|---|---|
| 1, days 1–2 | Topics 1 + 2 (propulsion basics + EM) | 2–3 |
| 1, days 3–4 | Topics 3 + 4 (plasma + Hall effect) | 2–3 |
| 1, day 5 | Read Boeuf (2017) carefully | 3 |
| 2, days 1–2 | Topics 9 + 10 (ML + RL) | 2–3 |
| 2, day 3 | Topic 11 (SAC) + read train.py | 2 |
| 2, days 4–5 | Topics 6 + 7 (magnetic circuits + sputtering) + read het_env.py | 3 |

After two weeks, read the full test suite and run it. Then re-read EXPLAINER.md — everything in it should make sense.

---

## The 20-Minute Version

If you just need the minimum to follow a conversation:

1. **Hall thruster** = ionises gas with electrons, accelerates ions with electric field = thrust. Magnetic field traps electrons (makes them orbit).
2. **Cathode** = electron emitter, outside the thruster. High-energy ions bombard and erode it. This kills the thruster.
3. **Magnetic shielding** = shape the B-field so it repels ions away from the cathode. Works via an exponential relationship with the field gradient.
4. **Three coils** = inner, outer, trim. All rings. Trim coil is the primary shielding actuator because it shapes the near-cathode field.
5. **RL** = agent sees 9-dim state, outputs 3-dim coil adjustments, gets reward = 50% flux reduction + 30% thrust + 15% oscillation penalty + 5% power. Trains by trial and error. 200k steps = 22 minutes.
6. **Result** = 76% flux reduction, 0.05 mN thrust error. Conservative 4× lifetime improvement.
7. **Novel** = nobody has used RL with coil currents targeting cathode erosion. The framing is new.

---

*ẍ OÜ — Tallinn, Estonia*
