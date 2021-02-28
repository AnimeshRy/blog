# Enable GUI Support in WSL2

There must be a good reason for anyone to switch to WSL given how much hate the Windows community gets from developers. 

I was in the same boat a while ago and was using a dual boot setup with focal Ubuntu as my daily driver. My reason for switching was simple, I needed games to play, and as we all know about the bad support of games on Linux distributions dual-booting seems like the only option. 

BUT (of course, there's a but), switching between two distributions again and again was a very big hassle that I just couldn't handle anymore. 

**Finally, I switched to WSL**, it's about 6 months since I changed to WSL and it's been an absolute blast. One of the biggest reasons was the seamless integration between WSL and VSCode which is the absolute need for my workflow.

I installed the Ubuntu flavor on windows and to my surprise, almost everything seems to work as it used to in a normal distribution. 

**Except for the GUI support**. 

I mean working as a web developer, you don't really need sound and GUI support but still, it was a void that I really wanted to fill.

According to Microsoft, new windows insider builds do have GUI support but that's just what they've been telling us that for the past year now. 

The image below is just what microsoft seems to have promised us, check it out [yourself](https://devblogs.microsoft.com/commandline/whats-new-in-the-windows-subsystem-for-linux-september-2020/).

![https://devblogs.microsoft.com/commandline/wp-content/uploads/sites/33/2020/09/gif3.gif](https://devblogs.microsoft.com/commandline/wp-content/uploads/sites/33/2020/09/gif3.gif)

So I found a few tutorials online and the common method involved using X Server on windows.

X Server was easy and you could try it but for me, it was very laggy and a lot of limitations.

Therefore I used [VNC](https://www.realvnc.com/en/) which was very fast and smooth even with animations (*at least for me it was)* 

# Guide

*I tested this on ArchWSL but I'll also try to add steps for Ubuntu too.*

### GUI

**Prerequisites**

Firstly we need some important components, for Arch you can install `gnome`, and optionally `gnome-extra`, or any application suite that you prefer, and for the Debian Based Distros select and install components using `tasksel`. 

LightDM also needs to be installed. We also need the dotnet runtime, which Arch Users can install from the AUR.

Ubuntu users need to run the following in the terminal:

```bash
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt update
sudo apt install dotnet-runtime-3.1
```

**Setting up Systemd**

WSL2 doesn’t come with Systemd so we need a workaround for that too. 

I used [Subsystemctl](https://github.com/sorah/subsystemctl) on Arch,  [Systemd-genie](https://github.com/arkane-systems/genie) is also a good option. Both of them offer PKGBUILDS for Arch. 

Here are the instructions for those on Debian for installing the later:

```bash
deb_name=`curl -s https://api.github.com/repos/arkane-systems/genie/releases/latest | grep name | grep deb | cut -d '"' -f 4`
wget https://github.com/arkane-systems/genie/releases/latest/download/$deb_name
sudo dpkg -i $deb_name
```

**Setting up VNC**

You will need `tigervnc-standalone-server` in WSL and a VNC viewer in Windows

*I used RealVNC Viewer, and I recommend you do the same, to make following this tutorial easier, but any other should also work.*

Once done we need to set the VNC passwords for the user and root. If you use a login manager other than LightDM the method will be different. I went with LightDM. You can do this by running `vncpasswd` and `sudo -H vncpasswd` (for LightDM) in WSL.

Next, we need to replace the Xorg script with one that calls Xvnc. 

Be sure to back up the existing one with `sudo mv /usr/bin/Xorg /usr/bin/Xorg_old` and create a new one with `sudo nano /usr/bin/Xorg`, pasting the following in it (*you can tweak the geometry flag in the third last line to match the resolution you want*).

```bash
#!/bin/bash
for arg do
  shift
  case $arg in
    # Xvnc doesn't support vtxx argument. So we convert to ttyxx instead
    vt*)
      set -- "$@" "${arg//vt/tty}"
      ;;
    # -keeptty is not supported at all by Xvnc
    -keeptty)
      ;;
    # -novtswitch is not supported at all by Xvnc
    -novtswitch)
      ;;
    # other arguments are kept intact
    *)
      set -- "$@" "$arg"
      ;;
  esac
done

# Here you can change or add options to fit your needs
command=("/usr/bin/Xvnc" "-geometry" "1024x768" "-PasswordFile" "${HOME:-/root}/.vnc/passwd" "$@") 

systemd-cat -t /usr/bin/Xorg echo "Starting Xvnc:" "${command[@]}"

exec "${command[@]}"
```

Remember to mark it as an executable with `sudo chmod 0755 /usr/bin/Xorg` .

## **Starting it up**

If you used Subsystemctl use the following commands to start it up

```bash
sudo Subsystemctl start
Subsystemctl shell
```

Otherwise for systemd-genie

```bash
genie -i > /dev/null &
genie -s
```

Also we need to start LightDM and enable it (so we don’t have to repeat this)

```bash
sudo systemctl start lightdm.service
sudo systemctl enable lightdm.service
```

Now you can head over to your VNC viewer, connect to `localhost:5900` and enter the password you set for VNC (the second one), and Voila you have your GUI (although it's not pretty right now, we’ll do that in another post). You can log in to use Gnome (or whatever WM you installed). 

Also, ***if you’re colors are messed up you need to change the output quality in your VNC Viewer’s settings.***

<details>
  <summary>Next Post</summary>
    I am currently looking in mutliple places for implementing sound and will post about the best method I found in enabling sound support for wsl.
</details>
<br>

This article was posted on the Official Meusec Website. Check it [out!](https://www.meusec.com/how-to/enabling-gui-support-in-wsl2-kick-off-vmware/)
