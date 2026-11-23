# ArduiPi_OLED: W0CHP's changes (from WPSD)

Two changes, as WPSD ships them for MMDVM-Display's OLED screens:

1. **I2C through the kernel's i2c-dev driver** (`/dev/i2c-1`;
   `ioctl(I2C_SLAVE)` and `write()`) instead of the bcm2835 library's
   direct register access. That access needs `/dev/mem`, so root, and
   the BCM2835 I2C block does not exist on a Pi 5. With i2c-dev the
   display works for any user in the `i2c` group, on every board. I2C
   OLED modules have no reset line, so no GPIO is touched in I2C mode.
   SPI displays still use bcm2835. Load the `i2c-dev` module at boot.

2. **The 1.3" (SH1106, OLED type 6) display scrolls, without tearing.**
   The SH1106 has no hardware scroll; the SSD1306 scroll commands
   MMDVM-Display sends mean other things to it (column and start-line
   commands), which is why the text tore across the screen. For an I2C
   SH1106, `startscroll*()` and `stopscroll()` now run a software scroll:
   a thread rotates the scrolled pages of the last frame `display()`
   sent, about 25 columns a second, and the diagonal scroll's vertical
   part uses the SH1106's display start line. One lock serializes every
   I2C write, so a new frame and a scroll step never interleave. The
   SSD1306 keeps its hardware scroll, unchanged.

Tested on a 1.3" display (left scroll, status bar, diagonal, a new frame
while scrolling) and on a 0.96" one (hardware scroll, unchanged).
