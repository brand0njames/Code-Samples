// Final Project Version3: Musical Keypad Lock
#include <hidef.h>      /* common defines and macros */
#include <mc9s12dg256.h>     /* derivative information */
#pragma LINK_INFO DERIVATIVE "mc9s12dg256b"
#include "main_asm.h" /* interface to the assembly module */
#include "final.h" /*set of functions defined in this program*/

//Define note & pitch
#define   c     2867
#define   d     2554
#define   e     2276
#define   f     2148
#define   g     1914
#define   a     1705
#define   b     1519
#define   C     1434
#define   D     1277
#define   E     1138
#define   F     1074
#define   G     957
#define   A     853
#define   B     760
#define   CC    717
#define   DD    639

//Define true and false keywords
typedef int bool;
#define true 1
#define false 0

char keyInput[8];                   //char array for keypad input
char unlockInput[8];                //char array for unlock input
char masks[8] = {                   //LED bitmasks for remaining entries
   0x7F, 0x3F, 0x1F, 0x0F, 0x07, 0x03, 0x01, 0x00
};

int pressInput[8];                  //array for durations of input key presses
int pitch_val[16] = {               //array for 16 pitch values
   d, A, B, CC, D, E, F, g, a, b, DD, G, C, f, c, e
};

int i,                              //for loop counter
    pitch,                          //pitch value of key press
    ticks,                          //counter for time of button press
    lockAttempt=3;                  //number of unlock attempts
   
bool setFlag=false,                 //stops array input once full
     fFlag=false,                   //goes high after complete entry failure
     sFlag=false,                   //goes high after code entry is successful
     wFlag;                         //waiting flag, goes false to exit loop
    
// Real Time Interrupt Service Routine
void interrupt 7 handler(){
  ticks++;        //counts time of key press
  clear_RTI_flag();
}

//Timer channel 5 interrupt service routine
void interrupt 13 handler1(){
  tone(pitch);    //set pitch value of sound   
}

void main(void) { 
  PLL_init();                       //initialize system clock frequency to 24MHz
  keypad_enable();                  //enable keypad
  lcd_init();                       //enable LCD
  SW_enable();                      //enable switches 1-5
  led_enable();                     //enable LEDs
  seg7_disable();                   //disable 7-segment display  
  newCodePrompt();                  //display start message on LCD
   
  while(1){
    
    //Switch 2 sets the passcode  
    if(SW2_down() && (!fFlag&&!sFlag)){ //sw2 pressed & not failed or succeeded
      setting_Message();                //  display set message
      keypad_Set();                     //  run function to set the passcode
    }
    
    //Switch 5 unlocks the device       
    if(SW5_down() && (!fFlag&&!sFlag)
       && setFlag==true){               //sw5 pressed & not failed or succeeded
      code_Prompt2();                   //  display code prompt message
      keypad_Unlock();                  //  run function to unlock the device
    }
  }
  
}//main

//-------------------------------------------
// keypad_Set
// 
// Keypad input is accepted and stored as the
// passcode
//-------------------------------------------
void keypad_Set(void){
  while(setFlag == false){               //as long as code is not set
    leds_on(0xFF);                       //  turn on all LED lights
    for(i=0; i<8; i++){                  
      ticks = 0;                         //key press counter reset  
      keyInput[i] = getkey();            //key presses stored in array
      playNote(i,1); 			 //plays appropriate key note
      RTI_init();                        //initialize real-time interrupts
      wait_keyup();                      //  to save duration of key press
      RTI_disable();                     //disable real-time interrupts
      sound_off();                       //stop playing the note
      pressInput[i] = ticks;             //store key press duration in array
      PORTB &= masks[i];                 //displays remaining entries on LED
    }
    confirm();                           //asks user to save or enter new code        
  }
  code_Prompt1();                        //prompt to unlock
}

//-------------------------------------------
// keypad_Unlock
// 
// Keypad input is accepted and compared to
// the stored passcode, displaying failures
// and success
//-------------------------------------------
void keypad_Unlock(){
  if(lockAttempt > 0){ 
    for(i=0; i<8; i++){
      ticks = 0;                          //key press counter reset
      unlockInput[i] = getkey();          //key presses stored in array
      playNote(i,2);                      //plays appropriate key note
      RTI_init();                         //initialize real-time interrupts
      wait_keyup();                       //  to save duration of key press
      RTI_disable();                      //disable real-time interrupts
      sound_off();                        //stop playing the note
   
      if((unlockInput[i] != keyInput[i])  //if unlock input does not match key press
          ||(pressInput[i]<(ticks-20)     //  or not close enough to press duration
          ||pressInput[i]>(ticks+20))){
                                                 
        i = 8;                            //exits current for loop
        lockAttempt -= 1;                 //decrements number of remaining attempts
        
        switch(lockAttempt){
        
          case 2: playNote(0,3);          //incorrect attempt causes error sound,
                  PORTB &= 0x03;          //  unlights an LED using proper bitmask,
                  failure_Message1();     //  and displays appropriate failure msg,
                  keypad_Unlock();        //  then repeats function with new values
                  break;                  //  for cases 2 and 1
                  
          case 1: playNote(0,3);
                  PORTB &= 0x01;          
                  failure_Message1();
                  keypad_Unlock();              
                  break;
                  
          case 0: playNote(0,3);         //third failed attempt causes error sound,
                  PORTB &= 0x00;         //  unlights final LED light,
                  failure_Message2();    //  displays complete failure message,
                  alarm();               //  sounds the alarm,
                  fFlag = true;          //  and sets the failure flag
                  break;
        }
      } else if(i==7){
          success_Message();             //success msg displayed when input correct
          leds_off();                    //shuts off LEDs
          success_Tune();                //plays success tune
          sFlag = true;                  //sets success flag 
          ms_delay(3000);                //3 second delay
          menu();                        //menu prompt for new code or lock device
      }
    }
  }
}


//-------------------------------------------
// playNote
// 
// Plays the note that corresponds to the key
// that was pressed, based on the pitch index
//-------------------------------------------
void playNote(int pIndex, int num){
  switch(num){
    //sets notes for initial key presses
    case 1: pitch = pitch_val[keyInput[pIndex]];
            sound_init();
            sound_on();     
            break;
    //sets notes for unlock key presses
    case 2: pitch = pitch_val[unlockInput[pIndex]];
            sound_init();
            sound_on();  
            break;
    //sets error note        
    case 3: pitch = 3100;
            sound_init();
            sound_on();
            ms_delay(1000);
            sound_off();                             
            break;
  } 
}

//-------------------------------------------
// success_Tune
// 
// Plays a success tune when the correct code
// is input. (c#, d#, a#)
//-------------------------------------------
void success_Tune(){
  //plays note c# for .2 seconds
  pitch = 1353;
  sound_init();
  sound_on();
  ms_delay(200);
  sound_off();
  
  //plays note d# for .2 seconds
  pitch = 1206;
  sound_init();
  sound_on();
  ms_delay(200);
  sound_off();
  
  //plays note a# for .5 seconds
  pitch = 805;
  sound_init();
  sound_on();            
  ms_delay(500);            
  sound_off();
}

//-------------------------------------------
// alarm
// 
// Sounds an alarm after 3 incorrrect code
// attempts. (repeats a#, f#)
//-------------------------------------------
void alarm(){
  for(i=0; i<8; i++){
    //plays note a# for .3 seconds
    pitch = 805;
    sound_init();
    sound_on();
    ms_delay(300);
    sound_off();
    
    //plays note f# for .3 seconds
    pitch = 1014;
    sound_init();
    sound_on();
    ms_delay(300);
    sound_off();
  }
}

//-------------------------------------------
// newCodePrompt
// 
// Displays prompt message to the LCD screen
// asking the user to input a new lock code
//-------------------------------------------
void newCodePrompt(){
  set_lcd_addr(0x04);
  type_lcd("Welcome!");
  ms_delay(2000);           //2 second delay
  set_lcd_addr(0x00);
  type_lcd("Press switch 2");
  set_lcd_addr(0x40);
  type_lcd("to set new code");
}

//-------------------------------------------
// confirm
// 
// Displays a prompt to the user to either
// save the code he/she just input, or retry
// a new code
//-------------------------------------------
void confirm(){
  wFlag = true;
  clear_lcd();
  set_lcd_addr(0x00);
  type_lcd("Save  ---  Retry");
  set_lcd_addr(0x40);
  type_lcd("   SW3 - SW4    ");
  
  while(wFlag){
    if(SW3_down()){           
      setFlag = true;         //code has been set
      wFlag = false;          //ends wait loop
    } else if(SW4_down()){
      setting_Message();      //returns to code set
      wFlag = false;          //ends wait loop
    }
  }
}

//-------------------------------------------
// menu
// 
// Gives user the choice to either set a new
// code or to lock the device again
//-------------------------------------------
void menu(){
  wFlag = true;
  sFlag = false;
  lockAttempt = 3;
  clear_lcd();
  set_lcd_addr(0x00);
  type_lcd("1. New Code(SW2)");
  set_lcd_addr(0x40);
  type_lcd("2. Re-Lock(SW3)");
  
  while(wFlag){
    if(SW2_down()){           
      setFlag = false;        //unsets the new code flag
      setting_Message();      //displays setting message
      keypad_Set();           //set new code
      wFlag = false;          //ends wait loop
    } else if(SW3_down()){
      clear_lcd();
      set_lcd_addr(0x00);
      type_lcd("Device is Locked");
      ms_delay(3000);         //3 second delay
      code_Prompt1();         
      wFlag = false;          //ends wait loop
    }
  }
}

//-------------------------------------------
// code_Prompt1
// 
// Displays prompt message to the LCD screen
// asking the user to input the code that will
// unlock the device
//-------------------------------------------
void code_Prompt1(){
  clear_lcd();
  set_lcd_addr(0x01);
  type_lcd("Press switch 5");
  set_lcd_addr(0x41);
  type_lcd("to enter code"); 
}

//-------------------------------------------
// code_Prompt2
// 
// Displays prompt message to the LCD screen
// asking to enter the code and turning on
// the right 3 LEDs
//-------------------------------------------
void code_Prompt2(){
  clear_lcd();
  set_lcd_addr(0x01);
  type_lcd("Enter the code");
  if(lockAttempt==3){
    leds_on(0x07);    //turn on right 3 LEDs
  }
}

//-------------------------------------------
// setting_Message
// 
// Displays message on LCD screen that the
// code is still being set and entries left
// to be displayed on the 7-seg display
//-------------------------------------------
void setting_Message(){
  clear_lcd();
  set_lcd_addr(0x04);
  type_lcd("Set code"); 
}

//-------------------------------------------
// failure_Message1
// 
// Displays message to the LCD screen
// indicating failed code input attempt, then
// redisplays input prompt
//-------------------------------------------
void failure_Message1(){
  clear_lcd();
  set_lcd_addr(0x01);
  type_lcd("Incorrect Code");
  set_lcd_addr(0x43);
  type_lcd("Try again");
  ms_delay(1500);         //1.5 second delay
  code_Prompt2();
}

//-------------------------------------------
// failure_Message2
// 
// Displays message to the LCD screen
// indicating complete input failure
//-------------------------------------------
void failure_Message2(){
  clear_lcd();
  set_lcd_addr(0x02);
  type_lcd("Code Failure");
  set_lcd_addr(0x42);
  type_lcd("Device Locked");
}

//-------------------------------------------
// success_Message
// 
// Displays message to the LCD screen
// indicating correct lock code input
//-------------------------------------------
void success_Message(){
  clear_lcd();
  set_lcd_addr(0x04);
  type_lcd("Success!");
  set_lcd_addr(0x40);
  type_lcd("Device Unlocked");
}
