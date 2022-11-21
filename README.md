# Python Code Style Configs
This is an introduction and proposal for how to format and assert a certain style of Python code and maintain a clean git repository.
The goal of this guide is to prevent the need for you or anyone who is responsible for
maintaining the repo to revoke pushed commits or any other manual git labor.

# Using .gitignore
Probably the most important thing to mention is to use a very restrictive .gitignore.
Having someone for example to push the whole virtual environment into the
repo is probably one of the most annoying things that can happen.
Luckily GitHub provides a very good template for Python projects that can be found [here](https://github.com/github/gitignore/blob/main/Python.gitignore)

Depending on the IDEs that are used and the systems I would suggest activating the ".idea" line in the template and
to add the following lines to the .gitignore.
Also it does not hurt to ignore files that contain words like
"credentials" or "secrets" since it is not a good idea to track passwords etc. in git.
If you use VS-Code it is maybe preferable to share the project
settings but nothing else in ```.vscode```.
```

# General
.DS_Store
.AppleDouble
.LSOverride

# Icon must end with two \r
Icon

# Thumbnails
._*

# Files that might appear in the root of a volume
.DocumentRevisions-V100
.fseventsd
.Spotlight-V100
.TemporaryItems
.Trashes
.VolumeIcon.icns
.com.apple.timemachine.donotpresent

# Directories potentially created on remote AFP share
.AppleDB
.AppleDesktop
Network Trash Folder
Temporary Items
.apdisk

### VisualStudioCode ###
.vscode/*      # Maybe .vscode/**/* instead - see comments
!.vscode/settings.json
!.vscode/tasks.json
!.vscode/launch.json
!.vscode/extensions.json

### VisualStudioCode Patch ###
# Ignore all local history of files
**/.history

# End of https://www.gitignore.io/api/visualstudiocode

# Ignoring Files that contain "secrets" or "credentials"
*[Ss]ecrets*
*[Cc]redentials*
```

# Using pre-commit hooks
For ensuring similiar code style from different developers working in the same project,
a pre-commit hook that is used by everyone is highly advised.
In a pre-commit hook one can basically define scripts or tests that are executed
before the staged files are actually committed.
If tests should fail the commit hook tells one so and the commit will be aborted.
The developer gets the chance to correct the errors in his local code, before committing it.

To install pre-commit hooks in a project do the following:
1. Install pre-commit hooks in your environment with:
    ```
    pip install pre-commit
    ```

2. In projects that use pre-commit hooks,
add the following line to requirements.txt:
    ```
    pre-commit
    ```

3. Add a file called ```.pre-commit-config.yaml``` to the root of your project.
In this file you can specify what steps are executed when issuing a commit.
For a general Python project I would propose the following jobs/tests:

    ```yaml
    repos:
    -   repo: https://github.com/pre-commit/pre-commit-hooks
        rev: v2.3.0
        hooks:
        -   id: check-added-large-files
        -   id: requirements-txt-fixer
        -   id: trailing-whitespace
        -   id: end-of-file-fixer
        -   id: check-yaml
            args: [--allow-multiple-documents]
        -   id: check-xml
        -   id: check-toml
    -   repo: https://gitlab.com/bmares/check-json5
        rev: v1.0.0
        hooks:
        - id: check-json5
    -   repo: https://github.com/pycqa/isort
        rev: 5.10.1
        hooks:
        - id: isort
          name: Sorting Python imports with isort
    -   repo: https://github.com/ambv/black
        rev: stable
        hooks:
        - id: black
          name: Formatting code
          language_version: python3.10
    -   repo: https://github.com/PyCQA/flake8
        rev: 4.0.1
        hooks:
        - id: flake8
          name: Linting with flake8
    ```
    **Info about the steps**:
    - **pre-commit hooks**:  Check out the [documentation](https://pre-commit.com/hooks.html)  about what each
        step does and modify as necessary. This is pretty standard stuff. 
        I recommend to allow multiple documents in one yaml file becuase one gets many kubernetes 
        deployment files with small snippets otherwise.
    - **check-json5**: Checks if json files are valid.
        This one is fine with comments in json files,
        which the default one from pre-commit is not.
    - **isort**: Orders imports on Python files after PEP 8 guidelines.
    - **black**: Pretty strict Python formatter.
        It does not provide much stuff to configure, which
        is a blessing and a curse at the same time.
        What it does though is to prevent unecessary endless discussion
        about marginal code-style questions.
        If it is too strict or too unflexible for you, substitute it with any other
        Python formatter like [pep8](https://pypi.org/project/autopep8/).
    - **flake8**: Flake8 is a Python linter, which checks your code for
        unused imports, variables, trailing whitespaces etc.

    **About the order of the execution tasks**: The order of the executed task does
    matter for the success of the commit. The order can be generally described
    as the following:
    - Check general stuff and other file types
    - Python Code
        - Order imports (isort)
        - Format code (black)
        - Lint (flake8)

    If the linter would be exeucuted before the formatting one would probably
    get a lot of errors, which would be not ideal to say the least.
    So if you decide to swap out some stuff, keep an eye on the order of the
    tasks.

    **Note**: If you want more info on pre-commit and maybe add your own commands, 
    check out the [documentation](https://pre-commit.com/index.html).

4. Add a file called ```pyproject.toml``` to the root of your project.
isort and Black do not work hand in hand out of the box.
Some configuratiion is needed to ensure a successful integration.
For the configuration proposed in step 3. add the following to ```pyproject.toml```:
    ```toml
    [tool.black]
    line-length = 88
    exclude = '''
    /(
        \.git
    | \.hg
    | \.mypy_cache
    | \.tox
    | \.venv
    | _build
    | buck-out
    | build
    | dist
    )/
    '''

    [tool.isort]
    profile = "black"
    ```
    **About the content**: Blacks' default line-length is 88. We keep it that way
    so we can use the Black-profile provided by isort to make the integration as easy as possible.
    We also exclude some directories from formatting.

5. Add a ```.flake8``` config file to the root of your project.
Same as with isort, Black and flake8 do not work hand in hand out of the box.
To make them compatible with each other add the following lines
to your flake8 config:
    ```ini
    [flake8]
    max-line-length = 88
    extend-ignore = E203, E227
    ```
    **About the content**: Since we use the default line-length of Black which is 88,
    the number deviates from the PEP 8 max-line-length of 79.
    Also we ignore the error of "whitespace before ':' since the warning is not PEP 8
    compliant.

6. Check everything you have done until now. Your project structure should look
like the following:
    ```
    project
    │   .flake8
    │   .gitignore
    │   .pre-commit-config.yaml
    │   .pyproject.toml
    │   requirements.txt
    │   (README.md)
    │
    └───source
    │   │   __init__.py
    │   │   some_code.py
    │   │   ...
    │   │
    │   └───some_module
    │       │   __init__.py
    │       │   some_module_file.py
    │       │   ...
    │
    └───tests
        │   __init__.py
        │   test_my_code.py
        │   ...
    ```

7. Install your new pre-commit hook with:
    ```
    pre-commit install
    ```
    
8. Update the dependencies of the hook with:
    ```
    pre-commit autoupdate
    ```    
    
9. Before committing everything you should test your pre-commit hook and
validate that it does what it should with:
    ```
    pre-commit run --all-files
    ```
**NOTE**: All proposed template files can be found in this repository.

P.S.: Don't worry if the formatters and linter give you the feeling that
you are the worst dev that has ever walked on this planet.
It is always like that if you introduce linters to your project.
