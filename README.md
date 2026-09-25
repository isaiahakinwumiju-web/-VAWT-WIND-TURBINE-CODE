hall = 0
test = 0
running = False
H = 0
rpm = 0
MWh = 0
p2 = 0
p1 = 0
# Setup pull-ups for buttons to avoid floating pins
pins.set_pull(DigitalPin.P1, PinPullMode.PULL_UP)
pins.set_pull(DigitalPin.P2, PinPullMode.PULL_UP)
lastP1 = 1
lastP2 = 1
basic.show_leds("""
    # . . . #
    . # # # .
    # # # # #
    . # # # .
    # . . . #
    """)
makerbit.connect_lcd(39)
makerbit.show_string_on_lcd1602("MWh", makerbit.position1602(LcdPosition1602.POS1), 16)
makerbit.show_string_on_lcd1602("rpm", makerbit.position1602(LcdPosition1602.POS7), 16)
makerbit.show_string_on_lcd1602("H", makerbit.position1602(LcdPosition1602.POS15), 16)

def on_forever():
    global p1, p2, MWh, rpm, H, running, lastP1, lastP2, test, hall
    # Read button states (active LOW)
    p1 = pins.digital_read_pin(DigitalPin.P1)
    p2 = pins.digital_read_pin(DigitalPin.P2)
    # Button B (P2) - RESET
    if p2 == 0 and lastP2 == 1:
        basic.show_leds("""
            # . . . .
            # . . . .
            # . . . .
            # # # # #
            # # # # #
            """)
        MWh = 0
        rpm = 0
        H = 0
        makerbit.clear_lcd1602()
        makerbit.show_string_on_lcd1602("MWh", makerbit.position1602(LcdPosition1602.POS1), 16)
        makerbit.show_string_on_lcd1602("" + str(MWh),
            makerbit.position1602(LcdPosition1602.POS17),
            16)
        makerbit.show_string_on_lcd1602("rpm", makerbit.position1602(LcdPosition1602.POS7), 16)
        makerbit.show_string_on_lcd1602("" + str(rpm),
            makerbit.position1602(LcdPosition1602.POS24),
            16)
        makerbit.show_string_on_lcd1602("H ", makerbit.position1602(LcdPosition1602.POS15), 16)
        makerbit.show_string_on_lcd1602("" + str(H),
            makerbit.position1602(LcdPosition1602.POS30),
            16)
        # debounce
        basic.pause(200)
    # Button A (P1) - TOGGLE running state
    if p1 == 0 and lastP1 == 1:
        basic.show_leds("""
            # . . . #
            # # . . #
            # # # # #
            # # . # #
            # . . . #
            """)
        running = not (running)
        # debounce
        basic.pause(200)
    lastP1 = p1
    lastP2 = p2
    # Read sensors
    test = input.acceleration(Dimension.STRENGTH)
    hall = pins.analog_read_pin(AnalogPin.P4)
    H = input.magnetic_force(Dimension.STRENGTH)
    # Update LCD with latest values
    makerbit.show_string_on_lcd1602("" + str(MWh),
        makerbit.position1602(LcdPosition1602.POS17),
        16)
    makerbit.show_string_on_lcd1602("" + str(rpm),
        makerbit.position1602(LcdPosition1602.POS24),
        16)
    makerbit.show_string_on_lcd1602("" + str(H),
        makerbit.position1602(LcdPosition1602.POS30),
        16)
    # Only update counters if running
    if running:
        basic.pause(1000)
        MWh += 0.02
        rpm += 1
    # Output to serial for logging / debugging
    serial.write_value("MWh", MWh)
    serial.write_value("rpm", rpm)
    serial.write_value("H", H)
    serial.write_value("test", test)
basic.forever(on_forever)

