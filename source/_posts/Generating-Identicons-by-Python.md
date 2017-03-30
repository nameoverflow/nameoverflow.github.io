---
title: Generating Identicons by Python
date: 2015-03-08 04:53:35
tags:
  - python
---

If a github's user has not upload an avatar, github will use gvatar or generate an image basing on the email(gvatar also offers this service).This kind of image called identicon.

This code does the similar thing in a simple way,using the md5 of a string to generate an identicon.

\#英语渣表示写这段话好吃力啊= =

example:

input: hello world

generated:

![](https://dn-hcyue.qbox.me/img/bbe01eeed093cb22bb8f5acdc3.png)

<!--more-->

```python
from hashlib import md5
from PIL import Image, ImageDraw

IMAGE_SIZE = 5
SQUARE_SIZE = 10

class newIcon(object):
    """generate new identicon"""
    def __init__(self, str, bgc = '#fff', size = IMAGE_SIZE):
        self.h = size
        if self.h % 2 == 0:
            self.w = self.h / 2
        else:
            self.w = (self.h + 1) / 2
        self.hash = self.gMD5(str)
        self.image = Image.new('RGB', (IMAGE_SIZE*SQUARE_SIZE, IMAGE_SIZE*SQUARE_SIZE), bgc)
        self.draw = ImageDraw.Draw(self.image)

    def gMD5(self, str):
        """get a md5 hash"""
        hash = md5()
        hash.update(str)
        return hash.hexdigest()

    def generate(self):
        """caculate and generate the image"""
        #usng the first six hexadecimal num to decide the color
        color = tuple([int(x, 16) for x in (self.hash[0:2], self.hash[2:4], self.hash[4:6])])
        print(color)
        self.hash = self.hash[6:]
        hash_list = [int(x, 16) for x in self.hash]

        for x in range(int(self.w)):
            for y in range(int(self.h)):
                if hash_list[y * IMAGE_SIZE + x] & 1:
                    #Draw the left side
                    self.draw.rectangle(    
                        (x*SQUARE_SIZE, y*SQUARE_SIZE, (x+1)*SQUARE_SIZE, (y+1)*SQUARE_SIZE),
                        fill=color,
                        outline=color)
                    #Draw the mirror side
                    self.draw.rectangle(    
                        ((self.h-x)*SQUARE_SIZE, y*SQUARE_SIZE, (self.h-(x+1))*SQUARE_SIZE, (y+1)*SQUARE_SIZE),
                        fill=color,
                        outline=color)

    def save(self):
        """save and show the image"""
        self.generate()
        self.image.save(self.hash+'.png', 'PNG')

pic = newIcon(input("the string:").encode('utf8'))
pic.save()```