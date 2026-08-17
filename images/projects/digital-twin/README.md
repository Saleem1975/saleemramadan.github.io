# Two-Tank Mixing System Digital Twin

A modular Python and Streamlit minimum working digital twin for two independent mixing tanks. Each tank contains a centrifugal pump, agitator motor, agitator-shaft bearing, electrical heater, process model, virtual sensors, and fault schedule.

This release covers **Phase 1 engineering design** and **Phase 2 minimum working implementation**. It deliberately does not contain machine-learning predictive-maintenance models.

## Main capabilities

- Independent Tank 1 and Tank 2 simulation
- Tank level mass balance and lumped temperature energy balance
- Pump affinity-law approximation
- Agitator motor electrical, mechanical, vibration, and thermal variables
- Bearing lubrication, friction, wear, vibration, and thermal variables
- On/off or proportional heater control
- True values and noisy virtual sensor measurements
- Sudden, gradual, intermittent, random, and permanent fault profiles
- Start, pause, resume, stop, reset, and single-step controls
- Live Plotly diagrams and trends
- Synchronized data logging and CSV export
- JSON configuration download, upload, and restore
- Common data-source protocol for future MQTT, OPC UA, Modbus, serial, REST, or CSV replay adapters

## 1. Install Python

Use Python 3.11 or later. Python 3.11 or 3.12 is recommended for the broadest package compatibility.

## 2. Create a virtual environment

From the project directory:

### Windows PowerShell

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### Windows Command Prompt

```bat
python -m venv .venv
.venv\Scripts\activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### macOS or Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

## 3. Run the application

```bash
streamlit run app.py
```

Streamlit will open the dashboard in the default browser. If it does not, copy the local URL displayed in the terminal.

## 4. First validation exercise

1. Open **Overview**.
2. Press **Single step** and confirm that two synchronized records are created.
3. Press **Start** and let the model run.
4. Confirm that both levels remain stable because the default process inlet and outlet flows are zero. The pumps operate as recirculation/transfer-loop assets by default and therefore still produce monitored flow.
5. Open **Faults** and add `PARTIAL_BLOCKAGE` to Pump 1 with 60% severity and a start time equal to the current simulation time.
6. Return to **Overview** and confirm that Pump 1 monitored flow decreases while Tank 2 remains unaffected.
7. To model a feed pump, open **Parameters**, select **Use pump delivered flow as tank inlet**, and set a matching outlet flow for stable-level operation.
8. Export the combined CSV from **Data & configuration**.

## 5. Run automated tests

```bash
pytest -q
```

The tests verify level balance, heater response, pump blockage and cavitation symptoms, motor load behavior, bearing lubrication effects, two-tank logging, and CSV output.

## Folder guide

- `app.py`: Streamlit orchestration only
- `simulation/`: physics, equipment, faults, sensors, and two-tank coordinator
- `interface/`: diagrams, parameter forms, fault forms, status cards, and charts
- `storage/`: CSV logging and JSON validation
- `config/`: default operating parameters and reserved engineering limits
- `docs/PHASE1_ENGINEERING_DESIGN.md`: architecture, assumptions, equations, sensor map, fault matrix, data dictionary, UI design, and validation plan
- `tests/`: automated engineering-behavior tests
- `data/generated_data/`: optional automatic CSV output
- `data/saved_configurations/`: location reserved for user configurations

## Important scientific limitation

The tank mass and energy balances are physics-based at a lumped level. The default configuration treats each pump as a monitored recirculation/transfer-loop pump; a checkbox can instead make its delivered flow the tank inlet. The pump, motor, bearing, heater-fault, vibration, and sensor-disturbance relationships are simplified engineering approximations. They generate internally consistent synthetic data but are not a validated representation of a particular industrial asset.

Before operational use, calibrate the model with actual tank dimensions, fluid properties, pump curves, motor nameplate and efficiency data, heater specifications, bearing data, sensor specifications, process measurements, maintenance records, and failure history.

## Common problems

### `streamlit` is not recognized

Activate the virtual environment, then run:

```bash
python -m streamlit run app.py
```

### The dashboard does not update automatically

Confirm that the installed Streamlit version satisfies `streamlit>=1.59`. Run:

```bash
python -c "import streamlit; print(streamlit.__version__)"
```

### A configuration does not load

Only load trusted JSON files that retain the root keys `simulation` and `tanks`, with exactly two complete tank entries. Download the current configuration first and edit that copy.

### Temperature changes slowly

The default tanks contain thousands of liters of water-like liquid. A 10–12 kW heater therefore changes bulk temperature gradually, as expected from the energy balance. Reduce the initial level for a faster demonstration, but keep the physical interpretation in mind.
