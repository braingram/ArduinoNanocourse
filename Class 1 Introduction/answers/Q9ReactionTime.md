As with any project it is a good idea to try and break up problems into smaller, testable, verifiable sub-components. One way to break up this question is into the following:

1) LED flash to signal the button should be pressed
1) Trial timing
1) Button press to signal response
1) 'Fast' response indicator


Rather than trying to code the entire answer from the start, let's make and verify the sub-components above.

LED flash
-----
The goal is to turn on an led to indicate to the subject that the button should be pressed. This should feel familiar (since you've completed questions 1 and 2) and we can copy what you wrote for the earlier questions. For this trial start signal, let's use an external LED. Let's put together a simple sketch that blinks an external led and wire up an LED (with current limiting resistor) to pin 12 (see the circuit part of the answer key for a diagram of the circuit).

```c++
// create variables for the led pin number
int external_led_pin = 12;

void setup() {
  // set the led pin as an output
  pinMode(external_led_pin, OUTPUT);
  
  // set the initial value/state of the pin
  digitalWrite(external_led_pin, LOW);
}

void loop() {
  digitalWrite(external_led_pin, HIGH);
  delay(1000);
  digitalWrite(external_led_pin, LOW);
  delay(1000);
}
```

See 01BlinkLED for the complete code.

Load this onto your Arduino and confirm that the external LED blinks on and off as expected.

Trial timing
-----

The question states that trials should occur every 30 seconds. Additionally, we know the final sketch will need to report the time between the time when the LED turned on and when the button was pressed (which we will add later). Let's create a copy of our led flash sketch and modify it to:

1) turn on an LED
1) wait for a fixed period of time (instead of waiting for a button press)
1) report the amount of time we waited (our 'reaction time')
1) wait the remainder of the trial duration

Add a variable to define the duration of a trial

```c++
// create a variable for the 30 second (30000 millisecond) trial duration
unsigned long trial_duration = 30000;
```

Next let's modify the loop function for our new trial structure. Remember that loop is called repeatedly forever (when it finishes, it automatically restarts). For this sketch we will use this feature to repeatedly run trials (1 per call to loop). It is sometimes helpful to first add comments before writing your code to help organize your thoughts.

```c++
void loop() {
  // save time of the start of the 'trial'
  // turn on the external led (to indicate the button should be pressed)
  // wait for a fixed time (instead of waiting for a button press)
  // save the current time (when the button was pressed)
  // turn off the external led
  // compute the reaction time
  // finally, wait the remainder of the trial
}
```

Now that we have written down our logic we can add code to match the comments.

```c++
void loop() {
  // save time of the start of the 'trial'
  unsigned long trial_start_time = millis();
  
  // turn on the external led (to indicate the button should be pressed)
  digitalWrite(external_led_pin, HIGH);
  
  // wait for a fixed time (instead of waiting for a button press)
  delay(1000);

  // save the current time (when the button was pressed)
  unsigned long button_press_time = millis();

  // turn off the external led
  digitalWrite(external_led_pin, LOW);

  // compute the reaction time
  unsigned long reaction_time = button_press_time - trial_start_time;

  // finally, wait the remainder of the trial
  
  // compute the current trial time (the difference between the current and start times)
  unsigned long trial_time_so_far = millis() - trial_start_time;
  
  // compute the time remaining for this trial
  long time_remaining = trial_duration - trial_time_so_far;
  
  // a slow response can cause this trial to take more than the allowed trial duration
  // only delay if there is some trial time remaining  
  if (time_remaining > 0) {
    delay(time_remaining);
  };
}
```

See 02TrialTiming for the complete code.

Load this onto your Arduino and confirm that the LED blinks for 1 second every 30 seconds.

Button Press
-----

Now that we have our basic trial structure, let's add a button to our circuit connected to pin 2 (see the circuit diagram for how to wire up the button). Note that the button has a 10K pull-up resistor which will set it's default "un-pressed" state to HIGH.

You may be tempted to start by adding the button to your work-in-progress reaction time sketch. However, it is good practice (and can make testing and debugging easier) to write a simple sketch to verify that your button circuit is working. In this case, let's have the sketch print out 'Button pressed' every 500 ms when the button is pressed.

```c++
// create a variable for the button pin number
int button_pin = 2;

void setup() {
  // set button pin as input
  pinMode(button_pin, INPUT);
  
  // start the serial connection to the computer
  Serial.begin(9600);
}

void loop() {
  // read button state until it is pressed (returns LOW)
  while (digitalRead(button_pin) != LOW) {
    // this while loop checks the button state and repeats until
    // the call to digitialRead returns LOW
  }
  
  // report the button was pressed
  Serial.println("Button pressed");
  
  // wait
  delay(500);
};
```

See 03Button for the complete code.

Load this onto your Arduino and confirm that your button press circuit and code is working.

Now that we've verified that our button circuit is working, let's update our reaction time sketch. You will need to:

1) Add the button pin variable towards the top of the sketch
1) Add the button setup and serial.begin lines to your setup function
1) Replace the "wait for a fixed time" delay in your loop function with the button press checking code
1) Add some Serial.print lines (see below) to report the reaction time

```c++
  // report the reaction time to the serial port
  Serial.print("Reaction time: ");
  Serial.print(reaction_time);
  Serial.println(" ms");
```

See 04ButtonTiming for the complete code.

Load this onto your Arduino and confirm that your sketch reports the time between the LED flash and your button press.

'Fast' response indicator
-----
To add the final piece of the sketch we need to decide on what to use as our 'Fast' response indicator. Feel free to be creative but for simplicity we're going to use the interal LED in this answer. As with the button, don't assume that your new circuit works and instead start by writing (or reusing) a test sketch to verify our internal LED is working as expected. For this answer, reuse the sketch we wrote to test the external led and modify it to use pin 13 (the internal led).

We also have to decide how to use this LED to indicate a fast response. For this answer we want to blink the internal led 3 times whenever a response is considered 'fast' (where the reaction time is less than 1000 milliseconds). Modify our reaction time sketch:

1) Adding a variable for the internal led pin
```c++
int internal_led_pin = 13;
```
2) Add lines to your setup function to set the pin mode and initial state of the led pin
```c++
  pinMode(internal_led_pin, OUTPUT);
  digitalWrite(internal_led_pin, LOW);
```
3) Finally add code to blink the internal led 3 times only for fast responses after you print the reaction time
```c++
  // if response was 'fast' (less than 1000 milliseconds)
  // blink the internal led three times
  if (reaction_time < 1000) {
    for (int i=0; i<3; i++) {
      digitalWrite(internal_led_pin, HIGH);
      delay(250);
      digitalWrite(internal_led_pin, LOW);
      delay(250);
    };
  };
```

See the ReactionTime sketch for the final code.
