---
title: "Image to ASCII for 16x2 Character Display Matrix"
date: 2025-07-14
draft: false
tags: ["Python", "Electronics", "Raspberry Pi"]
---

A while ago, I embarked on a personal project that combined my love for electronics and creative coding: building a large-scale display matrix out of 96 individual character LCD screens. The result was a massive 8x12 grid capable of displaying text, but I quickly realised I wanted to show more than just words—I wanted to display images.

The challenge? These are text-only displays. You can't just send them a JPEG. My solution was to go retro and use a classic form of digital art: **ASCII art**.

This blog post details my journey in developing a system to convert any image into beautiful, computationally efficient ASCII art, specifically designed to run on low-powered hardware like a Raspberry Pi.

![8x12 LCD matrix that I built and used for this project](project-images/lcd.jpg)

I will be making another post explaining how this project got started and how I got the hardware and firmware working for this.

### The Challenge: Images on a Text-Only Screen

Character displays are simple, affordable, and easy to control with a microcontroller or a Single Board Computer (SBC). However, their limitation is that they only understand characters. To display a picture, I needed to "translate" image data into a grid of ASCII characters.

My primary goals for this project were:
1.  **Design an efficient conversion system** that could handle various image formats.
2.  **Evaluate different conversion methods** to see what produced the best-looking results.
3.  **Optimise it for speed**, with a target of running in under a second on a Raspberry Pi.
4.  **Successfully display the art** on my custom-built LCD matrix.

### The Two Faces of ASCII Art: Tone vs. Structure

After some research, I found that ASCII art generation generally follows two main approaches:

*   **Tone-Based:** This method focuses on the brightness or "tone" of different parts of an image. Dark areas are replaced with dense, dark characters (like `@`, `#`, or `$`), while bright areas get sparse characters (like `.`, `'`, or just a space). It’s like painting with characters.
*   **Structure-Based:** This approach focuses on finding the edges and contours of an image. It uses characters that mimic lines and curves (`/`, `\`, `|`, `_`) to outline the shapes in the picture.

My plan was to implement and test both methods to see which was better suited for my display.

### Developing the Conversion Algorithms

The entire process was written in Python, using the powerful OpenCV library for image processing. The conversion process was broken down into three main stages: Image Sizing, Processing, and Character Selection.

#### 1. Sizing the Image
First, any input image had to be resized to match the resolution of my display (which, after some tweaking for aspect ratio, was effectively 640x360 pixels). I created two options: one that crops the image and one that adds blank space to prevent distortion, similar to the "letterboxing" you see on TVs.

#### 2. Processing the Image
This is where the two approaches diverge.
*   For the **tone-based** method, the process was simple: just convert the image to greyscale.
*   For the **structure-based** method, I needed to detect edges. I used the Canny edge detection algorithm, a feature well-supported in OpenCV. This method is great at identifying sharp changes in intensity, effectively outlining the features in an image and producing a clean, black-and-white result of just the lines.

#### 3. Choosing the Characters
This was the most critical step. I created four different methods to see what worked best:
1.  **Average Luminance (Tone-Based):** This was my fastest method. It divides the image into small sections (one for each character on the final display) and calculates the average brightness of each section. It then maps that brightness value to a character from a predefined list, from darkest to brightest (`@%#*+=-:. `).
2.  **Structure-Based (Large Character Set):** This method compares each section of the processed edge-detected image directly against *every single character* available on the LCD screen (64 of them). It picks the character that has the most pixel-for-pixel overlap. While thorough, it's incredibly inefficient.
3.  **Structure-Based (Small Character Set):** To optimise the above, I analysed which characters were chosen most frequently from a huge random dataset. It turned out only a handful of characters did most of the work. I created a much smaller, curated list of the 16 most effective characters (`$#%&gd()*|_-.=`). This dramatically reduced the number of comparisons needed.
4.  **Machine Learning Model (An Experiment):** I also experimented with training a Random Forest machine learning model to predict the best character. While a fun idea, the models were either too inaccurate or too large and slow to run on a Raspberry Pi. This approach was ultimately a dead end for this project.

### Putting the Art to the Test

To figure out which method produced the "best" art, I couldn't just rely on my own eyes. I needed data.
First, I ran a survey with 50 respondents, showing them the output of each method for five different test images and asking them to choose the best one.

{{< center >}}
*Here are the 5 test images chosen for the survey.*
{{< /center >}}
{{< image-gallery gallery_dir="/project-images/lcd-gallery" >}}

 The results were overwhelming:
*   The **Average Luminance** method was the clear favourite, chosen **62% of the time**. It proved to be the most versatile and consistently produced pleasing results.
*   The structure-based methods that relied on Canny edge detection were the least popular, collectively chosen only 12% of the time. The edge detection, while good in theory, often created results that were too sparse or missed important details.

However, the survey also showed that the "best" method could depend on the image. For the simple "circle" image, the structure-based method was the clear winner because the image is *all* structure. For more complex photographic images like "Lenna" or the "lane", the nuance of the tone-based approach was heavily preferred.

![LCD matrix output of the car image using the popular average luminance method](project-images/car_onscreen.jpg)

### Performance: Can it Run in Real-Time?

The final test was speed. Running the code on an Intel Core i5 PC, every method completed in under a second. But the real test was on the SBCs.
*   **Raspberry Pi 3B:** The fast Average Luminance method took about **0.6 seconds**. The large structure-based method took a sluggish **17 seconds**, but the optimised small structure-based method was a more respectable **3.5 seconds**.
*   **OKdo Rock 4C+ (a more powerful SBC):** The Average Luminance method was even faster here, at just **0.14 seconds**. This is quick enough for potential real-time video conversion at around 7 frames per second (Which is close to the refresh speed of the character displays)!

![Comparison of ASCII conversion results for the PC, Raspberry Pi 3B and OKdo Rock 4C+](project-images/EachMethodSpeed.png)

### Conclusion

This project was a fantastic journey into the practicalities of image processing on resource-constrained hardware. I successfully created a system that can generate visually appealing ASCII art and display it on a unique, custom-built screen.

The key takeaway was that while complex approaches like structure detection and machine learning are powerful, sometimes a simpler, more elegant solution like the **average luminance method** provides the best balance of speed, efficiency, and quality. It proved to be a robust and versatile technique that brought my giant LCD matrix to life.