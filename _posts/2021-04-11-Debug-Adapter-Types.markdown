---
layout: post
title:  "Debug Adapter Types"
date:   2021-04-11 
categories: Debug Adapter Protocol
---
Hello, I have started working on an debug adapter in [Rust](https://www.rust-lang.org/) for my [debugger](https://github.com/Blinningjr/master-thesis).
A debug adapter is a program that is between the debugger and the development tools that the user uses, this can be an [IDE](https://en.wikipedia.org/wiki/Integrated_development_environment), editor or some other tool.
The adapter handles the communication between the two and the idea is that the adapter enables any tool to communicate with the debugger.
Thus making it easier for IDE and debug developers to support multiple IDE:s/debuggers.
The specification for the debug adapter and the communication is given by [Microsoft Debug Adapter Protocol](https://microsoft.github.io/debug-adapter-protocol/)(DAP).


When reading the specifications for DAP I realised that I would have to create many different structs for all the different events, requests, responses and more.
So I had a look at how it was done in the  [probe-rs/vscode](https://github.com/probe-rs/vscode) project and found that they used a lib called [debugserver_types](https://docs.rs/debugserver-types/) which defines all of the structs found in the DAP specification.
But there are still some structs that I need to create my self because DAP have some arguments that are debug adapter specific.


Thus if you happen to be creating a debug adapter yourself I recommend checking out the rust crate [debugserver_types](https://docs.rs/debugserver-types/).

