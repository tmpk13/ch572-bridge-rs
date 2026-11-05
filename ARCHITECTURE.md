# Architecture

`main.rs` starts the chip and runs one polling loop with interrupts off.
Only `hw.rs` touches registers. The rest reaches the hardware through
three traits, `UsbRegs` (`usb.rs`), `SwdIo` (`swd.rs`) and `UartPort`
(`bridge.rs`), which `sim.rs` also implements for the host tests (dotted
lines). Smaller pieces: `touch.rs` spots the 1200 baud touch and
`linecode.rs` turns CDC line coding into UART divisors, both for
`bridge.rs`; `ring.rs` is the UART buffering in `hw.rs`; `regs.rs` holds
the register names.

```mermaid
flowchart LR
    subgraph PC["Host PC"]
        Tools["probe-rs, OpenOCD,<br/>serial terminal"]
        ISP["isp.py<br/>flasher"]
        Sim["sim.rs<br/>host tests: USB controller model,<br/>SWD target, UART"]
    end
    subgraph Chip["CH572D"]
        subgraph FW["Bridge firmware"]
            Main["main.rs<br/>startup, polling loop"]
            Bridge["bridge.rs<br/>composite device"]
            CDC["usbd-serial<br/>CDC-ACM"]
            DAPIF["dapclass.rs<br/>CMSIS-DAP v2 interface"]
            DAP["dap-rs<br/>DAP commands"]
            SWD["swd.rs<br/>bit-banged SWD"]
            Bus["usb.rs<br/>usb-device bus driver"]
            HW["hw.rs<br/>clock, USB, UART, pins"]
        end
        ROM["ROM bootloader"]
    end
    Wio["Wio-E5"]

    Main --> Bridge
    Bridge --> CDC --> Bus
    Bridge --> DAPIF --> Bus
    Bridge --> DAP --> SWD
    Bus -->|UsbRegs| HW
    SWD -->|SwdIo| HW
    Bridge -->|UartPort| HW
    Bus -.->|UsbRegs| Sim
    SWD -.->|SwdIo| Sim
    Bridge -.->|UartPort| Sim
    Tools <-->|"USB on PA0/PA1,<br/>1209:0001"| HW
    HW <-->|"UART PA8/PA9<br/>to USART2"| Wio
    HW -->|"PA5 SWCLK, PA7 SWDIO,<br/>PA6 nRESET"| Wio
    HW -->|"1200 baud touch:<br/>reset into the ROM"| ROM
    ISP <-->|"USB, 1a86:55e0"| ROM
    ROM -->|"programs flash,<br/>resets into it"| Main
```

## Flashing a blank chip

Why `mise run flash` starts before the plug-in: the bootloader's idle
timeout and isp.py's wait, from NOTES.md. The plug-in and start times
are examples.

```mermaid
gantt
    title Catching the bootloader of a blank chip
    dateFormat mm:ss
    axisFormat %M:%S
    todayMarker off
    section Plug in, then flash
    Bootloader on USB, nothing talks to it : crit, 00:00, 10s
    mise run flash polls, finds nothing    : 00:12, 30s
    section Flash, then plug in
    mise run flash polls                   : 00:00, 5s
    Plug in, isp.py takes the bootloader   : milestone, 00:05, 0s
```
