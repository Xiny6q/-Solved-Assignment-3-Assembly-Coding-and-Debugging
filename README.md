Download link :https://programming.engineering/product/solvedassignment-3-assembly-coding-and-debugging/

# -Solved-Assignment-3-Assembly-Coding-and-Debugging
(Solved)Assignment 3: Assembly Coding and Debugging
1 Introduction
This assignment will feel somewhat familiar in that it is nearly identical to the preceding assignment: there is a coding problem and a puzzle-solving problem. The major change is that everything is at the assembly level:
• Problem 1 re-works the Thermometer functions in assembly rather than C
• Problem 2 involves analyzing a binary executable to provide it with the correct input to “defuse” the executable
Working with assembly will get you a much more acquainted with the low-level details of the x86-64 platform and give you a greater appreciation for “high-level” languages (like C).
2 Download Code and Setup
Download the code pack linked at the top of the page. Unzip this which will create a project folder. Create new files in this folder. Ultimately you will re-zip this folder to submit it.
File State Notes
Makefile Provided Problem 1 Build file
thermo.h Provided Problem 1 header file
thermo_main.c Provided Problem 1 main() function
thermo_sim.c Provided Problem 1 thermometer drawing functions
thermo_update.c Create (?) Problem 1 C functions, may copy from previous assignment
thermo_update_asm.s Create Problem 1 Assembly functions, re-code C in x86-64

test_thermo.c Testing Problem 1 binary tests for thermo_update_asm.s
test_thermo_main.sh Testing Problem 1 shell tests for thermo_main
test_thermo_main_data.sh Testing Problem 1 shell test data for thermo_main
bombNN.tar Download Problem 2 Debugging problem, download from server
bombNN/bomb.c Unpack Problem 2 Unpack from .tar file, main() for bomb
bombNN/bomb Unpack Problem 2 Unpack from .tar executable to debug
bombNN/README Unpack Problem 2 Unpack from .tar contains “owner” of the bomb
input.txt Edit Problem 2 Input for bomb, fill this in
3 Problem 1: Thermometer Assembly Functions
The functions in this problem are identical to a previous assignment in which code to support digital thermometer was written. These functions are:
int set_temp_from_ports(temp_t *temp)
Read global variables corresponding to sensor and mode information and set the fields of a temp_t structure accordingly.
int set_display_from_temp(temp_t temp, int *display)
Given a temp_t struct, reset and alter the bits pointed to by display to cause a proper temperature display.
int thermo_update()
Update global THERMO_DISPLAY_PORT using the above functions.
The big change in this iteration will be that the functions must be written in x86-64 assembly code. As C functions each of these is short, 30-50 lines maximum. The assembly versions will be somewhat longer as each C line typically needs 1-4 lines of assembly code to implement fully. Coding these functions in assembly give you real experience writing working assembly code and working with it in combination with C.
The code setup and tests are identical for this problem as for the previous C version of the problem. Refer to original Thermometer Problem description for a broad overview of the thermometer simulator and files associated with it.
3.1 Write Your Assembly
As discussed in class, one can generate assembly code from C code with appropriate compiler flags. This can be useful for getting oriented and as a beginning to the code your assembly versions of the functions. However, code that is clearly compiler-generated with no hand-tweaking will
• Receive little credit on manual inspection
• Receive penalties on testing which lowers credit associated with that portion
Do not let that dissuade you from looking at compiler-generated assembly code from you C solution to the functions. Make sure that you take the following steps which are part of the manual inspection criteria.
Base your Assembly code on your C code
The files to be submitted for this problem include
• thermo_update.c: C version of the functions
• thermo_update_asm.s: Assembly version of the functions
Graders will examine these for a correspondence between to the algorithm used in the C version to the Assembly version. Compiler generated assembly often does significant re-arrangements of assembly code with many intermediate labels that hand-written code will not have.
If you were not able to complete the C functions for the thermometer problem from the previous assignment, see a course staff member who will help you get them up and running quickly.
Annotate your Assembly Thoroughly
Comment your assembly code A LOT. While good C code can be quite self-explanatory with descriptive variable names and clear control structures, assembly is rarely so easy to understand. Include clear commentary on your assembly. This should include
• Subdividing functions into smaller blocks with comments describing what the blocks accomplish.
• Descriptions of which “variables” from the C side are held in which registers.
• Descriptions of most assembly lines and their effect on the variables held in the registers.
• Descriptions of any data such as bitmasks stored in the assembly code.
Use Division
While it is a slow instruction that is cumbersome to set up, using division is the most human-readable means to compute several results needed in the required functions. Compiler generated code uses many tricks to avoid integer division so a lack of assembly instructions along this line will be a clear sign little effort has been put into the assembly code.
3.2 General Cautions when coding Assembly
1. Careful with constants: forgetting a $ in constants will lead to a bare, absolute memory address which will likely segfault your program. Contrast:
movq $0,%rax # rax = 0
movq 0, %rax # rax = *(0): segfault
# bare 0 is memory address 0 – out of bounds
Running your programs, assembly code included, in Valgrind can help to identify these problems. In Valgrind output, look for a line number in the assembly code which has absolute memory addresses or a register that has an invalid address.
2. Be disciplined about your register use: comment what “variables” are in which registers as it is up to you to keep track. Comments here are as helpful to you as to other readers.
3. Recognize that in x86-64 function parameters are passed in registers for up to 6 arguments. These are arranged as follows
1. rdi / edi / di (arg 1)
2. rsi / esi / si (arg 2)
3. rdx / edx / dx (arg 3)
4. rcx / ecx / cx (arg 4)
5. r8 / r8d / r8w (arg 5)
6. r9 / r9d / r9w (arg 6)
and the specific register corresponds to how argument sizes (64 bit args in rdi, 32 bit in edi, etc). The functions you will write have few arguments so they will all be in registers.
4. Use registers sparingly. The following registers (64-bit names) are “scratch” registers or “caller save.” Functions may alter them freely (though some may contain function arguments).
rax rcx rdx rdi rsi r8 r9 r10 r11 # Caller save registers

No special actions need to be taken at the end of the function regarding these registers except that rax should contain the function return value.
Remaining registers are “callee save”: if used, their original values must be restored before returning from the function.
rbx rbp r12 r13 r14 r15 # Callee save registers

This is typically done by pushing the callee registers to be used on the stack, using them, them popping them off the stack in reverse order. Avoid this if you can (and you probably can in our case).
5. Be careful to adjust the stack pointer using pushX and popX calls. Keep in mind the stack must be aligned to 16-byte boundaries for function calls to work correctly. Above all, don’t treat rsp as a general purpose register.
3.3 Iterative Development Strategy
With working C versions of the three required functions, you should be able to employ the an iterative strategy while developing assembly versions: focus on one assembly function while using working C functions for the remaining code. The Makefile and support code are specifically set up for this and details iterative development are as follows.
1. In thermo_update_asm.s, comment all code/declarations of global symbols except for set_temp_from_ports.
2. In thermo_update.c, comment out the definition of the set_temp_from_ports() function. This leaves working C versions of the other two functions.
3. Write some assembly code for set_temp_from_ports. Attempt to compile it on its own with
> make thermo_update_asm.o
gcc -Wall -g -c thermo_update_asm.s
which will report assembler errors if any are present
4. When there appear to be no assembler errors, create a hybrid main program which will combine the uncommented assembly and C functions
> make hybrid_main
gcc -Wall -g -c thermo_main.c
gcc -Wall -g -c thermo_sim.c
gcc -Wall -g -c thermo_update_asm.s # compiles assembly ..
gcc -Wall -g -c thermo_update.c # and C version to produce executable
gcc -Wall -g -o hybrid_main thermo_main.o thermo_sim.o thermo_update_asm.o thermo_update.o
5. One can then experiment with the hybrid_main to see if the written assembly function is working correctly.
> ./hybrid_main 1000 c
THERMO_SENSOR_PORT set to: 1000
set_temp_from_sensors(&temp );
temp is {
.tenths_degrees = 1084
.is_fahrenheit = 9
}
…

Something clearly looks wrong in the above so some debugging of the assembly function seems in order.
6. When ready, run the “hybrid tests” which will do automated testing on the hybrid codes.
> make test-hybrid
gcc -Wall -g -c test_thermo_update.c
gcc -Wall -g -o test_hybrid test_thermo_update.o …
===TESTS for Hybrid===
Running binary tests for hybrid
./test_hybrid
Test 1: set_temp_from_sensors() zero-c : OK
Test 2: set_temp_from_sensors() zero-f : OK
Test 3: set_temp_from_sensors() 64-c : FAIL
…
7. When all seems to be working correctly with the first assembly function, move on. Comment another C function and uncomment the corresponding assembly, write some code and repeat.
3.4 Structure of thermo_update_asm.s
Below is a rough outline of the structure of thermo_updat_asm.s. Consider copying this file as you get started and commenting parts of it out as needed.
.text
.global set_temp_from_ports

## ENTRY POINT FOR REQUIRED FUNCTION
set_temp_from_ports:
## assembly instructions here

## a useful technique for this problem
movX SOME_GLOBAL_VAR(%rip), %reg # load global variable into register
# use movl / movq / movw / movb
# and appropriately sized destination register

### Data area associated with the next function
.data

my_int: # declare location an single int
.int 1234 # value 1234

my_array: # declare multiple ints in a row
.int 10 # for an array. Each are spaced
.int 20 # 4 bytes from each other
.int 30

.text
.global set_display_from_temp

## ENTRY POINT FOR REQUIRED FUNCTION
set_display_from_temp:
## assembly instructions here

## two useful techniques for this problem
movl my_int(%rip),%eax # load my_int into register eax
leaq my_array(%rip),%edx # load pointer to beginning of my_array into edx

.text
.global thermo_update

## ENTRY POINT FOR REQUIRED FUNCTION
thermo_update:
## assembly instructions here
3.5 set_temp_from_ports
int set_temp_from_ports(temp_t *temp);
// Uses the two global variables (ports) THERMO_SENSOR_PORT and
// THERMO_STATUS_PORT to set the temp structure. If THERMO_SENSOR_PORT
// is above its maximum trusted value, associated with +50.0 deg C,
// does not alter temp and returns 1. Otherwise, sets fields of temp
// based on converting sensor value to degrees and checking whether
// Celsius or Fahrenheit display is in effect. Returns 0 on successful
// set. This function DOES NOT modify any global variables but may
// access global variables.
//
// CONSTRAINT: Uses only integer operations. No floating point
// operations are used as the target machine does not have a FPU.
Note that this function uses a temp_t struct which is in thermo.h described here:
// Breaks temperature down into constituent parts
typedef struct{
short tenths_degrees; // actual temp in tenths of degrees
char is_fahrenheit; // 0 for celsius, 1 for fahrenheit
} temp_t;
Assembly Implementation Notes set_temp_from_ports
1. The function takes a single argument, a pointer in rdi.
2. Return values or functions are to be placed eax for 32 bit quantities as is the case here (int).
3. To access global symbols/variables which are not defined in the assembly file, use the relative position from the instruction pointer register which allows the linker to handle the task. Specifically relevant examples are
movw THERMO_SENSOR_PORT(%rip), %dx # copy global var to reg dx (16-bit word)
movb THERMO_STATUS_PORT(%rip), %cl # copy global var to reg cl (8-bit byte)
movl %r8d,THERMO_DISPLAY_PORT(%rip) # copy reg r8d to global var (32-bit long-word)
4. Use comparisons and jump to a separate section of code that is clearly marked as “error” if you detect a bad arguments. Be careful to use appropriate assembly instructions for the type of data being compared.
◦ cmpX performs comparison based on subtraction; pick cmpq / cmpl / cmpw / cmpb according to the size of data being compared.
◦ Jump instructions such as jg assume prior comparison was done on signed quantities.
◦ Jump instructions like ja assume prior comparison was done on unsigned quantities.
5. To do the initial temperature conversion of the temperature sensor one must divide by 64. Avoid the division and use a shift instead. The remainder can also be found by masking / anding low order bits that would be shifted off which will allow for rounding
6. If the temperature must be converted to Fahrenheit, make use of division instructions to achieve fahrenheit = (9 * celsius) / 5 + 32. Keep in mind that the idivX instruction must have rax as the dividend and rdx sign-extended from it. This may involve use of the following sequence of instructions:
cwtl # sign extend ax to long word
cltq # sign extend eax to quad word
cqto # sign extend ax to dx
Any register can contain the divisor. After the instruction, rax / eax / ax will hold the quotient and rdx / edx / dx the remainder. In this function, a single division will be sufficient.
7. A pointer to a temp_t struct can access its fields using the following offset table which assume that %reg holds a pointer to the struct (substitute an actual register name).
Destination Assembly
C Field Access Offset Size Assign 5 to field
temp->tenths_degrees 0 bytes 2 bytes movw $5,0(%reg)
temp->is_fahrenheit 2 bytes 1 byte movb $5,2(%reg)
You will need to use these offsets to set the fields of the struct near the end of the routine.
3.6 set_display_from_temp
int set_display_from_temp(temp_t temp, int *display);
// Alters the bits of integer pointed to by display to reflect the
// temperature in struct arg temp. If temp has a temperature value
// that is below minimum or above maximum temperature allowable or if
// an improper indication of celsius/fahrenheit is given, does nothing
// and returns 1. Otherwise, calculates each digit of the temperature
// and changes bits at display to show the temperature according to
// the pattern for each digit. This function DOES NOT modify any
// global variables but may access global variables.
Assembly Implementation Notes set_display_from_temp
1. Arguments will be
◦ a packed temp struct in rdi
◦ an integer pointer in rsi
2. The packed temp_t struct is entirely in the 64-bit rdi register which has the following layout.
Bits Shift
C Field Access in rdi Required Size
temp.tenths_degrees 00-15 None 2 bytes
temp.is_fahrenheit 16-24 Right by 16 1 byte
To access individual fields of the struct, you will need to do shifting and masking to extract the values from the rdi register.
3. Use comparisons and jump to a separate section of code that is clearly marked as “error” if you detect bad fields in the temp struct argument such as temperature values that are outside the minimum/maximum values allowed for Fahrenheit or Celsius.
4. As was the case in the C version of the problem, it is useful to create a table of bit masks corresponding to the bits that should be set for each display digit (e.g. digit “1” has bit pattern 0b0000110). In assembly this is easiest to do by using a data section with successive integers. An example of how this can be done is below.
.section .data
array: # an array of 3 ints
.int 0b101 # array[0] = 0b101
.int 0b010 # array[1] = 0b010
.int 0b111 # array[2] = 0b111
const:
.int 17 # special constant

.section .text
.globl func
func:
leaq array(%rip),%r8 # r8 points to array, rip used to enable relocation
movq $2,%r9 # r9 = 2, index into array
movl (%r8,%r9,4),%r10d # r10d = array[2], note 32-bit movl and dest reg
movl const(%rip),%r11d # r11d = 17 (const), rip used to enable relocation
Adapt this example to create a table of useful bit masks for digits. The GCC assembler understands binary constants specified with the 0b0011011 style syntax.
5. Make sure to check for a negative temperature and adjust the display to contain a negative sign in the correct position. It may be useful to then negate a temperature below zero so that later divisions always result in positive values.
6. Make use of division instructions to compute “digits” for the tenths, ones, tens, and hundreds place for the thermometer. With cleverness, you should only need 3-4 divisions. Use these digits to reference into the table of digit bit masks you create to progressively build up the correct bit pattern for the display.
7. Use shifts and ORs to combine the digit bit patterns to create the final display bit pattern.
3.7 thermo_update
int thermo_update();
// Called to update the thermometer display. Makes use of
// set_temp_from_ports() and set_display_from_temp() to access
// temperature sensor then set the display. Checks these functions and
// if they indicate an error, makes not changes to the display.
// Otherwise modifies THERMO_DISPLAY_PORT to set the display.
//
// CONSTRAINT: Does not allocate any heap memory as malloc() is NOT
// available on the target microcontroller. Uses stack and global
// memory only.
Assembly Implementation Notes for thermo_update
1. No arguments come into the function.
2. Use the syntax described earlier to access global symbols/variables such as THERMO_SENSOR_PORT.
3. Call the two previous functions to create the struct and manipulate the bits of an the display. Calling a function requires that the stack be aligned to 16-bytes; there is always an 8-byte quantity on the stack (previous value of the rsp stack pointer). This means the stack must be extended with a pushq instruction before any calls. A typical sequence is
pushq %rdx # push any 64-bit register onto the stack
call some_func # stack aligned, call function
## return val from func in rax or eax
popq %rdx # restore the stack
4. If several function calls will be made, a single push is all that is needed as in the below
pushq %rdx # push any 64-bit register onto the stack
call some_func1 # stack aligned, call function
## return val from func in rax or eax

## do some more stuff

call some_func2 # stack aligned, call function
## return val from func in rax or eax

popq %rdx # restore the stack
5. In order to call the set_temp_from_ports() function, this function will need to allocate space on the stack for a temp_t. As described previously, this struct can be packed to fit in 8 bytes so a pushq $0 will put a “zero” temp_t struct on the stack and %rsp is then a pointer to it which can be copied to other registers.
6. Similarly, to call the set_display_from_temp() function, one will need a packed temp_t in a register. If the preceding set_temp_from_ports() call succeeded, this packed struct can be read from memory into a register with a movq instruction. That stack space can be re-used if needed.
7. Keep in mind that you will need to do error checking of the return values from the two functions: if they return non-zero values jump to a clearly marked “error” section and return a 1. If an error occurs, don’t forget to pop any values off the stack that have been pushed before returning.
3.8 Grading Criteria for Problem 1 GRADING
Weight Criteria
AUTOMATED TESTS run via make test-p1

15 test_thermo.c run via make test-p1a
Provides 30 tests for functions in thermo_update_asm.s
0.5 points per test
Deductions for memory problems identified by Valgrind

5 test_thermo_main.sh run via make test-p1b
5 Tests of the thermo_main which uses functions from thermo_update_asm.s
1 point per test passed
Deductions for memory problems identified by Valgrind
MANUAL INSPECTION CRITERIA

10 set_temp_from_ports()
Clear signs of hand-crafted assembly are present.
Detailed documentation/comments are provided showing the algorithm used in the assembly
There is a clear relation of the code to the C algorithm used in thermo_update.c
High-level variables and registers they occupy are described.
Error checking on the input values is done with a clear “error” section/label

The initial division by 64 is done using a bitwise shift instruction.
Remainders from the division by 64 are obtained through bitwise-AND on the low-order bits.
Division is used to compute Fahrenheit temperature conversions.
There is a clearly documented section which updates struct fields in memory
No function calls are made that would alter the stack contents

10 set_display_from_temp()
Clear signs of hand-crafted assembly are present.
Detailed documentation/comments are provided showing the algorithm used in the assembly
There is a clear relation of the code to the C algorithm used in thermo_updat.c
High-level variables and registers they occupy are described.
Error checking on the input values is done with a clear “error” section/label

There is a clearly documented data section setting up useful tables of bitmasks
Struct fields are unpacked from an argument register using shift operations
Division is used to compute quotients and remainders that are needed.
No function calls are made that would alter the stack contents

10 thermo_update()
Clear signs of hand-crafted assembly are present.
Detailed documentation/comments are provided showing the algorithm used in the assembly
There is a clear relation of the code to the C algorithm used in thermo_updat.c
High-level variables and registers they occupy are described.
Error checking on the return values is done with a clear “error” section/label

Memory is pushed onto the stack for local variables that must be passed by reference
Function calls to the earlier two functions are made with appropriate arguments passed
4 Problem 2: The Binary Bomb

4.1 Quick Links
Available only on Lab Machines or Vole
Download Bombs http://apollo.cselabs.umn.edu:15213/
Score Board http://apollo.cselabs.umn.edu:15213/scoreboard
2021 GDB Quick Guide/Assembly https://www-users.cs.umn.edu/~kauffman/2021/gdb#orgdb7b10c
More details on these are described in subsequent sections.
4.2 Overview
The nature of this problem is similar to the previous assignment’s puzzlebox: there is a program called bomb which expects certain inputs from a parameter file or typed as input. If the inputs are “correct”, a phase will be “defused” earning points and allowing access to a subsequent phases. The major change is that the bomb program is in binary so must be debugged in assembly.
Below is a summary of useful information concerning the binary bomb.
Bombs are Individual
The bomb you will download contains subtle variations so that the solution to yours will not work on other bombs. Feel free to discuss general techniques with classmates but know that you’ll need to ultimately defuse your own bomb.
Bombs are Binary
A small amount of C code with the main() function is included but the bulk of the code is binary which will require using gdb to debug the assembly code.
Bombs only Run on Lab Machines
To stay in contact with the scoring server, bombs won’t run on your laptop. You’ll need to work on them on lab machines.
Bombs Take Input
Similar to puzzlebox, create an input.txt file which will contain your answers. Run bombs with this input file. Note that if the bomb runs out of input, you can type input directly into the bomb though this may look a little funny in the debugger.
Defusing Phases Earns Points
As with the earlier puzzlebox, points for this problem are earned based on how many phases are completed. Each phase that is completed will automatically be logged with the scoring server
Bomb Explosions Lose Points
If incorrect input is entered and the bomb runs to completion, it will “explode” which causes credit to be deducted. See the scoring system for details. This can be prevented by setting breakpoints prior to the explosion sequence and restarting the bomb when those breakpoints are hit.
4.3 Machines on which bombs run
The binary bomb makes frequent contact with a scoring server so you can only run it on a list of prescribed machines. These comprise most of the valid CSELabs machines and are listed in the table below.
Machine Login Address Location
apollo csel-apollo.cselabs.umn.edu Machine Room
atlas csel-atlas.cselabs.umn.edu Machine Room
Vole csel-vole-01.cselabs.umn.edu Virtual
csel-vole-02.cselabs.umn.edu
…
csel-vole-48.cselabs.umn.edu
4-250 Lab csel-kh4250-01.cselabs.umn.edu Keller 4-250
…
csel-kh4250-49.cselabs.umn.edu
4-240 Lab csel-kh4240-01.cselabs.umn.edu Keller 4-240
…
csel-kh4240-10.cselabs.umn.edu
Lind Lab csel-lind40-01.cselabs.umn.edu Lind Hall 40
…
csel-lind40-43.cselabs.umn.edu
Attempting to run a bomb on an un-authorized machine will error out immediately as in
> ./bomb
Initialization error: illegal host ‘ck-laptop’.
Legal hosts are as follows:
csel-apollo
csel-atlas
csel-vole-01
csel-vole-02
…
4.4 Bomb Download and Setup
• Download your bomb from the following web address
◦ http://apollo.cselabs.umn.edu:15213/
• This site must be accessed from wired UMN machines as it is behind the campus firewall. Using a browser on Vole is the easiest way get a bomb onto your CSELabs account (and will let you tell friends “I’ve used a browser inside a browser.”).
• Enter your UMN information in the required fields. If you fail to enter your official information, you may not get a grade for this portion.
• The bomb will download as a .tar file, an archive format. On Unix machines, extract the contents using the command untar as in
> ls
bomb10.tar

> tar xfv bomb10.tar
bomb10/README
bomb10/bomb.c
bomb10/bomb

> ls
bomb10.tar bomb10/

> cd bomb10
> ls
bomb* bomb.c README
• The resulting bomb is unique for the downloader and the owner is in the README and logged on the download server.
• The file bomb (sometimes listed with a * to indicate it is executable) is a compiled binary so employ your assembly gdb skills to cracking it.
• Create a file input.txt. The bomb can be run with it as in
> ./bomb input.txt

but you’ll likely want to do this in gdb to avoid exploding the bomb.
• Unlike previous puzzles, if input.txt runs out of input, the bomb will prompt for you to type input. This can be a way to explore ahead a little bit in the bomb after solving a phase.
4.5 Scoring and Scoreboard (50%) GRADING
Scoring is done according to the following table.
Pts Phase
8 Phase 1
8 Phase 2
9 Phase 3
9 Phase 4
8 Phase 5
8 Phase 6
50 Total
Explosion Penalty: 0.5 points are deducted for each explosion up to 20 explosions (maximum -10 points).
On successfully defusing stages, the bomb will contact a server which tracks scores by number. The scoreboard is here:
• http://apollo.cselabs.umn.edu:15213/scoreboard
• The server is reachable only on UMN hardwired machines such as lab machines or Vole
You’ll need to know your bomb number to see your score but can also see the scores of others.
Examples of Scoring
Phases Final
Defused Explosions Computation Score Notes
6 1 50 – floor(0.5*1) 50 1 explosion for free
6 4 50 – floor(0.5*4) 48
6 10 50 – floor(0.5*10) 45
6 20 50 – floor(0.5*20) 40
5 7 42 – floor(0.5*7) 39 Round down for penalty
4 4 34 – floor(0.5*4) 32
1 0 8 – floor(0.5*0) 8
0 20 0 – (floor(0.5*20) -10
0 30 0 – (floor(0.5*20) -10 Max 20 explosions counted
Getting Credit for the Problem
• Ensure that the score listed on the Scoreboard site reflects your progress.
• Ensure your input.txt along with your bombNN/ directory are in your project directory with the rest of your code.
4.6 WARNING on Downloading Multiple Bombs
It is possible to download multiple bombs but this will NOT reset your explosion count. Quite the opposite: the default scoring system for the server uses the following conventions.
• Only the maximum phase defused in any bomb adds points
• Total explosions across all bombs subtract points with each separately downloaded bomb contributing up to -10.
Since more bombs likely means more explosions, you are strongly advised to download a single bomb and work with it.
4.7 Advice
• If you accidentally run the bomb from the command line, you can kill it with the Unix interrupt key sequence Ctrl-c (hold control, press C key).
> ./bomb
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
^C
So you think you can stop the bomb with ctrl-c, do you?
Well…OK. :-)
>
• Most of the time you should run the bomb in gdb as in
> gdb ./bomb

Refer to the Quick Guide to GDB if you have forgotten how to use gdb and pay particular attention to the sections on debugging assembly.
• Figure out what the explosion routine is called and always set a breakpoint there. This will allow you to stop the bomb
• Make use of other tools to analyze the binary bomb aside from the debugger. Some of these are described at the end of the Quick Guide to GDB. They will allow you to search for “interesting” data in the executable bomb. The author of the bomb is encoded in the binary as a string somewhere which may be relevant to inputs for some phases.
• Feel free to do some internet research. The “bomb lab” assignment has a long history and there are some useful guides out there that can help you through rough patches. Keep in mind that your bomb will differ but the techniques to defuse it may be similar to others.

