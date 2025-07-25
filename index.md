# Object Tracking Robot

This robot uses a color detection process from OpenCV masking and contour functions, along with a proximity sensor and Pi Camera, to follow the ball and park when within a certain proximity of the ball. 

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Derek L | Irvington High School | Electrical Engineering / Quantum Computing / LLM Research | Incoming Junior

<img src="https://github.com/user-attachments/assets/6bad424a-9478-41a8-83da-4c46421c0a5b" width="300" height="400">

<!---

# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/F7M7imOVGug" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE
-->

## Summary

The final milestone involves the complete integration of the modules from the previous two milestones (the Raspberry Pi camera and the Pi 4B computer), along with the complete mechanical, electrical, and software assembly of the project. This mainly involved putting together new modules and the coding portions for them. However, do note that most of this mainly revolved around the Pi camera's input, which effectively serves as the system's eye. First of all, I put together the dual-motor system with the L298N motor drive board, which controls speed, current, and direction of the motors, along with system installation of the motors themselves. By obtaining the center coordinates of the contour rectangle (from milestone 2) and comparing it to the camera frame's total pixels, I was able to find out whether the robot should go forward, left, or right (based on how far off the center is from the x-value center of the camera) by setting a buffer. Plus, by using the Enable pins on the drive board and controlling the PWM, indirect adjustment to the motor's speed could be achieved. 

I also implemented the ultrasonic distance sensor, which would detect distance using ultrasound pulses. There are 4 pins -- aside from VCC (voltage common collector) and ground to provide power, there is the trigger and echo. Trigger sends out a set of 8 ultrasound pulse every 10 microseconds; Echo, on the other hand, recieves the pulses and calculates the average time it took for all 8 to bounce back. Halving this duration and multiplying it by the speed of sound, we can find an estimation of how far the robot is from anything directly in front of it. This way, when the ball is close enough to the robot, the robot will know to "park", or stop moving. Using the implemented motors and sensor, I can make the robot successfully follow the ball. Additionally, I added a feature such that if the robot does not find the ball, it circles idly in a given direction until some red contour with area large enough to surpass threshold (a custom variable I set and tweaked) appears in the camera frame. Just like that, my project is complete! 

## Challenges (oh boy)

As most engineers would know, engineering is not all sunshine and rainbows -- just when you think everything is going great, something, just _something_, has to pop up -- and this is quite literally what I faced. 

# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/A1Gcot1nlMw?si=G1XRxM19wIM34EA7" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Summary
This milestone was mainly getting the color masking to work. For the robot to actually detect the ball, it needs a camera, and with a camera it needs to be able to detect a certain color, in this case red. I will do this by using OpenCV masking, which in essence replaces all pixels other than the color detected with black, and pixels with the color detcted red. Then, a contour function creates contours around each blob of red detected in the image, turning them into individual objects. Afterwards, a largest contour foreloop automatically cycles out any smaller contours and only leaves the largest contour on the screen, and then draws a box around it to show its location and approximate center (thus an approximate direction to move in). Finally, the box is re-applied onto the original image frame. All this is done live and continuously via a while(true) loop, so that the video will automatically update itself each time. 

Here is what the Pi sees as a result, with the mask shown below and the largest blob of red boxed in the original frame. 


<img src="https://github.com/user-attachments/assets/05f28ae5-ddb5-4759-b697-380c30a26aea" width="800" height="600">
<img src="https://github.com/user-attachments/assets/3c5fa87e-7fbb-4b77-b76d-939059608a23" width="800" height="600">


## Code

```python
from picamera2 import Picamera2
import cv2
import time

def maskon(dframe):
    global redmask
    lower = np.array([0, 0, 150])   # Bound range for red color
    upper = np.array([100, 100, 255])  


    # lower = np.array([150, 0, 0]) # If you want a BLUE ball
    # upper = np.array([255, 100, 100])  


    redmask = cv2.inRange(dframe, lower, upper) # For color in range, highlight

    (h,w) = redmask.shape # Height and Width of entire mask
    return redmask


def largestcontour(dmask):
           
    contours, hierarchy = cv2.findContours(dmask, cv2.RETR_CCOMP, cv2.CHAIN_APPROX_SIMPLE) # Function from OpenCV to detect contours


    contourslist = [] # List of contours, later to be filtered
   
    for cnt in contours:
        contourslist.append(cnt) # Add contours to contourslist
       
    if(len(contourslist) > 0): # If contours EXIST
        maxcontour = contourslist[0] # Initialize
        maxarea = cv2.contourArea(contourslist[0]) # Maximum area 


       
        for i in range(len(contourslist)): # For loop to sort out largest contour
            nowarea = cv2.contourArea(contourslist[i])
            if (nowarea > maxarea):
                maxarea = nowarea
                maxcontour = contourslist[i]


        return maxcontour 


def drawrectangle():
    global frame
    global center_x
    global center_y


    frame = camra.capture_array() # Captures video of camera
    frame = np.asarray(frame)
    height = frame.shape[0]
    width = frame.shape[1]


    center_x = 0
    center_y = 0
    frame = cv2.cvtColor(frame, cv2.COLOR_RGB2BGR)
    maskred = maskon(frame)


   
    conter = largestcontour(maskred)
   
    (x,y,w,h) = cv2.boundingRect(conter)
    cv2.rectangle(frame, (x,y), (x+w,y+h), (0, 255, 0), 2)


    ballcoords[0] = x+w/2 # Coordinates of center of contour that is on the ball (hopefully)
    ballcoords[1] = y+h/2


    rectanglesize[0] = w # Width and height of the rectangle, for use in thresholds later
    rectanglesize[1] = h



```


## New Components Used 
- Raspberry Pi Camera Module

## Challenges
This milestone came with some pretty big challenges. First of all, I had cluttered my code with a lot of redundant information irrelvant to the goal. This is because I had never coded with Python before (but I have experience with Java and C++, so I have a general idea of logic and concepts such as for-loops but not syntax), and as a result I derived most of my code from the exemplar portfolio and websites on color detection. In the end, I managed to code my own program from the examples given, and it is much simpler and readable than the other ones. Lesson learned: you must write your own code and make your own calculations to truly understand how it works. It is better to use other sources for reference to get an idea of what to do, if I have no idea on how to get started. 

Second of all, the Pi and camera module kept 'mysteriously' quitting each time. Sometimes, it was software issues, and other times it was just my general lack of knowledge on how Pi cameras work. The first time, the camera kept not being found, even after I power-cycled multiple times. After 10 minutes of internet surfing, I finally found out that I needed to 'awaken' the camera via libcamera-hello command in terminal. Other issues like this involve similar ideology: when I switched Wifi for my Pi due to malfunctions with the prior one, I forgot to change my IP address when logging in, and this took around 15 minutes for me to find out what was going on. On the bright side, though, challenges like this have taught me a lot about how these microcontrollers and equipment work, and each time I face an obstacle, as cheesy as it sounds, I am learning something new. 

## Next Up
For the final milestone, I will finish the mechanical assembly of the robot and finish testing and integration of the final 2 systems, the motor drive system and the proximity sensor, along with the general wiring together of the robot, which I hope will be a straightforward process. 

# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/SzkFIkVHrx8?si=s0iFrylLdsMsdEdA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Summary
This first milestone, I aimed to finish the setup of the Raspberry Pi computer, ensuring basic functionality for ease of access later during the coding portion. This process was a tumultuous one, and I ran into multiple issues. Everything worked in the end, but I will nevertheless need to fix the Pi more over the next milestone. First, I flashed the SD card to access the Pi from my computer. Next, I used PuTTY to SSH into the Pi -- I ran into some issues with the Pi password from my computer terminal, so I had to walk around it with PuTTY. Afterwards, I used VS Code and related python modules to code SSH into the Pi, proving that the camera functions. I demonstrated taking a picture using the Pi camera during the demo to prove that the camera works.

An image of the newborn(!) Pi from my VNC is below



<img src="https://github.com/user-attachments/assets/4befc68e-30eb-465f-bdf0-facae4d67537" width="800" height="600">


## Components Used
- Raspberry Pi microcomputer
- Extra set of Keyboard and Mouse

## Challenges
Before I could access the Pi, I had to flash the SD Card. The SD Card flashing innately takes a long time -- adding on to the issue was that day the School internet was especially faulty. As a result, after flashing until 74%, it stopped several times, got stuck, and I had to restart out of fear of corruption due to the internet closing down mid-flash. The internet was on-off that day, and as a result I attempted restart three times, wasting precious time.

Another challenge I faced would be the SSH accessing process. My terminal would not boot into the Pi, no matter if I put the correct password or not. Thus, SSH via computer terminal was futile, so I had to do a walkaround with the PuTTY SSH, in which the password somehow worked.

## Next Up
Next milestone will involve the main coding portion of this project, where I will complete the bulk of the image recognition and robot maneuver code, mostly done in Python. The final milestone will involve putting it all together, where I build the chassis and wire the robot together, completing the hardware portion.

# Bill of Materials

Here are the components necessary for my ball tracking robot, along with their uses and price.

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Breadboard | Reusable prototyping circuit board for circuit drafts, planning, and testing | 0.68$ | <a href="https://www.cytron.io/p-breadboard-8.5x5.5cm-400-holes"> Link </a> |
| Robot Chassis Kit | Backbone of robot hardware, holds robot together and supplies the wheels | 13.99$ | <a href="https://www.amazon.com/Smart-Chassis-Motors-Encoder-Battery/dp/B01LXY7CM3"> Link </a> |
| Raspberry Pi Model Case | Camera for the robot to locate red ball | 8.99$ | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| 3 Pack Proximity Sensor | Proximity Sensor for robot to detect surroundings | 6.99$ | <a href="https://www.amazon.com/Ferwooh-Ultrasonic-Distance-Measuring-Mounting/dp/B0D1MDP9V3"> Link </a> |
| Lighter Core Power Bank | Provides power for the robot | 17.99$ | <a href="https://www.amazon.com/INIU-High-Speed-Flashlight-Powerbank-Compatible/dp/B07CZDXDG8?th=1"> Link </a> |
| HDMI Video Capture | Captures and converts camera content | 9.98$ | <a href="https://www.amazon.com/Audio-Express-AXHDCAP-Broadcasting-Conference/dp/B0C2MDTY8P?source=ps-sl-shoppingads-lpcontext&ref_=fplfs&psc=1&smid=A2WUJZ46ZAD8LN&gQT=1"> Link </a> |
| Raspberry Pi 4 Starter Kit | Central computer for the robot | 119.95$ | <a href="https://www.canakit.com/raspberry-pi-4-starter-kit.html?srsltid=AfmBOoqIiOnjV3ixvPdr8wr3ZbNEp-_VpxdP3QmLGXa8yDdUkf-fmDZp"> Link </a> |
| Amazon Keyboard and Mouse | Required to manually control Pi | 22.49$ | <a href="https://www.amazon.com/AmazonBasics-Wireless-Keyboard-Mouse-Combo/dp/B0787CVBWP"> Link </a> |

<!---

# Other Resources/Examples
One of the best parts about Github is that you can view how other people set up their own work. Here are some past BSE portfolios that are awesome examples. You can view how they set up their portfolio, and you can view their index.md files to understand how they implemented different portfolio components. 
- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)

-->

# Starter Project Presentation: Mini Retro Arcade Console

<iframe width="560" height="315" src="https://www.youtube.com/embed/4dgTTYITtgQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

The arcade was an advanced soldering project that I completed as a warm-up. Just like the name implies, this is a LED based arcade console with a retro fit. Two 8x8 LED matrix boards serve as a game screen, and 6 buttons were soldered in for the user to play. The acrylic casing protects the console and adds an aesthetic feel to the console, making it feel more solid and easier to hold. It is powered by 3 AAA batteries and contains 5 types of games, along with a mute and brightness setting option. 

In the end, the project worked perfectly. A big challenge was soldering the two 8x8 LED matrices onto the PCB board -- since the board was very intricate, I needed to solder joints within close proximity of each other, which meant that the solder would easily goop out of place and connect two joints instead of one. This would cause a short circuit if not spotted in time -- so I had to be extra careful during this step. In the end, here is what it looks like.

<img src="https://github.com/user-attachments/assets/419d1657-0833-454c-92bb-a9e95db0d1f5" width="300" height="300">

# Starter Schematics

<img src="https://github.com/user-attachments/assets/09c17658-9248-4e3c-97f1-e4670ab90f49" width="600" height="600">


## Bill of Materials

All the parts came in the Console Soldering Kit. The parts are listed below. 

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Acrylic Encasing | Protects console and adds aesthetic feel | Estimated 15$ | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Main PCB Board | Operating center of the game | Estimated 20$ | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| LED 8x8 Matrix Board | Displays the game | Estiamted 30$ (according to Amazon) | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Buttons and Power Switch | Allows user to interact with game | Estimated 10$ | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
