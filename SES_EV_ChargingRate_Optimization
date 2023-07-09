import gurobipy as gp
import numpy as np
from gurobipy import GRB
import matplotlib.pyplot as plt


# Define the daily data and parameters
T = 24  # Number of time intervals in a day
nighttime_intervals = [0, 1, 2, 3, 4, 5, 18, 19, 20, 21, 22, 23]  # Example nighttime intervals (adjust as needed)

# Building load data (assumed to be constant for simplicity)
P_load = [110, 105, 100, 95, 90, 80, 90, 100, 110, 130, 140, 150, 160, 180, 200, 210, 190, 180, 170, 160, 150, 140, 130, 120]

# PV solar generation data
P_solar = [0, 0, 0, 0, 0, 40, 60, 80, 100, 120, 150, 180, 200, 210, 210, 180, 150, 120, 90, 60, 0, 0, 0, 0]

# Grid electricity price data
P_grid_buy = [0.12, 0.11, 0.13, 0.14, 0.13, 0.12, 0.11, 0.13, 0.15, 0.17, 0.18, 0.17, 0.15, 0.14, 0.13, 0.14, 0.15, 0.16, 0.18, 0.2, 0.19, 0.17, 0.15, 0.14]
P_grid_sell = [0.08, 0.07, 0.06, 0.05, 0.06, 0.07, 0.08, 0.09, 0.1, 0.11, 0.12, 0.13, 0.14, 0.15, 0.16, 0.17, 0.18, 0.19, 0.2, 0.21, 0.22, 0.23, 0.24, 0.25]

# Battery and EV parameters
SOC_initial_EV = 0.2   # Initial state of charge for EV
SOC_initial_BESS = 0.2   # Initial state of charge for BESS
SOC_min_EV = 0.20   # Minimum allowable SOC for EV
SOC_max_EV = 0.95   # Maximum allowable SOC for EV
SOC_min_BESS = 0.10   # Minimum allowable SOC for BESS
SOC_max_BESS = 0.95   # Maximum allowable SOC for BESS
Efficiency = 0.95   # Charging efficiency
Battery_degradation_cost = 0.01   # Cost per unit of battery degradation
Battery_degradation_factor = 0.001   # Battery degradation factor
BESS_capacity = 100   # Maximum capacity of the BESS
EV_capacity = 100   # Maximum capacity of the EV
BESS_charging_power_max = 20   # Maximum charging power for BESS
BESS_discharging_power_max = 20   # Maximum discharging power for BESS
EV_charging_power_max =20   # Maximum charging power for EV
EV_discharging_power_max =20   # Maximum discharging power for EV

# Create the optimization model
model = gp.Model("OptimizedChargingScheme")

# Create decision variables
P_ev_charge = model.addVars(T, lb=0 , ub=EV_charging_power_max , name="P_ev_charge")   # EV charging power
P_ev_discharge = model.addVars(T , lb=0 , ub=EV_discharging_power_max , name="P_ev_discharge")   # EV discharging power
P_bess_charge = model.addVars(T , lb=0 , ub=BESS_charging_power_max , name="P_bess_charge")   # BESS charging power
P_bess_discharge = model.addVars(T , lb=0 , ub=BESS_discharging_power_max , name="P_bess_discharge")   # BESS discharging power
P_grid_import = model.addVars(T , lb=0 , name="P_grid_import")   # Power imported from the grid
P_grid_export = model.addVars(T , lb=0 , name="P_grid_export")   # Power exported to the grid
SOC_ev = model.addVars(T , lb=SOC_min_EV , ub=SOC_max_EV , name="SOC_ev")   # EV battery SOC
SOC_bess = model.addVars(T , lb=SOC_min_BESS , ub=SOC_max_BESS , name="SOC_bess")   # BESS SOC

# Set objective function
obj = (sum(P_grid_import[t]*P_grid_buy[t] for t in range(T)) -
       sum(P_grid_export[t]*P_grid_buy[t] for t in range(T)) +
       Battery_degradation_cost * sum((P_bess_charge[t] + P_bess_discharge[t] + P_ev_charge[t] + P_ev_discharge[t])
                                      * Battery_degradation_factor for t in range(T)))

model.setObjective(obj , GRB.MINIMIZE)

# Add constraints
for t in range(T):
    # Energy balance constraint
    model.addConstr(P_solar[t] + P_grid_import[t] - P_grid_export[t] + P_bess_discharge[t] ==
                    P_load[t] + P_ev_charge[t] - P_ev_discharge[t] + P_bess_charge[t],
                    name=f"EnergyBalance_{t+1}")

    # SOC evolution for EVs
    if t > 0:
        model.addConstr(SOC_ev[t] == SOC_ev[t-1] + Efficiency * (P_ev_charge[t] / EV_capacity) - (1 / Efficiency) * (P_ev_discharge[t] / EV_capacity),
                        name=f"SOC_evolution_{t+1}")
    else:
        model.addConstr(SOC_ev[t] == SOC_initial_EV - (P_ev_discharge[t]/ EV_capacity), name=f"Initial_SOC_ev")

    # SOC evolution for BESS
    if t > 0:
        model.addConstr(SOC_bess[t] == SOC_bess[t-1] + Efficiency * (P_bess_charge[t] / BESS_capacity) - (1 / Efficiency) * (P_bess_discharge[t] / BESS_capacity),
                        name=f"SOC_evolution_{t+1}")
    else:
        model.addConstr(SOC_bess[t] == SOC_initial_BESS - (P_bess_discharge[t]/ BESS_capacity), name=f"Initial_SOC_bess")
        
    # EV discharging time constraint (excluding daytime)
    if t in nighttime_intervals:
        model.addConstr(P_ev_charge[t] <= EV_charging_power_max, name=f"EVcharging_{t}")
        model.addConstr(P_ev_discharge[t] <= EV_discharging_power_max, name=f"EVDischarging_{t}")
    else:
        model.addConstr(P_ev_discharge[t] == 0, name=f"NoEVDischarging_{t}")


# Optimize the model
model.optimize()

# Check the optimization status
if model.status == GRB.OPTIMAL:
    print("Optimization successful")
    print("Objective value:", model.ObjVal)

    # Get the optimized values
    P_ev_charge_opt = [P_ev_charge[t].x for t in range(T)]
    P_ev_discharge_opt = [P_ev_discharge[t].x for t in range(T)]
    P_bess_charge_opt = [P_bess_charge[t].x for t in range(T)]
    P_bess_discharge_opt = [P_bess_discharge[t].x for t in range(T)]
    P_grid_import_opt = [P_grid_import[t].x for t in range(T)]
    P_grid_export_opt = [P_grid_export[t].x for t in range(T)]
    SOC_ev_opt = [SOC_ev[t].x for t in range(T)]
    SOC_bess_opt = [SOC_bess[t].x for t in range(T)]

    # Print theoptimized results:

    print("EV charging power:", P_ev_charge_opt)
    print("EV discharging power:", P_ev_discharge_opt)
    print("BESS charging power:", P_bess_charge_opt)
    print("BESS discharging power:", P_bess_discharge_opt)
    print("Power imported from the grid:", P_grid_import_opt)
    print("Power exported to the grid:", P_grid_export_opt)
    print("EV battery SOC:", SOC_ev_opt)
    print("BESS SOC:", SOC_bess_opt)

    # Plot EV battery's state over time
    plt.figure()
    plt.plot(range(T), SOC_ev_opt, linestyle='--',marker='o')
    plt.xlabel('Time Interval')
    plt.ylabel('EV Battery SOC')
    plt.title('EV Battery State of Charge over Time')
    plt.xticks(range(T))
    plt.grid(True)
    plt.show()

    # Plot BESS state over time
    plt.figure()
    plt.plot(range(T), SOC_bess_opt, linestyle='--', marker='o')
    plt.xlabel('Time Interval')
    plt.ylabel('BESS SOC')
    plt.title('Battery Energy Storage System State of Charge over Time')
    plt.xticks(range(T))
    plt.grid(True)
    plt.show()

    # Plot energy purchase and sell-back over time
    plt.figure()
    plt.plot(range(T), P_grid_import_opt, linestyle='--', marker='s', label='Energy Purchase')
    plt.plot(range(T), P_grid_export_opt, linestyle='-.', marker='o', label='Energy Sell-back')
    plt.xlabel('Time Interval')
    plt.ylabel('Power (kW)')
    plt.title('Energy Purchase and Sell-back over Time')
    plt.xticks(range(T))
    plt.legend()
    plt.grid(True)
    plt.show()

    # Plot EV charging scheme
    plt.figure()
    plt.plot(range(T), P_ev_charge_opt, linestyle='--', marker='s', label='Charging')
    plt.plot(range(T), P_ev_discharge_opt, linestyle='-.', marker='o', label='Discharging')
    plt.xlabel('Time Interval')
    plt.ylabel('EV Charging Power (kW)')
    plt.title('EV Charging Scheme')
    plt.xticks(range(T))
    plt.legend()
    plt.grid(True)
    plt.show()
    
    # Plot BESS Charging Scheme
    plt.plot(range(T), P_bess_charge_opt, linestyle='--', marker='s', label='BESS Charging Power')
    plt.plot(range(T), P_bess_discharge_opt, linestyle='-.',marker='o', label='BESS Discharging Power')
    plt.xlabel('Time Intervals')
    plt.ylabel('BESS Charging Power (kW)')
    plt.title('BESS Charging Scheme')
    plt.xticks(range(T))
    plt.legend()
    plt.grid(True)
    plt.show()
    
    
    # Plot Building load and Solar Outputs
    plt.plot(range(T), P_load, linestyle='--', marker='s', label='Building Load')
    plt.plot(range(T), P_solar, linestyle='-.',marker='o', label='Solar Generation')
    plt.xlabel('Time Intervals')
    plt.ylabel('Power (kW)')
    plt.title('Building load and Solar Output')
    plt.xticks(range(T))
    plt.legend()
    plt.grid(True)
    plt.show()

else:
    print("Optimization failed")
