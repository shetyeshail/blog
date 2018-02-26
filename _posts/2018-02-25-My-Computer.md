---
published: false
---
My main computer is my laptop: a Lenovo ThinkPad T450s.

I dual-boot both Windows 10 and Linux on it. I need both because I enjoy using Linux more, but I do need to use Windows fairly often for programs in my engineering courses, and certain utilities/programs which only run on Windows. Sometimes I wish I had a Mac, so I could also use OS X but I'm happy with my ThinkPad.

I was using Ubuntu 16.04 for the longest time, but I started getting those annoying Ubuntu errors that seem to plague everyone. I got very fed up with it all, and decided to recently do a fresh Linux reinstall. I'll be walking through everything I do to customize it to my liking, such as any settings, configurations, applications, etc.

##Installing Elementary OS

I chose [Elementary OS](https://elementary.io/) for my Linux distro this time around, rather than the Ubuntu Gnome I have been running for the past few years. I've used Elementary "Luna" in the past and liked the aesthetics of it, and I'm now installing the latest release (at the time of this writing) which is called "Loki". Elementary is relatively straightforward to install, and uses the standard Ubuntu installer when you run as a live CD/USB. I used [Rufus](https://rufus.akeo.ie/) to create my bootable USB from the ISO image I donwloaded from the Elementary website. At this point, I made sure to back up everything on my Ubuntu partition that I would need, since fresh installing Elementary on that partition would delete all my pre-existing data.

Once I restart my computer, I can hit a key which takes me to a menu to select the drive to boot from. I select my flash drive from the list, and voila! I have booted into Elementary running on my flash drive. By clicking on "Install Elementary" on the wizard you can follow through step-by-step and select how you would like to configure it to install. I chose to install it over my existing Ubuntu 16.04 partition. I'll have to copy my data back from my backup I took.

##Configuring my Terminal & ZSH

The terminal is a very important application on any Linux system, and it can be very useful to learn. Since I spend so much time on it, it's important to me that it's setup perfectly. There's a few tweaks do whenever I setup a Linux system for myself.

I am using the standard Pantheon Terminal which comes with Elementary. Out of the box, it looks like this demo I found on the Elementary website.

![Elementary's standard terminal]({{site.baseurl}}/_drafts/elementary_terminal_standard.png)

It gets the job done, but personally, I don't like the color scheme, would prefer additional color highlighting, and a few other tweaks. Running the following commands allows us to change the color palette it uses, along with setting it to follow the last tab.

![Setting the Terminal color palette.]({{site.baseurl}}/_drafts/carbon_terminal_colors.png)


	gsettings set org.pantheon.terminal.settings palette "#070736364242:#DCDC32322F2F:#858599990000:#B5B589890000:#26268B8BD2D2:#D3D336368282:#2A2AA1A19898:#EEEEE8E8D5D5:#00002B2B3636:#CBCB4B4B1616:#58586E6E7575:#65657B7B8383:#838394949696:#6C6C7171C4C4:#9393A1A1A1A1:#FDFDF6F6E3E3"
	gsettings set org.pantheon.terminal.settings foreground "#838394949696"
	gsettings set org.pantheon.terminal.settings background "#00002B2B3636"
	gsettings set org.pantheon.terminal.settings cursor-color "#838394949696"
	gsettings set org.pantheon.terminal.settings follow-last-tab "true"


##This post is a work-in-progress.
