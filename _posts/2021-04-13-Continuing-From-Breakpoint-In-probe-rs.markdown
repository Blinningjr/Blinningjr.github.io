---
layout: post
title:  "Continuing From Breakpoint In probe-rs"
date:   2021-04-13 
categories: probe-rs
---
Hello, when I was working on my [debugger](https://github.com/Blinningjr/master-thesis) in [Rust](https://www.rust-lang.org/) I noticed that calling `core.run()` and `core.step()` does nothing if the core is halted because of a breakpoint.
`core.run()` is a function call that is suppose to resume the execution of the program running on the debugee and it is from a lib called [probe-rs](https://github.com/probe-rs/probe-rs), which I use in my debugger to flash and debug the debugee.


To solve this problem I made a function that fixed this problem by first removing the breakpoint from the current location which solves the problem, but now I have to add back the breakpoint.
But before I can do that I have to step one instruction so that I don't get the same problem again.
Then I add back the breakpoint and the problem is solved, if I was going to call `core.run()` I need to do that now to continue running the program on the debugee.


Here is the code of my implementation of this fix:
```
fn continue_fix(core: &mut Core) -> Result<CoreInformation, probe_rs::Error>
{
    match core.status()? {
        probe_rs::CoreStatus::Halted(r)  => {
            match r {
                probe_rs::HaltReason::Breakpoint => {

                    let pc = core.registers().program_counter();
                    let pc_val = core.read_core_reg(pc)?;
                    core.clear_hw_breakpoint(pc_val)?;

                    let res = core.step();

                    core.set_hw_breakpoint(pc_val)?;

                    return res;
                },
                _ => (),
            };
        },
        _ => (),
    };

    core.step()
}
```

If you want to checkout how I use this function have a look [here](https://github.com/Blinningjr/master-thesis/blob/bb9e85ef037a3943d48b9c413b5ea315a7f6ec81/embedded-rust-debugger/src/commands.rs#L174-L203).

