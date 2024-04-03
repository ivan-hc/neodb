This is the complete guide to creating your own applications database manageable by "[AM](https://github.com/ivan-hc/AM)" and "[AppMan](https://github.com/ivan-hc/AppMan)", the APT/PacMan/DNF style AppImage package managers.

-------------------------------
- [Step 1: create an online repository](#step-1-create-an-online-repository)
- [Step 2: create the script "neodb"](#step-2-create-the-script-neodb)
  - [Which options use the above variables?](#which-options-use-the-above-variables)
  - [Where should I place the "neodb" script?](#where-should-i-place-the-neodb-script)
- [Step 3: create an installation script](#step-3-create-an-installation-script)
  - [Is it mandatory to use templates for "AM"?](#is-it-mandatory-to-use-templates-for-am)
  - [How do "AM" installation scripts work?](#how-do-am-installation-scripts-work)
- [Step 4: the list](#step-4-the-list)
- [Step 5: the markdown](#step-5-the-markdown)
- [Conclusions](#conclusions)
-------------------------------

# Step 1: create an online repository
The application installation and management modules in "AM"/"AppMan" use `wget` to download scripts and `curl` to determine their existence.

Below is the structure of a repository for x86_64 applications:
```
https://your-domain.net
  |
  |__ apps # this is a name for the directory containing the installation scripts that we will use as an example
  |    |__ app1                     
  |    |__ app2
  |    |__ appN
  |
  |__ x86_64-apps # this is a text file, the list of the apps available in the directory "apps" above, this is the name you must give to this list
  |
  |__ info # this is the directory containing the info files in Markdown format that will be managed by the option "-a" or "about"
      |__ app1.md
      |__ app2.md
      |__ appN.md
  ```
By convention, Markdorn files will have the same name as their respective script, but with the .md extension.

NOTE: You can give the directories any names you like, but the list must necessarily be called "**x86_64-apps**".

-------------------------------

# Step 2: create the script "neodb"
With the data from step 1 (above) available, here's what the "neodb" script should look like:
```
#!/usr/bin/env bash

APPSDB="https://your-domain.net/apps"
APPSLISTDB="https://your-domain.net/x86_64-apps"
AMCATALOGUE="https://your-domain.net"
AMCATALOGUEMARKDOWNS="https://your-domain.net/info"
#NEODBFLAG=nowarn
```
remember to make it executable, command `chmod a+x neodb`.

### Which options use the above variables?
- **APPSDB** is used by `-i`/`install`, `-d`/`download` and `-s`/`sync`
- **APPSLISTDB** is used by `-l`/`list` and `-q`/`query`
- **AMCATALOGUE** is used by `-h`/`help` and from the main CLI "AM"/"AppMan" itself, but only to identify **and notify** the source of the apps
- **AMCATALOGUEMARKDOWNS** is used by `-a`/`about`
- **NEODBFLAG** instead is a dumb variable, if uncommented, you will not receive any notification message about using a third-party database (see "AMCATALOGUE")

### Where should I place the "neodb" script?
The "$AMPATH" variable that you often find in the CLI and in modules indicates the path:
- For "AM" the path is always /opt/am;
- For "AppMan" instead is the "appman" directory into the path you decided to install the apps in your "$HOME", for example, if you've choosen "Applications", then the path will be $HOME/Applications/appman

-------------------------------

# Step 3: create an installation script
"AM"/"AppMan" have an inbuilt option named `-t` or `template` that takes some scripts from https://github.com/ivan-hc/AM/tree/main/templates depending on your needings.

They are structured to be modified depending on whether you use "AppMan", but by default they are scripts for "AM". Only via the `-i`/`install` option can they be adapted to "AppMan", but they can be run completely standalone for a system-wide installation ("AM") but with the path to the launchers in / usr/share/applications (and this is why it is advisable to use "AM" to install them, in case /usr/share is read-only, the path will be changed to /usr/local/share).

### Is it mandatory to use templates for "AM"?
No, if you have in mind a way to install and integrate applications in a better way, you are welcome... as long as you take into account the patches that the [install.am](https://github.com/ivan-hc/AM/blob/main/modules/install.am) module contains to modify them.

### How do "AM" installation scripts work?
For this topic, I wrote a [wiki](https://github.com/ivan-hc/AM/wiki) in the early days, when "AM" was in its infancy. But I will try to summarize as much as possible:
1. creation of the "remove" script, which lists the location of the app directory, its link/script in "$PATH" and the location of the launcher/.desktop file (it is the first step, necessary to cancel the installation if something goes wrong);
2. download of the application, the URL must be obtained via command to intercept the latest release, without further manual intervention on the script (unlike AUR's PKGBUILD);
3. place the file in the destination directory, if it is not an AppImage but an archive, the appropriate commands must be entered to extract/place/patch the files inside it;
4. create a link in "$PATH", or a script (it is better, as it can be extended with environment variables, for example by increasing the font size or forcing the system theme);
5. creation of the AM-updater script, containing functions to update the application, and most of the time includes steps 2 and 3 just mentioned;
6. creation and positioning of launcher and icon, if it is an AppImage, the script will try to extract them from the bundle itself with the "`--appimage-extract` option", otherwise the launcher will have to be included in the script and the icon searched among files extracted or downloaded.

There are also two other optional/obsolete steps, which are more oriented towards standalone use of the script, one is changing the permissions in the application directory (to allow updates with "AM" and without root privileges), the other it's a final message about the origin of the app... but no one knows "AM"/"AppMan" anyway 😉

-------------------------------

# Step 4: the list
To allow other modules to use the names from the list (for example, including them among the terms to use with "`bash-completion`" or to create other lists) this formatting is needed:
```
◆ app1 : Description for this application
◆ app2 : Description for another app
◆ appN : Also this is a description
```
NOTE the spaces around the application name. Those are important in order to separate names from symbols and description.

As for the length of the description, you are free to use as much space as you like. I usually try not to exceed 80 characters in total (maximum space occupied by default by many terminal windows).

If you used the `-t` option to create the script, the message will definitely be in the "list" file created near the x86_64 directory (on the desktop, in the "am-scripts" directory).

NOTE: remember to call the list "x86_64-apps".

In our example we are working on a 64bit/amd64 architecture. If you intend to use "AM"/"AppMan" to manage programs for other architectures, remember to name the file including that architecture (for example, on 32bit systems the file should be called "i686-apps", for ARM64 instead " aarch64-apps", and so on...). Use the "`echo "$HOSTTYPE"`" command to discover your architecture. it is already included in the main CLI (see "$arch" environment variable) to manage multiple architectures if the application database includes multiple architectures.

-------------------------------

# Step 5: the markdown
This file is also created with the `-t` option, a template can be found at https://github.com/Portable-Linux-Apps/Portable-Linux-Apps.github.io/blob/main/apps/.template , which is the same one that is used as the basic model for compiling the pages of my catalogue, https://portable-linux-apps.github.io

However, the use we have to make of it with "neodb" is to get information on the screen, from the terminal. If you don't load that file, using the `-a`/`about` option will give you an error message. This option (contained in the [database.am](https://github.com/ivan-hc/AM/blob/main/modules/database.am) module) removes the images, videos and buttons at the bottom (the ones you see in the template I linked to you), to display only the title, description and URL, but also adding the " status", i.e. whether the program is installed (and therefore also shows the occupied space) or not.

-------------------------------

# Conclusions
I started making these changes to "AM"/"AppMan" for future commitments that might keep me away from my repositories, which are now too many to handle.

If I look at my database today I can see applications that I didn't even know existed. And there are many more out there to upload that many of you would like to see in the repository.

Maybe many of you will not agree with the fact that "AM"/"AppMan" manages both portable and AppImage archives, and perhaps you would like only AppImage packages or only archived programs... it's a matter of taste.

Maybe some of you will have your own personal applications and would like a shorter and less invasive list, and a good way to integrate them into the system, without having to install daemons and runtimes of any kind without your knowledge.

Maybe many of you are wary of github and prefer to use other sites, and don't have a Reddit account or any way to contact me and say "hey, I really wish you could publish this application".

Maybe to some of you I may seem like a Tyrant, maintaining a database by myself and renaming applications in a different way than you would like.

Maybe you would like to keep a brand in the launcher names, and do not want it to be renamed with the AM extension.

Maybe, one day, my account will be deactivated (again) and you will still have a copy of "AM" or "AppMan" on your PC, to be able to draw from some database other than mine.

Maybe you would like to open an organization and allow more people to decide what to upload and what not.

Maybe you only want free and non-proprietary software, and vice versa.

Maybe... maybe... maybe...

Maybe you like "AM" or "AppMan", simply, despite everything.

I wrote them because I was tired of waiting for someone to write them for me.

"AM"/"AppMan" was born from a free choice, mine, and it is to your freedom of choice that I entrust you with what it can manage.
