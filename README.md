# Dockerized Flask Cat GIF App
<img width="713" height="801" alt="image-12" src="https://github.com/user-attachments/assets/a3f2b040-5965-4110-853a-15ac244df145" />


## Project Overview


A fun little Flask App that displays a random cat gif every time it is loaded, because you know, who doesn’t like cats? built as a hands-on exercise in writing a Dockerfile from scratch, layer caching, and running a Python web app inside a container.

Every real-world containerised service, a microservice on ECS, a batch job on Fargate, an API behind a load balancer, starts from the exact same skeleton as this project: a base image, a dependency install step, an app copy step, an exposed port, and a start command. Getting comfortable with this pattern on a tiny, low-stakes app makes it second nature when the app itself gets more complex.

## Architecture Workflow


1. **Base Image** — The Dockerfile starts `FROM python:3.12-slim`, a lightweight, official Python 3.12 image with the OS-level packages needed to run Python already baked in.
2. **Working Directory** — `WORKDIR /usr/src/app` sets the directory inside the container where all subsequent commands run and files are copied to.
3. **Dependency Layer** — `COPY requirements.txt .` followed by `RUN pip install` happens *before* the rest of the app code is copied in. This is the key layer-caching trick: Docker only re-runs `pip install` when `requirements.txt` changes, not on every code edit.
4. **App Layer** — `COPY . .` copies the remaining files (`app.py`, `templates/`) into the image.
5. **Exposed Port** — `EXPOSE 5000` documents that the container listens on port 5000 (this is metadata only — it doesn't actually publish the port; that happens at `docker run` time with `-p`).
6. **Container Start** — `CMD ["python", "./app.py"]` is the command that runs when the container starts, launching the Flask development server.
7. **Random GIF Logic** — Inside `app.py`, Flask's `render_template` renders `index.html`, injecting a URL randomly picked from a hardcoded list of cat GIFs hosted on Firebase Storage. Every page refresh re-runs the route handler, so a new random GIF is picked each time.

## Tools & Services Used


- **Docker** — builds and runs the containerised app
- **Python 3.12 (slim base image)** — runtime language for the Flask app
- **Flask** — lightweight Python web framework serving the HTTP route
- **Docker Hub** *(optional)* — registry to publish the built image publicly

## Preparation



The project consists of four files:

- **`Dockerfile`** — instructions for building the image: base image, working directory, dependency install, app copy, exposed port, and startup command
- **`app.py`** — the Flask application; defines a single `/` route that picks a random GIF URL from a list and renders it into the HTML template
- **`requirements.txt`** — pins the exact Flask and Werkzeug versions needed, so the image is reproducible
- **`templates/index.html`** — the HTML page Flask renders, styled with simple inline CSS, displaying the chosen GIF

## Steps



### **1. Write the Dockerfile**

Defined a lightweight Python 3.12 slim base image, set the working directory, copied `requirements.txt` separately and installed dependencies with `pip install --no-cache-dir` (the `--no-cache-dir` flag skips caching pip's own download cache inside the image layer, keeping the image smaller), then copied the rest of the app code, exposed port 5000, and set the container's startup command.

<img width="690" height="415" alt="image-10" src="https://github.com/user-attachments/assets/3ca02431-093b-4a13-8490-7025fd9ab557" />


### **2. Write the Flask application (`app.py`)**

The app defines a list of cat GIF URLs and a single route (`/`) that uses Python's `random.choice()` to pick one URL each time the page loads, then passes it into the `index.html` template for rendering. The app also reads a `PORT` environment variable (defaulting to 5000) so the listening port can be overridden at runtime without changing code.

<img width="1261" height="684" alt="image-11" src="https://github.com/user-attachments/assets/98d4f79a-a88d-4d03-b097-f42811ce9cfa" />


### **3. Build the Docker image**

```
docker build -t flask-catgif-image:v1 .
```

This reads the Dockerfile in the current directory (`.`) and builds an image tagged `flask-catgif-image:v1`. The `-t` flag assigns a human-readable name and version tag instead of relying on the auto-generated image ID.

### **4. Run the container**

```
docker run -d -p 8888:5000 --name flask-catgif-container flask-catgif-image:v1
```

`-d` runs the container in detached mode (in the background). `-p 8888:5000` maps port 8888 on the host machine to port 5000 inside the container — this is why the app is reachable at `localhost:8888` even though Flask itself listens on 5000. `--name` gives the running container a friendly name for later `docker stop`/`docker logs` commands.

## Validation & Testing

---

#### **1. Verify the container is running**

Confirms the container's status, name, and port mapping.

```
docker ps
```

#### **2. Load the app in a browser**

Opened http://localhost:8888 and refreshed the page multiple times to confirm a different random cat GIF loads each time. Proof the `random.choice()` logic and port mapping both work end-to-end.

<img width="713" height="801" alt="image-12" src="https://github.com/user-attachments/assets/09956da0-b706-46b2-ae73-06912952ee17" />


#### **3. Check container logs**

Useful if the page fails to load — shows the Flask server's startup output and any request errors.

```
docker logs flask-catgif-container
```


