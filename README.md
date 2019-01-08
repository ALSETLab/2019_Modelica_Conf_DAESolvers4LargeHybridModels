# Modelica Models from "DAE Solvers for Large-Scale Hybrid Models"

This repository contains specialized versions of the the Nordic44 (N44) power system model, from the [OpenIPSL](https://github.com/OpenIPSL/OpenIPSL) library, which has been modified and used for simulation in the following paper (to be) presented in the [International Modelica Conference 2019](https://modelica.org/events/modelica2019):

> Erik Henningsson, Hans Olsson and Luigi Vanfretti, "DAE Solvers for Large-Scale Hybrid Models," Proceedings of the 13th International Modelica Conference, Regensburg, Germany, March 4–6, 2019.

Please see the full paper on the scope of usage for the models. You can download the full paper from the [conference website](https://modelica.org/events/modelica2019) when it becomes available. A pre-print is available under `./Doc/`; note that this is the version submitted for review, and not the final version of the paper, which you can obtain in the website cited above.


## How to Simulate it?
The DAE mode functionality demonstrated by the models in this repository was introduced in Dymola 2019 and 3DEXPERIENCE 2019x (DBM, Dynamic Behavior Modeling). We exemplify how to simulate the models using Dymola.

Using Dymola 2019 or newer, follow the steps below:
- Load the OpenIPSL library distributed with this repository under `./Models/` by uncompressing the .zip file `OpenIPSL-master.zip`, in your local drive.
- In Dymola, File/Open `./OpenIpSL-master/OpenIPSL/package.mo`
- In Dymola, File/Open the N44 package `./N44/package.mo`
- In Dymola, load the `Nordic44_DAEModeTestCases.mo` package from File/Open. The package browser should look like this:

![N44package](./Figs/00_package.png 'N44 State Events Package')

- Right click `Nordi44_Original_Case_Bus_Fault` and select "Simulation model".

![N44package_model](./Figs/00_main.png 'N44 State Events - Sample Model')

- Simulate and you shall obtain the results shown below.

![N44package_sim](./Figs/01_sim.png 'N44 State Events - Sample Simulation')


## Models used for the publication
The models that were used in the experiments section of the article can be found in the package `Nordic44_DAEModeTestCases.mo`. These are `Nordic44_Original_Case_Line_Opening`, `Nordic44_Original_Case_Bus_Fault`, and `Nordic44_Base_Case_StateEvents3`. The Dymola experiment annotations have been set up in these models as to reproduce the DAE mode test cases in the publication.

These three models can be directly used out-of-the-box to recreate the experiments of the article. However, in the following sections we also outline instructions on how to modify the original N44 models to recreate these test cases.


## Reproducing results from previous work
- To reproduce results from the original N44 model publication [here](https://www.sciencedirect.com/science/article/pii/S2352711018300050?via%3Dihub)
Insert a PwFault component (see procedure below) in `Nordic44_Original_Case` and connect it to `bus_3100` as in the figure below. Set the fault parameters as: `pwFault.R = 0`, and `pwFault.X = 0`.

![original fault](./Figs/02_original_fault.png 'Original Fault')

- This scenario already triggers state events. When the fault at `bus_3100` is applied, the excitation control systems inside several of the generators reach their max limit, the maximum AVR output `V_RMAX0`. This triggers state events and re-initializations of the internal states in the `SimpleLagLim` non-wind-up transfer functions. Because the desired set point can’t be reached these integral controllers continue to wind-up and several resets are required, thus resulting in several state events.

![original sim](./Figs/03_original_sim1.png 'State Events')

![original sim](./Figs/04_original_sim2.png 'State Events')

- To perform further analysis on state events the following modifications were made to the original model.

## Modifications made to the N44 model to introduce state events
- To create a state event a new electric power generator at bus 5610 was included. The generator is equipped with a controller modeled using the ``EXST1`` "IEEE Type AC2A Excitation System" model from OpenIPSL, shown below.

![fault](./Figs/10_faultbus.png 'Fault Bus')

- The EXST1 controller model includes an ``if-elseif-else`` statement that will change the output of the model depending on the value of the generator field voltage. This creates an event that depends on the value of the state of `Vm1` in the figure below.

![ecs](./Figs/11_ECS.png 'ECS')

Inspecting the text view of the model, it can be observed that the value of the parameter `K_C` has been changed from 0.2 to 1.5 so that the condition `EFD > ECOMP*VRMAX – K_C*XADIFD` is fulfilled more easily and therefore trigger a state event quickly.

![ecs](./Figs/12_ECS_code.png 'ECS')

- To trigger the state event, a fault model (pwFault) was added to bus 5603 near this new generator, as shown in the first figure of this section. The fault settings are shown next:
![fault](./Figs/13_fault.png 'ECS Code')

- The fault is applied at `t = 61.050` s and cleared at `t = 61.15`. After this time, the variable EFD present a state event at `t = 61.5066` s as indicated by the event log and the plot of the variable EFD.
![fault](./Figs/14_sim1.png 'State Events')
![fault](./Figs/15_sim2.png 'State Events')

- Note: The system is simulated first without the fault from 0.0 s to 60 s to let the system arrive to equilibrium.  This is because the initial conditions have changed change due to the inclusion of the new generator. See the next section on how to perform the simulation required for initialization.

## Simulating State Events at t=60
- Step 1: The model `Nordic 44 Base_Case_StateEvents2` which does not include the fault is used to generate a new set of initial conditions. This is required because of the inclusion of the new generator at bus 5610 which change the operating point of the whole system. This model is simulated from t = 0.0 s to t = 60 s, time at which it is consider the transient response has extinguished. The figure shows the behavior of the variable of interest.

![fault](./Figs/20_step1.png 'Step 1')

- Step 2: Make sure the parameter `K_C` is equal to 1.5 in the `EXST1 controller` model of the new generator at bus 5610 (`N44.Base_Case.Generators.GenEventTest`) in the model `Nordic 44 Base_Case_StateEvents`.

![fault](./Figs/21_step2.png 'Step 2')

- Step 3: The result file of the simulation above can be used to start a new simulation by selecting `Simulation > Continue > Import initial...`) .

![fault](./Figs/22_step3.png 'Step 3')

The simulation executed in Step 1 is used to continue the simulation using the model `Nordic 44 Base_Case_StateEvents` from t = 60 s to t= 65 s. In this case the fault is applied. The result of the simulation is shown in the following figure.

![fault](./Figs/23_step3.png 'Step 3')

- Step 4: The above steps are equivalent to simulate the model `Nordic 44 Base_Case_StateEvents` from t = 0 s to t = 65 s. the fault is applied at t = 61.05 s. This long time  is necessary to ensure that the system is in steady-state before the application of the fault, due to the connection of the new generator. The following figure shows the result.

![fault](./Figs/24_step4.png 'Step 4')

- Note: All the simulations were performed using the "DAE mode" and the following settings:

![fault](./Figs/25_settings.png 'Simulation Settings')

## Development and contribution

The models in this repository are maintained by [Erik Henningsson](https://www.linkedin.com/in/erik-henningsson-0638839/) and [Luigi Vanfretti](https://github.com/lvanfretti).

Contributions are welcome via pull requests.

## License - No Warranty

This Modelica package is free software and the use is completely at your own risk; it can be redistributed and/or modified under the terms of the GNU Public License version 3.

Copyright (C) 2018, Erik Henningsson and Luigi Vanfretti.
