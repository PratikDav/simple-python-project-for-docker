# Python Docker Practice Project

A simple Flask-based Python app I built to properly learn how Docker works — from writing a Dockerfile line by line to deploying the container on an AWS EC2 instance.

## Why I built this

I already had some background in Python, so instead of jumping into something complicated, I wanted to slow down and actually understand **what's happening inside a Dockerfile** — what each instruction does and why it's needed — rather than just copy-pasting one from a tutorial. So I kept the app itself intentionally simple and put all my focus into the containerization part.

## What the app does

Just two routes, nothing more:

- `/` → returns `Hello from python inside Docker`
- `/about` → returns `This is my Docker practice project.`

The idea wasn't to build something feature-rich — it was to have an app simple enough that Docker itself stayed the main focus.

## Dockerfile — what each line does

This is how I built the image, layer by layer:

```dockerfile
FROM python:3.x-slim          # Base image — gives the container Python out of the box
WORKDIR /app                  # Sets the working directory inside the container
COPY requirements.txt .       # Copies requirements.txt into the container
RUN pip install -r requirements.txt   # Installs the dependencies
COPY app.py .                 # Copies the app code into the working directory
EXPOSE 5000                   # Documents that the app listens on port 5000
CMD ["python", "app.py"]      # Command that runs when the container starts
```

Breaking down what I learned each keyword actually does:

| Instruction | What it does |
|---|---|
| `FROM` | Sets the base image the container is built on |
| `WORKDIR` | Sets the working directory inside the container so later commands run from there |
| `COPY` | Copies files from my local machine into the container |
| `RUN` | Runs a command *while the image is being built* — used here to install dependencies |
| `EXPOSE` | Documents which port the container listens on |
| `CMD` | The default command that runs when the container starts |

## Build & Run

```bash
docker build -t python-docker-practice .
docker run -d -p 5000:5000 python-docker-practice
```

## Deploying to AWS EC2 (and the problem I hit)

Once the container was running fine locally, I pushed it to an EC2 instance. The container itself was up and running with no errors — but when I hit the EC2 public IP in the browser, nothing loaded.

Turned out the problem had nothing to do with Docker or the app. I had forgotten to open **port 5000 in the EC2 instance's Security Group (Inbound Rules)** — by default AWS blocks external traffic to any port that isn't explicitly allowed.

**Fix:** added an inbound rule allowing TCP traffic on port 5000, and the app immediately became accessible from the public IP.

Honestly this was the most useful part of the whole project — it taught me that `EXPOSE` in a Dockerfile only documents the port, it doesn't open anything to the outside world. That's a separate, cloud-level networking step.

## What I actually learned

- How to write a Dockerfile from scratch and what each instruction is doing
- The difference between build-time steps (`RUN`, `COPY`) and the runtime step (`CMD`)
- How to build an image and run it as a container
- `EXPOSE` ≠ public access — cloud firewall/security group rules are a separate layer
- Basic troubleshooting when a deployed app doesn't behave the way it does locally

## Tech Stack

Python (Flask) · Docker · AWS EC2

## Next steps

- Add a `.dockerignore` file
- Try a multi-stage build to shrink the image size
- Set up a basic CI/CD pipeline so builds deploy automatically on push
