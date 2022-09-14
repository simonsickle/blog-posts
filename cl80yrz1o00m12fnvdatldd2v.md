## Drawing QR Codes in Jetpack Compose

*👋 Hey there! I'm Simon, a senior Android engineer at [Block](https://block.xyz). I have been hacking around on Android roughly since its creation: from kernel mods to app creation, I have done a little bit of it all. I also dabble in other projects, like home automation and helpful utilities to make my life a little bit easier.*


*If you have questions or comments, you can reach me on [Twitter](https://twitter.com/ssickle42)!*

## I want my QR code and I want it *now*

> "Once a new technology rolls over you, if you’re not part of the steamroller, you’re part of the road"
>
> &mdash; <cite>Stewart Brand</cite>


For a recent project, I needed to show a QR code to allow a user to scan and retrieve data from another device. While there are *tons* of examples of how to do exactly this in Android, there are not many examples of how to do this within Jetpack Compose. As any engineer would do, I decided to write out the pros and cons of 3 approaches I had considered:
1. Using a static asset inside of my application **(Easy)**
2. Using an API to request a QR code image **(Moderate)**
3. Generate the QR code on device and draw it with **(Hard)**

#### Static Asset

Using a static asset seems like the easiest solution here and is certainly worth a mention. Why write a ton of code when you can just drop an SVG into your app's resources and display it like you would any other image? The drawbacks here are that you will be increasing your application size and preventing the flexibility of updating this code's contents on the fly. 

#### API / Network Call

Using a network call is certainly another option here: we already have an endpoint that can generate and then return a PNG of a QR code to my specification. The upside is flexibility in changes to the design of this code and it's contents, but at the expense of needing to make a network call and for the device to be online.

#### Generation

This option *almost* scared me away as I have *never* worked with the Android Canvas APIs before but had heard horrors from fellow engineers about the quirks and pain of working with the Canvas. This all changes with Jetpack Compose, however! Google took the time to make improvements to these Canvas APIs - even further, anything that supports Skia could use the same code to draw with. This is a huge win for Compose Multiplatform, which allows writing the same UI code to run across multiple targets. This generation would be much more performant than a network call, keep the APK size down, and would be "future proof" if I needed to update the contents of the QR code. *This is my winner.*


## Hand me my paintbrush

At a glance, I am going to need to do the following:
1. Create a project in Android Studio with support for Jetpack Compose
2. Create an activity to show my QR code right in the middle
3. Create a Composable function that hosts a `Canvas` Composable
4. Use the `DrawScope` to draw the QR code data onto the canvas

### Drawing a QR code

As you may have noticed, step 4 is pretty vague... How does one draw a QR code? What even makes these magical bits of color work? Let's dive in so we can determine how one should be drawn.

#### The finders

If you don't know what the thing you are searching for looks like, how will you find it? When the computer processes the frames of data coming from your phone's camera to look for a QR code, it is looking for a distinct spacing of 3 squares. These 3 squares are known as the finder pattern. Most QR scanners also require a bit of a quiet zone around these finder patterns where there is *nothing* else - around 16 pixels usually works fine.

#### The data bits

Now that you have found what you are looking for, you can figure out what it means to you. In this case, the computer will read the data in form of columns and rows and then use an algorithm to decode the meaning of these little bits of data. This data, luckily, does not need to be perfect as a certain level of data correction is built in (and is configurable in most QR encoders).


### Generation of a barcode

To generate the instructions we need to be able to draw a QR code on the canvas, we are going to need to use a library called ZXing - the tried and true java library for making barcodes of many different types. You may add it to your project by adding the following dependency.

```kotlin 
implementation("com.google.zxing:core:3.5.0")
```

Now to generate a QR code in a format we can use, you need to call the ZXing Encoder with the information you wish to define the code (see `QrCode.createQrCode` in sample). Next we draw the finder patterns - you can see  `QrCode.kt` and `QrCodeFinderDrawing.kt` for examples of how to do this; the overview is that you want to determine the position of each of these markers by using a size ratio of the generated matrix's width and height.

Finally, you will need to draw the data into the canvas. To do this, you will need to look through the ByteMatrix and check to see if a bit is enabled or disabled at that location. If it is, draw a simple rectangle on the canvas at the appropriate offset with a size matching the row and column. (see `QrCodeDataDrawing.kt`)


### Lessons learned
This project was by no means easy, but I learned a lot along the way. Drawing may seem intimidating at first, but just remember that you are essentially telling the computer to do what you already know how to do - move a pen/painter across a canvas. If something doesn't work, break down the parts into small consumable pieces which can be easily achieved.

### View My Code

You can take a look at my demo app, embedded below, and copy any code as you please! See something that could be improved upon? Feel free to submit a PR as well

%[https://github.com/simonsickle/ComposableBarcode/tree/main]