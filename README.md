# MLCLOUD
Scripts for using the MLCloud cluster with JupyterLab.
https://portal.mlcloud.uni-tuebingen.de/user-guide/

## Server Setup
### SSH Keys
Under the `$HOME` directory (e.g., `/home/<group>/<your-id>/`), copy the private key you have authorized into the `.ssh` folder (i.e., `$HOME/.ssh`).
This step is necessary to manually log in to the computational node from the login node.

### Setup .bashrc
For Galvani, set `$WORK`:
```sh
export WORK="/mnt/lustre/work/<group>/<user>"
```
It is also helpful to keep local installations and config files in that directory:
```sh
export XDG_CONFIG_HOME=$WORK/.config/
export XDG_CACHE_HOME=$WORK/.cache/
export XDG_DATA_HOME=$WORK/.local/share/
export XDG_STATE_HOME=$WORK/.local/state/
```
Then add local executables to `$PATH`:
```sh
if ! [[ "$PATH" =~ "$HOME/.local/bin:$HOME/bin:" ]]
then
    PATH="$HOME/.local/bin:$HOME/bin:$PATH"
fi
export PATH
```
You may also need to add CUDA binaries to `$PATH`:
```sh
PATH=$PATH:/usr/local/cuda/bin
```
It is also important to set the directory for your logs:
```sh
export LOGDIR="/path/to/logs"
```
Useful aliases:
```sh
alias v='nvim'
alias q='squeue --me'
alias p='sinfo -S+P -o "%18P %8a %20F"'
alias g='scontrol show node | grep "CfgTRES.*gres"'
```

### Essentials (neovim, uv, tmux, rsync)
```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
echo 'eval "$(uv generate-shell-completion bash)"' >> ~/.bashrc
```
```sh
wget https://github.com/neovim/neovim/releases/latest/download/nvim-linux-x86_64.tar.gz
```
For a Neovim release built with an older glibc, go [here](https://github.com/neovim/neovim-releases).

## Client Setup
Move the `mlcloud` script into a standard executable location, for example `~/.local/bin/`.
Then source it from your `.bashrc` or `.zshrc`:

```sh
source ~/.local/bin/mlcloud
```

To run jobs on the cluster, you need a config file that defines the requested resources and a command file that defines the commands to execute.
You can find the templates in `./config/config-template` and `./cmd/cmd-template`, respectively.

## Function Overview
All functions expect `-u <user>`. The cluster selector is `-g` for Galvani or `-f` for Ferranti. The optional `-l 1|2` flag selects the login node and defaults to `1`.

- `mlcloud-ln`: open an SSH session on a login node or run a command there.
```sh
mlcloud-ln -g -u <user>
mlcloud-ln -f -u <user> -l 2 squeue --me
```

- `mlcloud-job`: submit a batch job using a settings file and either a command file or an inline command.
```sh
mlcloud-job -g -u <user> -o <path/to/config> -c <path/to/jupyter/cmd>
mlcloud-job -f -u <user> -l 2 -o <path/to/config> python train.py
```

- `mlcloud-cn`: attach to the compute node of a queued or running job. With `-i`, start an interactive job first.
```sh
mlcloud-cn -g -u <user>
mlcloud-cn -f -u <user> -l 2 -i
```

- `mlcloud-jupyter`: print the Jupyter URL from the job log and open the SSH tunnel to the compute node.
```sh
mlcloud-jupyter -g -u <user>
mlcloud-jupyter -f -u <user> -l 2 my-job-name
```

- `mlcloud-sync`: copy files with `rsync`. Use `g:` or `f:` prefixes for remote paths.
```sh
mlcloud-sync -u <user> <client/path> g:<server/path> # client -> server
mlcloud-sync -u <user> -l 2 f:<server/path> <client/path> # server -> client
```

## Jupyter Notebook
To run Jupyter, create a command script like this:
```sh
<do stuff before running jupyter>
jupyter-lab --no-browser --port 8080
<do stuff after closing jupyter>
```
Then run it as a job, for example:
```sh
mlcloud-job -g -u <user> -o <path/to/config> -c <path/to/jupyter/cmd>
```
A job will be submitted.
Now run:
```sh
mlcloud-jupyter -g -u <user> job-name
```
If there is only one running job, `job-name` can be omitted.
This prints the Jupyter URL from the job log and opens the SSH tunnel to the compute node.
