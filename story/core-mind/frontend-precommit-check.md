# Pre-commit with husky & lint-staged
In the most modern projects, will do some scripts to keep the team's code style uniformly, such as prettier, es-lint, style-lint and unit test and so on. Those scripts are individual, we only can run them one by one in the terminal before we commit the code, which is inconvenient.

So let's import the husky & lint-staged to help us run those scripts at the same time automatically.

## Husky
Let's set up the husky first.

Husky is a popular Git hook tool that helps developers enforce and automate certain actions and scripts before committing or pushing code to a Git repository. It works by intercepting Git commands and executing predefined scripts or actions. It supports various hooks, such as pre-commit, pre-push, pre-rebase, and more. These hooks allow developers to run tasks like linting code, running tests, formatting code, or performing other custom actions before certain Git operations.

### Step 1
Walk in [Husky website](https://typicode.github.io/husky/).

### Step 2
Install the husky with its document guide:

```
npm install husky --save-dev
```

### Step 3
Add a new script in `package.json`:
```json
{
    "script": {
        "prepare": "husky install"
    }
}
```
**Notice**, normally, this script will install a husky shell script in the root folder. **But** if your project's git is not in the **front-end folder(where package.json is)**, then you should change the script.
```json
{
    "script": {
        "prepare": "cd /path/to/where-.git-folder-is && husky install /path/to/where-husky-need-to-install/.husky"
    }
}
```
**For Example**, If my project folder constructor is like this:
```
------project
    |--.git/
    |--backend/
    |--frontend/
        |--package.json
```
In this situation, the script in the `package.json` should be:
```json
{
    "script": {
        "prepare": "cd .. && husky install frontend/.husky"
    }
}
```

### Step 4
Run `prepare` script, the husky will create a `.husky` folder where you set.
```
npm run prepare
```
In this folder, you can find a `pre-commit` file like this:
```shell
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run pre-commit
```
**Notice**, if your project's git is not in the **front-end folder(where package.json is)**, you also need to change this file:
```shell
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

cd frontend && npm run pre-commit
```
So far, the husky is set down, but I believe you may find out that we don't mention the `npm run pre-commit` because this is the next part we will go through.

## Lint-staged
In the last part, we talked about `npm run pre-commit`, so let's add the `pre-commit` script in the `package.json` first:
```json
{
    "script": {
        "prepare": "husky install",
        "pre-commit": "lint-staged" // the new tool
    }
}
```
Now you can see a new tool `lint-staged` comes up.

`lint-staged` is a tool that runs code checks during the pre-commit stage in Git. It allows you to configure code-checking tools, such as ESLint and Prettier, to run only on the staged files before they are committed. This ensures that only code that adheres to the defined code standards is committed to the version control system.

By using `lint-staged`, you can apply custom code-checking scripts to the staged files before committing. This helps improve code quality, maintain consistent code styles, and reduce code conflicts among team members.

Typically, you would install `lint-staged` in your project and define the code-checking tools and their corresponding configurations in the project's configuration file, such as package.json. Once configured, every time you run the git commit command, `lint-staged` automatically runs the specified code-checking tools on the **staged** files. If the code checks fail, the commit is blocked, allowing you to fix the issues before attempting to commit again.

In summary, `lint-staged` is a tool that facilitates running code-checking tools before committing code. It helps improve code quality and consistency and assists teams in better managing code during collaborative development.

### Step 1
First, install it:
```
npm install --save-dev lint-staged
```

### Step 2
Now you can config it in your `package.json`, here is the [guide doc](https://github.com/lint-staged/lint-staged):
```json
{
    "lint-staged": {
        "file-path": ["script1", "script2"],
        "RegExp-pattern": ["script1", "script2"],
    }
}
```
**For example**
```json
{
    "lint-staged": {
        "src/**/*.{ts,tsx}": [
            "npm run test",
            "npm run prettier"
        ]
    }
}
```

#### Notice 1
You can set multiple patterns to match different files for different scripts.

#### Notice 2
Lint-staged only works on staged files, which means it only runs the script for files that are added and changed.

### Step 3
Now you can try to commit your code to see what will happen.

## In the end
Those two tools help us to run the lint scripts before we push the code to the remote repository, to help us not let 💩 slip into our code base, they are very useful.

Thx for reading, that's all my sharing.