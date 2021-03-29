---
layout: post
title:  "Explaining Pieces in Dwarf"
date:   2021-03-29 
categories: Dwarf Format
---
# Background
Working on my embedded Rust debugger I found it a bit tricky to evaluate the value of a variable using Dwarf pieces and the variable type, so I am going to explain the relevant parts of the Dwarf format and how I solved it.
Fist of all I use the library [gimli-rs](https://github.com/gimli-rs/gimli) in my debugger for reading the Dwarf format, which makes it a lot easier to work with. Thus some of the information on how I solved it will be specific to gimli-rs and Rust, but it should not be hard to translate it over to other languages and libraries.

To describe the information that a Dwarf piece contains I will use the definition in the gimli-rs [documentation](https://docs.rs/gimli/0.23.0/gimli/read/struct.Piece.html), were a piece is defines as follows:
```
pub struct Piece<R, Offset = <R as Reader>::Offset>
where
    R: Reader<Offset = Offset>,
    Offset: ReaderOffset,
  {
      pub size_in_bits:	Option<u64>,
      pub bit_offset: Option<u64>,
      pub location: Location<R, Offset>,
  }
```

From this definition it is clear that a piece has three attributes, all of the attributes can are optional and don't need to have a value, even the location attribute.
The first attribute called `size_in_bits` describes the size of the value the piece represent, the size is described in bits.
The second attribute called `bit_offset` describes the offset to the value and is only present if the `location` attribute is a address or registry.
The last attribute called `location` describes the value of the piece or a location where the value of the piece is stored.


# The Problem
To find the value of a variable using Dwarf two important pieces of information is needed, one of them is the type of the variable.
The other one is one or more Dwarf piece objects, these piece objects contains the whole or part of a objects value or location.
Thus the main problem here is to find out which Dwarf piece object corresponds to the correct part of the type and to also read the correct value from memory or registry if the Dwarf piece describes a location.

The type information for a variable is found in the attribute `DW_AT_type` and the most common die for variables is the die with the tag `DW_TAG_variable`.
The Dwarf piece objects needed comes from evaluating a expression that is most commonly found in the attribute `DW_AT_location` under the same die as the type information is found.


# Solution
My solution to the problem of finding out which piece corresponds to the correct part of the variable type is to go through the type tree in the order it is stored in memory until a base type(a number, string, boolean or address) is reached.
Then I use the fist piece in the list of pieces I have to find the value of the base type and I remove it from the list pieces if it is not needed anymore (I explain later how I know the piece is not needed anymore).
Then I repeat this until I have reached every base type in the tree, note that all the pieces should be used at this point and the algorithm should never run out of pieces.
This algorithm finds which piece corresponds to the correct part of the type because the path to the base type in the tree describes what value that piece represent.

To determine if a piece is not needed any more I use attribute `size_in_bits` of the piece.
If that attribute is empty then the piece must be the only piece and it represents the whole object, thus it should not be removed at any point. 
But if it has a value then I will subtract it with size of the base type, if the resulting `size_in_bits` value is zero then I remove it from the list of pieces.


This post is not finished, I am going to update it with more information.
