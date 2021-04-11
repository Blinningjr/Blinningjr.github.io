---
layout: post
title:  "First Post"
date:   2021-03-29 
categories: My blog
---
Hello, my name is Niklas Lundberg and I am currently a computer science student at Luleå University of Technology. This blog is a place for me to share what I am doing and to post information on subjects I think lacks information or good explanations.

I am currently doing my master thesis work which involves creating a debugger in Rust for embedded Rust applications. It uses the [probe-rs](https://probe.rs/) library for communicating with the embedded system, which already gives a lot of the debug functionality I intend to implement for my debugger. But the main part I am implementing for my debugger is the ability to read the values of variables and parse them to the correct type. I am also trying to make my debugger give a better experience in debugging optimized Rust code then other debuggers, such as GDB. Another part of my master thesis work is to create a debug adapter that will further improve on the user experience of the debugger. Here is a link to the repository [link](https://github.com/Blinningjr/master-thesis).
