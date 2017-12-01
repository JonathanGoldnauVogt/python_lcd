Snektek: Modified I2C and GPIO LCD library based on work by DHylands.
https://github.com/dhylands/python_lcd


Instructions for use with SnekTek AIR director plus and 16x2 I2C LCD:

1. Copy:
```
lcd_api.py
director_plus_i2c_lcd.py
director_plus_i2c_lcd_test.py
script.py
```
to director plus.

2. Reset director plus.

3. Look in 
```
director_plus_i2c_lcd_test.py
```
for ideas on how to use the LCD.


Custom characters
=================

The HD44780 displays come with 3 possible CGROM font sets. Japanese, European and Custom.
Test which you have using:

```
lcd.putchar(chr(247))
```

If you see Pi (π), you have a Japanese A00 ROM.
If you see a division sign (÷), you have a European A02 ROM.

Characters match ASCII characters in range 32-127 (0x20-0x7F) with a few exceptions:

* 0x5C is a Yen symbol instead of backslash
* 0x7E is a right arrow instead of tilde
* 0x7F is a left arrow instead of delete

Only the ASCII characters are common between the two ROMs 32-125 (0x20-0x7D)
Refer to the HD44780 datasheet for the table of characters.

The first 8 characters are CGRAM or character-generator RAM.
You can specify any pattern for these characters.

To design a custom character, start by drawing a 5x8 grid.
I use dots and hashes as it's a lot easier to read than 1s and 0s.
Draw pixels by replacing dots with hashes.
Where possible, leave the bottom row unpopulated as it may be occupied by the underline cursor.

```
Happy Face (where .=0, #=1)
.....
.#.#.
.....
..#..
.....
#...#
.###.
.....
```

To convert this into a bytearray for the custom_char() method, you need to add each row of 5 pixels to least significant bits of a byte (the right side).

```
Happy Face (where .=0, #=1)
..... == 0b00000 == 0x00
.#.#. == 0b01010 == 0x0A
..... == 0b00000 == 0x00
..#.. == 0b00100 == 0x04
..... == 0b00000 == 0x00
#...# == 0b10001 == 0x11
.###. == 0b01110 == 0x0E
..... == 0b00000 == 0x00
```

Next, add each byte from top to bottom to a new byte array and pass to custom_char() with location 0-7.

```
happy_face = bytearray([0x00,0x0A,0x00,0x04,0x00,0x11,0x0E,0x00])
lcd.custom_char(0, happy_face)
```

`custom_char()` does not print anything to the display. It only updates the CGRAM.
To display the custom characters, use putchar() with chr(0) through chr(7).

```
lcd.putchar(chr(0))
lcd.putchar(b'\x00')
```

Characters are displayed by reference.
Once you have printed a custom character to the lcd, you can overwrite the custom character and all visible instances will also update.
This is useful for drawing animations and graphs, as you only need to print the characters once and then can simply modify the custom characters in CGRAM.

Examples:

```
# smiley faces
happy = bytearray([0x00,0x0A,0x00,0x04,0x00,0x11,0x0E,0x00])
sad = bytearray([0x00,0x0A,0x00,0x04,0x00,0x0E,0x11,0x00])
grin = bytearray([0x00,0x00,0x0A,0x00,0x1F,0x11,0x0E,0x00])
shock = bytearray([0x0A,0x00,0x04,0x00,0x0E,0x11,0x11,0x0E])
meh = bytearray([0x00,0x0A,0x00,0x04,0x00,0x1F,0x00,0x00])
angry = bytearray([0x11,0x0A,0x11,0x04,0x00,0x0E,0x11,0x00])
tongue = bytearray([0x00,0x0A,0x00,0x04,0x00,0x1F,0x05,0x02])

# icons
bell = bytearray([0x04,0x0e,0x0e,0x0e,0x1f,0x00,0x04,0x00])
note = bytearray([0x02,0x03,0x02,0x0e,0x1e,0x0c,0x00,0x00])
clock = bytearray([0x00,0x0e,0x15,0x17,0x11,0x0e,0x00,0x00])
heart = bytearray([0x00,0x0a,0x1f,0x1f,0x0e,0x04,0x00,0x00])
duck = bytearray([0x00,0x0c,0x1d,0x0f,0x0f,0x06,0x00,0x00])
check = bytearray([0x00,0x01,0x03,0x16,0x1c,0x08,0x00,0x00])
cross = bytearray([0x00,0x1b,0x0e,0x04,0x0e,0x1b,0x00,0x00])
retarrow = bytearray([0x01,0x01,0x05,0x09,0x1f,0x08,0x04,0x00])

# battery icons
battery0 = bytearray([0x0E,0x1B,0x11,0x11,0x11,0x11,0x11,0x1F]))  # 0% Empty
battery1 = bytearray([0x0E,0x1B,0x11,0x11,0x11,0x11,0x1F,0x1F]))  # 16%
battery2 = bytearray([0x0E,0x1B,0x11,0x11,0x11,0x1F,0x1F,0x1F]))  # 33%
battery3 = bytearray([0x0E,0x1B,0x11,0x11,0x1F,0x1F,0x1F,0x1F]))  # 50%
battery4 = bytearray([0x0E,0x1B,0x11,0x1F,0x1F,0x1F,0x1F,0x1F]))  # 66%
battery5 = bytearray([0x0E,0x1B,0x1F,0x1F,0x1F,0x1F,0x1F,0x1F]))  # 83%
battery6 = bytearray([0x0E,0x1F,0x1F,0x1F,0x1F,0x1F,0x1F,0x1F]))  # 100% Full
battery7 = bytearray([0x0E,0x1F,0x1B,0x1B,0x1B,0x1F,0x1B,0x1F]))  # ! Error
```
