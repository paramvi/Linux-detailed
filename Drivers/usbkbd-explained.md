# usb_kbd_irq (uki)

this doc is for the usb_kbd_irq(uki) function. This function is the brains behind the whole keyboard program. So kbd->new (new) is the buffer and it receives the input data from keypresses. Now this is parsed in the function of 'uki'. 

In the first loop, special keys: both Shift, Alt, Ctrl are handled. The way they are handled are as follows:

1. When these keys are pressed new[0] gets the data, instead of usual new[2].        There scancodes are listed below.

    * L-shift --> \x2
   
    * L-Ctrl ---> \x1
   
   * L-Alt ----> \x4
   
   * R-Alt ----> \x40
   
   * R-Ctrl ---> \x10
   
   * R-shift --> \x20
   
    Now the loop handles the special keys by right shifting the new[0] appropriately.

    `input_report_key(kbd->dev, usb_kbd_keycode[i + 224], (kbd->new[0] >> i) & 1);`

    `usb_kbd_keycode[224+0]` is 29, 
which is *L-Ctrl* and usb_kbd_keycode[224+1] is '42', which is *L-shift*.

    Example:
    
    Usually when you press say `Shift_L + a` which is Capital `a`, what happens is the 'new' string is like as follows:	
    
    `new: 2 0 4 0 0 0 0`

    In the first loop
   for i = 1, `new[0]>>1 & 1 == true`, 
    so input_report_key reports usb_kbd_keycode[224+1] to the kernel, which is     'shift'. 
    
    'new[2]' is handled in the next loop.
   
2. Second Loop is for getting to work the non-special keys... keys which independently works. Keys like 'k', 's' etc.
Here the loop is from 2 to 8. Why's that? 

    A: special keys have reserved 0th place. 1st place is not known for?
       
    How does the loop work?
   
    First IF condition 

    * old[i] > 3, ... this avoids the scancodes which have values less than 3 ( But then what is values 1-3 for in the new[2] ?).

    * memscan condition: check the 'new' string from the 2nd place to the end of it for the character 'old[i]'. For not present, 
         (returns the address of the next byte of the 'new' which is 'new+8')
         
     If condition 'a' and 'b' is true, it means 'old[i]' key has been released.     
     Second IF condition works the same as well  
     
     Example:  Carrying on with the above example

    ` new: 2 0 4 0 0 0 0`
    
     Now that 'new[0]' is already taken care of, now is the turn of new[2] to new[7].
     
     For this new[2]=4. 
     So it will check the above conditions and if both true, then input_report_key will report the usb_kbd_keycode[4] to the kernel.
     

Some unanswered questions...
1. Why's there input data in new[0] and new[2]. I mean if this is it, then why do we need the other 6 bits for?

2. When I set the packet size to < 8, status code returned is 75, -EOVERFLOW.
   When I set the packet size to > 8 (100), status code returned is 75, -EOVERFLOW. Why?
