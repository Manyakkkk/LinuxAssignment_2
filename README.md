# SHELL SCRIPTING 
---

## Overview 

**Q**- What is shell scripting?

**Answer** - It is a practice of write small commands in a command-lin interpreter that allows communication with the OS (shell) to automate tasks.
---

This assignment includes two project that focuses on automatyed system setup and user creation using Bash shell scripts. The scripts created in individual projects will help streamline administrative tasks, making it easier to manage configuration files and users on a new system.

### THE PROJECTS ARE:

### Project 1 - System Setup


Includes installation of packages and symbolic linking of respective configuration files.

### Project 2 - New User


Includes creation of a new user with customized home directory, shell setting and group access permissions.
---

## PROJECT 1: System Setup Script 

We will start by creating a script that includes user-defined list of packages that is to be installed which are ***kakoune*** and ***tmux***.

### Script 1 - *generate_list_packages* 

- Here we create an array of the packages we want to install in our server.

```package = ("kakoune" "tmux") ```

- Then we want to create a separate .txt file and use for loop to store the list of packages in it and clearing any additional content if the file alreeady exists.

```  
for package in "${packages[@]}"; do
    echo "$package" >> packages_list.txt
    echo "- $package"
done 

```

- After this we echo a confirmation message to make sure that we know the packages list has been saved.

- Lastly call the function to generate the list by simply writing the name of the file 

``` generate_package_list ```

---

### Script 2 - *install_packages* 

This script willl install the packages listed in the .txt file

1. We will start by ensuring that if our script is running as root.

We will do this by creating a function- **check_root()** followed by an if statement.

```

check_root() {
        if [[$EUID -ne 0]] ; then
            echo "This script must run as root"
            exit 1
        fi
} 

```

- `$EUID` : this will store the user ID of. The root user has an ID of **0** so if this command is not equal to zero, it will display the `echo` message and will exit (indicating an error)

2. Next we will create a function to check if our .txt file exists. This will be done by using an if statement.

```

check_package_list() {
    if [[ ! -f list_packages.txt ]]; then
        echo "Error: list_packages.txt not found!"
        exit 1
    fi
}

```

- here `-f` is a flag that checks if a file exists and if false will show the user an error message.

3. After checking the above two conditions, the user is ready to install all the packages as in the .txt file.

```

install_packages() {
    while IFS= read -r package; do
        if [[ -n "$package" ]]; then
            echo "Installing $package..."
            pacman -S --noconfirm "$package" || echo "Failed to install $package"
        fi
    done < list_packages.txt
}

```

- `IFS`: is an Internal Field Seperator to ssplit the data wherever it finds the whitespace.
- `read -r package`: will read each line from the text file into the package variable.
- `if [[ -n "$package" ]]`: this will check that the gven string is not empty.
- `pacman -S --noconfirm "$package"`: this tells pacman to installthe specified package from the repository or update one if already installed. And `nonconfirm` will answer to all the prompted question during installation as yes.
- `||`: or


4. Then we will call each of the function in the correct oder i.e.

```

check_root()
    check_package_list()
    install_pakage()

```

---

### Script 3 - *symbolic_links* 

**Q** - What are symbolic links? 

**Answer** - These are shortcuts that makes it easier to reach a file/folder from different locations on your computer.

In this script we will create symbolic links.

1. Start by storing the path of your current directory in a variable to let know the script where all of our files are located.

    We will also make a variable that will take symbolic link as second arguments.

``` SOURCE_DIR=$(pwd) DEST_LINK=$1 ```

2. Now we will create a function to create the symbolic links and check if the path exists.

``` 

create_symlink() {
    if [[ -e "$SOURCE_DIR" ]]; then
        if ln -sf "$SOURCE_DIR" "$DEST_LNK"; then
            echo "Successfully created symlink: $DEST_LNK -> $SOURCE_DIR"
        else
            echo "ERROR: Failed to create symlink for $SOURCE_DIR"
        fi
    else
        echo "ERROR: Source $SOURCE_DIR does not exist"
    fi
}

```

- `[[ -e "$SOURCE_DIR" ]]`: this will check this directory exists. `-e` flag returns true if that directory exists. If the source does not exist, an error message is printed.If the source does not exist, an error message is printed.
- `-s`: will create a symbolic link rather than hard link.
- `-f`: this will overwrite any existing file at the current directory. 
- If successful, it prints a success message, otherwise, it prints an error message.


3. Then we create symbolic links for the binary files. For this we use the following commands:

```
ln -sf "$SOURCE_DIR/bin/sayhi" ~/bin/sayhi
ln -sf "$SOURCE_DIR/bin/install-fonts" ~/bin/install-fonts

```


- `-s`: specifies to create a symbolic link rather than a hard link
- `-f`: overwrites any existing link at the destination and creates a new one

- We are linking the files from **bin** directory in our current path with the **~/bin** directory in our home folder

- It links `sayhi` from `SOURCE_DIR/bin` to `~/bin/sayhi`. Same goes for the `install-fonts`.

3. Now we make a configuration directory to in our home directory to store all these config files. For same, we will use the `mkdir` command.

``` mkdir -p ~/.config ```

- `-p`: this makes sure that no error is thrown if the directory already exists.

4. Since we already created links for our binary files, we will do the same for their respective configuration files.

```
ln -sf "$SOURCE_DIR/config/kak" ~/.config/kak
    ln -sf "#SOURCE_DIR/config/tmux" ~/.config/tmux

```

5. End the script by dispslaying a confirmation message saying to let user know that all the steps have done successfully and were no errors.

---

### Script - 4: *call_scripts* 

This script is designed to handle two main tasks: installing packages and creating symbolic links, based on user-provided options. It uses command-line flags to determine which task(s) to perform

1. Start by initializing three variables for our three scripts, and set them both to false. These will later be used to determine which tasks to execute based on user input.

```
generate_list_package=false
install_package=false
symbolic_link=false

```
2. Then we will start a while loop that processes the command-line options passed to the script. `getopts` is used to parse short options (`g` , `-i` , `-l`). The opt variable will store each option passed to the script.

``` while getopts "gil" opt; do ```

- After that begins the `case` statement which checks the value of `opt`, followed byblock of code sets that will set all the above created variable to true, indicating that the user wants to do that respective tasks. Like below:

``` 
    i)
    install_packages=true
    ;;

```

- We will also add a wildcard case that will handle any other options or errors. If an invalid option is passed, that function will be called to display usage instructions to the user, and end the code with `esac
done`

```
    *)
    user_guide
    ;;
  esac
done

```
3. Now we will create a code that checks if the `-g` , `-i` and `-l` option was provided by the user (indicating list generation, installation and creation of symbolic link resp) If the defined variables is set to true, the script proceeds to run the scripts. Before running it, a message is displayed indicating the start of the creation process. If neither options was provided, it calls the `user_guide` function to display usage instructions.

```

if $symbolic_link; then
  echo "Starting symbolic link creation..."
  if ./symbolic_links; then
# If this script is executed successfully, proceed
    echo "Symbolic link creation completed successfully."
  else
  # If exit code is not 0, error occured.
    echo "Error: Symbolic link creation failed."
  fi
fi

# If neither option is given by user, show user_guide to display syntax
if ! $generate_list_packages && ! $install_package && ! $symbolic_links; then
  user_guide
fi

```

# PROJECT 2 - Creating new user

1. We will check if the user is runnig as root.

2. We will define a function that will print correct usage of the script. 

```

usage() {
  echo "Usage: $0 -u username -s shell -h home_directory [-g additional_groups]"
  echo "  -u   Specify the username for the new user"
  echo "  -s   Specify the shell for the new user (e.g., /bin/bash)"
  echo "  -h   Specify the home directory for the new user"
  echo "  -g   Specify additional groups (comma-separated, no spaces)"
}

```


- `$0`: This variable holds the name of the script.

3. We will use `getopts` - a built-in command to parse command-line options.

A small code below explains how to do it:

```

while getopts ":u:s:h:g:" opt; do
   case ${opt} in 
     g)
      groups=${OPTARG} # will set the additional groups
      ;;
    \?)
      echo "Invalid option: -$OPTARG" 
      usage
      ;;
  esac
done

```

- `\?)`: Handles invalid options and calls the `usage` function.

4. Then we will make sure that `username` and `home_directory` variable is not empty because we need a name to create a new user. So if it is empty, we will throw an ERROR.

5. We also have to make sure that our shell is not empty too. So if the user has not provided with anything the script assigns a default value i.e. (/bin/bash) and will inform the user that the default value shell is being used.

6. After confirming the above conditions we will now create the user with the specified shell and home directory

```
useradd -m -d "$home_directory" -s "$shell" "$username"
useradd: A command used to create a new user.

```

- `-m`: Creates the home directory if it does not exist.

- `-d "$home_directory"`: Specifies the path to the home directory.

- `-s "$shell"`: Specifies the user's login shell.

```

7. Make sure you tell the user if the user was created successfullly.


8. Copy the contents of /etc/skel to the new user's home directory by using `cp` command

9. Then by the use of `-aG` command we will add the user to any additional groups if specified

```
 usermod -aG "$groups" "$username"

```
-a: Appends the user to the supplementary groups.

-G "$groups": Specifies the supplementary groups.

10. Setting the ownership of the home directory comes next.

``` chown -R "$username":"$username" "$home_directory" ```

11. Now we will allow user to set the password of using the `passwd` command and print them a confirmation message indicating the user was created successfully.

## CONCLUSION

Using both the **System Setup Script** and the **User Creation Script** makes it easier and faster to manage important tasks on a new system. The System Setup Scripts help by automatically installing the software we need and setting up configuration files, saving time and keeping things organized. The User Creation Script takes care of setting up new users by giving them the right permissions, groups, and a customized environment.

