# Focus-Study-Timer
# Micro:Bit Prototype
# By Anna Gil-Munoz


# Overview

The Focus Study Timer is a simple timer built with the Micro:Bit to help students manage their study sessions using the Pomodoro technique. It’s designed to promote focus and reduce distractions by providing a physical, offline timer instead of relying on phones or laptops.
- Study duration: 25 minutes  
- Break duration: 5 minutes  
- LED feedback: 1 LED lights up per minute, building cumulatively over 25 minutes  
This prototype demonstrates physical computing with Python, including button input, LED control, and sound output.

# How It Works

1. Button A- Starts a new 25-minute study session.  
2. LED Grid- Displays progress; 1 LED lights up per minute until all 25 are lit.  
3. Button B- Stops the timer immediately.  
4. End of Study- Displays a happy face, plays a short sound, and signals the 5-minute break.  

# Hardware Required

- 1 x Micro:Bit
- USB cable (for programming and power)  
- Battery pack for portable use  

# Software Requirements

- MicroPython environment for Micro:Bit  
- Micro:Bit libraries:  
  - microbit
  - music

# How to Run

1. Connect your Micro:Bit to your computer via USB.  
2. Copy the `focus_timer.py` script to the Micro:Bit (Option to connect Battery pack after this for portability) 
3. Press Button A to start the timer.  
4. Press Button B at any time to stop/reset the timer.  
5. Observe the LED grid light up cumulatively over 25 minutes.  
6. At the end of the session, a happy face appears, scroll writing and a sound plays to signal break time.  

# Code Features

- Cumulative LED display for visual study progress  
- Immediate stop/reset with Button B  
- Break timer with heart LED display  
- Sound feedback at the end of study sessions  
- Fully commented for beginner-friendly understanding  

# Future Improvements

- Adjustable study/break durations  
- Optional vibration or buzzer alert  

## References

- micro:bit - Getting started - Get coding. (n.d.). Microbit.org. https://microbit.org/get-started/getting-started/get-coding/
- Micro:bit Educational Foundation. (2020, October 12). Sensing and making sound on the BBC micro:bit. YouTube. https://www.youtube.com/watch?v=waIdGCitbH4
- micro:bit Python Editor. (n.d.). Python.microbit.org. https://python.microbit.org/v/3
- Nicholas H. Tollervey. (2017). Programming with MicroPython. O’Reilly Media, Inc.
- Noteberg, S. (2010). Pomodoro Technique Illustrated: The Easy Way to Do More in Less Time (1st ed.). Pragmatic Programmers, LLC, The.



# MicropPython Code Below

# Focus Study Timer using the Pomodoro Technique by Anna Gil-Munoz
# Button A starts the timer, Button B resets it / stops it early


from microbit import *
import music # sound to play at completion of 25 minute focus


# Variables 
study_minutes = 25 # Duration of study session
break_minutes = 5 # Duration of break 
led_steps = 25 # Number of LED Updates (1 per minute)
running = False # Track whether timer is active 
current_step = 0 # Tracks how many pminutes have passed during the session


# Function- show cumulative progress on LEDs
def show_progress(step):
    # Light only the next LED without clearing the previous ones
    x = (step - 1) % 5 # LED Column position
    y = (step - 1) // 5 # LED Row position
    display.set_pixel(x,y,9) # Brightness 0-9 (9=full)

# Function- show break timer (5 minutes)
def show_break_timer():
    display.scroll("RELAX")  # Tell the user it's break time
    for minute in range(break_minutes):
        for _ in range(600):  # 600 * 100ms = 1 minute
            if button_b.is_pressed():  # Allow early stop
                display.scroll("STOPPED")
                display.show(Image.NO)
                return
            sleep(100)
        display.show(Image.HEART)  # Show heart each minute to mark time (Per Minute)
        sleep(500)
    display.scroll("READY?")  # End of break message
    display.show(Image.ASLEEP) # 


# Function- study complete
def study_complete():
    display.show(Image.HAPPY) # Smiley face on LED at end of focus time
    music.play(music.POWER_UP) # Sound to alert completion 
    display.scroll("BREAK TIME!") # LED "BREAK TIME" 
    show_break_timer()  # Start 5 minute break

    
# Function- start study timer
def start_timer(): # Starts timer when button A is pressed
    global running, current_step # Ensures LED go up each minute 
    running = True # Shows the timer is active (study session had begun)
    current_step = 0 # Resets the LED back to 0 per session
    display.scroll("FOCUS!") # LED text to confirm the session has begun

    while current_step < led_steps:
        # 1 minute = 600 * 100ms = 60,000 ms
        for _ in range(600):
            if button_b.is_pressed():  # Stop/reset immediately
                running = False # Stops the timer and LED'increasing 
                display.scroll("STOPPED") # Shows user their action has worked and the timer has "STOPPED"
                display.show(Image.NO) # Display of an X
                return
            sleep(100)  # 100ms chunk
        current_step += 1
        show_progress(current_step)  # Light 1 LED per minute (for 25 minutes)

    if running: # checks whether the timer finished naturally (it wasn't stopped early)
        running = False # After completion of study period, timer is switched off
        study_complete() # To play sound and LED's to signal completion of study period


# Main Loop
while True: # This loop makes sure the microbit responds to buttons being pressed
    if button_a.was_pressed() and not running: # Only starts a new study session if Button A is pressed and no timer is currently active (prevention of accidental restarts in the middle of a session)
        start_timer() # Starts study timer and LED displays 
    sleep(100)  # Short pause to ensure a smooth main loop

