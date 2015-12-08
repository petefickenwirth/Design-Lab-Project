# Design-Lab-Project Maze Code
#include <IRremote.h>
#include <Wire.h>
#include <Adafruit_MotorShield.h>
#include "utility/Adafruit_PWMServoDriver.h"
#include <Servo.h>
//this tells the arduino that it will be using a motor shield and two DC motors run through ports 1 and 2
Adafruit_MotorShield AFMS = Adafruit_MotorShield();
Adafruit_DCMotor *myMotor1 = AFMS.getMotor(1); //left motor
Adafruit_DCMotor *myMotor2 = AFMS.getMotor(2); //right motor
#define leftoutsensor  	A1 //left outside line sensor
#define leftinsensor  	A2 //left inside line sensor
#define centersensor  	A3 //center line sensor
#define rightinsensor  	A4 //right inside line sensor
#define rightoutsensor  A5 //right outside line sensor
#define doublecheck  	200
int leftoutReading;
int leftinReading;
int centerReading;
int rightinReading;
int rightoutReading;

void setup()
{
	AFMS.begin(); //begins running through the motor shield
	myMotor1->run(RELEASE); //begins running the motor in port 1
	myMotor2->run(RELEASE); //begins running the motor in port 2
	Serial.begin(9600); //starts monitoring the values from the analog input
}

void loop()
{

	readSensors(); //reads sensors
    //return;
	if(leftoutReading < 200 && rightoutReading < 200 && centerReading > 200) 
	{
		straight();
	}
	else
	{
		LeftHandRule();
	}
}

void readSensors()
{
	leftoutReading 	= analogRead(leftoutsensor);
	leftinReading	= analogRead(leftinsensor);
	centerReading	= analogRead(centersensor);
	rightinReading	= analogRead(rightinsensor);
	rightoutReading	= analogRead(rightoutsensor);
}

void LeftHandRule()
{
	if(leftoutReading > 200 && rightoutReading > 200) //both sides detect line
	{
		inch();

		if(leftoutReading > 200 || rightoutReading > 200) //either side detects line again
		{
			done();
		}
		if(leftoutReading < 200 && rightoutReading < 200) //neither side detects line
		{
			turnLeft();
		}
	}

	if(leftoutReading > 200)    //left side detects line
	{
		inch();

		if(leftoutReading < 200 && rightoutReading < 200) //
		{
			turnLeft();
		}
		else
		{
			done();
		}
	}

	if(rightoutReading > 200)
	{
		inch();

		if(leftoutReading > 200)
		{
			delay(doublecheck - 30);
			readSensors();

			if(rightoutReading > 200 && leftoutReading > 200)
			{
				done();
			}
			else
			{
				turnLeft();
				return;
			}
		}
		delay(doublecheck - 30);
		readSensors();

		if(leftoutReading < 200 && rightoutReading < 200 && centerReading < 200)
		{
			turnRight();
			return;
		}
		readSensors();
		if(leftoutReading < 200 && leftinReading < 200 && centerReading < 200
		        && rightinReading < 200 && rightoutReading < 200)
		{
			turnAround();
		}
	}
}

void inch ()
{
	myMotor1->run(FORWARD);
	myMotor2->run(FORWARD);
	myMotor1->setSpeed(50);
	myMotor2->setSpeed(50);
	delay(doublecheck);
	readSensors();
}

void done()
{
	myMotor1->run(FORWARD);
	myMotor2->run(FORWARD);
	myMotor1->setSpeed(0);
	myMotor2->setSpeed(0);
}

void turnLeft()
{
	while (analogRead(centersensor) > 200)
	{
		myMotor1->run(BACKWARD);
		myMotor2->run(FORWARD);
		myMotor1->setSpeed(50);
		myMotor2->setSpeed(50);
		delay(2);
		myMotor1->setSpeed(0);
		myMotor2->setSpeed(0);
		delay(1);
	}
	while(analogRead(centersensor) < 200)
	{
		myMotor1->run(BACKWARD);
		myMotor2->run(FORWARD);
		myMotor1->setSpeed(50);
		myMotor2->setSpeed(50);
		delay(2);
		myMotor1->run(BACKWARD);
		myMotor2->run(FORWARD);
		myMotor1->setSpeed(0);
		myMotor2->setSpeed(0);
		delay(1);
	}
}

void turnRight()
{
	while(analogRead(centersensor) > 200)
	{
		myMotor1->run(FORWARD);
		myMotor2->run(BACKWARD);
		myMotor1->setSpeed(50);
		myMotor2->setSpeed(50);
		delay(2);
		myMotor1->run(FORWARD);
		myMotor2->run(BACKWARD);
		myMotor1->setSpeed(0);
		myMotor2->setSpeed(0);
		delay(2);
	}

	while(analogRead(centersensor) < 200)
	{
		myMotor1->run(FORWARD);
		myMotor2->run(BACKWARD);
		myMotor1->setSpeed(50);
		myMotor2->setSpeed(50);
		delay(2);
		myMotor1->run(FORWARD);
		myMotor2->run(BACKWARD);
		myMotor1->setSpeed(0);
		myMotor2->setSpeed(0);
		delay(2);
	}
}

void straight()
{
	if (analogRead(leftinsensor) < 200)
	{
		myMotor1->run(FORWARD);
		myMotor2->run(FORWARD);
		myMotor1->setSpeed(50);
		myMotor2->setSpeed(50);
		delay(1);
		myMotor1->run(FORWARD);
		myMotor2->run(FORWARD);
		myMotor1->setSpeed(50);
		myMotor2->setSpeed(0);
		delay(5);
	}

	if (analogRead(rightinsensor) < 200)
	{
		myMotor1->run(FORWARD);
		myMotor2->run(FORWARD);
		myMotor1->setSpeed(50);
		myMotor2->setSpeed(50);
		delay(1);
		myMotor1->run(FORWARD);
		myMotor2->run(FORWARD);
		myMotor1->setSpeed(0);
		myMotor2->setSpeed(50);
		delay(5);
	}

	myMotor1->run(FORWARD);
	myMotor2->run(FORWARD);
	myMotor1->setSpeed(50);
	myMotor2->setSpeed(50);
	delay(4);
	myMotor1->run(FORWARD);
	myMotor2->run(FORWARD);
	myMotor1->setSpeed(0);
	myMotor2->setSpeed(0);
	delay(1);
}

void turnAround()
{
	myMotor1->run(FORWARD);
	myMotor2->run(FORWARD);
	myMotor1->setSpeed(50);
	myMotor2->setSpeed(50);
	delay(150);

	while(analogRead(centersensor) < 200)
	{
		myMotor1->run(FORWARD);
		myMotor2->run(FORWARD);
		myMotor1->setSpeed(50);
		myMotor2->setSpeed(50);
		delay(2);
		myMotor1->run(FORWARD);
		myMotor2->run(FORWARD);
		myMotor1->setSpeed(0);
		myMotor2->setSpeed(0);
		delay(1);
	}
}
/*
Peter Fickenwirth's Code
22.201 Robot Project
Fall Semester 2014=5
*/
