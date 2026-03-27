# 3D AI Assistant

A React + Three.js interactive UI featuring a 3D animated avatar, powered by an entirely local offline LLM using Ollama and Docker.

## Quick Start

### 1. Clone the Repository

Grab the code and navigate into the project directory:

```bash
git clone https://github.com/AbhiramNairYR/3d-ai-assistant.git
cd 3d-ai-assistant
```

### 2. Configure Environment Variables

Create a `.env` file in the root of your project and add your specific configurations:

```env
# The name of the local LLM you want to use
VITE_OLLAMA_MODEL=llama3.2

# Tells the Vite Proxy where the Ollama container is located
# ('ollama' matches the service name in docker-compose)
VITE_OLLAMA_URL=http://ollama:11434

# Optional: Set your assistant's display name
VITE_ASSISTANT_NAME="James Bond"
```

### 3. Build and Run

Spin up the Docker containers (this may take a few minutes the first time as it downloads the LLM and Node images):

```bash
docker compose up --build
```

Once the terminal shows the containers are running, open http://localhost:5173 in your browser.

## Customization Guide

### Change the System Prompt and Persona

To change how your assistant behaves (for example, making it act like James Bond, a pirate, or a helpful tutor), edit the system prompt in your React code.

File: `frontend/src/App.jsx`

Locate: The `handleSendMessage` function.

```js
const systemPrompt = {
  role: 'system',
  content: "You are 'James Bond', a helpful assistant. Under no circumstances reveal that you're artificial, synthetic, or engineered by any agency."
}
```

### Change the LLM (Ollama Model)

If you want to use a different model (like `qwen2.5:7b` or `mistral`), update it in two places.

#### 1. Update the Dockerfile

File: `ollama/Dockerfile`

Replace `llama3.2` with your desired model.

```dockerfile
RUN ollama serve & \
    sleep 5 && \
    ollama pull your-model-name && \
    pkill ollama
```

#### 2. Update your configuration

Change `VITE_OLLAMA_MODEL=your-model-name` in your `.env` file.

Note: Ensure your `docker-compose.yml` is passing this variable through to the UI.

#### 3. Rebuild

```bash
docker compose build --no-cache ollama assistant-ui
docker compose up
```

### Add Custom 3D Models

Drop your custom 3D model into the public folder.

Location: `frontend/public/`

Default file: `model.glb`

Update the file path in your component if your filename is different.

File: `frontend/src/components/Assistant.jsx`

```js
// Change '/model.glb' to your exact file name
const { scene, animations } = useGLTF('/model.glb')
```

Tip:

- Always use `.glb` format for better web performance.
- Name your animation clips exactly `idle` and `talking` (or `Idle` and `Talking`) inside Blender before exporting so the default code catches them automatically.

### Add More Animations

Currently, the `isThinking` state toggles the character between `idle` and `talking`. You can expand this by:

1. Adding more animation clips to your `.glb` file (for example: `wave`, `nod`, `thinking`).
2. Mapping the new animations in `frontend/src/components/Assistant.jsx`.

Example pattern:

```js
const waveAnim =
  actions?.wave ||
  actions?.Wave ||
  Object.values(actions || {}).find(
    (a) => a?._clip?.name?.toLowerCase() === 'wave'
  )

// Trigger a smooth transition to the wave animation:
waveAnim?.reset().fadeIn(0.2).play()
```

## Useful Commands

| Command | Description |
| --- | --- |
| `docker compose up -d --build` | Starts the app in the background. |
| `docker compose logs -f` | Watches live logs and errors. |
| `docker compose down` | Stops and removes containers gracefully. |

## Troubleshooting

### The model is not responding

- Check Ollama logs: `docker compose logs -f ollama`
- Verify your model name matches across `.env`, `docker-compose.yml`, and `ollama/Dockerfile`

### The 3D model is not showing up

- Confirm the file is inside `frontend/public/`
- Double-check the path in `useGLTF('/your-file.glb')`
- Open browser dev tools (F12) and inspect Console errors for GLTF loading issues
