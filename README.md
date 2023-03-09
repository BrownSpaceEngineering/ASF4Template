# bse-fsw-template
A template project for BSE flight software projects. Built to work with Atmel START generated ASF4 projects, on the Atmel SAMD21J18A processor.

Note: the Atmel START library in this repo is the bearbones minimal configuration; projects using this template should add required components and libraries.

## Building
Make sure you have the toolchain installed (see below)
Run `make`. The produced binaries will end up in `build/`

## Uploading to, running & debugging microcontroller
To upload your code to the microcontroller, and then run or debug it using gdb:
1. Make sure you've built the code with `make`
2. Make sure you're connected to the board or debugger
3. From within this project directory, start `openocd`
4. In a new terminal within this project directory, run `make connect` (it's fine if an error about "No executable specified" comes up)
5. At the `(gdb)` prompt run `load` to upload the binary to the microcontroller
6. Run `monitor reset halt` to reset the microcontroller and start debugging
7. Type `c` to run (continue) the program
   - If you want to step through your code, set any breakpoints you need before this (ex. via `break main`)

Tips: 
- To halt the code and get back to the debugger, do `Ctrl-C`
- To rebuild the code without closing gdb, run `make` at the `(gdb)` prompt and repeat steps 5-7

## Installing the Toolchain
### Windows
Windows toolchain installation works by using WSL to build and run gdb, and the windows OS to run openocd.
When building, run openocd on the windows host system, and everything else (make/gdb) inside of WSL

###Inside WSL:
1. Ensure you have WSL installed, or install it by running PowerShell as Administrator and typing `wsl --install`
2. Once inside WSL, clone the repository into the WSL filesystem rather than in the windows filesystem (it's over 10x faster this way, but can work in the windows filesystem as well)
3. Edit the `openocd.cfg` file in the repository, and add the line: `bindto 0.0.0.0` at the very top.
4. run the `hostname` command and take note of the output
5. Edit the `Makefile` file in the repository, and replace every instance of "localhost" with "<hostname>.local" (where <hostname> is the output from the `hostname` command)
6. Install all nesecary packages by running `sudo apt install gcc-arm-none-eabi gdb-multiarch make build-essential -y`
7. run `sudo ln -s /usr/bin/gdb-multiarch /usr/bin/arm-none-eabi-gdb`
8. run `make` inside the repository you wish to build -- With any luck it should build without any errors!
Notes: You can run VScode from inside wsl by typing `code .`

###In Windows:
1. Download and extract the newest Windows version of OpenOCD from [here](https://github.com/xpack-dev-tools/openocd-xpack/releases).
   - Make sure to add the bin folder to your path (e.g. `C:\Users\<username>\<path>\openocd-0.10.0\bin`). 
   - If you have trouble with that version you can also use [this site](http://www.freddiechopin.info/en/download/category/4-openocd) (you'll need something like [7-Zip](https://www.7-zip.org/)).
2. Create a firewall rule to allow WSL to communicate with openocd:
   - Open the Windows Firewall Advanced Security console by typing "wf.msc" into the Windows search box and pressing enter.
   - In the console, click on "Inbound Rules" in the left-hand pane, and then click on "New Rule" in the right-hand pane.
   - In the "New Inbound Rule Wizard" that appears, select "Port" as the rule type and click "Next".
   - In the "Protocol and Ports" screen, select TCP as the protocol, and 3333 as the port number, and click "Next".
   - Continue through the rest of the application, making sure that the rule is enabled on both public and private networks


### Mac OSX
0. If you have an Apple Silicon Mac, follow the ARM Mac Instructions first
1. Install homebrew [here](https://brew.sh/)
2. Install openocd: `brew install openocd`
3. Install ARM developer tools: 
   ```
   brew tap PX4/homebrew-px4
   brew update
   brew search px4
   brew install gcc-arm-none-eabi-80
   ```
   If the last command fails but tells you to "Install the Command Line Tools" using `xcode-select --install`, do so.
   
## ARM (M1) Mac Instructions (In addition to normal instructions):
NOTE: This assumes that zsh is the default terminal. Using bash may cause issues.
1. Add the following lines to the end of the ~/.zshrc file (use `nano ./zshrc` to edit this file, and `CTRL`+`X`, then `Y`, then `Enter` to quit and save):
```
alias arm="env /usr/bin/arch -arm64 /bin/zsh --login"
alias intel="env /usr/bin/arch -x86_64 /bin/zsh --login"
```
2. run `source ~/.zshrc`
3. switch into an intel terminal by running the `intel` command you just created, and verify that the `arch` command returns `i386`
IMPORTANT: All commands beyond this point **MUST** be executed with an 'intel' terminal. You will need to switch to intel mode every time.
4. Install brew in the intel terminal (if you already have brew installed in an ARM environment, hope and pray that there are no conflicts) by running the script at https://brew.sh/ and following the prompts
5. Complete the rest of the MacOS instructions.

### Linux (Ubuntu) -- (Not been tested for a while but probably still works)
1. Install the "Arm GNU Toolchain for 32-bit Devices" for Linux from [here](https://www.microchip.com/mplab/avr-support/avr-and-arm-toolchains-c-compilers).
   - The Microchip link asks you to make an account, so either ask your BSE leader for the BSE account credentials or create your own account.
   - To avoid dealing with an account AND to get some newer GCC and (importantly) GDB features, use [this link](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads) instead. You may need to use the second-to-latest version if you can't find a 64-bit linux installer (i.e. "64-bit Tarball").
   - Make sure to add (or symlink) the bin folder into your path (e.g. add `export PATH=/path/to/arm-none-eabi/bin/:$PATH` to the end of `~/.bashrc` and restart your terminal or run `source ~/.bashrc`).
2. Install openocd: `sudo apt install openocd`. Unfortunately this doesn't seem to add required udev rules, so you'll need to:
    1. Download the udev rules: `wget https://repo.or.cz/openocd.git/blob_plain/HEAD:/contrib/60-openocd.rules`
    2. Set the rules' owner to root: `sudo chown root:root 60-openocd.rules`
    3. Set the rules' permissions: `sudo chmod 644 60-openocd.rules`
    4. Move the rules: `sudo mv 60-openocd.rules /etc/udev/rules.d/`
    5. Add yourself to the plugdev group: `sudo usermod -a -G plugdev $USERNAME`
    6. Probably log out or reboot. Just running `sudo udevadm control --reload` might also work.

### Linux (other distributions)
1. Install the Arm GNU Toolchain as above.
2. openocd is probably in your package manager, but if not there are instructions [here](http://openocd.org/getting-openocd/). Make sure to put (or symlink) the bin folder in your path.
2. You may need to follow the instructions above to add the udev rules.

### Using Visual Studio Code **(Untested!)**
You can use the vscode debugger interface instead of the gdb textual interface to debug programs by installing the [Cortex-Debug](https://github.com/Marus/cortex-debug) vscode extension from the marketplace.

To upload, run and debug using vscode:
1. Make sure you've built the code with `make`
2. Make sure you're connected to the board or debugger
3. Under the "Run and Debug" vscode menu on the left, click the green arrow to start debugging
   - Make sure the "Debug (OpenOCD)" configuration is selected in the dropdown to the right of the arrow
4. Wait for the debugger to start up (your bottom bar will turn orange once it does)
   - If it doesn't start up, check the "Output" and "Debug Console" panes of vscode for error messages
5. On the hovering toolbar that comes up, click the play button to run (continue) the program
   - If you want to step through your code, set any breakpoints you need before this using the interface to the left of any code file you have open in vscode

## Changing ASF library configuration
Follow this procedure if you'd like to add Atmel Start ASF libraries, change chip configuration or pinout, etc.
1. Go to [start.atmel.com](https://start.atmel.com)
2. Click on "Load project from file" and upload `./asf-samd21/atmel_start_config.atstart`
4. As desired, add software components, configure pinmux, package, or clocks
5. Click "Export project", make sure "Makefile (standalone)" is selected, and click "Download pack"
8. Change the `.atzip` file extension on the file you downloaded to `.zip` and extract it
10. Delete the `main.c` file in the downloaded directory
11. Delete the contents of `./asf-samd21/` and copy all files in the downloaded directory there
12. Modify `ATMEL_SRC_DIRS` and `ATMEL_INCLUDE_DIRS` in `./Makefile` according to the instructions there. You may need to make further modifications in `./Makefile` based on changes made by Atmel Start to `./asf-samd21/gcc/Makefile` (to see what changed, do `git diff ./asf-samd21/gcc/Makefile`).
