# UTS DRC 2022

> This project is not maintained and may be broken.

## Summary 📝

This repo is a cleaned-up and commented version of the logic code written by Jakob Tonnaer Lewis for the [UTS Robotics Society's](http://utsroboticssociety.org/) submission to [QUT's Droid Racing Challenge](https://qutrobotics.com/droid-racing-challenge/) in 2022. Successfully deployed on "Autonomous Delicacy", it helped our team claim 3rd place!

Microcontroller code is not included in this repo.

## Installation 📦

This project requires Python above version 3.5 with the following packages; `pip3 install opencv-python numpy pyserial`.

## Usage 🏁

```python
from src import Vision, link, unlink, send_data

# Initialise video source and sideline colours using default values (see Vision.reload_configuration() for more)
vision = Vision.start()

# Connect to the microcontroller
link(vision.settings["serial"]["port"])

# Run until the video source stops providing frames
while vision.next_frame():

    # Extract relevant colours
    vision.set_masks()

    # Find contours/obstacle shapes
    vision.find_contours()

    # Slice up contours
    vision.find_bounds()

    # Find gaps between obstacles, ignoring "illegal paths"
    vision.generate_path_blocks()

    # Generate a path forward
    vision.find_path()

    # Attempt to smooth path
    path = vision.optimise_path()

    # Turn toward the most immediate path segment
    vision.add_turn_destination(path[0][0])

    # Check if we've crossed the finish line
    vision.finish_line_detection()

    # If we've completed the track, reset our steering, stop the droid and exit
    if vision.info["track_complete"]:
        send_data(
            vision.info["neutral_steering_position"],
            vision.settings["serial"]["stop_speed"]
        )
        exit()

    # If we are turning by less that 40% of the screen width, drive at "boost_speed"
    current_speed = vision.settings["serial"]["boost_speed"] if vision.turning_certainty() < 0.4 else vision.settings["serial"]["go_speed"]
  
    # Transmit turning and speed values to the microcontroller
    send_data(
        vision.current_steering(),
        current_speed 
    )

```
