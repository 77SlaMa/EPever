# EPever

My attempts to make solar data from EPever Tracer3210AN collected and visualised using Orange Pi PC 2 under Armbian (4.14.48).

1. Default driver (CDC_ACM) for USB - RS-485 wasn't working properly. I had to compile it from vendor (Exar) sources
available at https://www.exar.com/design-tools/software-drivers
Newest version is 1C as of 2018-08-07. Driver had to be patched to work exclusively in RS-485 mode.
(source: https://github.com/kasbert/epsolar-tracer/blob/8c21f4afdfd6acd77b6adad59a4dabe5cbf2b947/xr_usb_serial_common-1a/xr_usb_serial_hal.c )

Installation of kernel headers needed for module compilation wasn't working in armbian-config, so I downloaded them and installed
manually. https://apt.armbian.com/pool/main/l/linux-4.14.48-sunxi64/

Compiled module wasn't loading properly - extraversion wasn't set in headers Makefile - set it to "-sunxi64".

After that Luca Soltoggio's scripts (https://github.com/toggio/PhpEpsolarTracer) started getting data from EPever controller.
Remember to edit each file and replace any reference to USB port (/dev/ttyUSB0 for example) to Exar specific (ttyXRUSB0). 

I've followed many steps of Colin HickeyColin Hickey covered in his video https://www.youtube.com/watch?v=_VcWwJIq4XU and on his git https://github.com/chickey/Epever-influxdb

2. It turned out, that EPever controller is reporting some parameters in non-obvious way. For example - Battery-charging-current is positive even if battery is delivering current. So in order to monitor curent and it's direction you have to monitor Stat-net-battery-current instead.
Here I encountered another quirk - as long as current is flowing into battery, measurement is fine. As soon as battery starts discharging, value jumps to millions or more. I discovered, that routine which calculates values and sign is not working properly (anyway on my setup) - there is some problem with string/integer/hex conversion in "private function convertData". PhpEpolarTracer.php here is modified and works well at least for me.
There was also a need to add "22" (Stat-net-battery-current) to array of values "suspected" to be negative.
