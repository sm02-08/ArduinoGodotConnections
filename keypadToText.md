# 4x4 Keypad to Text
Functionality: Pressing and releasing a key on the 4x4 Matrix Membrane Keypad outputs a text ("\[Key] pressed" or "\[Key] released") in Godot.

*Required:* 
- Hardware: Breadboard, soldered arduino nano onto it, 8 male-to-male jumper wires, USB-C to USB-C (or USB-C to whatever connects to your laptop), 4x4 Matrix Membrane Keypad. 
- Software: Godot (install with the .NET and make sure to have .NET installed like in and of itself), Arduino IDE

[https://www.youtube.com/watch?v=nOKno82_gd0](https://www.youtube.com/watch?v=ePO3SE1ExXw) for some more help 

## Godot Script

`New Scene` -> `Node2D` -> `Add Node` -> `RichTextLabel` -> Add script to Node2D -> Script should have template, open with C#

Godot script full code: 
```
using Godot;
using System;
using System.IO.Ports;

public partial class MatrixkeypadNumbers : Node2D
{
	SerialPort serialPort; 
	RichTextLabel text; 

	public override void _Ready()
	{
		// this matches the RichTextLabel node in screen but if u change it (e.g. to like "text") then change the name here as well
		text = GetNode<RichTextLabel>("RichTextLabel"); 
		
		serialPort = new SerialPort(); 
		serialPort.PortName = "COM4"; // adjusted to actual device port on arduino ide
		serialPort.BaudRate = 9600; 
		serialPort.ReadTimeout = 50; 
		
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
				// read the lines from arduino (e.g. the "1 rpessed" stuff) and output it
				string rawMessage = serialPort.ReadLine().Trim();
				
				// msg splits at underscore
				string[] parts = rawMessage.Split('_');

				if (parts.Length == 2)
				{
					string keyLabel = parts[0];   // e.g., "1"
					string keyAction = parts[1];  // e.g., "PRESSED" or "RELEASED"

					// format "PRESSED" -> "pressed" or "RELEASED" -> "released"
					string formattedAction = keyAction.ToLower();

					// combine text in output
					text.Text = $"{keyLabel} {formattedAction}";
				}
			}
			catch (TimeoutException)
			{
				// wait for serial ticks
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
#include <Keypad.h>

const byte ROWS = 4; 
const byte COLS = 4; 

// this is a mapping of the keys. if you have a 3x3 keypad instead of a 4x4, adjust this and the rows/columns variable accordingly (shift from 4 to 3).
char hexaKeys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

byte rowPins[ROWS] = {9, 8, 7, 6};   // Row 0, Row 1, Row 2, Row 3
byte colPins[COLS] = {5, 4, 3, 2}; // Col 0, Col 1, Col 2, Col 3

// initialize class "Keypad"
Keypad customKeypad = Keypad(makeKeymap(hexaKeys), rowPins, colPins, ROWS, COLS); 

void setup() {
  Serial.begin(9600);
  
  // event listener catches the press and releases
  customKeypad.addEventListener(keypadEvent); 
}
  
void loop() {
  // the background matrix scanning has to run constantly so it runs in this loop
  customKeypad.getKey();
}

// triggers automatically when something updates (e.g. key pressed)
void keypadEvent(KeypadEvent key) {
  switch (customKeypad.getState()) {
    case PRESSED:
      Serial.print(key);
      Serial.println("_PRESSED");
      break;
      
    case RELEASED:
      Serial.print(key);
      Serial.println("_RELEASED");
      break;
  }
}
```

Have the breadboard & arduino setup connected via USB-C to your computer. How the breadboard should look like will be uploaded in pictures below.
On Arduino IDE, press the "-->" button and then the check button. 
In Godot, press the ▶️ play button.
Press buttons on your keypad and see the Godot output. 

Now for how the breadboard should look like (a visual representation): go to the "Images" folder


How the wires are set up: (also can see a visual representation in the "Images" folder)

From left to right on the keypad...
- Insert the wires from D9 to D2. (For example, the leftmost wire should go into the D9 hole, the second leftmost into D8, etc etc until the rightmost wire goes to D2).
