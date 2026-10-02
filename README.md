# Unity Docker Compose

A reusable Docker Compose scaffold for running the Unity Editor remotely on a Linux server.

The project provides a containerized Unity development environment that can be accessed through a web browser, allowing Unity projects to be developed remotely without installing the Unity Editor on the local development computer.

## Purpose

This repository is intended to act as a starting point for remote Unity projects.

The Unity Editor, Android development environment, graphical desktop, and browser-based remote access are handled inside Docker while the Unity project files remain stored on the host.

This allows the same development environment to be reused across multiple Unity projects.

## Features

- Unity Editor running inside Docker
- Browser-based Unity Editor access
- noVNC remote desktop
- Docker Compose based workflow
- Host-mounted Unity project files
- Unity C# script recompilation
- Android Build Support
- Persistent Unity project data
- Linux GPU access through `/dev/dri`
- No Unity installation required on the client computer
- Suitable for remote and headless Linux servers

## Requirements

The host system requires:

- Docker
- Docker Compose
- Linux
- A Unity license
- A web browser on the client device

For hardware accelerated graphics, the host should expose:

```text
/dev/dri
```

## Unity Version

The environment currently uses:

```text
Unity 6000.3.25f1
```

with the UnityCI Android editor image:

```text
unityci/editor:6000.3.25f1-android-3.2.2
```

The Android image includes the Unity Android build environment required for producing Android builds.

## Project Structure

```text
.
├── docker-compose.yml
└── src/
    ├── Assets/
    ├── Packages/
    └── ProjectSettings/
```

The `src` directory contains the Unity project and is mounted into the Unity development container.

Unity-generated project data can remain local to the development environment rather than being committed to Git.

## Starting the Environment

Start the Unity development environment:

```bash
docker compose up -d
```

View running services:

```bash
docker compose ps
```

Follow the Unity container logs:

```bash
docker compose logs -f unity
```

## Accessing Unity

The Unity Editor is available through noVNC in a web browser.

Open:

```text
http://<server-ip>:5400
```

The remote display stack provides browser access to the graphical Unity Editor without requiring a VNC client on the development computer.

Internally, the environment uses:

```text
Unity Editor
    ↓
X11 display
    ↓
x11vnc
    ↓
websockify / noVNC
    ↓
Web browser
```

## Development Workflow

Source files can be edited from outside the Unity container using tools such as VS Code or a remote SSH development environment.

Unity works directly against the mounted project under:

```text
src/
```

When C# scripts are changed, Unity detects the project changes and recompiles the scripts.

A typical workflow is:

```text
Edit code
    ↓
Save changes
    ↓
Unity detects changes
    ↓
C# scripts compile
    ↓
Unity reloads the project
    ↓
Test in the Editor
```

The Unity Editor does not need to be restarted for normal script changes.

## Remote Development

The environment is designed so that the development computer does not need to run Unity locally.

A typical setup is:

```text
Development Computer
├── Browser
├── VS Code
└── SSH
        │
        ▼
Linux Development Server
├── Docker
├── Unity Editor
├── Unity project
├── Android build environment
└── noVNC
```

The browser is used when graphical Unity Editor access is required, while most code can be written directly through the remote development environment.

## Android Development

The Unity image includes Android Build Support.

This allows Android builds to be created on the remote Linux server without installing the Android Unity modules on the client computer.

Unity projects can therefore be developed and built for Android entirely from the remote environment.

## Stopping the Environment

Stop and remove the running container:

```bash
docker compose down
```

Project files under `src/` remain on the host and are not removed when the container is stopped.

## Restarting

Restart the environment:

```bash
docker compose restart
```

Or recreate it:

```bash
docker compose down
docker compose up -d
```

## Using as a Scaffold

The repository is intended to be reused for new Unity projects.

Clone the scaffold:

```bash
git clone <repository-url> my-unity-project
cd my-unity-project
```

Remove the existing Git history:

```bash
rm -rf .git
git init
```

The `src` directory can then be used for the new Unity project while retaining the existing remote development environment.

## Architecture

The environment is split between the client computer and the remote Linux development server.

### Client Computer

The client is used for:

- Browser access to the Unity Editor through noVNC
- Source code editing
- SSH access
- Git operations
- General project management

Unity itself does not need to be installed on the client.

### Linux Development Server

The server is responsible for:

- Running Docker
- Running the Unity development container
- Storing the Unity project files
- Running Unity imports and compilation
- Producing Android builds
- Providing GPU access to Unity
- Hosting the remote graphical session

### Unity Container

The Unity container provides:

- Unity Editor
- Android Build Support
- Android SDK, NDK, and OpenJDK
- X11 graphical environment
- x11vnc
- websockify
- noVNC

The Unity project is mounted from the host into the container, allowing the project files to persist independently of the container lifecycle.

### Remote Access

The Unity graphical session is exposed through noVNC.

The browser connects to the noVNC service, which forwards input and display data to the VNC server running inside the container.

This allows the complete Unity Editor to be controlled remotely without requiring a native VNC client.

### Project Storage

The Unity project is stored on the Linux host under:

```text
src/
## Project Philosophy

The goal of this scaffold is to separate the development workstation from the resources required to run Unity.

The server handles the Unity Editor, builds, project imports, and graphical environment while the client computer is primarily used for code editing and browser access.

This makes it possible to maintain a consistent Unity development environment while keeping Unity development workloads off the primary computer.
