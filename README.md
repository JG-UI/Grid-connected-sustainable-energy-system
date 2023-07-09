# Optimized Charging Scheme of Grid-connected Sustainable Energy System
This repository contains Python code that implements an optimized charging scheme for an Electric Vehicle (EV) and a Battery Energy Storage System (BESS). The optimization aims to minimize the cost of electricity consumption while considering various constraints and parameters.
It was created on June 28, 2023 for a simple testing buliding system with several assumptions. The optimization problem in the provided code is a mixed-integer linear programming (MILP) problem. The battery can not be charging or discharging simultaneously，which is constrained by binary variables, but not included in the provdied codes.

**Prerequisites**
To run the code, you need to have the following dependencies installed:

1. Gurobi: Gurobi is a mathematical optimization solver used in this code. You can install it by following the instructions provided by Gurobi on their official website.

2. NumPy: NumPy is a library for numerical computing in Python. You can install it using pip or any other package manager.

3. Matplotlib: Matplotlib is a plotting library used to visualize the results. You can install it using pip or any other package manager.

**Code Overview**
The code consists of the following sections:

1. Importing Dependencies: The required libraries, namely gurobipy, numpy, and matplotlib.pyplot, are imported.

2. Define Data and Parameters: Daily data and parameters such as time intervals, building load data, PV solar generation data, and grid electricity price data are defined.

3. Create the Optimization Model: A Gurobi optimization model named "OptimizedChargingScheme" is created.

4. Create Decision Variables: Decision variables for EV charging power, EV discharging power, BESS charging power, BESS discharging power, grid power import, EV battery state of charge (SOC), and BESS SOC are defined.

5. Set Objective Function: The objective function is set to minimize the cost, considering grid power import, grid power export, and battery degradation cost.

6. Add Constraints: Various constraints are added to the model, including energy balance, SOC evolution for EVs and BESS, and EV discharging time constraints.

7. Optimize the Model: The model is solved using the Gurobi optimizer.

8. Check Optimization Status: The optimization status is checked, and if it is optimal, the optimized results are printed and visualized using matplotlib.

**How to Use the Code**
To use the code, follow these steps:

1. Install the required dependencies mentioned in the "Prerequisites" section.

2. Copy the provided code into a Python script or Jupyter Notebook.

3. Run the script.

**Results**
Upon successful optimization, the code prints and visualizes the optimized results, including the EV charging power, EV discharging power, BESS charging power, BESS discharging power, power imported from the grid, power exported to the grid, EV battery SOC, and BESS SOC. Additionally, it generates several plots to visualize the results:

EV Battery SOC over Time
BESS SOC over Time
Energy Purchase and Sell-back over Time
EV Charging Scheme
BESS Charging Scheme
Building Load and Solar Output
These plots provide insights into the optimized charging scheme and the overall energy consumption and generation pattern.

**Contributing**
Contributions to the code are welcome. Feel free to open issues or submit pull requests to suggest improvements or add new features.

**License**
This code is released by JG-UI on July 08, 2023. You can freely use and modify the code for both commercial and non-commercial purposes.
