# Table of Contents
- [Connecting to remote machine via VS Code and ssh](#connecting-to-remote-machine-via-vs-code-and-ssh)
- [Enable Source Control in VS Code and use Git](#enable-source-control-in-vscode-and-use-git)
- [Clone an existing repository](#clone-an-existing-repository-using-vs-code-ssh)
- [Setup Python project on a remote server using VS Code ssh](#setup-python-project-on-a-remote-server-using-vs-code-ssh)


# Connecting to remote machine via VS Code and ssh:
1. Open VS Code.
2. Install the "Remote - SSH" extension if you haven't already.
3. Click on the `Remote Explorer` icon in the Activity Bar on the side of the window.
4. Click on the `+` icon to add a new SSH target.
5. Enter the SSH connection details for your server (e.g., `user@hostname` or `user@IP_address`).
6. Click on the `Connect` button to connect to your server.
7. Once connected, navigate to the directory where your `working folder` is located, and open it in VS Code.

# Enable Source Control in VS Code and use Git:
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

# Clone an existing repository using VS Code SSH:
1. If you already have a remote repository and want to clone it to your server, you can use the following command:
   ```bash
   git clone <remote_repository_url>
   ```

2. After cloning, you can open the cloned directory in VS Code and start working on your files. You can use the Source Control panel in VS Code to manage your commits, branches, and other Git operations.

# Setup Python project on a remote server using VS Code ssh:

1. Ensure that Python is installed on your remote server. You can check this by running:
   ```bash
   python3 --version
   ```
   If Python is not installed, you can install it using the package manager for your operating system. For example, on Ubuntu, you can run:
   ```bash
   sudo apt update
   sudo apt install python3 python3-pip python3-venv
   ```

2. Create a virtual environment for your Python project:
   ```bash
   python3 -m venv .venv
   ```

3. Activate the virtual environment:
   ```bash
   source .venv/bin/activate
   ```
   You should see the virtual environment name in your terminal prompt as `(.venv) username@hostname:`.

4. Install the required Python packages for your project using `pip`. If you have a `requirements.txt` file, you can install the packages by running:
   ```bash
   pip install -r requirements.txt
   ```

5. Run the Python script:
   ```bash
   python your_script.py
   ```


