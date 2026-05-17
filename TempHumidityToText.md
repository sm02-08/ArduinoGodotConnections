# Temperature & Humidity Sensors to Text
Functionality: Every 2 seconds, use a DHT11 to get a reading of the temperature outside. Then, get text output in Godot (can be changed to other output too)!

*Required:* 
Hardware: Breadboard, soldered arduino nano onto it, 3 male-to-female jumper wires, USB-C to USB-C (or USB-C to whatever connects to your laptop), DHT11 module (not just sensor). 
Software: Godot (install with the .NET and make sure to have .NET installed like in and of itself), Arduino IDE

https://www.youtube.com/watch?v=nOKno82_gd0 for some more help 

## Godot Script

`New Scene` -> `Node2D` -> `Add Node` -> `RichTextLabel` -> Add script to Node2D -> Script should have template, open with C#

Godot script full code: 
```
using Godot;
using System;
using System.IO.Ports;

public partial class HumidityDht11 : Node2D
{
	SerialPort serialPort; 
	RichTextLabel text; 

	public override void _Ready()
	{
		text = GetNode<RichTextLabel>("RichTextLabel"); 
		
		serialPort = new SerialPort(); 
		serialPort.PortName = "COM4"; // make sure this matches the port in arduino ide
		serialPort.BaudRate = 9600; 
		
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

		if (serialPort.BytesToRead > 0)
		{
			string serialMessage = serialPort.ReadExisting(); 

			if (serialMessage.Contains("ALERT_ABOVE_80"))
			{
				text.Text = "Temperature above 80F has been hit.";
			}
			else if (serialMessage.Contains("TEMP_NORMAL"))
			{
				text.Text = "Temperature is stable.";
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
#include "DHT.h"

#define DHTPIN 5        // data pin connects to d5
#define DHTTYPE DHT11   // the sensor type if dht11 (ALSO MAKE SURE U DOWNLOAD THE DHT LIBRARY BEFORE RUNNING)

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(9600);
  dht.begin(); // Initialize the DHT11 sensor
}

void loop() {
  // wait 2 seconds between measurements
  delay(2000);

  // true = Fahrenheit
  float fahrenheit = dht.readTemperature(true);

  // if the sensor fails 
  if (isnan(fahrenheit)) {
    return;
  }

  // if temperature crosses threshold, tell godot
  if (fahrenheit > 10.0) {
    Serial.println("ALERT_ABOVE_80");
  } else {
    Serial.println("TEMP_NORMAL");
  }
}
```

Have the breadboard & arduino setup connected via USB-C to your computer. How the breadboard should look like will be uploaded in pictures below.
On Arduino IDE, press the "-->" button and then the check button. 
In Godot, press the ▶️ play button.
Press the button on your DHT11 and see text output on Godot.

Now for how the breadboard should look like: go to the "Images" folder
