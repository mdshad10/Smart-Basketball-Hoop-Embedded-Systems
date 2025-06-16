<h1 style="text-align:center; font-size:2.5rem;">
  Smart Basketball-Hoop Scoring Tracker
</h1>
#### Overview 
Our final project is a smart basketball-hoop scoring tracker where a player has to make as many shots on the basketball hoop in a set period of time. We are using a low-power infrared break beam sensor which is mounted below the rim; every time the light connection is interrupted by the ball passing through and covering the laser, the microcontroller instantly registers a make, increments an internal score counter, and provides feedback in three ways. For every make that is registered, one of the serial LEDs will light up green for a second, the buzzer will play a sequence of frequencies representing a "Cha-Ching!" sound, and the updated score is displayed on the LCD. These features allow the player to know when they score and appreciate their make. The timer for how long the round will last is chosen by the player. Switch 1 is used to increment the timer by 5 seconds, and if it is held for 2 seconds or longer, it is set to the default 30 seconds for a round. Switch 3 is pressed when the player is satisfied with the amount of time set, and to start the game. Then a 3-second timer runs to allow the player to get ready, and when the buzzer plays a loud high high-pitched sound, it signals the start of the round. When the round starts, the timer starts counting down from the time set by the player, and the timer is displayed on the LCD. Once the time expires, which means the round has ended. 

Our inspiration for this project came from a smart basketball hoop that was recently installed at Tompkins Square Park in New York City. That system allows players to track how many shots they make within a certain time period while providing immediate feedback. Additionally, the same company, Huupe, has developed smart mini hoops that offer similar features for indoor use, combining shot tracking with visual and audio feedback to enhance the user experience. We wanted to recreate a similar experience using our own microcontroller and by applying the techniques we learned in embedded systems, including interrupt handling, real-time timing, and hardware control.

#### Final Project Video
<div style="margin: 2rem 0; border-radius: 8px; overflow: hidden;">
  <iframe src="https://drive.google.com/file/d/19WirKPllfCPig7lkLNfI8Ul6PIVjNu1b/preview"
          width="100%" height="480" style="border:0;" allowfullscreen></iframe>
</div>

#### System Diagram
<div style="margin:1.5rem 0; border-radius:8px; overflow:hidden;">
  <iframe src="https://drive.google.com/file/d/1VFXiUNhIHJkyfCTLeu_h4EKW3pZBmH1o/preview"
          width="100%" height="480" style="border:0;" allowfullscreen></iframe>
</div>

#### Technical Approach 
Our design works based on a finite-state machine (FSM) which cycles through IDLE to PREP to RUN to END. The FSM is event-driven, where we have a SysTick that provides a millisecond-accurate system clock, debounces buttons, and drives the loops for the timer and main countdown. There are also PORTC interrupts which handles when the SW1 and SW3 are pressed and manipulates the preset and running flags without using polling. There is an ADC-driven break-beam where each frame of the code sends a 12-bit conversion within channel 11 (PTC2). A valid score is counted when the result is >900 counts and the ISR-safe function of check_breakbeam() in our code updates a score variable, flashes the LED green, and triggers the buzzer to play a sound. The audio is synthesized using buzzer_play_tone(), which will toggle PTC1 at the necessary frequency while there are delays set in between each sound. The display is made using helper functions to convert integers into segments so they can be used to display the time and score independently. Regarding electrical usage, every peripheral used, such as the IR LED, photodiode, and piezo disk, operates with the 3.3 V rail, so no level shifting will be required. Then, a single-sided strip board mounts the beam hardware and connects the +3.3 V, GND, and the photodiode output to the PTC2. We decided not to use interrupts for the buzzer and instead use a blocking statement. If we did not use a blocking statement, the amount of time the laser is off will update the score almost continuously during that time, which is an issue because we only made 1 basket. Utilizing the blocking buzzer code adds a delay after the beam is broken, only updating the score once while the beam is off. Since the LED board takes up both 3.3V pins on the board, we had to find another way to provide 3.3V to the laser and photoresistor voltage divider. After some research, the pins on J9 provide 3.3V. We soldered some breakaways on and used them to power.

The KL46Z’s built-in four-digit LCD refreshes itself, so we only had to send new segment data when a value changed. Two helper functions allowed this: lcd_show_timer() wrote the remaining seconds to the left-hand pair of digits, while lcd_show_score() wrote the current score to the right-hand pair. Controlling the timer relies on two on-board buttons wired as edge-triggered GPIO interrupts. SW1 ( pin PTC3) adjusts the round length—each quick tap adds five seconds (up to 60 s), and holding it for at least two seconds snaps the timer to a 30-second “arcade” default. SW3 (pin PTC12) starts the action: once a time is selected, one press moves the finite-state machine out of IDLE, runs a three-second countdown, and begins play. Because both buttons are handled inside the shared PORTC_PORTD_IRQHandler, they are fully debounced and processed immediately, with no polling to slow down LCD updates or break-beam sampling.


##### Circuit Explanation
<div style="margin: 1.5rem 0; border-radius: 8px; overflow: hidden;">
  <iframe src="https://drive.google.com/file/d/1nXagdMgyYwe7BKZioC4WUuTYo7xBmQa5/preview"
          width="100%" height="400" style="border:0;" allowfullscreen></iframe>
</div>

##### Our Soldering Process
<div style="margin: 1.5rem 0; border-radius: 8px; overflow: hidden;">
  <iframe src="https://drive.google.com/file/d/1T0qnLg4qKQGk8JZTNSfdBw6wwczZrTkE/preview"
          width="100%" height="400" style="border:0;" allowfullscreen></iframe>
</div>

##### Our Setup 
<div style="margin: 1.5rem 0; border-radius: 8px; overflow: hidden;">
  <iframe src="https://drive.google.com/file/d/1W06Vmte9UukinsX4BY7eWaIZe00g4g5m/preview"
          width="100%" height="400" style="border:0;" allowfullscreen></iframe>
</div>

#### Testing and Debugging
We followed an incremental testing approach to ensure system stability at every step and isolate bugs efficiently. From the outset, we broke our project down into modular components, photoresistor-based breakbeam sensing, buzzer output, LCD display, and game timer logic, so we could build and verify each part individually before integrating them into the complete system. This strategy enabled us to spot and resolve issues early, avoiding the complexity of debugging a fully combined system all at once.

To begin, we tested the photoresistor breakbeam sensor independently. We connected the sensor to the microcontroller and printed its ADC values to the serial monitor. Through physical testing, dropping a ball through the hoop, we determined that light levels dropped significantly when the beam was interrupted, typically falling below 700 out of the 0 – 1023 ADC range. This empirical threshold became our scoring condition: if the value dropped below 700, we counted it as a valid score. These tests helped calibrate our check_breakbeam() logic and confirmed that the ball reliably disrupted the beam under normal play conditions.

Next, we tested the buzzer output. Using the buzzer_play_tone() function, we experimented with different frequencies and durations to generate a satisfying “Cha-Ching!” effect upon scoring. We also developed distinct tones for game start, end, and victory states. Fine-tuning these tones required iterative testing, as we adjusted frequencies and delays to make sure the sounds were easily recognizable and appropriately timed.

The LCD display testing came next. We used the lcd_show_timer() and lcd_show_score() helper functions to display both the countdown timer and the player’s current score. Initially, we manually updated score and timer values in code (e.g., incrementing the score every 5 seconds) to verify that the correct digits appeared on the right segment positions. This validated both digit conversion and the display. We also ensured that clearing and updating digits didn't cause flickering or ghosting on the LCD.

For game mechanics and timing, we tested the timer state machine independently of the breakbeam and buzzer components. We simulated a game by using hardcoded timers and updating the score in software every few seconds. This allowed us to confirm that the countdown logic, timer display, and end-of-game behavior all functioned correctly, even across different preset durations. We also stress-tested the game by toggling between different timer values and validating that score resets and game-end transitions happened cleanly.

Once each subsystem functioned as expected, we combined all the components. Integration went smoothly due to our modular testing. We refined timing interactions (e.g., ensuring that the blocking buzzer logic didn’t interfere with score registration) and added polish to the game flow, like including a victory sound if the score was greater than zero. This integration phase also allowed us to tweak the check_breakbeam() delay and LED feedback timing for better responsiveness and user experience.

When we physically mounted the system on the basketball hoop, we encountered only one significant concern: mechanical vibration. We were initially worried that the hoop's rattling might dislodge the laser from the photoresistor, falsely triggering or missing scores. However, thanks to careful hot glue application during assembly, the breakbeam sensor remained stable. The only minor issue we observed was that the net sometimes interfered with the laser beam and triggered false scores. We resolved this by taping the net to the side, preventing it from swinging into the beam’s path.

We also encountered an interesting design tradeoff while working with the buzzer implementation. Our buzzer_play_tone() function uses a blocking delay loop that toggles the output pin at a given frequency, effectively halting the CPU from running other tasks during the sound playback. Initially, we were concerned this could interfere with other real-time components of the game, most notably, the countdown timer or the responsiveness of the breakbeam sensor. We feared that during the time the CPU was locked in a buzzer routine, we might miss additional inputs or affect game timing accuracy.

However, this behavior actually worked in our favor. Because the breakbeam is momentarily blocked by the ball as it passes through the hoop, the sensor can sometimes register multiple beam breaks in rapid succession for a single shot. The blocking buzzer function unintentionally created a natural debounce period, preventing the check_breakbeam() logic from running again until the buzzer finished playing. This ensured that only one score was counted per make, even if the beam remained broken for a short duration. We confirmed through testing that the SysTick-based timer continued counting down correctly in the background, meaning the game’s length wasn’t affected. In this case, our initially suboptimal buzzer implementation provided a simple and elegant solution to an otherwise tricky edge case in scoring logic.

Ultimately, our systematic and incremental testing strategy ensured that the entire system functioned reliably during real gameplay. Each shot was accurately tracked, feedback was immediate and consistent, and the timer behaved predictably throughout the game.

#### Team Work 
To divide the workload efficiently, we split the project based on our individual strengths and interests. Kaelem focused on the analog and hardware-side tasks, which included designing the circuit, setting up and reading values from the photoresistor-based breakbeam sensor, and implementing the buzzer system. He was responsible for testing these components individually, determining reliable ADC thresholds for detecting a made basket, and fine-tuning the buzzer sound sequences to ensure they were responsive and recognizable.

On the other hand, Md concentrated on the digital and logic-side tasks, such as programming the countdown timer, managing the LCD display, and implementing the core game mechanics through a finite-state machine. He handled the setup and testing of the state transitions, display updates, and switch interactions to ensure the timer logic was robust and user-friendly. Md also worked on the buzzer sounds within the game, helping to fine-tune the basket indicator sound and compose the victory melody to align with the pacing and tone of the gameplay.

Throughout the project, we regularly communicated our progress and shared intermediate results to ensure compatibility between subsystems. Once both the analog and digital portions were independently tested and confirmed to work, we worked together to integrate them into the full game loop. This clear division of responsibilities allowed us to make steady progress while minimizing overlap and confusion, ultimately contributing to the success of the final system.

#### Outside Resources 
The FRDM-KL46Z datasheet was particularly helpful in determining which pins to use for different components and understanding where to solder connections for peripherals like the photoresistor, laser, and buzzer. It guided our hardware setup and ensured we were interfacing correctly with the microcontroller’s ADC and GPIO ports.

We also used generative AI tools to supplement our knowledge in areas not covered during class. For example, we used AI to learn how to read analog signals in C using the KL46Z’s ADC hardware, since analog-to-digital conversion was not directly covered in the lab curriculum. AI tools were also instrumental in helping us generate and fine-tune buzzer tones and melodies. We received help designing the sequences for different game events, such as the basket indicator sound, the game start beep, and the victory jingle, and made additional adjustments ourselves to fit the timing and tone we wanted in our game. Moreover, AI was used to clean up our language and improve the clarity in our explanations throughout the website, video and code. 

In addition, we utilized the provided demo code and libraries for the LCD and onboard switches to simplify integration and ensure proper functionality. These resources allowed us to easily display the score and countdown timer on the LCD screen and handle user input using Switches 1 and 3 for setting and starting the game timer.
