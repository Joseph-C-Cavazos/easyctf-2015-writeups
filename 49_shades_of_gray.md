# 49 Shades of Gray  (125)

## Problem

We only have 49 shades of gray D:

\#000000 to \#F5F5F5... there's one shade missing! Find the hex value of the missing shade. Pound sign optional.

[Image](images/shades.png)

## Hint

How can we tell which color is which?

## Writeup

This challenge was another fun one.
Just by looking at the problem, I immediately defaulted to ImageMagick to be my savior and sure enough, it was.
Since we were seeking which shade was missing, my first thought was to take a look at the histogram of the image.  

```sh
bill@windows ~/easy $ convert shades.png -format %c histogram:info:-
      5420: (  0,  0,  0) #000000 black
      5425: (  5,  5,  5) #050505 grey2
      5405: ( 10, 10, 10) #0A0A0A grey4
      5438: ( 15, 15, 15) #0F0F0F grey6
      5286: ( 20, 20, 20) #141414 grey8
      5266: ( 25, 25, 25) #191919 srgb(25,25,25)
      5284: ( 30, 30, 30) #1E1E1E srgb(30,30,30)
      5316: ( 35, 35, 35) #232323 srgb(35,35,35)
      5263: ( 40, 40, 40) #282828 srgb(40,40,40)
      5318: ( 45, 45, 45) #2D2D2D srgb(45,45,45)
      5301: ( 50, 50, 50) #323232 srgb(50,50,50)
      5269: ( 55, 55, 55) #373737 srgb(55,55,55)
      5366: ( 60, 60, 60) #3C3C3C srgb(60,60,60)
      5306: ( 65, 65, 65) #414141 srgb(65,65,65)
      5362: ( 70, 70, 70) #464646 srgb(70,70,70)
      5408: ( 75, 75, 75) #4B4B4B srgb(75,75,75)
      5406: ( 85, 85, 85) #555555 srgb(85,85,85)
      5421: ( 90, 90, 90) #5A5A5A srgb(90,90,90)
      5396: ( 95, 95, 95) #5F5F5F srgb(95,95,95)
      5321: (100,100,100) #646464 srgb(100,100,100)
      5389: (105,105,105) #696969 DimGray
      5421: (110,110,110) #6E6E6E grey43
      5401: (115,115,115) #737373 grey45
      5317: (120,120,120) #787878 grey47
      5280: (125,125,125) #7D7D7D grey49
      5310: (130,130,130) #828282 grey51
      5418: (135,135,135) #878787 grey53
      5390: (140,140,140) #8C8C8C grey55
      5354: (145,145,145) #919191 grey57
      5210: (150,150,150) #969696 grey59
      5454: (155,155,155) #9B9B9B srgb(155,155,155)
      5226: (160,160,160) #A0A0A0 srgb(160,160,160)
      5342: (165,165,165) #A5A5A5 srgb(165,165,165)
      5375: (170,170,170) #AAAAAA srgb(170,170,170)
      5375: (175,175,175) #AFAFAF srgb(175,175,175)
      5374: (180,180,180) #B4B4B4 srgb(180,180,180)
      5326: (185,185,185) #B9B9B9 srgb(185,185,185)
      5472: (190,190,190) #BEBEBE grey
      5347: (195,195,195) #C3C3C3 srgb(195,195,195)
      5354: (200,200,200) #C8C8C8 srgb(200,200,200)
      5401: (205,205,205) #CDCDCD srgb(205,205,205)
      5223: (210,210,210) #D2D2D2 srgb(210,210,210)
      5346: (215,215,215) #D7D7D7 srgb(215,215,215)
      5259: (220,220,220) #DCDCDC gainsboro
      5413: (225,225,225) #E1E1E1 srgb(225,225,225)
      5320: (230,230,230) #E6E6E6 srgb(230,230,230)
      5381: (235,235,235) #EBEBEB grey92
      5368: (240,240,240) #F0F0F0 grey94
      5321: (245,245,245) #F5F5F5 grey96

```  

That looks what we expected.
To be certain, let's make sure that these are only the 49 expected shades.  

```sh
bill@windows ~/easy $ convert shades.png -format %c histogram:info:- | sed '/^$/d' | wc -l
49
```  

Using sed to delete that last empty line, we used wc with the -l option to tell us the number of lines AKA the number of shades.
With that settled, now we need to find which one is missing.
First things first, we need to strip out all the extra info.
That can be done with a pipe to sed.  

```sh
bill@windows ~/easy $ convert shades.png -format %c histogram:info:- \
> | sed -n 's/^.*#\([^ ][^ ]*\) .*$/\1/p'
000000
050505
0A0A0A
0F0F0F
141414
191919
1E1E1E
232323
282828
2D2D2D
323232
373737
3C3C3C
414141
464646
4B4B4B
555555
5A5A5A
5F5F5F
646464
696969
6E6E6E
737373
787878
7D7D7D
828282
878787
8C8C8C
919191
969696
9B9B9B
A0A0A0
A5A5A5
AAAAAA
AFAFAF
B4B4B4
B9B9B9
BEBEBE
C3C3C3
C8C8C8
CDCDCD
D2D2D2
D7D7D7
DCDCDC
E1E1E1
E6E6E6
EBEBEB
F0F0F0
F5F5F5
```  

Now that we've got everything looking pretty, let's go ahead and shove it into a script.
I'm a Perl fan, so into Perl it went.
Since the histogram had already sorted the colors and they were also the same hexadecimal number repeated three times, we can be a little lazy.
Looking through the head and tail of the output from the histogram, we can also see that the shades are in increments of 5.
With this, the basic goal of the script is to have it run alongside the output we have so far and point out any difference.
If all assumptions are correct, this should highlight the missing shade of gray.  

```sh
bill@windows ~/easy $ convert shades.png -format %c histogram:info:- \
> | sed -n 's/^.*#\([^ ][^ ]*\) .*$/\1/p' \
> | perl -e '$i = 0;
> while (<>)
> {
> $tmp = sprintf "%02X", $i;
> $i += 5;
> die $tmp x 3 if not /$tmp/;
> }'
505050 at -e line 1, <> line 17.
```  

Obviously not a very pretty solution, but it works for all that we needed.  

Flag: `505050`
