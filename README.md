# ZMK Analog Input Driver

This driver groups ADC io channels into single input event for input subsystem. It provides config for thumbstick input with mid-point alignment, min-mav limitation, deadzone, sampling rate, reporting rate, multiplier, divisor, invertor, etc.

## Installation

Only GitHub actions builds are covered here. Local builds are different for each user, therefore it's not possible to cover all cases.

Include this project on your ZMK's west manifest in `config/west.yml`:

```yml
manifest:
  ...
  projects:
    ...
    - name: zmk-analog-input-driver
      remote: badjeff
      revision: main
    ...
```

Now, update your `board.overlay` adding the necessary bits (update the pins for your board accordingly):

```dts
#include <zephyr/dt-bindings/input/input-event-codes.h>
/* Reference: https://docs.zephyrproject.org/apidoc/latest/group__input__events.html */

&adc {
	status = "okay";
};

/ {
	anin0: analog_input_0 {
		compatible = "zmk,analog-input";
		sampling-hz = <100>;
		x-ch {
			io-channels = <&adc 2>;
			mv-mid = <1630>;
			mv-min-max = <1600>;
			mv-deadzone = <10>;
			scale-multiplier = <1>;
			scale-divisor = <70>;
			invert;
			evt-type = <INPUT_EV_REL>;
			input-code = <INPUT_REL_X>;
		};
		y-ch {
			io-channels = <&adc 3>;
			mv-mid = <1630>;
			mv-min-max = <1600>;
			mv-deadzone = <10>;
			scale-multiplier = <3>;
			scale-divisor = <4>;
			invert;
			evt-type = <INPUT_EV_REL>;
			input-code = <INPUT_REL_Y>;

			/* enable report mdoe for gamepad axix or knob */
			/* to only call input_report on quantquantized value is updated */
			report-on-change-only;

		};
	};
};
```

Now enable the driver config in your `<shield>.config` file (read the Kconfig file to find out all possible options):

```conf
# Enable Analog Input
CONFIG_ADC=y
# Use async mode (Optional)
CONFIG_ADC_ASYNC=y

# Enable Analog Input Module
CONFIG_ANALOG_INPUT=y
# CONFIG_ANALOG_INPUT_LOG_LEVEL_DBG=y
# CONFIG_ANALOG_INPUT_REPORT_INTERVAL_MIN=22

# Enable logging for pre/post processed value
CONFIG_ANALOG_INPUT_LOG_DBG_RAW=y
CONFIG_ANALOG_INPUT_LOG_DBG_REPORT=y

```

## Troubleshooting

If you are running on nrf52840 board and analog reading get stuck after some moment, you need to ground all `uint8_t adc_sequence::oversampling` to zero in your ZMK branch in respect to `oversampling` setting is unsupported by given ADC hardware in a specific mode. [Reference](https://docs.zephyrproject.org/apidoc/latest/structadc__sequence.html#a233e8b20b57bb2fdbebf2c85f076c802)

