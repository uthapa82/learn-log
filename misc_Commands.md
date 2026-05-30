**uv package manager**

```bash
    # Initialize the project, default --app 
    $ uv init 
    or 
    $ uv init new_app

    # flag to create distributable python library 
    $ uv init new_app --lib

    # pyproject.toml --> modern way to configure python project 

    # upon installing the following packages uv updated the toml dependencies list 
    # as well as as creates uv.lock files which records the exact version of all the
    # packages currently installed, ensures our environment is perfectly reproduceable

    $ uv add flask requests

    # visualize dependencies, directory structure 
    $ uv tree 

    # running program, automatically selects the appropriate virtual environment 
    # even if the virtual environment doesnot exist it will recreate virtual env 
    $ uv run main.py   

    # reproducing same environment in another machine based on lock file 
    $ uv sync 

    # remove the dependencies, update the lock file and toml file  
    $ uv remove flask 

    # using pip along with uv 
    $ uv pip flask 
    $ uv pip list 

    # add from requirements.txt file 
    $ uv add -r requirements.txt 

    # linting using ruff, can use globally for any projects 
    $ uv tool install ruff 
    $ which ruff 
    $ ruff check 

    # remove a tool
    $ uv tool uninstall ruff 

    # just to test tool, without even installing it 
    $ uv tool run ruff check 
    or 
    $ uvx ruff check

    $ uv tool list 

    $ uv tool upgrade --all 
```

* Smart global caching system 

***

* To include the env folder to gitignore 
    
    `$ echo 'env' > .gitignore`

* Store the installed packages in requirements.txt file 
    
    `$ pip freeze > requirements.txt`

* Install the requirements required to run the project 
    
    `$ pip install -r requirements.txt`

* ip configuration on windows device 
    
    `C:>netsh interface ipv4 set address name="Ethernet" static <ip-address> <subnet-mask> <gateway>`

* Change DNS server on windows device 
    
    `C:>netsh interface ipv4 set dns name="Ethernet" static <ip-address> `

* save the output of ipconfig in a file Windows 

    `> ipconfig /all >$env:USERPROFILE:DESKTOP\ipconfig.txt`


