/*
 * line_robot.h
 *
 *  Created on: 05-Apr-2024
 *      Author: Rajpatel_AbidemiAgboola
 */

#include <msp430.h>

#define MOTOR1_PIN1 BIT2 // Motor 1 control pin 1 (P2.2)

#define MOTOR1_PIN2 BIT4 // Motor 1 control pin 2 (P2.4)

#define MOTOR2_PIN1 BIT3 // Motor 2 control pin 1 (P2.3)

#define MOTOR2_PIN2 BIT5 // Motor 2 control pin 2 (P2.5)

#define IR_SENSOR1 BIT3 // IR Sensor 1 input pin (P1.3)

#define IR_SENSOR2 BIT4 // IR Sensor 2 input pin (P1.4)

//void initializePWM()    // set up PWM for motor control

//void initializeIRSensor()//set up IR sensor pin as input

int isBlack(int sensorValue, int threshold) {
    return (sensorValue < threshold); // Sensor readings below threshold indicate black

intblackthreshold(calculate the threshold value according to the surroundings);

