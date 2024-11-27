1 - Best Case Scenario:
1.1 File Setup
Retrieve all the files from the project_NAO archive and unzip it. The folder structure should look like this:
- ROB311_Project_NAO_DEMAZURE_Varganyi:
    - NAO_exp:
        - behaviours_1 (folder)
        - translations (folder)
        - manifest.xml
        - NAO_exp.pml
    - PC_server_v2.py

1.2 Open the Required Files
Open PC_server_v2.py and the Choregraphe project on your computer.

1.3 Connect NAO to Choregraphe
Wake up NAO and connect it to your Choregraphe instance using the "use fixed IP/hostname" method:

Listen to NAO's address after pressing the button on its chest.
Manually type NAO's address into Choregraphe.
Ensure that the address is valid.
Once the address is confirmed, copy and paste it into line 13 of the PC_server_v2.py code (variable : NAO_IP)

1.4 Execute the Programs
Run the program in Choregraphe. As soon as you hear NAO say "setting up connection," run PC_server_v2.py.

1.5 Verify
The sequence should execute successfully.


2. Program Description and Expected Working Sequence

2.1 TCP/IP Communication
2.1.1 Choregraphe Python Blocks

Communication TCP/IP:
This block is responsible for setting up the server and managing communication with the PC server.

"Position Actuel" and "TIMEOUT Switch":
These blocks are added for convenience.

"Position Actuel": This block saves the last position input received to determine which position needs to be detected and when. Its onStart method sends a message to the "Communication TCP/IP" block to initiate detection on the PC server.
"TIMEOUT Switch": A SEND_TIMEOUT method is added to send a "TIMEOUT" signal to the last game diagram used. Without this, all game diagrams would restart simultaneously.

2.1.2 PC_server_v2.py

This algorithm relies on two AI models from Mediapipe that use computer vision to detect the human body and map it with landmarks on joints and articulations. The AI models are trained to recognize hands and the full body.
In this code, a function is created for each position to be recognized. When launched, the server waits for signals from NAO to trigger the corresponding function, starting the detection sequence. It then sends feedback to NAO via TCP.

2.2 Application Timeline
2.2.1 Greeting

In this diagram, NAO greets the user by waving and saying "hello" every 30 seconds until the user waves back.

The sequence:
- NAO executes a waving animation and speech block.
- NAO uses the "Position Actuel" block to instruct the "Communication TCP/IP" block to send "waving" to the server.

The server responds with:
"TIMEOUT": If no wave is detected within 30 seconds, the sequence repeats.
"wave_validated": If the correct movement is detected, NAO confirms with a phrase and moves to the next sequence.

2.2.2 Rules

NAO asks the user if they want to hear the rules.
"Yes": The rules are explained.
"No": The rules are skipped, and the program proceeds to the next step.

2.2.3 Gamemode

The gamemode sequence follows the same principles as other camera-detection-based blocks (e.g., "Greeting").
NAO asks the user to show either one or two fingers and uses the communication blocks to send "count_finger" to the server.
The server responds with:
"one_finger" or "two_finger": Launches the "Static Game" or "Dynamic Game" diagram accordingly.
"wrong_nb_fingers": If the user shows an incorrect number of fingers, NAO asks the user to retry.

2.2.4 Games

Static Game (1 finger) :
This block introduces speech recognition, allowing the user to choose between "position_O" and "position_K."
The chosen position is saved to replay the correct one if a "TIMEOUT" signal is received.

The server responds with:
"TIMEOUT": Restarts the program.
"position_validated": Proceeds to the endgame sequence.


Dynamic Game (two fingers) :
This block operates similarly to the "Greeting" block:
NAO performs the bending position while saying, "It's Showtime."
NAO sends "bowing" to the server to initiate detection.

The server responds with:
"TIMEOUT": Restarts the program.
"bowing_validated": Proceeds to the endgame sequence.
If the server sends "arm_near_body," NAO provides appropriate feedback to alert the user.

2.2.5 Endgame

This diagram is similar to the "Rules" explanation but asks the player if they want to play another round:
"Yes": The "Gamemode" diagram is executed again.
"No": The program ends.

3. Tips for position to be detected

3.1 Waving : 
To ensure the waving function works correctly : 
Wave with one hand (either hand works). After several back-and-forth motions, the movement should be detected.
If it is not detected, try adjusting the angle of your arm while waving (this issue should rarely occur).

3.2 Counting Fingers : 
To ensure the finger-counting function works correctly : 
Hold your palm facing the camera and raise your fingers. Stay close to the camera for accurate detection.
To validate the number of fingers, hold the chosen count steadily for a few seconds. Both hands can be used for this feature.

3.3 Position "K" :
To perform the "K" position:
Raise your left arm straight up to a 180-degree angle.
Keep your left leg straight along your body.
Position your right leg at an outward angle to the right.
Extend your right arm outward to the right as well.
Hold the pose for a few seconds to validate the position.

3.4 Position "O" :
To perform the "O" position:
Join your arms above your head to form an "O" shape.
Hold the pose for a few seconds to validate the position.
If detection fails, it may be due to the angle of your arms relative to the camera.
Adjust the angle by slightly raising or lowering your arms over your head, ensuring your hands are close together, for better detection.

3.5 Bowing (inclination) : 
To perform the bowing movement, follow these steps:

Initial Position:
Place your arms along the sides of your body throughout the entire movement.
First Step: Bring your arms to rest along your body (transition from state none to init_timer). Hold this position for 1.5 seconds (transition from state init_timer to init).

Bowing Movement:
Lower your upper body to perform the bowing motion.
At the lowest point, hold the position for at least 1 second (transition from state init to descending_complete).

Returning to Standing Position:
Straighten your body back to the initial position. The movement is considered complete.

Important Note:
If the state is init or descending_complete and the arms are no longer aligned with the sides of the body, a message arm_near_body will be triggered.

4. Troubleshooting

4.1 Python Environment and Library Compatibility
Ensure that the following libraries are installed in your Python environment: tensorflow, opencv, numpy, mediapipe.
Note: The Mediapipe library may require a specific Python version to work correctly. For us, it worked with Python versions 3.12.7 and 3.10.15, but not with version 3.12.4.

4.2 Speech Recognition Issues and Overrides
Speech recognition can occasionally be unreliable, but it can be overridden:

4.2.1 Rules Explanation:
If NAO does not correctly recognize your response, you can override it by clicking the blue output box corresponding to the desired answer and typing any text into the box.

4.2.2 Static Game Diagram:
To select the position you want, NAO must recognize the letters "K" or "O". If recognition fails, override it as follows:
Click the blue output box for the input.
Manually type either "K" or "O" (in uppercase), depending on the position you want to attempt.

4.2.3 Endgame Block:
Similarly, for the final speech recognition in the "Endgame" block, override the response by typing "yes" or "no" in the corresponding textbox.

4.3 Speech Recognition Blocks Not Working
On Kylian's computer, the speech recognition blocks failed to function, which prevented the program from running.

We were unable to identify a solution for this issue. If you encounter the same problem, the best workaround is to:

Remove the speech recognition blocks from the program.
Manually connect the blocks according to the desired sequence or gameplay mode.
