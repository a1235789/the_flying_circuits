#!/usr/bin/python3

###  PiBot class library
#
#    Python version 3.0
#
#    PWM version

import RPi.GPIO as GPIO
import time


#  Motor pin numbers
#
#  Left forwards   - 19
#  Left backwards  - 21
#  Right forwards  - 24
#  Right backwards - 26
#
#  States:
#          LF  LB  RB  RF
#           0   0   0   0      Stop
#           1   0   0   1      Forward
#           0   1   1   0      Back
#           0   0   0   1      Turn left
#           1   0   0   0      Turn right
#           0   1   0   1      Spin left
#           1   0   1   0      Spin right
#

class Motors():
    # Set variables for the GPIO motor pins
    #         RF  RB  LB  LF
    pins  = [ 24, 26, 21, 19 ]
    ports = [ None, None, None, None ]
    speed = [ 0, 0, 0, 0 ]
    debug = False
    state = 0
    trim  = 0
    minimumTrim = -25
    maximumTrim = 25

    ##  setState - set motor state
    #
    def setState(self, state, text):
        self.state = state
        for i in range(4):
            self.ports[i].ChangeDutyCycle(0)
        for i in range(4):
            if (state & 1):
                self.ports[i].ChangeDutyCycle(self.speed[i])
            state = state >> 1
        if self.debug == True:
            print(text)


    # Initialisation method
    #
    def __init__(self, debug = False, speed=50, trim=0):
        for i in range(4):
            self.speed[i] = speed
        self.trim  = trim
        self.debug = debug

        GPIO.setwarnings(False)
        GPIO.setmode(GPIO.BOARD)       #  Use board numbering

        for i in range(4):
            GPIO.setup(self.pins[i], GPIO.OUT)
            self.ports[i] = GPIO.PWM(self.pins[i], 20)
            self.ports[i].ChangeDutyCycle(0)
            self.ports[i].start(0)

        if self.debug == True:
            print("PiBot init method run")


    def Setup(self):
        if self.debug == True:
            print("setup motors run")


    def Shutdown(self):
        self.Stop()
        GPIO.cleanup()
        if self.debug == True:
            print("Motors shut down")

    def Drive(self, left, right, duration=0):
        #    RF  RB  LB  LF
        d = [ 0,  0,  0,  0 ]

        #  Check range
        if left > 100:
            left = 100
        elif left < -100:
            left = -100
        if right > 100:
            right = 100
        elif right < -100:
            right = -100

        #  Set up control matrix
        if left < 0:
            d[2] = -left
        else:
            d[3] = left
        if right < 0:
            d[1] = -right
        else:
            d[0] = right

        #  Activate motors
        for i in range(4):
            self.ports[i].ChangeDutyCycle(d[i])

        #  Debug required?
        if self.debug == True:
            print("Motor set to: " + str(d))

        #  If duration > 0 then wait for this amount of time and stop the motors
        if duration > 0:
            time.sleep(duration)
            self.Stop()

    def Stop(self):
        self.setState(0, "Stopped")


    def SetTrim(self, trim):
        if trim < self.minimumTrim:
          trim = self.minimumTrim
        if trim > self.maximumTrim:
          trim = self.maximumTrim
        self.trim = trim
        self.SetSpeed(self.speed[2])


    def SetSpeed(self, speed):
        if speed > 100:
            speed = 100
        elif speed < 0:
            speed = 0
        if (speed+self.trim) > 100:
            speed = 100 - self.trim
        elif (speed - self.trim) < 0:
            speed = self.trim
        self.speed[0] = speed + self.trim  #  Right forward
        self.speed[1] = speed - self.trim  #  Right back
        self.speed[2] = speed              #  Left back
        self.speed[3] = speed              #  Left forward
        if self.state != 0:
            self.setState(self.state, "Speed change")

    def Forward(self):
        self.setState(9, "Forward")


    def Backward(self):
        self.setState(6, "Backward")


    def TurnLeft(self):
        self.setState(1, "Turn Left")


    def TurnRight(self):
        self.setState(8, "Turn Right")


    def SpinLeft(self):
        self.setState(5, "Spin Left")


    def SpinRight(self):
        self.setState(10, "Spin Right")
                

class Led():
    pinLED = 31
    led = None
    debug = False

    
    # Initialisation method
    #
    def __init__(self, debug=False, pin=31, name=None):
        self.debug = debug
        self.pinLED = pin
        self.name = name
        if self.name == None:
            self.name = "(" + str(pin) + ")"
        GPIO.setmode(GPIO.BOARD)
        GPIO.setup(self.pinLED, GPIO.OUT)
        self.led = GPIO.PWM(self.pinLED, 100)
        self.led.start(0)
        print("LED interface '" + self.name + "' initialised using pin: " + str(self.pinLED))

    def On(self):
        self.led.ChangeDutyCycle(100)
        if self.debug == True:
            print("Turning '" + self.name + "' LED on")

    def Off(self):
        self.led.ChangeDutyCycle(0)
        if self.debug == True:
            print("Turning '" + self.name + "' LED off")

    def Intensity(self, value):
        self.led.ChangeDutyCycle(value)
        if self.debug == True:
            print("Setting '" + self.name + "' LED intensity to " + str(value))


class Ultrasonic():
    pinTrigger = 11
    pinEcho    = 12
    samples    = 5
    debug      = False
    
    # Initialisation method
    #
    def __init__(self, debug = False, trigger=11, echo=12, samples=5, name=None):
        self.debug = debug
        self.pinEcho = echo
        self.pinTrigger = trigger
        self.samples = samples
        self.name = name
        if self.name == None:
            self.name = "(" + str(self.pinTrigger) + "," + str(self.pinEcho) + ")"
        GPIO.setmode(GPIO.BOARD)
        GPIO.setup(self.pinTrigger, GPIO.OUT)
        GPIO.setup(self.pinEcho, GPIO.IN)
        print("Ultrasonic interface " + self.name + " initialised using pins: " + str(trigger) + " and " + str(echo))

    def Distance(self, samples=None):

        if samples != None:
            if samples < 1:
                samples = 1
            elif samples > 100:
                samples = 100
            self.samples = samples

        distance = 0
        count = 0
        for i in range(self.samples):
            # Send a 10 micro-second trigger pulse
            GPIO.output(self.pinTrigger, True)
            time.sleep(0.00001)
            GPIO.output(self.pinTrigger, False)
   
            # get the start and end times (wait until pin goes high)
            startTime = time.time()
            endTime = startTime
            while (GPIO.input(self.pinEcho) == 0) and ((startTime - endTime) < 0.04):
                startTime = time.time()
   
   
            # Wait for the echo to go low
            endTime = time.time()
            while (GPIO.input(self.pinEcho) == 1) and ((endTime-startTime) < 0.04):
                endTime = time.time()
   
            elapsedTime = endTime - startTime
            if (elapsedTime < 0.04) and (elapsedTime > 0):

                # Calculate elapsed distance and scale
                distance = distance + (elapsedTime * 17163)
                count = count + 1

            # Small delay to ensure read frequency never exceeds 40Hz
            time.sleep(0.025)

        # Samples read so average
        if count == 0:
            print("Distance sensor " + self.name + " unable to return value")
            return -1
        distance = distance / count
        if self.debug == True:
            print("Distance from sensor " + self.name + " is: " + str(distance))
        return distance

class LineFollower():
    # Set variables for the line detector GPIO pin
    pinLineFollower = 22
    debug = False;
    
    # Initialisation method
    #
    def __init__(self, debug=False, pin=22, name=None):
        self.debug = debug
        self.pinLineFollower = pin
        self.name = name
        if self.name == None:
            self.name = "(" + str(self.pinLineFollower) + ")"
        GPIO.setmode(GPIO.BOARD)
        GPIO.setup(self.pinLineFollower, GPIO.IN)
        print("Line follower interface " + self.name + " initialised using pin: " + str(self.pinLineFollower))

    def Read(self):
        # Read the sensor
        d = GPIO.input(self.pinLineFollower)
        if self.debug == True:
            print("Line follower " + self.name + " returned: " + str(d))
        return d
    
class Pins():

    # States are:  0 - not defined, 1 - input, 2 - output, 3 = PWM
    # Pins are referenced as Jumper (9 or 10) and pin 1 - 4
    # or for PWM pins Jumper (5 or 6) and pin 3


    validation = [ (9, [ (1, 7), (2, 15), (3, 16), (4, 13) ] ),
                   (10,[ (1, 18), (2, 23), (3, 29), (4, 33) ] ),
                   (5, [ (3, 35) ] ),
                   (6, [ (3, 36) ] )
                 ]
    INPUT    = 1
    OUTPUT   = 2
    PWM      = 3
    mode     = 0
    header   = 0
    position = 0
    port     = 0
    pwmPort  = 0
    state    = 0
    debug    = False

    # Initialisation method
    #
    def __init__(self, header, position, mode, debug=False, name=None):
        self.debug = debug
        self.name = name
        GPIO.setmode(GPIO.BOARD)
        self.state = -1
        for v in self.validation:
            if v[0] == header:
                for p in v[1]:
                   if p[0] == position:
                       self.header = header
                       self.position = position
                       self.mode = mode
                       self.port = p[1]
                       self.state = 0
                       break
        if self.state == 0:
            if self.mode == self.INPUT:
                GPIO.setup(self.port, GPIO.IN)
            elif self.mode == self.OUTPUT:
                GPIO.setup(self.port, GPIO.OUT)
            else:
                if (header != 5) and (header != 6):
                    print("PWM can only be used with J5 and J6")
                    return
                GPIO.setup(self.port, GPIO.OUT)
                self.pwmPort = GPIO.PWM(self.port, 20)
                self.pwmPort.ChangeDutyCycle(0)
                self.pwmPort.start(0)
            if self.name == None:
                self.name = "(J" + str(self.header) + ",P" + str(self.position) + ")"
            print("Interface " + self.name + " on header J" + str(self.header) + ", position " + str(self.position) + " initialised")
        else:
            print("Invalid pin definition")


    def Output(self, value=0):
        if value != 0:
          value = 1
        if self.mode == self.OUTPUT:
            GPIO.output(self.port, value)
            if self.debug == True:
                print("Output " + str(value) + " to Pin " + str(self.position) + " on header J" + str(self.header))
        elif self.debug == True:
            print("Pin not configured for output")
        return

    def Input(self):
        value = 0
        if self.mode == self.INPUT:
            value = GPIO.input(self.port)
            if self.debug == True:
                print("Read " + str(value) + " from Pin " + str(self.position) + " on header J" + str(self.header))
        elif self.debug == True:
            print("Pin not configured for output")
        return value

    def Pwm(self, value=0):
        if value < 0:
          value = 0
        if value > 100:
          value = 100
        if self.mode == self.PWM:
            self.pwmPort.ChangeDutyCycle(value)
            if self.debug == True:
                print("Pin " + str(self.position) + " on header J" + str(self.header) + " set to " + str(value) + "%")
            return
        elif self.debug == True:
            print("Pin not configured for PWM")
        return

import sys
import tty
import termios

class Keyboard():
    def __init__(self):
        print("keyboard interface initialised")

    def ReadChar(self):
        fd = sys.stdin.fileno()
        old_settings = termios.tcgetattr(fd)
        try:
            tty.setraw(sys.stdin.fileno())
            ch = sys.stdin.read(1)
        finally:
            termios.tcsetattr(fd, termios.TCSADRAIN, old_settings)
        if ch == '0x03':
            raise KeyboardInterrupt
        return ch
    
    def ReadKey(self, getchar_fn=None):
        getchar = getchar_fn or self.ReadChar
        c1 = getchar()
        if ord(c1) != 0x1b:
            return c1
        c2 = getchar()
        if ord(c2) != 0x5b:
            return c1
        c3 = getchar()
        return chr(0x10 + ord(c3) - 65)  # 16=Up, 17=Down, 18=Right, 19=Left arro
    
      
