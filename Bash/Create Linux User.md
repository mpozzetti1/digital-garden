#bash 

```bash
#!/bin/bash

# Get the new username
read -p "Enter the new username: " new_username

# Check if the user already exists
if id -u "$new_username" >/dev/null 2>&1; then
    echo "User $new_username already exists."
    exit 1
fi

# Become root for privileged operations
sudo su -  << EOF

# Create the user
useradd -m -s /bin/bash "$new_username"

# Create the .ssh directory
mkdir -p /home/"$new_username"/.ssh

# Create the authorized_keys file
touch /home/"$new_username"/.ssh/authorized_keys

# Generate the SSH key pair (no passphrase, keyfile named after the user)
ssh-keygen -t rsa -b 4096 -f /home/"$new_username"/.ssh/"$new_username" -q -N ""

# Append the public key to authorized_keys
cat /home/"$new_username"/.ssh/"$new_username".pub >> /home/"$new_username"/.ssh/authorized_keys

# Set appropriate permissions
chmod 700 /home/"$new_username"/.ssh
chmod 600 /home/"$new_username"/.ssh/authorized_keys
chown -R "$new_username":"$new_username" /home/"$new_username"/.ssh

# Add the user to the sudoers file (using visudo for safer editing)
echo "$new_username  ALL=(ALL:ALL) NOPASSWD: ALL" | EDITOR='tee -a' visudo
cat /home/"$new_username"/.ssh/"$new_username"
rm -rf /home/"$new_username"/.ssh/"$new_username"

EOF
```