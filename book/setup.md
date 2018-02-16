# Development Environment Setup
There are a few things you should have set up on your machine before moving forward with the lab.

### AWS CLI
The easiest way to get the CLI installed is via pip, the python package manager:
```bash
$ pip install awscli --upgrade --user
```
After installation, you'll need to configure the CLI to use your IAM credentials:
```bash
$ aws configure
```
Enter your credentials as prompted.  For default region and output format, you can just leave them empty.
```bash
AWS Access Key ID [None]: AKIA******************** 
AWS Secret Access Key [None]: **************************** 
Default region name [None]: 
Default output format [None]: 
```

### npm
As we'll be using Typescript to author the lambda functions, as well as Angular for the client, you'll need npm (which comes by default with nodejs).  Homebrew is easiest for mac, but there are instructions for other platforms [here](https://www.npmjs.com/get-npm).
```bash
$ brew install node
```
While you're at it, install the Angular CLI:
```bash
$ npm install -g @angular/cli
```

### Yeoman
To save some time, I've created a *Yeoman Generator* for the backend API project.  Like the Angular CLI, this will help us by initializing a project for the SAM application, saving a lot of manual configuration.

You'll need to install Yeoman itself, as well as the generator:
```bash
$ npm install -g yo
$ npm install -g generator-aws-sam-typescript
```

### Visual Studio Code
Other editors will work fine, obviously, but since we'll be working with Typescript, I highly recommend VSCode for the language support which, as far as I've seen, is unparalleled.  Get it [here](https://code.visualstudio.com/download).

You'll also want to install the Typescript plugin, but you should be prompted to do so when editing a `.ts` file for the first time.