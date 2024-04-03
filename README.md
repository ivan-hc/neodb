This is the complete guide to creating your own application's database manageable by "[AM](https://github.com/ivan-hc/AM)" and "[AppMan](https://github.com/ivan-hc/AppMan)", the APT/PacMan/DNF style AppImage package managers.

-------------------------------
- [Step 1: create an online repository](#step-1-create-an-online-repository)
- [Step 2: create the script "neodb"](#step-2-create-the-script-neodb)
  - [Which options use the above variables?](#which-options-use-the-above-variables)
  - [Where should I place the "neodb" script?](#where-should-i-place-the-neodb-script)
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

