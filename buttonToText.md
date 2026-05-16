# Push a Button
Functionality: Push a button (KY 004), get text output in Godot (can be changed to other output too)!

*Required:* 
Hardware: Breadboard, soldered arduino nano onto it, 3 male-to-female jumper wires, USB-C to USB-C (or USB-C to whatever connects to your laptop). 
Software: Godot (install with the .NET and make sure to have .NET installed like in and of itself), Arduino IDE

https://www.youtube.com/watch?v=nOKno82_gd0 for some more help 

## Godot Script

`New Scene` -> `Node2D` -> `Add Node` -> `RichTextLabel` -> Add script to Node2D -> Script should have template, open with C#

Godot script full code: 
```
using Godot;
using System;
using System.IO.Ports;

public partial class Arduino1 : Node2D
{
	SerialPort serialPort; 
	RichTextLabel text; 

	// Called when the node enters the scene tree for the first time.
	public override void _Ready()
	{
		// execution 
		text = GetNode<RichTextLabel>("RichTextLabel");
		serialPort = new SerialPort(); 
		serialPort.PortName = "COM5"; // edit this to be whatever the arduino name is that shows up on your arduino IDE
		serialPort.BaudRate = 9600; 
		serialPort.Open(); // opening access to all com3 in he process 
	}

	// Called every frame. 'delta' is the elapsed time since the previous frame.
	public override void _Process(double delta)
	{
		if (serialPort.BytesToRead > 0)
		{
			string serialMessage = serialPort.ReadExisting(); 

			if (serialMessage.Contains("HelloGodot"))
			{
				text.Text = "Hello Arduino, I hear you :)";
			}
		}
	}
}
```

## Arduino IDE Script 
On Arduino IDE, make sure to connect the Arduino to its correct thing. 

Full script:
```
const int buttonPin = 4; 

int buttonState = 0; 

void setup() {
  Serial.begin(9600); 
  // INPUT_PULLUP turns on the internal resistor and sets the default state to HIGH
  pinMode(buttonPin, INPUT_PULLUP); 
}

void loop() {
  buttonState = digitalRead(buttonPin); 

  // Because of the pull-up, pressing the button pulls the signal to LOW
  if (buttonState == LOW) { 
    Serial.println("HelloGodot"); 
    delay(200); // A small delay keeps it from sending 1,000 messages per second while held
  }
}
```

Have the breadboard & arduino setup connected via USB-C to your computer. How the breadboard should look like will be uploaded in pictures below.
On Arduino IDE, press the "-->" button and then the check button. 
In Godot, press the ▶️ play button.
Press the button on your KY 004 and see text output on Godot.

Now for how the breadboard should look like: go to the "Images" folder
