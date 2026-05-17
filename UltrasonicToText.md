# Ultrasonic Sensors (Distance) to Text
Functionality: Every 150 milliseconds, get a reading of how far the nearest object is from the front of the sensor. It tells you if something's far or close (the two categories) and also the numerical value (e.g. "the object is 154 cm away").

*Required:* 
Hardware: Breadboard, soldered arduino nano onto it, 4 male-to-female jumper wires, USB-C to USB-C (or USB-C to whatever connects to your laptop), HC-SRO4 ultrasonic distance sensor. 
Software: Godot (install with the .NET and make sure to have .NET installed like in and of itself), Arduino IDE

https://www.youtube.com/watch?v=nOKno82_gd0 for some more help 

## Godot Script

`New Scene` -> `Node2D` -> `Add Node` -> `RichTextLabel` -> Add script to Node2D -> Script should have template, open with C#

Godot script full code: 
```
using Godot;
using System;
using System.IO.Ports;

public partial class UltrasonicsensorHcsro4 : Node2D
{
	SerialPort serialPort; 
	RichTextLabel text; 

	public override void _Ready()
	{
		text = GetNode<RichTextLabel>("RichTextLabel"); 
		
		serialPort = new SerialPort(); 
		serialPort.PortName = "COM5"; // this needs to be changed to be adjusted to the actual port each time in arduino ide
		serialPort.BaudRate = 9600; 
		serialPort.ReadTimeout = 50; // if data is incomplete the game doesnt just freeze completely
		
		try
		{
			serialPort.Open();
		}
		catch (Exception e)
		{
			GD.PrintErr($"Could not open serial port: {e.Message}");
		}
	}

	public override void _Process(double delta)
	{
		if (serialPort == null || !serialPort.IsOpen) return;

		while (serialPort.BytesToRead > 0)
		{
			try
			{
				// strip spaces and 
				string rawMessage = serialPort.ReadLine().Trim();
				
				// split msg at comma: index 0 is status, index 1 is distance
				string[] dataTokens = rawMessage.Split(',');

				if (dataTokens.Length == 2)
				{
					string pathStatus = dataTokens[0];
					string distanceValue = dataTokens[1];

					// format and output text directly as a RichTextLabel
					if (pathStatus == "CLEAR")
					{
						text.Text = $"Path is clear. Nearest object is {distanceValue} cm away.";
					}
					else if (pathStatus == "BLOCKED")
					{
						text.Text = $"Path is NOT clear! Nearest object is {distanceValue} cm away.";
					}
				}
			}
			catch (TimeoutException)
			{
				// if ReadLine reaches the limit before finishing run this
			}
			catch (Exception e)
			{
				GD.PrintErr($"Error parsing serial data: {e.Message}");
			}
		}
	}

	public override void _ExitTree()
	{
		if (serialPort != null && serialPort.IsOpen)
		{
			serialPort.Close();
		}
	}
}
```

## Arduino IDE Script 
On Arduino IDE, make sure to connect the Arduino to its correct thing. 

Full script:
```
const int trigPin = 5;      //  the first pin is connected to D5
const int echoPin = 6;      // the 2nd pin is connected to D6
const float threshold = 20.00; // the distance threshold to see if the path is near or far (near is <20 cm)

void setup() {
  Serial.begin(9600);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
}

void loop() {
  digitalWrite(trigPin, LOW); // clear trigger pin
  delayMicroseconds(2);
  
  // 10 microsecond pulse 
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  // bounce back duration
  long duration = pulseIn(echoPin, HIGH);
  
  // calculate distance in centimeters
  float distanceCm = duration * 0.0343 / 2.0;

  
  if (distanceCm > 0 && distanceCm < 400) { // any 0 or out of bounds readings are filtered out
    // step 1 is to set the distance threshold for blocked/clears
    if (distanceCm < threshold) {
      Serial.print("BLOCKED,");
    } else {
      Serial.print("CLEAR,");
    }
    
    // make sure to round to the hundredths place so that it's not really 64.000000000000000000001
    // print new line ln r/n/ at the end of msg
    Serial.println(distanceCm, 2); 
  }
  
  // 150ms delay between readings
  delay(150); 
}
```

Have the breadboard & arduino setup connected via USB-C to your computer. How the breadboard should look like will be uploaded in pictures below.
On Arduino IDE, press the "-->" button and then the check button. 
In Godot, press the ▶️ play button.
Press the button on your DHT11 and see text output on Godot.

Now for how the breadboard should look like: go to the "Images" folder


How the wires are set up: 
- `VCC` wire goes to the `5V` hole.
- `Trig` wire goes to any D2 to D10 hole.
- `Echo` wire goes to the hole + 1 that Trig is in (e.g. Trig is in D5, Echo is in D6).
- `GND` wire of course goes into the GND hole.
