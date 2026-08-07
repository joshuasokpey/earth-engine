## 🚀 Live Demo

Try the MVP here:

🔗 https://earth-engine-prototype.netlify.app

> **Note:** For the best experience, use a PC/laptop. The simulation is optimized for larger screens.


# 🌍 Earth-Engine

A living ecosystem simulation project created for the **GreenRes Hackathon**.

Earth-Engine is an interactive simulation that explores how environments evolve through the interaction between nature, resources, and human activities.

## 🎮 How to Use

1. Open the link above.
2. Explore the simulation interface.
3. Interact with the environment and observe how the ecosystem changes.

That's it — the simulation is designed to be intuitive. 🌱

## ✨ Features

- 🌳 Dynamic ecosystem simulation
- 🌎 Interactive environmental modeling
- 🌱 Visualization of natural processes
- 🧩 Real-time changes based on simulation interactions

## 🛠️ Built With

Rendering

- WebGL2 — the entire 3D scene
- GLSL ES 3.00 — eight hand-written shader programs (terrain, foliage, water, sky, people, birds, rain, rigid props)
- GPU instancing — ~10,000 trees, ~660 people, 90 birds, 16,000 rain streaks
- World generation (plain JavaScript, no libraries)

- Mulberry32 — seeded PRNG, so the village is identical every load
- Value noise / fBm — terrain height, ground colour variation, cloud layer
- Catmull-Rom splines — river course, roads, dirt tracks
- Procedural geometry — buildings, bridge arches, water tower, solar arrays, canoes, parasols built from primitives at load time

Interface

- HTML + CSS — control panel, district markers, theming
- DOM ↔ 3D projection — marker positions computed each frame by pushing world anchors through the camera's view-projection matrix

Delivery

- A single self-contained .html file — no framework, no build step, no dependencies, no network requests

## 📌 Project Background

Earth-Engine was developed as part of the **GreenRes Hackathon**, with the goal of creating a digital experience that promotes awareness of environmental systems and sustainability.

## 👥 Team

- Joshua Sokpey
- Ellis Acquaye

## 📄 License

This project is for educational and hackathon purposes.

