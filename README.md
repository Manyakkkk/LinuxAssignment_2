# SHELL SCRIPTING 
---

## Overview

### Q- What is shell scripting?

It is a practice of write small commands in a command-lin interpreter that allows communication with the OS (shell) to automate tasks.
---

This assignment includes two project that focuses on automatyed system setup and user creation using Bash shell scripts. The scripts created in individual projects will help streamline administrative tasks, making it easier to manage configuration files and users on a new system.

### The projects are:

### PROJECT 1- System Setup

Includes installation of packages and symbolic linking of respective configuration files.

### PROJECT 2- New User

Includes creation of a new user with customized home directory, shell setting and group access permissions.
---

<!!!!IMAGE!!!!>

## PROJECT 1: System Setup Script

We will start by creating a script that includes user-defined list of packages that is to be installed which are ***kakoune*** and ***tmux***.

### Script 1 - *generate_list_packages*

- Here we create an array of the packages we want to install in our server.

```package = ("kakoune" "tmux")

- Then we want to create a separate .txt file and use for loop to store the list of packages in it and clearing any additional content if the file alreeady exists.

``` for package in "${packages[@]}"; do
    echo "$package" >> packages_list.txt
    echo "- $package"
done

- After this we echo a confirmation message to make sure that we know the packages list has been saved.

- Lastly call the function to generate the list by simply writing the name of the file 

``` generate_package_list

### Script 2 - *install_packages*

This script willl install the packages listed in the .txt file

1. We will start by ensuring that if our script is running as root.

We will do this by creating a fnuction- **check_root()** followed by an if statement.

```check_root() {
        if [[$EUID -ne 0]] ; then
            echo "This script must run as root"
            exit 1
        fi
}

- `$EUID` : this will store the user ID of. The root user has an ID of 0 so if this command is not equal to zero, it will display the `echo` message and will exit (indicating an error)

2. Next we will create a function to check if our .txt file exists. This will be done by using an if statement.

```check_package_list() {
    if [[ ! -f list_packages.txt ]]; then
        echo "Error: list_packages.txt not found!"
        exit 1
    fi
}

- here `-f` is a flag that checks if a file exists and if false will show the user an error message.

3. After checking the above two conditions, the user is ready to install all the packages as in the .txt file.

```install_packages() {
    while IFS= read -r package; do
        if [[ -n "$package" ]]; then
            echo "Installing $package..."
            pacman -S --noconfirm "$package" || echo "Failed to install $package"
        fi
    done < list_packages.txt
}

- `IFS`: is an Internal Field Seperator to ssplit the data wherever it finds the whitespace.
- `read -r package`: will read each line from the text file into the package variable.
- `if [[ -n "$package" ]]`: this will check that the gven string is not empty.
- `pacman -S --noconfirm "$package"`: this tells pacman to installthe specified package from the repository or update one if already installed. And `nonconfirm` will answer to all the prompted question during installation as yes.
- `||`: or


4. Then we will call each of the function in the correct oder i.e.

```check_root()
    check_package_list()
    install_pakage()

### Script 3 - *symbolic_links*

#### Q - What are symbolic links?

Answer - These are shortcuts that makes it easier to reach a file/folder from different locations on your computer.

In this script we will create symbolic links.

1. Start by storing the path of your current directory in a variable to let know the script where all of our files are located.

``` SOURCE_DIR=$(pwd)

2. Then we create symbolic links for the binary files. For this we use the following commands:

```ln -sf "$SOURCE_DIR/bin/sayhi" ~/bin/sayhi
ln -sf "$SOURCE_DIR/bin/install-fonts" ~/bin/install-fonts


- `-s`: specifies to create a symbolic link rather than a hard link
- `-f`: overwrites any existing link at the destination and creates a new one

- We are linking the files from **bin** directory in our current path with the **~/bin** directory in our home folder

- It links `sayhi` from `SOURCE_DIR/bin` to `~/bin/sayhi`. Same goes for the `install-fonts`.

3. Now we make a configuration directory to in our home directory to store all these config files. For same, we will use the `mkdir` command.

``` mkdir -p ~/.config

- `-p`: this makes sure that no error is thrown if the directory already exists.

4. Since we already created links for our binary files, we will do the same for their respective configuration files.







