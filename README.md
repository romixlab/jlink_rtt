# SEGGER RTT Support for Rust

This repo implements support for the Real Time Transfer (RTT) debugger
extensions that are present in J-Link devices produced by SEGGER.

## Using it

Basic logging:

```
extern crate jlink_rtt;

fn boo() {
   let mut output = jlink_rtt::Output::new();
   let _ = writeln!("Hello {}", 42);
}
```

Handling panics:

```
#![no_std]

extern crate panic_rtt;

fn main() {
    panic!("message is logged to debugger");
}
```

## Using with OpenOCD
```
git clone http://openocd.zylin.com/openocd
cd ./openocd
git fetch http://openocd.zylin.com/openocd refs/changes/55/4055/8 && git checkout FETCH_HEAD
```

Watch out for that 8, there maybe new changes, see http://openocd.zylin.com/#/c/4055/

```
./bootstrap
./configure --prefix ~/openocd
make 
make install
```

If you have JLink, then create `~/openocd/share/openocd/scripts/interface/jlink_swd.cfg` with this lines:
```
interface jlink
transport select swd
```

And create this script, which would start and stop the rtt, when your mcu is running or halted. `~/openocd/share/openocd/scripts/interface/rtt.cfg`:
```
$_TARGETNAME configure -event resume-end {
        rtt start
}

$_TARGETNAME configure -event halted {
        rtt stop
}
```

Start with this arguments, do not forget to modify version of your stlink adapter or change it to jlink:
```openocd -f interface/stlink-v2.cfg -f target/stm32f1x.cfg -f interface/rtt.cfg -s tcl -c "rtt setup 0x20000000 1352 \"SEGGER RTT\"" -c "rttserver start 9090 0"```

Where 1352 is control structure size with buffers: (24 for ControlBlock structure + (24 bytes for Buffer structure and 1024 bytes is the actual Down buffer) + (24 + 256 as Up buffer).

Telnet server will be started on port 9090 for up/down channel 0 (which is the only one in this implementation). Connect with `telnet localhost 9090`.

If everything is ok you should see `RTT control block found at 0x...` when you flash and let mcu run.

This section is a modified translation of [this article](http://we.easyelectronics.ru/arhiv_6/rtt-s-pomoschyu-openocd-bez-j-link-i-bez-softa-ot-segger.html).
## More info

More information on RTT can be found here:
<https://www.segger.com/products/debug-probes/j-link/technology/about-real-time-transfer/>

The author of this repo is not affiliated with SEGGER, nor is
this repo supported by them.

## License

The implementation is derived from code produced by SEGGER Microcontroller GmbH
under BSD-3-Clause license.

