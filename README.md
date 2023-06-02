# tictactobot

## Table of Contents
* [Table of Contents](#TableOfContents)
* [Goal](#Goal)
* [Planning](#Planning)
* [CAD](#CAD)
* [Code](#Code)
* [Building](#Building)
* [End Result](#EndResult)
* [Problems](#Problems)
* [Reflection](#Reflection)
---
# `Goal`
The overall goal of this assignment was to create a robot arm, with at least two joints, to solve a problem. The problem we came up with is that tic-tac-toe sucks because it's always a draw, so we made a robot that will cheat in tic-tac-toe.

# `Planning`
## First Sketches
The design of our robot changed a lot during the first few days of planning. We originally planned to have a big rotating pole pinion which the arm would scale up and down while extending. It was a little ambitious, and after realizing that we would have to use multiple stepper motors, all reliant on a flimsy piece of acrylic, we decided to switch it up. The design we settled on was a cylindrical arm. We would have an arm that could extend outwards and be rotated. (see image on right). But you may have noticed that the arm depicted there can move up and down, and given that our bot has to be able to pick up pieces, we need some vertical movement. But given our sketches so far, there was no way to make a reliable elevation system that wouldn’t risk the stability of the arm. We came up with the idea of attaching a magnet to the end of the arm which could be raised and lowered to pick up pieces. This seemed much simpler than our original ideas of claws that would pick up the pieces.

## Requirements
The most important requirement to watch out for is your range of motion. Your robot may need to have far reach in a small area in front of it, or have less reach but all around it. Make sure to plan for the range of motion you need your robot to have. An important aspect of this is servo choice. 180 servos are small and somewhat accurate, but have a very limited range. Continuous servos can go for as long as they like, but in a very inaccurate and unpredictable way. Steppers will be your best choice if you need continuous motion with accurate positioning, but they are big and heavy, so plan accordingly. 
For this project the robot just needs to be able to move in a square shape in front of it. We went with a standard size 180 servo to spin the arm back and forth, and a micro 180 servo to extend the arm. This micro servo was connected to a 3:1 gear system which allowed it to extend the whole length of the board in under 180 degrees of rotation, thanks to the math we did in our planning.
Another thing related to the servos is how much force they can exert. Especially when working with micro servos, you have to be aware of their weight limits. This project was luckily very light on the arm, but if you’re going to be putting any real strain on the servos you should do the math to make sure they can handle it.
![image](https://github.com/jvaugha3038/tictactobot/assets/112961338/d77939f4-6d30-4ff8-927b-d7cd59fc573f)
If you’re ever struggling to think of things your robot needs to do (or even if you aren’t), I recommend looking back at the purpose of your robot and thinking about how you can achieve these things. This helped us realize that we hadn’t planned on how we would detect the pieces on the board (we wouldn’t want a blind tic-tac-toe robot would we?). Luckily for us, this was relatively early on, and we were given the idea of an infrared sensor by Mr. Helmstetter. This sensor can measure how reflective an object is by shining an invisible light on it, and by using different materials for our X’s and O’s, we can use it to differentiate between the board and those two.

# `CAD`
https://cvilleschools.onshape.com/documents/9f7a2ab06042344ad68077a3/w/577d4a05bb440caa0ac4fef6/e/6a6642626d928ba19f00adb1
## Proof of Concept
Our proof of concept was planned to be a small-scale rack and pinion gear that would represent the arm of the robot (A.K.A the extendo part), and the device we would use to pick up the pieces (the pick-uppy part). However, we did not get any string or magnets in time for the proof of concept, so we just created the small extendo arm.

# `Code`
<details>
<summary><b>Click to Show<b></summary>
     
<p>
    
```
import board
from time import sleep
import random
import pwmio
import servo
from analogio import AnalogIn
from digitalio import DigitalInOut, Direction


turn = 0 # whose turn it is, 0 is player, 1 is AI
round = 1 # which round of the game it is
end = 0 # if the game has ended
plan = 0 # where the AI is planning to move
extend = 0 # how far out the arm is
twist = 0 # angle of turn of the arm
offset = 1
armProg = 96
colorBase = 0
Nscan = 1

button = DigitalInOut(board.D8)
button.direction = Direction.INPUT
pwm = pwmio.PWMOut(board.A1, duty_cycle=2 ** 15, frequency=50)
pwm2 = pwmio.PWMOut(board.A5, duty_cycle=2 ** 15, frequency=50)
uppi = pwmio.PWMOut(board.D7, duty_cycle=2, frequency=50, variable_frequency=True) # Sets up a PWM output for the magnet servo as there are no more A timers
#uppy = pwmio.PWMOut(board.A0, duty_cycle=2 ** 15, frequency=50, variable_frequency=True)
color = AnalogIn(board.A2)


theBoard = {'7': ' ' , '8': ' ' , '9': ' ' , # The visual board, keeps track of placed O's and X's
            '4': ' ' , '5': ' ' , '6': ' ' ,
            '1': ' ' , '2': ' ' , '3': ' ' }

distBoard = {'7': 170 , '8': 157 , '9': 170 , # Holds all of the distance data to the board
            '4': 110 , '5': 96 , '6': 110 ,
            '1': 58 , '2': 30 , '3': 58 , '0': 118}
angleBoard = {'7': 110 , '8': 90 , '9': 70 , # Holds all of the angle data to the board
            '4':  122, '5': 90 , '6': 61 ,
            '1': 150 , '2': 90 , '3': 42 , '0': 175}
arm = servo.Servo(pwm) # The extendy arm servo
spinny = servo.Servo(pwm2) # The twisty servo
uppy = servo.Servo(uppi)
#                                                  ARM SERVO SHOULD HAVE ONE TOOTH SHOWING IN THE BACK WHEN AT '5' POSITION

def printBoard(board):
    print("")
    print(board['7'] + '|' + board['8'] + '|' + board['9'])
    print('-+-+-')
    print(board['4'] + '|' + board['5'] + '|' + board['6'])
    print('-+-+-')
    print(board['1'] + '|' + board['2'] + '|' + board['3'])

def checkWin():
            global end
            if theBoard['7'] == theBoard['8'] == theBoard['9'] and theBoard['9'] != ' ': # across the top
                end += 1      
                print("Across Top")     
            
            elif theBoard['4'] == theBoard['5'] == theBoard['6'] and theBoard['6'] != ' ': # across the middle
                end += 1
                print("Across Mid")       
            
            elif theBoard['1'] == theBoard['2'] == theBoard['3'] and theBoard['3'] != ' ': # across the bottom
                end += 1
                print("Across Bottom")
                
            elif theBoard['1'] == theBoard['4'] == theBoard['7'] and theBoard['7'] != ' ': # down the left side
                end += 1
                print("Down Left")
                
            elif theBoard['2'] == theBoard['5'] == theBoard['8'] and theBoard['8'] != ' ': # down the middle
                end += 1
                print("Down Mid")
                
            elif theBoard['3'] == theBoard['6'] == theBoard['9'] and theBoard['9'] != ' ': # down the right side
                end += 1
                print("Down Right")
                 
            elif theBoard['7'] == theBoard['5'] == theBoard['3'] and theBoard['3'] != ' ': # diagonal
                end += 1
                print("Diagonal 7-3")
                
            elif theBoard['1'] == theBoard['5'] == theBoard['9'] and theBoard['9'] != ' ': # diagonal
                end += 1
                print("Diagonal 9-1")

def grab(direction): # Code to pick up and drop the magnet
    if direction == 0: # 0 Picks Up
        uppy.angle = (0) # 0 Is lowest, 180 is highest
        sleep(1)
        for i in range(160):
            uppy.angle = (i)
        sleep(.25)
    elif direction == 1: # 1 Drops
        uppy.angle = (180)
        arm.angle += 10
        sleep(.1)
        arm.angle -= 10
        sleep(1)

def place(spot): # Code to move the arm to a specified place on the board
    armProg = arm.angle
    sleep(.25)
    while arm.angle != distBoard['0']: # code to move arm smoothly
        if abs(armProg - distBoard['0']) < 2:
            arm.angle = (distBoard['0'])
            break
        elif armProg < distBoard['0']:
            armProg += 1
        elif armProg > distBoard['0']:
            armProg -= 1
        arm.angle = (armProg) 
        sleep(.0001)
    armProg = spinny.angle
    spinny.angle = (angleBoard['0'] + offset)
    while spinny.angle != angleBoard['0'] + offset: # code to move arm turn smoothly
        if abs(armProg - angleBoard['0'] + offset) < 2:
            spinny.angle = (angleBoard['0'] + offset)
            break
        elif armProg < angleBoard['0'] + offset:
            armProg += 1
        elif armProg > angleBoard['0'] + offset:
            armProg -= 1
        spinny.angle = (armProg) 
        sleep(.0001)
    sleep(1.25)
    #   PICKUP
    grab(0)
    print("Pickup")

    
    theBoard[str(spot)] = "X"
    armProg = spinny.angle
    while spinny.angle != angleBoard[str(spot)] + offset: # code to move arm turn smoothly
        if abs(armProg - (angleBoard[str(spot)] + offset)) < 2:
            spinny.angle = (angleBoard[str(spot)] + offset)
            break
        elif armProg < angleBoard[str(spot)] + offset:
            armProg += 1
        elif armProg > angleBoard[str(spot)] + offset:
            armProg -= 1
        spinny.angle = (armProg) 
        print(str(angleBoard[str(spot)]) + "   " + str(armProg))
        sleep(.0001)
    print("moved")
    sleep(.25)
    armProg = arm.angle
    while arm.angle != distBoard[str(spot)]:#             Extend
        if abs(armProg - distBoard[str(spot)]) < 2:
            arm.angle = (distBoard[str(spot)] * 0.86925636203 + 5)
            break
        elif armProg < distBoard[str(spot)]:
            armProg += 1
        elif armProg > distBoard[str(spot)]:
            armProg -= 1
        arm.angle = (armProg * 0.86925636203 + 5) 
        sleep(.0001)

    #   DROP
    grab(1)
    print("Drop")

    armProg = arm.angle
    sleep(.25)
    while arm.angle != distBoard['0']: # code to move arm smoothly
        if abs(armProg - distBoard['0']) < 2:
            arm.angle = (distBoard['0'])
            break
        elif armProg < distBoard['0']:
            armProg += 1
        elif armProg > distBoard['0']:
            armProg -= 1
        arm.angle = (armProg) 
        sleep(.0001)
    armProg = spinny.angle
    spinny.angle = (angleBoard['0'] + offset)
    while spinny.angle != angleBoard['0'] + offset: # code to move arm turn smoothly
        if abs(armProg - angleBoard['0'] + offset) < 2:
            spinny.angle = (angleBoard['0'] + offset)
            break
        elif armProg < angleBoard['0'] + offset:
            armProg += 1
        elif armProg > angleBoard['0'] + offset:
            armProg -= 1
        spinny.angle = (armProg) 
        sleep(.0001)

def scan():
    Nscan = 1
    global offset
    offset -= 3
    spinny.angle = 90
    print ("offest: " + str(offset))
    print ("B: "+ str(colorBase))
    while Nscan < 10:
        if theBoard[str(Nscan)] != 'O' and theBoard[str(Nscan)] != 'X':
            if Nscan == 3:
                angleBoard['3'] = 30
                distBoard['3'] = 65
            armProg = arm.angle
            sleep(.25)
            while arm.angle != distBoard[str(Nscan)]-10: # code to move arm smoothly
                if abs(armProg - (distBoard[str(Nscan)]-10)) < 3:
                    arm.angle = (distBoard[str(Nscan)]-10)
                    break
                elif armProg < distBoard[str(Nscan)]-10:
                    armProg += 3
                elif armProg > distBoard[str(Nscan)]-10:
                    armProg -= 3
                arm.angle = (armProg) 
                print(armProg - (distBoard[str(Nscan)]-10))

            armProg = spinny.angle
            while spinny.angle != (angleBoard[str(Nscan)] + offset): # code to move arm turn smoothly
                if abs(armProg - (angleBoard[str(Nscan)] + offset)) < 4: 
                    spinny.angle = (angleBoard[str(Nscan)] + offset)
                    print("good")
                    break
                elif armProg < (angleBoard[str(Nscan)] + offset):
                    armProg += 4
                elif armProg > (angleBoard[str(Nscan)] + offset):
                    armProg -= 4
                spinny.angle = (armProg) 
                print(str(armProg - (angleBoard[str(Nscan)] + offset)))

            for e in range (1,10):                         # PROBLEM FIX PROBLEM
                if abs((color.value - colorBase) / colorBase*100) >25:
                    print("O at " + str(Nscan))
                    theBoard[str(Nscan)] = 'O'
                    Nscan = 20
                    break
                print(color.value)
                print(str(abs((color.value - colorBase) / colorBase*100)))
                e = 1
                sleep(.1)
            print(Nscan)

        if Nscan == 20:
            pass
        elif Nscan < 3:
            Nscan += 1
        elif Nscan == 3:
            Nscan = 6
        elif Nscan > 4 and Nscan < 7:
            Nscan -= 1
        elif Nscan == 4:
            Nscan = 7
        elif Nscan > 6:
            Nscan += 1
    if (Nscan > 9) and (Nscan != 20):
        scan()
    offset += 3
    angleBoard['3'] = 42
    distBoard['3'] = 58


arm.angle = distBoard['5']
spinny.angle = 90 + offset
uppy.angle = 90
sleep(.3)
for i in range(9):
    colorBase += color.value
    sleep(.05)
colorBase /= 10
print(colorBase)
print("Move with the numpad.")
sleep(.4)
uppy.angle = 0
sleep(1)
uppy.angle = 90
sleep(1)
uppy.angle = 180
sleep(1)

#uppy.duty_cycle = (3800)   #This can be used to adjust the height of the magnet, if needed (3800 is down, 5800 is up)
#sleep(.05)
#uppy.duty_cycle = (0)


for i in range(5): # The main tic tac toe loop
    plan = 0
    print("Round " + str(round) + ", Your Turn")
    turn = 0
    printBoard(theBoard)
    print("--------------------------------")

#Start
    '''
    print("Where Would you like to move?")
    move = input("")

    try:
        if move == "]":
            uppy.duty_cycle = (5800)
            sleep(.15)
            uppy.duty_cycle = (0)
            i-=1
            continue
        elif move == "[":
            uppy.duty_cycle = (3800)
            sleep(.15)
            uppy.duty_cycle = (0)
            i -=1
            continue
        elif int(move) < 10 and  int(move) > 0 and theBoard[move] == " ":
            theBoard[str(move)] = "O"
            round += 1
        else:
            print("Invalid move, try again")
            i -= 1
            continue
    except:
        print("Invalid move, try again")
        continue
    '''
# End Removal
    while button.value:
        sleep(.5)
    print("Pressed!")
    scan()
    checkWin()
    if end == 1:
        i = 6
        break
    elif round == 10:
        turn = 2
        break

    printBoard(theBoard)
    sleep(1)
    turn = 1
    if True:
        if theBoard['7'] == theBoard['8'] == "X" and theBoard['9'] == ' ': # across the top
            place(9)
            round += 1
            turn += 1
        elif theBoard['9'] == theBoard['8'] == "X" and theBoard['7'] == ' ': # across the top
            place(7)
            round += 1
            turn += 1
        elif theBoard['7'] == theBoard['9'] == "X" and theBoard['8'] == ' ': # across the top
            place(8)
            round += 1
            turn += 1
        elif theBoard['4'] == theBoard['5'] == "X" and theBoard['6'] == ' ': # across mid
            place(6)
            round += 1
            turn += 1
        elif theBoard['6'] == theBoard['5'] == "X" and theBoard['4'] == ' ': # across mid
            place(4)
            round += 1
            turn += 1
        elif theBoard['4'] == theBoard['6'] == "X" and theBoard['5'] == ' ': # across mid
            place(5)
            round += 1
            turn += 1
        elif theBoard['1'] == theBoard['2'] == "X" and theBoard['3'] == ' ': # across bottom
            place(3)
            round += 1
            turn += 1
        elif theBoard['3'] == theBoard['2'] == "X" and theBoard['1'] == ' ': # across bottom
            place(1)
            round += 1
            turn += 1
        elif theBoard['1'] == theBoard['3'] == "X" and theBoard['2'] == ' ': # across bottom
            place(2)
            round += 1
            turn += 1
        elif theBoard['7'] == theBoard['4'] == "X" and theBoard['1'] == ' ': # down left
            place(1)
            round += 1
            turn += 1
        elif theBoard['1'] == theBoard['4'] == "X" and theBoard['7'] == ' ': # down left
            place(7)
            round += 1
            turn += 1
        elif theBoard['7'] == theBoard['1'] == "X" and theBoard['4'] == ' ': # down left
            place(4)
            round += 1
            turn += 1
        elif theBoard['8'] == theBoard['5'] == "X" and theBoard['2'] == ' ': # down mid
            place(2)
            round += 1
            turn += 1
        elif theBoard['2'] == theBoard['5'] == "X" and theBoard['8'] == ' ': # down mid
            place(8)
            round += 1
            turn += 1
        elif theBoard['8'] == theBoard['2'] == "X" and theBoard['5'] == ' ': # down mid
            place(5)
            round += 1
            turn += 1
        elif theBoard['9'] == theBoard['6'] == "X" and theBoard['3'] == ' ': # down right
            place(3)
            round += 1
            turn += 1
        elif theBoard['3'] == theBoard['6'] == "X" and theBoard['9'] == ' ': # down right
            place(9)
            round += 1
            turn += 1
        elif theBoard['9'] == theBoard['3'] == "X" and theBoard['6'] == ' ': # down right
            place(6)
            round += 1
            turn += 1
        elif theBoard['9'] == theBoard['5'] == "X" and theBoard['1'] == ' ': # right diagonal
            place(1)
            round += 1
            turn += 1
        elif theBoard['9'] == theBoard['1'] == "X" and theBoard['5'] == ' ': # right diagonal
            place(5)
            round += 1
            turn += 1
        elif theBoard['1'] == theBoard['5'] == "X" and theBoard['9'] == ' ': # right diagonal
            place(9)
            round += 1
            turn += 1
        elif theBoard['7'] == theBoard['5'] == "X" and theBoard['3'] == ' ': # left diagonal
            place(3)
            round += 1
            turn += 1
        elif theBoard['3'] == theBoard['5'] == "X" and theBoard['7'] == ' ': # left diagonal
            place(7)
            round += 1
            turn += 1
        elif theBoard['7'] == theBoard['3'] == "X" and theBoard['5'] == ' ': # left diagonal
            place(5)
            round += 1
            turn += 1

        elif theBoard['7'] == theBoard['8'] == "O" and theBoard['9'] == ' ': # across the top
            place(9)
            round += 1
            turn += 1
        elif theBoard['9'] == theBoard['8'] == "O" and theBoard['7'] == ' ': # across the top
            place(7)
            round += 1
            turn += 1
        elif theBoard['7'] == theBoard['9'] == "O" and theBoard['8'] == ' ': # across the top
            place(8)
            round += 1
            turn += 1
        elif theBoard['4'] == theBoard['5'] == "O" and theBoard['6'] == ' ': # across mid
            place(6)
            round += 1
            turn += 1
        elif theBoard['6'] == theBoard['5'] == "O" and theBoard['4'] == ' ': # across mid
            place(4)
            round += 1
            turn += 1
        elif theBoard['4'] == theBoard['6'] == "O" and theBoard['5'] == ' ': # across mid
            place(5)
            round += 1
            turn += 1
        elif theBoard['1'] == theBoard['2'] == "O" and theBoard['3'] == ' ': # across bottom
            place(3)
            round += 1
            turn += 1
        elif theBoard['3'] == theBoard['2'] == "O" and theBoard['1'] == ' ': # across bottom
            place(1)
            round += 1
            turn += 1
        elif theBoard['1'] == theBoard['3'] == "O" and theBoard['2'] == ' ': # across bottom
            place(2)
            round += 1
            turn += 1
        elif theBoard['7'] == theBoard['4'] == "O" and theBoard['1'] == ' ': # down left
            place(1)
            round += 1
            turn += 1
        elif theBoard['1'] == theBoard['4'] == "O" and theBoard['7'] == ' ': # down left
            place(7)
            round += 1
            turn += 1
        elif theBoard['7'] == theBoard['1'] == "O" and theBoard['4'] == ' ': # down left
            place(4)
            round += 1
            turn += 1
        elif theBoard['8'] == theBoard['5'] == "O" and theBoard['2'] == ' ': # down mid
            place(2)
            round += 1
            turn += 1
        elif theBoard['2'] == theBoard['5'] == "O" and theBoard['8'] == ' ': # down mid
            place(8)
            round += 1
            turn += 1
        elif theBoard['8'] == theBoard['2'] == "O" and theBoard['5'] == ' ': # down mid
            place(5)
            round += 1
            turn += 1
        elif theBoard['9'] == theBoard['6'] == "O" and theBoard['3'] == ' ': # down right
            place(3)
            round += 1
            turn += 1
        elif theBoard['3'] == theBoard['6'] == "O" and theBoard['9'] == ' ': # down right
            place(9)
            round += 1
            turn += 1
        elif theBoard['9'] == theBoard['3'] == "O" and theBoard['6'] == ' ': # down right
            place(6)
            round += 1
            turn += 1
        elif theBoard['9'] == theBoard['5'] == "O" and theBoard['1'] == ' ': # right diagonal
            place(1)
            round += 1
            turn += 1
        elif theBoard['9'] == theBoard['1'] == "O" and theBoard['5'] == ' ': # right diagonal
            place(5)
            round += 1
            turn += 1
        elif theBoard['1'] == theBoard['5'] == "O" and theBoard['9'] == ' ': # right diagonal
            place(9)
            round += 1
            turn += 1
        elif theBoard['7'] == theBoard['5'] == "O" and theBoard['3'] == ' ': # left diagonal
            place(3)
            round += 1
            turn += 1
        elif theBoard['3'] == theBoard['5'] == "O" and theBoard['7'] == ' ': # left diagonal
            place(7)
            round += 1
            turn += 1
        elif theBoard['7'] == theBoard['3'] == "O" and theBoard['5'] == ' ': # left diagonal
            place(5)
            round += 1
            turn += 1

        else:
            if round == 2 and theBoard['5'] == ' ':
                if random.randint(1,2) == 1:
                    place(5)
                else:
                    place(random.randint(1,9))
                round += 1
            else:
                while plan != 10:
                    plan = random.randint(1,9)
                    if theBoard[str(plan)] == ' ':
                        place(plan)
                        plan = 10
                        round += 1

    checkWin()
    if end == 1:
        i = 6
        break

printBoard(theBoard)
print("\nGame Over.\n")   
if turn == 0:             
    print(" ****  You  win. ****")
elif turn == 2:
    print(" ****  AI  wins. ****")
    
else:
    print("0 wuns")

```
</p>  
    
</details>

# `Building`
## Pieces
The pieces were always going to be the standard X’s and O’s, so the only question we had to ask ourselves was what acrylic we should use to make the tops. Since we are using an infrared sensor to detect piece locations on the board, we decided to use a bronze-looking color for the O’s, as it is easy for the sensor to see. The other color didn't matter as much, so we went with gray because it looks nice. 
Each piece has a magnet in it.
In hindsight, having the black base of the X's be a circle, like it is for the O's, would likely be better, so the robot can actually place them in the slots on the board.
## Board
The board design is similar to the pieces, having a black base and a colorful top. The handle-looking things on the top fit around the robot itself, so the board is always stable and the robot can navigate consistently. The purple piece on the side is where the robot grabs pieces to place on the board. The blue squares aren't for anything in particular.
![image](https://github.com/jvaugha3038/tictactobot/assets/112961338/30af98e3-6448-4cf5-84fa-e925a3a747ba)
## Box and Extendo Arm
The box is pretty simple; it's a box with a servo in it, which spins the arm on top of the box. It can also hold the metro M4, a battery pack, and all of the wiring. It has a handle on the back of it so the batteries can be changed out.
The arm is a rack and pinion gear set that helps position the magnet thing along with the servo in the box. Note that one of the gears is 3D printed.
It was originally 2 big gears connected to 3 small gears to create the gear ratio.
![image](https://github.com/jvaugha3038/tictactobot/assets/112961338/6bdf4a24-5df9-48dc-8946-23744670ae74)
## The Magnet thing a.k.a the Pick-Uppy Part
This part is responsible for picking up and dropping pieces, and it's attached to the arm. The servo lowers a magnet attached to a piece of string, which then connects to the magnet in a piece. Then, it can raise the magnet, disconnecting the piece and letting it fall onto the board.
![image](https://github.com/jvaugha3038/tictactobot/assets/112961338/7e5bf1fa-a4a1-4e7e-ad3a-5fa31a9c543d)
## The Whole Thingy
![image](https://github.com/jvaugha3038/tictactobot/assets/112961338/8314c021-9074-407b-83d8-fbfc39e086c0)

# `End Result`
The final product is, admittedly, not exactly what we had hoped for. Yes, it can pick up pieces and place them (with some difficulty), but it is also not great at seeing pieces on the board. This means that it occasionally will completely miss a piece, and place one on top of it. This does mean that we can say "Oh it was supposed to cheat anyway" and move on, but besides that, the robot itself is very creaky for some reason.

# `Problems`
## Big problems
* The gears in the arm were inconsistent.
  * We 3D printed one of them to solve the problem.
* The servo powering the rack and pinion kept destroying itself (tried to spin too hard)
  * We used the servo horn instead of screwing the gear directly onto the servo.
* Servo responsible for rotating the arm could move around in the box.
  * Added a small 3D printed part to the front of the box that allows the servo to be screwed in place.
* We waited about 2 weeks to get string and magnets
  * yeah that one is just our fault, get the stuff early next time
* String is flimsy and keeps breaking.
  * Make the servo spin slower (and hope for the best)
## Less severe problems
* Servo responsible for rotating the arm wasn’t aligned properly.
  * Added an offset into the code to account for this.
* Infrared sensor wires interfered with the box when fully retracted
  * We turned it around.
* Can't screw the infrared sensor in place after turning it around.
  * Completely unnecessary. 
* The top part of the arm interfered with the gears
  * We made cool looking fillets on it.
* Wires cross over the board
  * Didn't have time to do this, but we could have zip-tied the wires to other wires to keep them out of the way.
* The pieces get caught on the board when picked up from the zone on the side of the board.
  * We can stack 2 of the blue squares (or really any acrylic things)

# `Reflection`
