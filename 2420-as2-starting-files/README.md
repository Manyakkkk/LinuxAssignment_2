# SHELL SCRIPTING 
---

## Overview
Q- What is shell scripting?
It is a practice of write small commands in a command-lin interpreter that allows communication with the OS (shell) to automate tasks.
---

This assignment includes two project that focuses on automatyed system setup and user creation using Bash shell scripts. The scripts created in individual projects will help streamline administrative tasks, making it easier to manage configuration files and users on a new system.

The projects are:
PROJECT 1- System Setup
Includes installation of packages and symbolic linking of respective configuration files.

PROJECT 2- New User 
Includes creation of a new user with customized home directory, shell setting and group access permissions.
---

<!!!!IMAGE!!!!>

## PROJECT 1: System Setup Script

We will start by creating a script that includes user-defined list of packages that is to be installed which are ***kakoune*** and ***tmux***.

### Script1 - *generate_list_packages*
- Here we create an array of the packages we want to install in our server.
```package = ("kakoune" "tmux")

- Then we want to create a separate .txt file and then use for loop to store the list of packages in it and clearing any additional content if the file alreeady exists.
``` for package in "${packages[@]}"; do
    echo "$package" >> packages_list.txt
    echo "- $package"
done

