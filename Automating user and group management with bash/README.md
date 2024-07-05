# Stage One Task Documentation
### Automating User and Group Management with Bash

As a SysOps engineer, efficiently managing user accounts is vital, especially when onboarding numerous new developers. This article introduces a Bash script tailored to automate the process of user creation, group assignment, home directory setup, and secure password management. Utilizing this script not only ensures consistency across user accounts but also significantly reduces the manual workload for system administrators.


### Step 1: create a log file in /var/log/user_management.log
```
sudo touch /var/log/user_management.log
```

### Step 2: create a file to save password in /var/secure/user_passwords.txt
```
sudo touch /var/secure/user_passwords.txt
```

### Step 3: Create the create_users.sh Script
1. **Create a script file**
```
touch create_user.sh
```
2. **Open the script file in a text editor**
```
vim create_user.sh
```
### Step 4: Write the script
copy and paste the following script into 'create_user.sh' file.
```
#!/bin/bash

# Define log and password files
LOG_FILE="/var/log/user_management.log"
PASSWORD_FILE="/var/secure/user_passwords.txt"

# Create log and password files if they don't exist
touch $LOG_FILE
mkdir -p /var/secure
touch $PASSWORD_FILE

# Function to log messages
log_message() {
    echo "$(date +'%Y-%m-%d %H:%M:%S') - $1" | tee -a $LOG_FILE
}

# Function to generate random password
generate_password() {
    tr -dc A-Za-z0-9 </dev/urandom | head -c 12 ; echo ''
}

# Check if the input file is provided
if [ $# -ne 1 ]; then
    echo "Usage: $0 <input_file>"
    exit 1
fi

# Read the input file
INPUT_FILE=$1

# Check if the input file exists
if [ ! -f $INPUT_FILE ]; then
    echo "Input file not found!"
    exit 1
fi

while IFS=';' read -r username groups; do
    # Remove leading and trailing whitespaces
    username=$(echo $username | xargs)
    groups=$(echo $groups | xargs)

    if id "$username" &>/dev/null; then
        log_message "User $username already exists. Skipping..."
        continue
    fi

    # Create a personal group for the user
    groupadd $username
    if [ $? -ne 0 ]; then
        log_message "Failed to create group $username."
        continue
    fi
    log_message "Group $username created successfully."

    # Create user and add to personal group
    useradd -m -g $username -s /bin/bash $username
    if [ $? -ne 0 ]; then
        log_message "Failed to create user $username."
        continue
    fi
    log_message "User $username created successfully."

    # Create additional groups if they don't exist and add user to groups
    IFS=',' read -ra group_array <<< "$groups"
    for group in "${group_array[@]}"; do
        group=$(echo $group | xargs)
        if [ -z "$group" ]; then
            continue
        fi
        if ! getent group $group >/dev/null; then
            groupadd $group
            if [ $? -ne 0 ]; then
                log_message "Failed to create group $group."
                continue
            fi
            log_message "Group $group created successfully."
        fi
        usermod -aG $group $username
        log_message "User $username added to group $group."
    done

    # Set up home directory permissions
    chmod 700 /home/$username
    chown $username:$username /home/$username
    log_message "Permissions set for home directory of $username."

    # Generate random password and store it
    password=$(generate_password)
    echo "$username:$password" | chpasswd
    echo "$username:$password" >> $PASSWORD_FILE
    log_message "Password set for user $username."

done < "$INPUT_FILE"

log_message "User and group creation process completed."

exit 0
```

![image](https://github.com/Dreyshantel/Devops-Projects/assets/109143806/abadd28d-0b25-4aeb-bf8d-2fe66b78511d)


### Step 5: Make The Script Executable
1. save and close the text editor
2. Make the script executable
```
chmod +x create_user.sh
```

### Step 6: Prepare The Users File
1.Create the users.txt file that contains the usernames and group names
```
touch employeefile.txt
```
2. open the 'employeefile.txt' in a text editor
```
vim employeefile.txt
```
3. Add user information in the format "user,group1, group2'
```
light; sudo.dev,www-data
idinma;dev
mayowa;dev, www-data
```
### Step 7: Execute the script
```
sudo bash ./create_user.sh employeefile.txt
```
![Screenshot (402)](https://github.com/Dreyshantel/Devops-Projects/assets/109143806/a7475f31-4452-4768-a605-a633ca8ad578)


Check the log file
```
 sudo cat /var/log/user_management.log
```
check the users' passwords
```
sudo cat /var/secure/user_passwords.txt
```

![Screenshot (403)](https://github.com/Dreyshantel/Devops-Projects/assets/109143806/5f207008-560d-43b1-97b3-0872b7d4e8b6)

