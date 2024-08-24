---
layout: post
title: Making a Benchtop Power Supply
date: 2024-08-23 20:00:00
categories: ["Electronnic","PowerSupply","DIY"]
---
##### All links are affiliate links
## 🎯Goal
For a long time now, I wanted a lab power supply to power prototypes and help me repair electronics devices, but pre-builts I found were either too expensive for my usage or not portable. So my DIY power supply must meet personal requirements.

#### 📋Requirements:
- Being portable
- Being powered by the same cable as my soldering iron
- Have a current limiter
- Easy to use
- Cheap

## 🔍 Identify The Right Parts
To meet the need to be inexpensive, I mostly search on Aliexpress for the different parts of the power supply.
For the main part of the power supply, I'll use this [SK35H 35W buck boost converter](https://www.aliexpress.com/item/1005007004741793.html) (10.34€).
To power it, I'll use this [USB C to DC5525](https://www.aliexpress.com/item/1005004744935148.html) (3.11€) that I already use to power my TS100 soldering iron. Since it uses the PowerDelivery protocol, it outputs a 12V 36W current which is perfect for the power supply.
On the power supply side, I'll use these [DC5525 sockets](https://www.aliexpress.com/item/1005007211080553.html) (2.24€ for 10pcs).
To connect standard 4mm banana cables, I bought this [pack of 30 banana sockets on Amazon](https://amzn.to/3ANy3q4) (11€)(it can be useful in the future). On Amazon, I also bought these [regulation modules](https://amzn.to/4g0DeDh) (13€ for 10pcs) to power a 5V fan (its dimensions are 30x30x10mm) from the 12V of the power input (I used one from an old Raspberry Pi, so I don't have a link for it).

## 🛠️ Building a Case
Since I don't want to move a bunch of wires, I've designed a case with Fusion 360.
![fusion360 render picture](https://github.com/Chachigo/chachigo.github.io/blob/main/all_collections/_posts/img/psuCase.png?raw=true)  
The STL files if you want to print it yourself ([Thingiverse](https://www.thingiverse.com/thing:6741616)):
- [Front](https://github.com/Chachigo/chachigo.github.io/blob/main/all_collections/_posts/files/Front_Supply_Case.stl)
- [Back](https://github.com/Chachigo/chachigo.github.io/blob/main/all_collections/_posts/files/Back_Supply_Case.stl)


## 🏆 Finished Product
![final product photo](https://github.com/Chachigo/chachigo.github.io/blob/main/all_collections/_posts/img/psuPhoto.jpg?raw=true)  
![final product photo Back](https://github.com/Chachigo/chachigo.github.io/blob/main/all_collections/_posts/img/psuBackPhoto.jpg?raw=true)