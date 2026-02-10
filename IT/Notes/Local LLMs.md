Just some quick notes on my experience running local LLM's with Ollama and Open WebUI on fedora Linux.

# Why Local?
Running AI's locally actually has a lot of downsides but with the recent updates to Open WebUI it might actually be worth it now(***Spoiler:** its not*).

## Pros and cons
Pros and cons of running your LLM's locally vs using ChatGPT, Gemini 3 or other public models.

**Pros:**
- **Privacy:** Using a public LLM is using your data for training when running locally you dont have that problem. Its not 100% confirmed but sending private information to a AI chatbot might result in that data being compromised in the future you can prevent that when running your local LLM as the data isnt being used for training.
- **Freedom:** You can configure your AI freely by being able to change the system prompt, voice, web search and much more. Open WebUI also makes this process a lot simpler and gives you a ton of options to create your own personalized models and change existing models to your preferences. Local LLM's will also be a lot more likely to answer *any* question with the right system prompt.
- **Integrations:** You can add a lot more integrations with Open WebUI for example being able to add your personal knowledge base or even custom scripts that the AI can use.

**Cons:**
- **Intelligence:** Public models like GPT-5.2 have a way higher parameter count than the ones we can run locally and are a lot smarter than what we have locally. They are way less likely to hallucinate basic facts and can give you way better responses with basic prompts.
- **Hardware Requirements:** Due to the recent rise in RAM prices running a local LLM has become way more expensive than it used to and to run smarter models(100B+ Parameters) you would need a rig worth thousands of euros the usual AI rig costs around 5000€ but usually even more. You can still run dumber models(around 20B Parameters) with lower end hardware but the lower the parameter count the dumber the model.

# Setup
The hardware and software side of my personal AI server. *Its not perfect but it works.*

## Hardware
my hardware isn't the best and probably the reason why I had struggles finding an actual use for my local LLM's. You can expect pretty mediocre results when running the same or a similar rig.

- **GPU:** `RTX3060 12GB VRAM` *12GB can barely fit a 20b model but running 8b models is a lot more reliable*
- **CPU:** `R5 5600X` *Running AI on the CPU is really slow using the GPU is smarter*
- **RAM:** `32GB DDR4 at 3200MT/s` *Solid RAM but useless unless running the AI on CPU*

## Software
Now finally the interesting part of this setup, I of course went with ollama for actually running the LLM's as its the easiest to work with and directly integrates with Open WebUI. Speaking of Open WebUI that's the application I use for managing and configuring AI's it offers a pretty extensive webui with options for changing system prompts, managing multiple users in case its set up as a centralized server and much more. Also this offers integration with different API's so theoretically you could use ChatGPT with it which is something I defenetly wanna look into. 

### Ollama
here is a quick guide on how I setup ollama on fedora Linux
> **Don't install via dnf** the dnf version for some reason doesn't setup the `ollama.service` file which we need for this project

**Using the official install script:**
*just copy paste this command from [ollama.com](ollama.com)*
```sh
curl -fsSL https://ollama.com/install.sh | sh
```

Next we need to make sure that Open WebUI can see our ollama server for that we need to edit the `ollama.service` file
```sh
sudo nano /etc/systemd/system/ollama.service
```

Then add this line to the `[Service]` Part:
```
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```

*Also make sure the service is enabled*
```sh
sudo systemctl enable --now ollama.service
sudo systemctl status ollama.service
```

### Open WebUI
Open WebUI offers the best experience when run inside of a container so we need docker. If you haven't setup docker yet you can check they're [Official guide](https://docs.docker.com/engine/install/fedora/)

For setting up Open WebUI I would also recommend using they're [Official guide](https://github.com/open-webui/open-webui?tab=readme-ov-file#quick-start-with-docker-) on GitHub

I would make one change though and that is to create a desktop shortcut for launching the container so it doesn't have to run in the background all the time. 
I made this small script to streamline the process of launching the Open WebUI container:
```bash
#!/bin/bash

CONTAINER="open-webui"
URL="http://localhost:3000"
ICON="open-webui-light"

# Check if the container is currently running
IS_RUNNING=$(docker inspect -f '{{.State.Running}}' $CONTAINER 2>/dev/null)

if [ "$IS_RUNNING" != "true" ]; then
    # Only notify and wait if we actually have to start it
    notify-send "Open WebUI" "Launching container..." --icon=$ICON
    
    docker start $CONTAINER

    # Wait for the backend to stop throwing 500 errors
    until $(curl -s -f -o /dev/null $URL); do
        sleep 1
    done
fi

# Open the site (instant if already running, or after the loop if just started)
xdg-open $URL
```

Now create this file and replace the path in the Exec line and the user in the User line:
`/home/porotscho/.local/share/applications/open-webui.desktop`
```
[Desktop Entry]
Version=1.0
Type=Application
Name=Open WebUI
Comment=Start Open WebUI
Exec=/path/to/launch-open-webui.sh
Icon=open-webui-light
Terminal=false
Type=Application
Categories=AI;
User=YOURUSERNAME
```

Now grab the Icon from [here](https://dashboardicons.com/icons/open-webui) and place it in `/home/porotscho/.local/share/icons` 