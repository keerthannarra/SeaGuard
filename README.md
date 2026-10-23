# SeaGuard

SeaGuard is a maritime operational technology (OT) security demo. It combines a browser-based network playground with a Flask backend that runs cyber-attack and defense simulations, streams results to the UI, and generates PDF reports.

The frontend lets you create a maritime network topology, choose a MAPPO or IPPO simulation, and review node state, attack events, MITRE ATT&CK techniques, the kill chain, and simulation statistics.

## Features

- Interactive network topology editor with maritime node types, vulnerabilities, and alert probabilities.
- MAPPO and IPPO simulation options.
- Live simulation updates over Socket.IO, including state, attack events, and kill-chain progress.
- Scenario save/load and PDF report generation.
- Landing, About, Contact, and user manual pages.

## Project layout

```text
SeaGuard/
├── backend/                  Flask API and simulation engine
│   ├── app.py                API routes, Socket.IO events, and reports
│   ├── agents/               MAPPO and IPPO agents
│   ├── adapters/             Model, observation, and action adapters
│   ├── envs/                 CyberBattleSim environment wrappers
│   ├── models/               MAPPO model checkpoint
│   ├── scenarios/            Backend scenario and latest attack log
│   └── requirements.txt
├── frontend/                 React application built with Vite
│   ├── package.json
│   └── src/
│       ├── components/       Shared UI and simulation visualizations
│       └── pages/            Landing, Playground, About, Contact, Manual
├── scenarios/                Scenario data at the repository root
└── CyberBattleSim/           Optional location for the upstream dependency
```

## Prerequisites

- Python 3.10 or newer.
- Node.js 18 or newer and npm.
- CyberBattleSim installed in the Python environment so that `import cyberbattle` works. The `CyberBattleSim/` directory in this checkout is empty; install or provide the upstream project separately, following its installation instructions.

`backend/requirements.txt` lists Flask, Flask-SocketIO, Flask-CORS, Flask-Mail, eventlet, and PyTorch. The backend also imports ReportLab, Gymnasium, NumPy, and CyberBattleSim, which are not listed there. Install the first three in the backend environment if they are not already available through your CyberBattleSim setup; install CyberBattleSim separately.

## Run locally

Open two terminals from the repository root.

### 1. Start the backend

```bash
cd backend
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m pip install gymnasium numpy reportlab
python app.py
```

If the CyberBattleSim package is not already available in that environment, install it separately before starting the backend. The server listens on `http://localhost:5000`. Start it from the `backend/` directory: the model checkpoint and scenario files are loaded using paths relative to that working directory.

On Windows PowerShell, activate the environment with:

```powershell
.\.venv\Scripts\Activate.ps1
```

### 2. Start the frontend

```bash
cd frontend
npm install
npm run dev
```

Open the local URL printed by Vite (usually `http://localhost:5173`). The playground and contact form currently call the backend at `http://localhost:5000`.

To create a production frontend build:

```bash
cd frontend
npm run build
```

## Using the playground

1. Open **Playground** and add maritime nodes to the canvas.
2. Connect nodes to describe the topology. Adjust node properties in the interface as needed.
3. Choose **MAPPO** or **IPPO**, then start the demo.
4. Follow live node state, events, MITRE ATT&CK technique tags, and kill-chain progress.
5. Download a PDF report after the simulation.

The backend persists the current scenario and latest attack log under `backend/scenarios/`. Running another simulation replaces the latest saved scenario/log.

## Backend API and live events

The frontend uses these backend HTTP endpoints:

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `POST` | `/save_scenario` | Save the submitted scenario. |
| `GET` | `/load_scenario` | Load the last saved scenario. |
| `POST` | `/run_demo` | Start a simulation using the selected algorithm. |
| `GET` | `/download_report` | Generate and download a PDF report. |
| `POST` | `/contact` | Submit a message from the Contact page. |

The backend streams simulation events over Socket.IO. The playground listens for `state_update` and `demo_done`; the backend also emits `event_occurred`, `killchain_update`, `stats_update`, and simulation error events.

## Notes

- The MAPPO checkpoint is `backend/models/final-torch.model` and is loaded by the backend.
- The simulation environment is backed by CyberBattleSim; the playground's maritime topology is presented in the UI and is not itself a separate CyberBattleSim installation.
- Contact form delivery depends on SMTP configuration in the backend.
