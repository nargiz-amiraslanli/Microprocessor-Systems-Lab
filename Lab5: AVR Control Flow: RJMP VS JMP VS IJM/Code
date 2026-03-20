#include <avr/io.h>        // Gives access to AVR registers like PORTB, DDRB, PINB
#include <util/delay.h>    // Allows us to use _delay_ms() for delays


void action0(void); // Function prototype for action0 (normal blink)
void action1(void); // Function prototype for action1 (reverse blink)
void action2(void); // Function prototype for action2 (short blink)
void action3(void); // Function prototype for action3


void run_mode(uint16_t on_time, uint16_t off_time); // Function that runs blinking mode
void modeA_entry(void); // Mode A: slow blinking
void modeB_entry(void); // Mode B: medium blinking
void modeC_entry(void); // Mode C: fast blinking


uint16_t g_on_time  = 500;   // Global variable storing LED ON time
uint16_t g_off_time = 500;   // Global variable storing LED OFF time
volatile uint8_t action_index = 0; // Stores which action is currently active


void delay_ms_var(uint16_t ms) // Function that delays for variable milliseconds
{
    while(ms--) _delay_ms(1);  // decreases by 1 every time the loop runs
}


void action0(void) // Action 0: LED ON then OFF (normal blink)
{
    PORTB |=  (1 << 5);        // Set PB5 HIGH → turn LED ON
    delay_ms_var(g_on_time);   // Wait for ON time

    PORTB &= ~(1 << 5);        // Clear PB5 → turn LED OFF
    delay_ms_var(g_off_time);  // Wait for OFF time

    asm volatile("rjmp mode_loop"); // Jump back to mode_loop to continue program
}

void action1(void) // Action 1: LED OFF then ON (reverse blink)
{
    PORTB &= ~(1 << 5);        // Turn LED OFF
    delay_ms_var(g_off_time);  // Wait OFF time

    PORTB |= (1 << 5);         // Turn LED ON
    delay_ms_var(g_on_time);   // Wait ON time

    asm volatile("rjmp mode_loop"); // Jump back to mode_loop
}

void action2(void) // Action 2: short LED pulse
{
    PORTB |= (1 << 5);         // Turn LED ON
    _delay_ms(30);             // Keep LED ON for 30 ms (very short blink)

    PORTB &= ~(1 << 5);        // Turn LED OFF
    delay_ms_var(g_on_time + g_off_time); // Wait full cycle time

    asm volatile("rjmp mode_loop"); // Jump back to mode_loop
}

void action3(void) // Action 3: reset actions
{
    action_index = 0;          // Reset action index to 0 (go back to action0)
    asm volatile("rjmp mode_loop"); // Jump back to mode_loop
}

//pointer to a function that takes no parameters and returns nothing
typedef void (*action_fn)(void); 
//This creates a table of function addresses.-jump table
action_fn action_table[4] = { action0, action1, action2, action3 }; 


void run_mode(uint16_t on_time, uint16_t off_time) // Function that runs the blinking mode with given ON and OFF times
{
    g_on_time  = on_time;      // Save ON time into global variable
    g_off_time = off_time;     // Save OFF time into global variable
    action_index = 0;          // Start with action 0

    asm volatile("mode_loop:"); // Create assembly label "mode_loop"

    while(1)                    // Infinite loop so program runs forever
    {

        if(!(PINB & (1 << 0)))  // If PB0 = LOW → button pressed
        {
            action_index++;     // Move to next action

            action_index %= 4;  // Keep value between 0–3

            while(!(PINB & (1 << 0))); // Wait until button is released

            _delay_ms(50);      // Debounce delay
        }

        // Jump to selected action
        asm volatile(
            "movw r31, %0\n\t"  // Copy function address into Z register (r31:r30)
            "ijmp\n\t"          // Jump to function stored in Z register
            :
            : "r" (action_table[action_index]) // Get address from action_table
        );
    }
}


void modeA_entry(void) { run_mode(1000,1000); } // slow blink Mode A
void modeB_entry(void) { run_mode(200,200); }   // Mode B: medium blink
void modeC_entry(void) { run_mode(50,50); }     // Mode C: very fast blink


int main(void)
{
    DDRB |=  (1 << 5);   // Set PB5 as OUTPUT (LED pin)
    DDRB &= ~(1 << 0);   // Set PB0 as INPUT (button pin)
    PORTB |= (1 << 0);   // Enable internal pull-up resistor for button

    
    asm volatile(
        "wait_btn:\n\t"         // Create label wait_btn
        "sbic %0,0\n\t"         // Skip next instruction if bit pressed = 0
        "rjmp wait_btn\n\t"     // If button not pressed, keep looping
        :
        : "I" (_SFR_IO_ADDR(PINB)) // So assembly knows which register to test.
    );

    //After first button press:
    PORTB |= (1 << 5); _delay_ms(200);  // Turn LED ON for 200 ms
    PORTB &= ~(1 << 5); _delay_ms(200); // Turn LED OFF for 200 ms

    
    uint8_t presses = 0;   // counts number of presses
    uint16_t timer = 0;    // counts how long the selection window lasts

    while(timer < 4000)    // Run loop for 4000 ms (4 seconds)
    {
        if(!(PINB & (1 << 0))) // If button pressed
        {
            presses++;         // Increase press count
            while(!(PINB & (1 << 0))); // Wait until button released
            _delay_ms(50);     // Debounce delay
        }

        _delay_ms(1);          // Wait 1 ms
        timer++;               // Increase timer
    }

    
    if(presses >= 3)      modeC_entry(); // If pressed 3 or more times → fast mode
    else if(presses == 2) modeB_entry(); // If pressed twice → medium mode
    else                  modeA_entry(); // Otherwise → slow mode

    while(1); // Infinite loop so program never exits
}
