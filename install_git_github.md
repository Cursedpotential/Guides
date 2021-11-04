# Initializing a Local Git Repo & Setting up GitHub

### Installing Git on Linux (Ubuntu 20.04 Used for Testing)

Download and install Git for Linux:
```shell
sudo apt-get install git
```

### Configuring GitHub
   
Once the installation has successfully completed, the next thing to do is to set up the configuration details of the GitHub user. 

GitHub has recently changed thier policies and no longer allows password based API login. Before we begin setting up our system for GitHub we have to create a pesonal access token. 

1. Log into GitHub
2. Go to settings
3. Scroll down to 'Developer Settings' in the left hand column
4. Click 'Personal Access Token' in the left column
5. Click 'Generate New Token'
6. Type a description of who/where this token is to be used and select the appropiate permissions. As well as an expiration time. For me this is not a production Repo and for testing only so I selected all the permissions and selected the token to never expire.
7. Copy newly created token and save for later. (I copied it to a Apple Note so i can refer back to it during this procedure.) 


We can now begin configuring our local Git install to communicate with GitHub. First, use the following three commands by replacing "user_name" with your GitHub username and replacing "email_id" with your email-id you used to create your GitHub account.

 ```shell
git config --global user.name "user_name"
git config --global user.email "email_id"
git config --global credential.helper.cache
```
> The third line will cache our credentials (Token) when we login the first time from our commandline so that we dont have enter it again each time we commit a change.


### Creating a local repository

Create a folder on your system. This will serve as a local repository which will later be pushed onto the GitHub website. 
```shell
mkdir git_repo
```

We will then initialize the directory with the following command

```shell
git init git_repo
```

If the repository is created successfully, then you will get the following line:
```shell
Initialized empty Git repository in /home/{user}/git_repo/.git/
```

This line may vary depending on your system.


So here, git_repo is the folder that is created and "init" makes the folder a GitHub repository. Change the directory to this newly created folder:

```
cd git_repo
```

### Creating a README file to describe the repository

Now create a README file and enter some text like "this is a git setup on Ubuntu". The README file is generally used to describe what the repository contains or what the project is all about. Example:

```
nano README
```
You can use any other text editors. I use nano. The content of the README file will be:
```
This is a git repo
```

### Adding repository files to an index

This is an important step. Here we add all the things that need to be pushed onto the website into an index. These things might be the text files or programs that you might add for the first time into the repository or it could be adding a file that already exists but with some changes (a newer version/updated version).

Here we already have the README file. So, let's create another file just to test our configuration.

```shell
touch new.file
```

So, now that we have 2 files; README and new.file, add it to the index by using the following 2 commands:

```shell
git add README
git add new.file
```

you can also use a wildcard if you want every file added to the index. This is helpful when setting up a new repo and you have added a lot of files. That looks like:

```shell
git add *
```

>Note that the "git add" command can be used to add any number of files and folders to the index. Here, when I say index, what I am referring to is a buffer like space that stores the files/folders that have to be added into the Git repository.

### Committing changes made to the index

Once all the files are added, we can commit it. This means that we have finalized what additions and/or changes have to be made and they are now ready to be uploaded to our repository. Use the command :
```shell
git commit -m "some_message"
```

"some_message" in the above command can be any simple message like "my first commit" or "edit in readme", etc.

### Creating a repository on GitHub
Create a repository on GitHub. Notice that the name of the repository should be the same as the repository's on the local system. In this case, it will be "git_repo". To do this login to your account on https://github.com. Then click on the "plus(+)" symbol at the top right corner of the page and select "create new repository". Fill the details as shown in the image below and click on "create repository" button.

Once this is created, we can push the contents of the local repository onto the GitHub repository in your profile. Connect to the repository on GitHub using the command:

```shell
git remote add origin https://github.com/user_name/git_repo.git
```

>Important Note: Make sure you replace 'user_name' and 'git_repo' in the path with your Github username and folder before running the command!

### Pushing files in a local repository to GitHub repository

The final step is to push the local repository contents into the remote host repository (GitHub), by using the command:

git push origin main

Enter the login credentials. Here is where you will need to have your token, it will prompt you for user name and password. Password is actually your Personal Access Token. If you use your GitHub Password it **WILL** fail. 

You should see your repo and files on GitHub!
