# Table of Contents
- [Connecting to Configuration File via VS Code](#connecting-to-configuration-file-via-vs-code)
    - [Enable Source Control in VS Code and use Git](#enable-source-control-in-vscode-and-use-git)


# Connecting to Configuration File via VS Code:
1. Open VS Code.
2. Install the "Remote - SSH" extension if you haven't already.
3. Click on the `Remote Explorer` icon in the Activity Bar on the side of the window.
4. Click on the `+` icon to add a new SSH target.
5. Enter the SSH connection details for your server (e.g., `user@hostname` or `user@IP_address`).
6. Click on the `Connect` button to connect to your server.
7. Once connected, navigate to the directory where your `working folder` is located, and open it in VS Code.

## Enable Source Control in VS Code and use Git:
1. Ensure that `Git` is installed on your server and your local machine. You can test this by running:
   ```bash
   git --version
   ```
   If Git is not installed, you can install it using the package manager for your operating system. For example, on Ubuntu, you can run:
   ```bash
   sudo apt update
   sudo apt install git
   ```
2. Initialize the Git repository in your working directory on your server:
   ```bash
   git init
   ```
3. [Optionally] Setup git identity if you haven't done so already:
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "email"
    ```
4. On your local machine, open `VS Code` -> open the `project folder` -> click on the `Source Control` icon in the Activity Bar on the side of the window -> click on `Initialize Repository` button. You can also use the below command in the terminal:
   ```bash
   git add .
   git commit -m "Initial commit"
   ```

5. To connect your local repository to a remote repository (e.g., GitHub, GitLab, Bitbucket), you need to add the remote URL. You can do this by running the following command in your terminal:
   ```bash
    git remote add origin <remote_repository_url>
    ```

6. To push your local commits to the remote repository, use the following command:
   ```bash
   git push -u origin main
   ```
   Replace `main` with the name of your branch if it's different.