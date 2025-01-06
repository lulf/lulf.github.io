---
layout: post
title: Debugging embedded rust
author: Ulf Lilleengen
categories: technical work rust embedded
---

Debugging can be a frustrating experience, where the self-confidence of the human debugger can vary from very low to very high. I've benefited from others writing about their experience, so I will do the same to hopefully help others. This time I'm writing about a recent experience at work where I was looking into one of those rare bugs that are hard to debug.

### The system

One of the Akiles products is a [WiFi-enabled Gateway](https://akiles.app/en/products/smart-lock-system-controller) that can be wired to different kinds of locks and inter-com systems. Once wired, it can be opened via a phone app, a browser, or [another device](https://akiles.app/en/products/smart-lock-system-pinpad).

All products, are updated with bugfixes and new features as we make them. But sometimes, even after a lot of testing, we find bugs in the firmware. To reduce the impact of that, we perform a gradual rollouts, which tends to catch most of the rare ones.

The firmware is written in Rust, and uses the [embassy](https://embassy.dev/) framework for accessing peripherals and scheduling async Rust tasks on the microcontroller. Having worked on large C and C++ code bases in the past, I'm 100% sure that Rust has prevented a lot of bugs. But, it turns out you can write bugs in Rust as well! 

### The symptoms

After the initial rollout of the firmware, we noticed that some products, in particular those with a wifi connectivity, crashed after 24-48 hours of operation.

After looking at the crash logs, we were left with scratching our heads for a while. The only message was:

```
TODO: Copy paste error message
error Watchdog expired
```

The firmware uses a watchdog to detect hangs, so the first hypothesis is that we have a task in the firmware that is busy-looping.

Sidenote: A watchdog is a peripheral in the hardware that you must 'pet' periodically within a configurable time limit, for instance at least once every 5 seconds. If it is not pet, it will cause the firmware to crash. This is a commonly used peripheral when you have systems that are remotely operated and must be able to recover from hangs and bugs in the firmware.

#### Problem 1: Getting a core dump

Normally, the panic handler (a function that is invoked on a crash) would output the processor registers and stack as well, but that did not happen.

To get to this information, we ended up creating a software watchdog that expired slightly before the hardware watchdog. The hypothesis being that the panic handler didn't have enough time to print the registers and stack before the chip was reset.

This turned out to be true, and after fixing that, the error message at least got a little better:

```
TODO: Example register dump
```

But this still leaves something to be desired in terms of displaying the information. Eager to see if the root cause could be found, however, the first step went looking at the firmware file using `objdump`, a handy tool for inspecting binaries that can show you what is located at different flash and memory addresses.

For instance, to dump the program code and demangle the symbols:

```
TODO: Example objdump output
```

* TODO: Write about return register pointing to WFI
* TODO: Write about trusting debug output

In our case, if we looked at the address of the `PC` (program counter) register, it turned out to point to the memory address running
the `WFI` instruction. WFI is a special instruction for ARM that puts the microcontroller to sleep until an interrupt wakes it up.

We were now left with a mystery: if the executor is not running anything, there are no tasks busy looping that could cause the watchdog to trigger. 

##### Interlude: getting a stack trace

When debugging, make sure you can debug it again more efficiently next time. What would have made core dump easier to reason about, is having the stack trace directly in the debug log. Luckily, [probe-rs](https://probe.rs/) makes that part fairly easy once you have the ELF file, the register values as well as a chunk of the stack.

This is almost worthy of a blog post on it's own, but the gist of it is first preparing the input data:

```rust
let elf = std::fs::read("app.elf");
let di = probe_rs::debug::DebugInfo::from_raw(&elf[..]).unwrap();

let instruction_set: InstructionSet = InstructionSet::Thumb2;
let supports_native_64bit_access: bool = false;
let core_type = CoreType::Armv7em;
let fpu_support = false;
let floating_point_register_count = None;
let dump = CoreDump {
    registers,
    data,
    instruction_set,
    supports_native_64bit_access,
    core_type,
    fpu_support,
    floating_point_register_count,
};
```

Where registers is a hash map of register identifiers and values (see the [docs](https://docs.rs/probe-rs/latest/probe_rs/debug/debug_info/struct.DebugInfo.html) for more details). The data is an array of memory addresses and corresponding slices of data at that location. The other input values depend on the chip.

It is then possible to iterate over the stack frames using the probe-rs API and format it as desired:

```rust
let initial_registers = dump.debug_registers();
let exception_handler = probe_rs::exception_handler_for_core(dump.core_type());
let instruction_set = Some(dump.instruction_set);
match di.unwind(&mut dump, initial_registers, exception_handler.as_ref(), instruction_set) {
    Ok(stack_frames) => {
        for (i, frame) in stack_frames.iter().enumerate() {
            let mut s = String::new();
            write!(&mut s, "Frame {}: {} @ {}", i, frame.function_name, frame.pc).unwrap();

            if frame.is_inlined {
                write!(&mut s, " inline").unwrap();
            }

            if let Some(location) = &frame.source_location {
                if location.directory.is_some() || location.file.is_some() {
                    write!(&mut s, "\n       ").unwrap();

                    if let Some(dir) = &location.directory {
                        write!(&mut s, "{}", dir.to_string_lossy()).unwrap();
                    }

                    if let Some(file) = &location.file {
                        write!(&mut s, "/{file}").unwrap();

                        if let Some(line) = location.line {
                            write!(&mut s, ":{line}").unwrap();

                            if let Some(col) = location.column {
                                match col {
                                    probe_rs::debug::ColumnType::LeftEdge => {
                                        write!(&mut s, ":1").unwrap();
                                    }
                                    probe_rs::debug::ColumnType::Column(c) => {
                                        write!(&mut s, ":{c}").unwrap();
                                    }
                                }
                            }
                        }
                    }
                }
            }

            println!("{}", s);
        }
    }
    Err(e) => {
        eprintln!("Error unwinding: {:?}", e);
    }
}
```

#### Instrumentation

Now that we had stack traces, it confirmed that the device was in fact not running any tasks at this point. 

To that end, the [`embassy-executor`](https://crates.io/crates/embassy-executor) already supported some notion of tracing via a feature named `rtos-trace`. After [expanding the functionality](https://github.com/embassy-rs/embassy/pull/3627) of the tracing to support multiple executors in the firmware, it was possible to build a table of the tasks last executed before the crash, and print them to the crash log.

To use the tracing feature, simply enable the `trace` feature of the embassy-executor crate. It will require you to implement the following functions in your firmware, but with it you can identify which executor and which task by looking up the memory addresses in the ELF file.

Example:
```
```

The trace revealed that:

1. The watchdog task had not run.
1. Several tasks had run after the watchdog task.
1. No tasks had run after the wifi driver task.

It looked like there was an issue within the executor. But, this code had not been modified for a long time, and been tested and used by not just Akiles, but many Embassy users.

Could it be memory corruption? More information was needed to peek into the the state of the device at the time of crash.

#### More debug info

In order to peek into the state of the device, we decided to add a mechanism to an affected device that would essentially write the contents of the RAM to the external flash, which could later be retrieved and analyzed. Obviously there are security risks of doing that, so we took care to only do this on device in which we had full control and responsibility.

#### Analyze

Finally we got hold of a complete RAM dump, combined with stack traces, executor instrumentation and more. 

Unfortunately, this can become quite overwhelming, and using objdump and hexdump is not for the faint of heart. Luckily, my co-worker pointed me towards [Ghidra](https://ghidra-sre.org/), a reverse engineering tool built by NSA. 

Ghidra looks a bit dated with the Java Swing UI, but I've made errors in underestimating tools based on their looks before. 

Ghidra allows you to load the ELF file of the firmware, and it will automatically build annotations into the memory locations of the binary and the RAM. The second part is to add the contents of RAM and let Ghidra work it's magic. After looking around the internets, we found a python script that we modified to run in Ghidra:

```python
```

We could now inspect the various memory locations, and more importantly, apply Rust type information from the ELF file to it. The result is shown below:

We could now inspect the full state of the system!

TODO: Screenshot

After poking around, we could confirm what the instrumentation was saying: the executor had a list of tasks that should have been run, but had not run! The run queue and timer queues all showed no signs of memory corruption, so the issue had to be somewhere else.

We started to look more at the time driver code, which is the thing that ensures anything using `embassy-time`in the application to wait a certain amount of time, will be awoken after the specified amount of time.

#### 

### Summary

