# Adaptive Temperature Control Algorithm
This documentation describes an adaptive temperature control algorithm designed for systems that monitor and control two temperature readings—one slow-responding (e.g., a vessel's internal temperature) and one fast-responding (e.g., ambient or external temperature). The algorithm calculates a weighted average of these temperatures, dynamically adjusting the weighting to avoid overshoot while ensuring timely responses to temperature changes.

## Inputs
* payload.t1: The internal, slow-responding temperature reading (e.g., the temperature inside a liquid vessel).
* payload.t2: The external, fast-responding temperature reading (e.g., the ambient temperature in a cabinet or surrounding environment).
* payload.target or payload.tset: The desired target temperature that the system aims to achieve. Denoted 𝑇set in the documentation.

## Outputs
* payload: The adjusted combined temperature of 𝑇1 and 𝑇2, that should be passed as temperature reading to the thermostat.
* t1: The latest 𝑇1 value
* t2: The latest 𝑇2 value
* target: The desired target temperature. Can be useful as pass-through to directly provide to the thermostat.


## Purpose and Use Case
When controlling a temperature-sensitive system, especially in cases where you have a more sluggish (slow-responding) internal temperature sensor and a faster-responding external sensor, there's often a risk of overshooting the internal temperature. This happens when the external environment is heated or cooled too quickly while the internal temperature lags behind. By dynamically adjusting the weight between the two temperature readings based on how close the internal temperature is to the setpoint, this algorithm helps:

* Prevent temperature overshoot by smoothly transitioning control focus from the faster external temperature to the more sluggish internal temperature.
* Enhance response time by leveraging the faster-responding temperature reading when the internal temperature is far from the target.
* Provide balanced control in systems with different temperature dynamics for internal and external environments.
![Overview diagram](https://github.com/pakerfeldt/node-red-contrib-adaptive-temperature-control/blob/main/overview-diagram.png?raw=true)

## How the Algorithm Works
The algorithm combines two temperature readings, 𝑇1 (internal) and 𝑇2 (external), and dynamically adjusts the weight assigned to each, with a focus on minimizing overshoot. The weight is determined by a function that changes based on how close the internal temperature is to the target setpoint 
 𝑇set.
 
### Key Formula
The algorithm is as follows:

![Mathematical expression](https://github.com/pakerfeldt/node-red-contrib-adaptive-temperature-control/blob/main/math-expression.png?raw=true)

Where:
* 𝛼(𝑇1,𝑇set) is the dynamic weighting factor and determines how much weight is given to the internal temperature 𝑇1.

## Parameters
The following parameters affects how the combined temperature is calculated and can be changed in the node configuration. The default values serve as a good starting point.

* baseline: Defaults to 0.65. A parameter that determines the minimum influence of 𝑇1 in the calculation. A baseline of 0.5 means that 𝑇1 will always have at least a 50% influence, even when the internal temperature is far from the setpoint.
* beta (𝛽): Defaults to 0.5. A steepness parameter that controls how quickly the algorithm shifts from relying on 𝑇2 to relying on 𝑇1. A higher 𝛽 value makes the transition sharper, while a lower 𝛽 value smooths the transition.
* delta (Δ𝑇): Defaults to 3.5. A threshold value that controls how close 𝑇1 needs to be to the setpoint before the algorithm starts prioritizing it over 𝑇2. A smaller Δ𝑇 value means the algorithm will start focusing on 𝑇1 only when it's very close to the setpoint, while a larger Δ𝑇 will cause the system to prioritize 𝑇1 earlier.

### Effects of Adjusting Parameters
1. Baseline:

    * Higher baseline (e.g., 0.6–0.7): Ensures that 𝑇1 (the internal temperature) has a higher influence on the combined temperature, even when it's far from the setpoint. This could result in slower system response but minimizes the risk of overshooting the target.
    
    * Lower baseline (e.g., 0.3–0.4): Allows 𝑇2 to have more influence when 𝑇1 is far from the setpoint, resulting in a faster response to temperature changes, but potentially increasing the risk of overshoot.

2. Beta:

    * Higher 𝛽 (e.g., 5-10): Causes a rapid shift from relying on 𝑇2 to 𝑇1 as 𝑇1 nears the setpoint. This makes the system more sensitive to changes in 𝑇1 as it approaches the target but could lead to less smooth control.

    * Lower 𝛽 (e.g., 1-3): Results in a smoother transition, making the system less responsive to sudden changes but providing more stable control.

3. Delta T:

    * Larger Δ𝑇 (e.g., 2-3°C): The system will begin prioritizing 𝑇1 earlier, when it's still relatively far from the setpoint. This reduces the risk of overshooting but may make the system slower to reach the desired temperature.

    * Smaller Δ𝑇 (e.g., 0.5-1°C): The system will rely on 𝑇2 for longer, only prioritizing 𝑇1 when it's very close to the setpoint. This makes the system more responsive but can lead to overshoot if  𝑇1 lags behind significantly.

### Example Use Case
Imagine you are controlling the temperature of a fermentation vessel inside a climate-controlled cabinet. The internal temperature of the liquid (𝑇1) is slow to change, while the cabinet's air temperature (𝑇2) can change more rapidly. Using this algorithm, you can:

* Allow the external air temperature (𝑇2) to influence the control system when the liquid temperature is far from the setpoint.
* Gradually shift the focus to the liquid temperature (𝑇1) as it nears the desired setpoint, avoiding overshooting the internal temperature.

This method ensures smooth, adaptive control, minimizing temperature fluctuations in sensitive environments like fermentation, laboratory experiments, or other temperature-critical applications.

## Appendix
### Latex expressions
#### Weighting factor 𝛼 
$$
\alpha(T_1, T_{\text{set}}) = \text{baseline} + (1 - \text{baseline}) \times \frac{1}{1 + e^{-\beta \cdot (\left|T_{\text{set}} - T_1\right| - \Delta T)}}
$$


#### Combined temperature
$$
T_{\text{combined}} = \alpha(T_1, T_{\text{set}}) \cdot T_1 + \left(1 - \alpha(T_1, T_{\text{set}})\right) \cdot T_2
$$
