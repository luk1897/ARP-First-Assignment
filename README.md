# ARP-First-Assignment

## Description

The project consists of controlling a hoist with two degrees of freedom. I have used several processes and will describe them one by one.

Preliminary remarks: Named pipes were used for the communication of all processes and thus for the exchange of almost all information. All signals were handled through the use of sigaction. In addition, logfiles were created for each process and an error-checking function that not only writes any errors on the screen, but also gives the number of the line in which the error occurred to make error resolution more efficient.

### MASTER
In the master I created all the necessary pipes and started all the processes using execvp. Then, via a wait, it waits for the first child to finish. When one of the child processes terminates, the master unlinks all the pipes and sends a SIGTERM to all the remaining processes, and finally terminates. There is a handler for the SIGINT signal to prevent the master from being terminated without having terminated all its children.

### CMD
This process handles the movement of the hoist in the two axes. There are three buttons that manage the vertical speed of the hoist and three buttons that manage the horizontal speed of the hoist. The user simply clicks on them with the mouse to give the command input. If the command receives a SIGUSR2 signal from the inspection then a reset is taking place and the commands are locked using a flag variable called flag_reset, when it receives a SIGUSR1 signal, from motor x, then either the reset has ended or a stop has been commanded from the console and in this case the flag_reset variable will be set negative and the command unlocked. The SIGINT signal has been disabled and process termination is handled via the SIGTERM signal.

### MOTOR X
In this process, commands are received from the command by which the horizontal speed is modified in few values (-3,-2,-1,0,1,2,3). Here I have calculated the horizontal position via the speed and it is sent to the worldxz process. I used a select for the readings to avoid to block the motor. In addition, here I have implemented a function which, via signals sent by the inspection, handles the reset and stop of the velocities of both axes. When there is a reset commanded by inspection, the latter sends a SIGUSR2 signal which enables a variable called reset that sets the speed equal to -3 until the x-position is equal to zero or a stop is commanded by inspection (in this case a variable stop is set to on). In the first and second case, the reset variable is sent false (as the variable stop) and the SIGUSR1 signal is sent to the command that unlocks the latter. The sending of the SIGUSR1 signal is only taken care of by this process because during a reset it will almost always be position x that comes second (after position z). The case where this is not the case is handled via a pipe that allows me to know the z position at any time, so if the x position is zero and the z position is non-zero, I can safely perform a reset without having to worry about the x axis moving before the reset is complete. The process termination is handled via the SIGTERM signal.

### MOTOR Z
Identical function to that of motor x. The differences are that it does not send a signal to the command to unlock the buttons when reset ends and that the vertical position of the hoist is calculated instead of the horizontal position. 

### WORLDXZ
In this process, an error is added to the estimated positions received from both motors, trying to make the calculation of the actual positions more realistic. After the positions are calculated, they are sent to the inspection. I used a select for the readings about the two motors to avoid the block of the process. The process termination is handled via the SIGTERM signal.

### INSPECTION
Here, the real-time position of the hoist, received from the worldxz process (select has been used), is shown graphically. In addition, there are two buttons (S and R) which correspond to the stop of both motors and the reset to the initial position. When the R button is clicked, the SIGUSR2 signal is sent to the two motors in order to set the speed to negative to reach the initial position (0.00,0.00) and to the command in order to disable it. 
When the S button is clicked, the SIGUSR1 signal is sent to the two motors to set the speed to zero and to the command to unlock the commands if a reset is in progress. In addition, I implemented a handler that handles the SIGUSR2 signal sent by the watchdog to reset the system. The SIGINT signal has been disabled and process termination is handled via the SIGTERM signal.

### WATCHDOG
This process checks that no command is sent from the command for 60 seconds, then ensures that the user does not use the system for at least 60 seconds. On the first start-up, the watchdog is not active because it would be useless to do a reset as at start-up the position is the initial one. The watchdog is first activated as soon as a move is performed because the command sends a SIGUSR2 signal which triggers a 60-second alarm. If no further SIGUSR2 signals are received within 60 seconds, then alarm will cause a SIGALRM signal to be sent to the watchdog itself, which in turn will cause a SIGUSR2 signal to be sent to the inspection which will cause the system to reset. The process termination is handled via the SIGTERM signal.


## Installing

### Install ncurses

`sudo apt-get install libncurses-dev` 

### Install Konsole

`sudo apt-get install konsole` 

Then

`chmod +x install.src` 

` ./install.src` 

## Run

`chmod +x run.src` 

`./run.src` 

## User guide

### KONSOLE COMMAND

Vx++: it increases horizontal velocity by 1
Vx--: it decreases horizontal velecity by 1
Vx stop: it sets the horizontal velocity to zero

Vz++: it increase vertical velocity by 1
Vz--: it decreases vertical velecity by 1
Vz stop: it sets the vertical velocity to zero

The upper and lower upper limit of velocity is +3/-3

### KONSOLE INSPECTION

S: it stops both engines

R: it resets the hoist to the initial position (0.00,0.00)


