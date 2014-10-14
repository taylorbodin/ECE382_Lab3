# Lab 3 - SPI - "I/O"

## Logic Analyzer
The answers to the logic analyzer section will be posted to GitHub along with the functionality code.

### Question 1
|Line|R12|R13|Purpose|
|:-:|:-:|:-:|:-:|
|66|NOKIA_DATA|0xE7|Writes a little bar with whole in the middle to the display|
|276|NOKIA_CMD|Row Number|Sets the row|
|288|NOKIA_CMD|Column Number|Sets the column|
|294|NOKIA_CMD|Setup call|"Write a command, setup call to make a copy of the TOS"|

### Question 2
|Line|Command/Data|8-bit packet|
|:-:|:-:|:-:|
|66|Data|11100111|
|276|Command|10110010|
|288|Command|00010001|
|294|Command|00000010|

### Question 2 Waveforms
Line 66
![LINE 66](1.jpg "Line 66")

Line 276
![LINE 276](2.jpg "Line 276")

Line 288
![LINE 288](3.jpg "Line 288")

Line 294
![LINE 294](4.jpg "Line 294")

### Question 3

Dec is a two operand instruction with an immediate source and register direct destination meaning it takes two cycles
and jumps always take two cycles. So, 4 cycles per iteration times 0xFFFF iterations is 262140 in decimal. The clock is running 
1 cycle per microsecond which means that reset is held low for approximately 26 miliseconds. 


### Question 4
![xor picture](bitblock.bmp)

## Code

### Setup

This section of code is almost completely borrowed from the provided code. The main changes made were to set R12 and R13 to
zero for a 0,0 cursor start. The second, and most signifcant is a call to #writeBlock which is a subroutine discussed later.

```
main:
	mov.w   #__STACK_END,SP				; Initialize stackpointer
	mov.w   #WDTPW|WDTHOLD, &WDTCTL  	; Stop watchdog timer
	dint								; disable interrupts

	call	#init						; initialize the MSP430
	call	#initNokia					; initialize the Nokia 1206
	call	#clearDisplay				; clear the display and get ready....

	clr		R10							; used to move the cursor around
	clr		R11

	clr		R12							; set up for 0,0 cursor start
	clr	    R13

	call	#setAddress
	call	#writeBlock					; writes the initial block in the upper left
```

### Polling

The first significant change I made to the provided code was to write four loops that polled each of the buttons on the 
display. The structure of each section is very similar so pollUp was chosen as an example. The first section places the 
pin in question into R14. A bit test is performed on the pin inquestion and if it is high (i.e. not pressed) a jump is 
performed to the next button. If it is depressed, a subroutine called wait for release is called. This subroutine takes 
R14 as an input and, not surprisingly, waits for that button to be released. After this housekeeping is taken care of then
the main work of the loop begins. In the case of pollUp, R12 (row) is decremeneted which moves the cursor up one row. 
Finally, #writeBlock is called. If R12 is negative that implies that it is at the top of the top of the display. When this condition is met the program simply jumps to the topOfDisplay which just sets the row to 0 and calls #writeBlock.  

```
pollUp:
	mov		#0x20,	R14
	bit.b	R14, &P2IN					; bit 5 (SW5) of P2IN set?
	jnz		pollDown					; no, poll the next button
	call	#waitForRelease
	dec		R12
	jn		topOfDisplay				; if dec sets the negative flag we're at the top
	call	#writeBlock
	jmp		pollUp

topOfDisplay:							; Keep it at the top of the display
	mov		#0x00, R12
	call	#writeBlock
	jmp		pollUp
```

### writeBlock

Write block is the subroutine that realizes the required functionality of this lab. First the display is cleared of the old
block using a call to #clearDisplay and the address of the cursor (which was passed into this subroutine by R12 and R13) is
set by setAddress. Next, writeBlock realizes a for loop that writes FF (a bar) to the display utilizing #writeNokiaByte. This nice thing is that the Nokia automatically wraps the cursor around for us so all we have to do is run this loop 8 times. 
writeBlock finally cleans up the stack and returns. 

```
writeBlock:
	call	#clearDisplay
	call	#setAddress
	push	R12							;Make a copy of these registers
	push	R13
	clr		R7
write:
	mov		#NOKIA_DATA, R12
	mov		#0xFF, R13					;FF is a solid bar
	call	#writeNokiaByte
	inc		R7							;While R7 > 8 keep writing bars, else jump to cleanup
	cmp	    #BlockWidth, R7
	jeq		cleanup
	jmp		write

cleanup:
	clr		R7
	pop 	R13
	pop		R12
	ret
```

## Debugging

This lab went smoother than expected and didn't require any debugging other than a few, simple syntax errors. 


## Functionality

A functionality was achieved in the lab. In the interest of saving time and space, it can be assumed that since A functionality
requires drawing an 8 by 8 block that required functionality was also achieved. Below is a video demonstrating A functionality. 

<a href="http://www.youtube.com/watch?feature=player_embedded&v=JVRB-ynRHow
" target="_blank"><img src="http://img.youtube.com/vi/JVRB-ynRHow/0.jpg" 
alt="A Functionality" width="240" height="180" border="10" /></a>

