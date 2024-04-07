#include <msp430.h> // Include the MSP430 header file

// Define pin mappings for motors and IR sensors
#define MOTOR1_PIN1 BIT2 // Motor 1 control pin 1 (P2.2)
#define MOTOR1_PIN2 BIT4 // Motor 1 control pin 2 (P2.4)

#define MOTOR2_PIN1 BIT3 // Motor 2 control pin 1 (P2.3)
#define MOTOR2_PIN2 BIT5 // Motor 2 control pin 2 (P2.5)

#define IR_SENSOR1 BIT3 // IR Sensor 1 input pin (P1.4)
#define IR_SENSOR2 BIT4 // IR Sensor 2 input pin (P1.5)

int miliseconds;
int distance;
long sensor;



// Function to initialize PWM for motor control
void initializePWM() {
    // Set up PWM for motor control
    P2DIR |= MOTOR1_PIN1 | MOTOR1_PIN2 | MOTOR2_PIN1 | MOTOR2_PIN2; // Set pins as outputs
    P2SEL |= MOTOR1_PIN1 | MOTOR1_PIN2 | MOTOR2_PIN1 | MOTOR2_PIN2; // Select peripheral function (PWM)

    TA1CTL = TASSEL_2 | MC_1 | TACLR; // SMCLK, Up mode, Clear Timer
    TA1CCR0 = 700; // PWM Period
    TA1CCR1 = 500;  // PWM Duty Cycle (50%)
    TA1CCR2 = 500;  // PWM Duty Cycle (50%)

    TA1CCTL1 = OUTMOD_7; // PWM output mode: Reset/Set
    TA1CCTL2 = OUTMOD_7; // PWM output mode: Reset/Set
}

// Function to initialize IR sensors
void initializeIRSensor() {
    // Set up IR sensor pins as inputs
    P1DIR &= ~(IR_SENSOR1 | IR_SENSOR2); // Set pins as inputs
    P1REN |= IR_SENSOR1 | IR_SENSOR2;    // Enable pull-up/down resistors
    P1OUT |= IR_SENSOR1 | IR_SENSOR2;    // Set pull-up resistors
}


int main(void) {

    BCSCTL1 = CALBC1_1MHZ;
     DCOCTL = CALDCO_1MHZ;                     // submainclock 1mhz
     WDTCTL = WDTPW + WDTHOLD;                 // Stop WDT

     CCTL0 = CCIE;                             // CCR0 interrupt enabled
     CCR0 = 1000;                  // 1ms at 1mhz
     TACTL = TASSEL_2 + MC_1;                  // SMCLK, upmode

     P1IFG  = 0x00;                //clear all interrupt flags
     P1DIR |= 0x01;                            // P1.0 as output for LED
     P1OUT &= ~0x01;                           // turn LED off

     _BIS_SR(GIE);                         // global interrupt enable


    initializePWM(); // Initialize PWM for motor control
    initializeIRSensor(); // Initialize IR sensors

    while(1) {
        P1IE &= ~0x01;          // disable interupt
            P1DIR |= BIT7;          // trigger pin as output
            P1OUT |= BIT7;          // generate pulse
            __delay_cycles(10);             // for 10us
            P1OUT &= ~BIT7;                 // stop pulse
            P1DIR &= ~BIT6;         // make pin P1.2 input (ECHO)
                P1IFG = 0x00;                   // clear flag just in case anything happened before
            P1IE |= BIT6;           // enable interupt on ECHO pin
            P1IES &= ~BIT6;         // rising edge on ECHO pin
                __delay_cycles(30000);          // delay for 30ms (after this time echo times out if there is no object detected)
                distance = sensor/58;           // converting ECHO lenght into cm
                if(distance < 20 && distance != 0)
                {
                    P1OUT |= 0x01;  //turning LED on if distance is less than 20cm and if distance isn't 0.
                    TA1CCR1 = 0;
                    TA1CCR2 = 0;

                }
                else{


        // Read IR sensor inputs
        int sensor1 = (P1IN & IR_SENSOR1) ? 1 : 0;
        int sensor2 = (P1IN & IR_SENSOR2) ? 1 : 0;

        // Control motors based on sensor inputs
        if(sensor1 && sensor2) {
            // Both sensors detect black line, continue moving forward
            TA1CCR1 = 300;
            TA1CCR2 = 300;
        } else if(!sensor1 && sensor2) {
            // Sensor 2 detects black line, turn right
            TA1CCR1 = 300; // Motor 1 continues moving forward
            TA1CCR2 = 10 ;   // Motor 2 turns right
        } else if(sensor1 && !sensor2) {
            // Sensor 1 detects black line, turn left
            TA1CCR1 = 0;   // Motor 1 turns left
            TA1CCR2 = 400; // Motor 2 continues moving forward
        } else {
            // Neither sensor detects a black line, stop motor
            TA1CCR1 = 200;
            TA1CCR2 = 0;
        }
    }
    }
}
#pragma vector=PORT1_VECTOR
__interrupt void Port_1(void)
{
    if(P1IFG&BIT6)  //is there interrupt pending?
        {
          if(!(P1IES&BIT6)) // is this the rising edge?
          {
            TACTL|=TACLR;   // clears timer A
            miliseconds = 0;
            P1IES |= BIT6;  //falling edge
          }
          else
          {
            sensor = (long)miliseconds*1000 + (long)TAR;    //calculating ECHO lenght

          }
    P1IFG &= ~BIT6;             //clear flag
    }
}

#pragma vector=TIMER0_A0_VECTOR
__interrupt void Timer_A (void)
{
  miliseconds++;
}

