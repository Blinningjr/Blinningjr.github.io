---
layout: post
title:  "Dwarf Location Pieces"
date:   2021-03-29 
categories: Dwarf Format
---
In the Dwarf format the location of the variables value is stored in one or more pieces.
A piece can have three attributes, the most important of these is the location attribute which holds the value of the piece or where the value is stored.
Then there can also be a attribute for the size of the data in this piece.
Lastly there can be a offset attribute that describes were to start reading the value if the location attribute is a address or register.

The piece attribute that describes the size of the value in a piece is very important for knowing which piece corresponds to a certain value in an object.
If that attribute is empty then the piece must be the only piece and it represents the whole object, otherwise the piece represents a part of the object.
The easies why I found to evaluate pieces that only represents a part of the object is to read the base types and subtract there size from the piece attribute.
Then when the size of the piece is zero I move on to the next piece.

This post is not finished, I am going to update it with more information on how to evaluate the value of an object.
