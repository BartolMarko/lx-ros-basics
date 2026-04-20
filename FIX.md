Here's the adapted guide for native Ubuntu, step by step:

1. Fix the Dockerfile
bashvim ~/.duckietown/recipes/lx-recipe-ros-basics/ente/Dockerfile.vscode
Change:
dockerfileFROM ${DOCKER_REGISTRY}/duckietown/${BASE_IMAGE}:${BASE_TAG} as base
To:
dockerfileFROM duckietown/dt-vscode:daffy-amd64 as base

Also:

Find the line:
dockerfileRUN dt-apt-install /tmp/dependencies-apt.txt
And add this directly before it:
RUN wget -qO - https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add - && apt-get update


Then clean Docker's build cache:
bashdocker buildx prune -f

2. Get your virtual robot's IP
bashdocker inspect dts-virtual-<your-bot-name> | grep IPAddress
Save this IP — you'll need it everywhere below (replacing X.X.X.X).

3. Add hostname resolution on your host
bashecho "X.X.X.X   <your-bot-name>.local <your-bot-name>" | sudo tee -a /etc/hosts
On native Ubuntu this is simpler than WSL since there's no Windows cert store to worry about.

4. Set up certificates
bashdts setup mkcert
mkcert -install
On native Ubuntu mkcert -install is sufficient — no Windows import needed.

5. Launch the code editor
Make sure VNC is running on the bot first, then:
bashdts code editor --distro daffy

6. Inside the container — fix ROS master
bashexport ROS_MASTER_URI=http://X.X.X.X:11311
And add the host resolution inside the container too:
bashdocker exec -it --user root <your-vscode-container-name> bash -c "echo 'X.X.X.X   <your-bot-name>.local <your-bot-name>' >> /etc/hosts"
Your vscode container name will be something like dts-lx-ros-basics-... — check with docker ps.

7. Verify roscore is running on the bot
bashdocker exec dts-virtual-<your-bot-name> bash -c "source /opt/ros/noetic/setup.bash && rosnode list"

