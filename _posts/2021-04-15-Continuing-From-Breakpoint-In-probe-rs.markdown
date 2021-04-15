---
layout: post
title:  "Continuing From Breakpoint In probe-rs"
date:   2021-04-15 
categories: probe-rs
---
Hello, when I was working on my [debugger](https://github.com/Blinningjr/master-thesis) in [Rust](https://www.rust-lang.org/) I noticed that calling `core.run()` and `core.step()` does nothing if the core is halted because of a breakpoint.
`core.run()` is a function call that is suppose to resume the execution of the program running on the debugee and it is from a lib called [probe-rs](https://github.com/probe-rs/probe-rs), which I use in my debugger to flash and debug the debugee.


There are two parts to this problem because there are two different types of breakpoints, hardware(HW) and software(SW) breakpoints.
The easier one of the two to solve is HW breakpoints so I will first explain how I solved it for them, then I will explain how I solved it for SW breakpoints and lastly how I used both of these solution to solve the whole problem.


# Hardware Breakpoint Solution
My solution of continuing executing from a HW breakpoint is to first remove the breakpoint at the current program counter(pc) if there is a HW breakpoint there.
To check if there is a program HW breakpoint there I search through my debuggers list of set HW breakpoint and compare the location to the current pc.
If there is no HW breakpoint set at that location because it has been removed then in that case `core.run()` and `core.step()` will work and no extra steps are needed.
But if a HW breakpoint is present then my fix is to remove it, then use `core.step()` to step one instruction and after that I add back the HW breakpoint.
This will solve the problem of not being able to continue execution when the core is halted by an HW breakpoint.


Here is some example code of my solution:
```
match core.status()? {
        probe_rs::CoreStatus::Halted(r)  => {
            match r {
                probe_rs::HaltReason::Breakpoint => {
                    let pc = core.registers().program_counter();
                    let pc_val = core.read_core_reg(pc)?;

                    for bkpt in breakpoints {
                        if pc_val == *bkpt {
                            core.clear_hw_breakpoint(pc_val)?;

                            let res = core.step();

                            core.set_hw_breakpoint(pc_val)?;

                            return res;
                        }
                    }
                    return core.step();
                },
                _ => (),
            };
        },
        _ => (),
    };

    core.step()
```
Note if I wanted to run `core.run()` I would call if after I have executed the code in the example.


# Software Breakpoint Solution
My solution of continuing executing from a SW breakpoint is to add the size of a breakpoint machine instruction to the current pc and that fixes the problem.
The hard part of the problem is that it is hardware specific in that the machine code for breakpoint is different and the size of the machine code is different.
Thus this solution requires a lot of research into different chipsets.


Here is some example code of my solution:
```
let pc = core.registers().program_counter();
let pc_val = core.read_core_reg(pc)?;

// NOTE: Increment with 2 because Cortex-M brekpoint instuction is 2 bytes.
let step_pc = pc_val + 0x2;

core.write_core_reg(pc.into(), step_pc)?;
```
Note that this example only works for [Cortex-M](https://en.wikipedia.org/wiki/ARM_Cortex-M) because the breakpoint machine code size is hard coded for Cortex-M, which is 2 bytes.


# Complete Solution
Now to solve the whole problem I combine the two solutions by checking if the breakpoint is a SW or HW breakpoint and use there respective solution.
The way I check which type of breakpoint it is, is by checking if the machine instruction at the current pc is a breakpoint instruction.
This requires that the debugee chipset is known and what machine code the breakpoint instruction has for that chipset.
Thus if the current instruction is a breakpoint instruction then I use my solution for SW breakpoints and otherwise I use my solution for HW breakpoints.


Here is some example code of my solution:
```
fn continue_fix(core: &mut Core, breakpoints: &Vec<u32>) -> Result<CoreInformation, probe_rs::Error>
{
    match core.status()? {
        probe_rs::CoreStatus::Halted(r)  => {
            match r {
                probe_rs::HaltReason::Breakpoint => {
                    let pc = core.registers().program_counter();
                    let pc_val = core.read_core_reg(pc)?;

                    let mut code = [0u8; 2];
                    core.read_8(pc_val, &mut code)?;
                    if code[1] == 190 && code[0] == 0 { // bkpt == 0xbe00 for Cortex-M
                        // NOTE: Increment with 2 because bkpt is a 2-byte instruction in Cortex-M.
                        let step_pc = pc_val + 0x2;

                        core.write_core_reg(pc.into(), step_pc)?;

                        return core.step();
                    } else {
                        for bkpt in breakpoints {
                            if pc_val == *bkpt {
                                core.clear_hw_breakpoint(pc_val)?;

                                let res = core.step();

                                core.set_hw_breakpoint(pc_val)?;

                                return res;
                            }
                        }
                        return core.step();
                    }
                },
                _ => (),
            };
        },
        _ => (),
    };

    core.step()
}
```
Note that this example only works for [Cortex-M](https://en.wikipedia.org/wiki/ARM_Cortex-M) because the breakpoint machine code is hard coded for Cortex-M, which is `0xbe00` and note that it is 2-bytes long.

