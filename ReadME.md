<!-- - - - - - - - - - - - - - - - 
Modder: Hieu
Last update: 2026.01.29

Modifications:  

- Rewrite explanation completely.  
- Explain the process and rationale in details.  
- Fix inaccurate or confusing sample code.  
- Add instructions for remote connections.  
- Add table of contents.  

To add:  
- TBD

- - - - - - - - - - - - - - - - -->


# Biol217-2026

<img src="./resources/Biol217_FIG.png" alt="Poster for biol217" width="800">


<!-- Table of Contents GFM -->

* [How to run interactive sessions](#how-to-run-interactive-sessions)
    * [Step 1 - Initiate an interactive session](#step-1---initiate-an-interactive-session)
    * [Step 2 - Run your interactive processes](#step-2---run-your-interactive-processes)
    * [Step 3 - Connect to the interactive web server](#step-3---connect-to-the-interactive-web-server)
* [How to connect to the CAU cluster from home](#how-to-connect-to-the-cau-cluster-from-home)

<!-- /Table of Contents -->


## How to run interactive sessions  

High-performance computing allows you to process a large amount of data, but command-line interfaces traditionally have very limited graphical displays. In order to visualize _**and**_ interact with your data in feature-rich graphical applications like `anvi'o`, you would need an interactive graphical environment like the everyday computer.  

Luckily, the CAU cluster does have [interactive compute nodes](https://www.hiperf.rz.uni-kiel.de/caucluster/status/#slurm-partition-interactive) in its architecture, but in order to see and click on the graphics that these interactive nodes generate, you would have to be there in person, looking at a screen that is physically connected to the compute nodes.  

Unless you follow the instructions below.  

_**Note:**_ All internative commands should be entered $\color{red}directly\ in\ the\ terminal$ (not in a batch script), because they are, well, _interactive_.  


### Step 1 - Initiate an interactive session  

Your `sunamNNN` node does not have much computing resources for graphical display. Therefore, you need to [request computing resources](https://www.hiperf.rz.uni-kiel.de/caucluster/slurm/#interactive-batch-usage) from the cluster to run interactive processes. The interactive nodes (n246-n248) are only accessible from within the CAU cluster, so after you log into your `sunamNNN` account, run:  

``` bash
srun --pty --x11 --partition=interactive --nodes=1 --tasks-per-node=1 --cpus-per-task=1 --mem=10G --time=01:00:00 /bin/bash
```

**Importantly**, write down which interactive node was allocated to you. When you run the command above, you will log into one of the interactive compute nodes as `sunamNNN@nNNN`. `nNNN` is the interactive node that the server allocated to you, and only it will display your results.  

This command also enables X11 forwarding, which allows graphical applications from a remote computer/server to be _forwarded_ to your own local computer.  

Here, the remote computer is the interactive compute node on the CAU cluster, and the "local" computer is the one that the account `sunamNNN` is using on the CAU cluster. If you logged into your `sunamNNN` account with `ssh -X`, then the graphical applications can be further forwarded from `sunamNNN`'s computer (on the CAU cluster) to the computer you are using right now in class.  


### Step 2 - Run your interactive processes  

In the terminal, activate the `anvi'o` environment:  

``` bash
module load gcc12-env/12.1.0
module load micromamba 2> /dev/null
cd $WORK
micromamba activate $WORK/.micromamba/envs/00_anvio/
```

Then, run your interactive commands, such as `anvi-display-contigs-stats`, `anvi-interactive`, or `anvi-refine`.  

The `anvi'o` interactive interface is built as a web application, so it will create a local web server to run the app. You will probably see an address like [http://127.0.0.1:8080/](http://127.0.0.1:8080/) being shown on the terminal.  

Since you are running interactive `anvi'o` from the interactive compute node `nNNN`, this web server is hosted on node `nNNN` and is local to node `nNNN` but is still remote to `sunamNNN` or to your own computer. Therefore, you now need to connect your local computer to this web server.  


### Step 3 - Connect to the interactive web server  

Since we initiated the interactive session with `srun`, the web server will be terminated as soon as we cancel the `srun` command. Thus, you have to connect to the `anvi'o` web server on node `nNNN` from _**a new terminal tab**_. Again, you will have to connect to `sunamNNN` first before having access to the interactive nodes.  

``` bash
ssh -L localhost:8080:localhost:8080 sunamNNN@caucluster.rz.uni-kiel.de
```

As `sunamNNN`, connect to node `nNNN` (both are part of the cluster):  

``` bash
ssh -L localhost:8080:localhost:8080 nNNN
```

Since we want to interact with the web app on node `nNNN` conveniently from the classroom, we need a way to _forward_ what we do with the web app locally to `sunamNNN`, and then from `sunamNNN` to node `nNNN`. That is what the [`-L`](https://www.man7.org/linux/man-pages/man1/ssh.1.html) (local port forwarding) option does.  

The first `localhost:8080` is interpreted relative to the computer that initiates the `ssh` connection, while the second `localhost:8080` is interpreted relative to the remote `ssh` connection target. With the first `ssh` command, if a program/process talks to port `8080` on your classroom computer (such as a browser opening the `anvi'o` web app), that conversation will be forwarded to `sunamNNN` at port `8080`. With the second `ssh` command, if a process talks to port `8080` on `sunamNNN`, that conversation will then be forwarded to node `nNNN`.  

Once the connections are established, you can start playing around with the data by accessing [http://127.0.0.1:8080/](http://127.0.0.1:8080/).

_**Note:**_ Check the web address carefully. If the web address is, for example, [http://127.0.0.1:8081/](http://127.0.0.1:8080/), then you have to use `ssh -L localhost:8081:localhost:8081` instead of `8080`. Sometimes you may need to use `8060` if `8080` is busy.  


## How to connect to the CAU cluster from home  

The CAU cluster can only be accessed by computers whose IP addresses are within the university network. Therefore, you must use a VPN client to make your personal device appear like a university computer. Just follow the instructions here: [DE](https://www.rz.uni-kiel.de/de/tipps/vpn) or [EN](https://www.rz.uni-kiel.de/en/hints-howtos/vpn17).  

**Note:** In order to access the CAU cluster, you will need to connect to the university network as `sunamNNN` instead of `stuNNNNNN` (unless, of course, your `stuNNNNNN` account has also been granted cluster access from other engagements). Once a VPN connection to the university network is in place, you can open your computer's terminal and log into your `sunamNNN` account as usual to run computations.  

For file transfers, you can use commands such as [`scp`](https://man7.org/linux/man-pages/man1/scp.1.html), [`rsync`](https://man7.org/linux/man-pages/man1/rsync.1.html), and [`sftp`](https://man7.org/linux/man-pages/man1/sftp.1.html) (see [here](https://tecadmin.net/transferring-files-over-ssh/), for example). If you are more GUI-inclined, you can install graphical SSH/SFTP clients to browse files visually on remote servers. Some of them even let you perform drag-and-drop magic. Below are a few options you can try:  

+ Windows-only (unless with [Wine](https://www.winehq.org/)): [MobaXterm](https://mobaxterm.mobatek.net/), [WinSCP](https://winscp.net/eng/index.php), [Solar-PuTTY](https://www.solarwinds.com/free-tools/solar-putty), or the built-in [VSCode](https://code.visualstudio.com/) terminal.  
+ Cross-platform: [Cyberduck](https://cyberduck.io/), [FileZilla](https://filezilla-project.org/?ref=sftptogo.com), [Termius](https://termius.com/).  
+ Linux-specific: mounting remote on Nautilus (the default file manager of GNOME) and Dolphin (the default file manager of KDE).  
