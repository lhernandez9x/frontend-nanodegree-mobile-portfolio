## Website Performance Optimization portfolio project

The objective of this project was to take a website and optimize it to achieve a rank of 90 on [Google's Page Speed](https://developers.google.com/speed/pagespeed/insights/) site. The second objective was to take the Pizza page *(`views/pizza.html`)*. and achieve a 60 fps on the background sliding pizzas and also to resize the pizzas in less than 5 ms. 

### Getting started

You can view the code in Github at this [repository](http://github.com/lhernandez9x/frontend-nanodegree-mobile-portfolio).

You can also view the live site [here](http://lhernandez9x.github.io/frontend-nanodegree-mobile-portfolio).

###Optimizations Made

1. The file was originally well bellow 60fps. in order to optimize I began with reducing and limiting DOM access. I moved DOM access like:
    ```var items = document.getElementsByClassName('mover');```
outside of for loops. This endured that DOM was accessed the least times as possible. Also in the above code I used **getElementsByClassName** instead of **querySelectorAll** that was originally used.

2. The sliding pizzas was originally written to load 200 pizzas for the entire site. You can only see a handful of these pizzas due to screen size. The following code was also changed:

  ```
document.addEventListener('DOMContentLoaded', function() {
  var cols = 8;
  var s = 256;
  for (var i = 0; i < 200; i++) {
    var elem = document.createElement('img');
    elem.className = 'mover';
    elem.src = "images/pizza.png";
    elem.style.height = "100px";
    elem.style.width = "73.333px";
    elem.basicLeft = (i % cols) * s;
    elem.style.top = (Math.floor(i / cols) * s) + 'px';
    document.querySelector("#movingPizzas1").appendChild(elem);
  }
  updatePositions();
}
  ```
I added the ability to get the users window size and create enough pizzas to only fill the screen, instead of creating 200 pizzas. I did this by changing a few lines:

  ```
  var cols = Math.floor(window.innerWidth / 200),
        rows = Math.floor(window.innerHeight / 230),
        s = 256,
        totalPizzas = cols * rows;
    for (var i = 0; i < totalPizzas; i++) {
        var elem = document.createElement('img');
        elem.className = 'mover';
        elem.src = "dist/images/pizza-sm.png";
        elem.style.height = "100px";
        elem.style.width = "73.333px";
        elem.basicLeft = (i % cols) * s;
        elem.style.top = (Math.floor(i / cols) * s) + 'px';
        document.getElementById("movingPizzas1").appendChild(elem);
    }
    updatePositions();
});
  ```

3. By running console.log on some of the calculations I found that some calculations were unnecessary. I changed these lines of code from:

  ```
  var items = document.querySelectorAll('.mover');
    for (var i = 0; i < items.length; i++) {
    var phase = Math.sin((document.body.scrollTop / 1250) + (i % 5));
    items[i].style.left = items[i].basicLeft + 100 * phase + 'px';
  }
  ```
  
  to:

  ```
    var items = document.getElementsByClassName('mover'),
        secondItems = [0, 1, 2, 3, 4],
        moduloItems = [secondItems.fill(secondItems.length + items.length - 1)],
        phaseOne = document.body.scrollTop / 1250;
    for (var i = 0, j = 0; i < items.length, j < secondItems.length; i++, j++) {
        var phase = Math.sin(phaseOne + j),
            slideLeft = -items[i].basicLeft + 1000 * phase + 'px';
        items[i].style.transform = 'translateX(' + slideLeft + ') translateZ(0)';
    }
  ```

4. I noticed that the last part of the phase calculation *`(i % 5)`* only created 5 numbers *[0, 1, 2, 3, 4]*. Instead of calcualating this everytime in the for loop, I hard coded these numbers and then created a way for these numbers to get repeated for the length of the pizza items. I found this code at [Stackoverflow.com](http://stackoverflow.com/questions/12503146/create-an-array-with-same-element-repeated-multiple-times-in-javascript).

  ```
  Array.prototype.fill = function(val) {
        var len = this.length;
        if (len < val) {
            for (var i = val - 1 - len; i >= 0; i--) {
                this[i + 1] = this[i % 1];
            }
        }
        return this;
    }
    ```

  These optimizations allowed for the moving pizzas to render at 60fps while the scroll event is happening.

5. The resizePizzas function made quite a bit of calculations, so it was pretty slow to change the size, avg. *300-400 ms*. I first started with console.log on all of the calculations.

  I noticed that most of these numbers were repeating and in some cases *(`var dx`)*, they were unnecessary. I eliminated all of the slow calculations and changed the resize calculation to use percentages. I started by eliminating these calculations. It made for a very clean function. The code that was removed was:

    ```
    function determineDx (elem, size) {
    var oldWidth = elem.offsetWidth;
    var windowWidth = document.querySelector("#randomPizzas").offsetWidth;
    var oldSize = oldWidth / windowWidth;
    var newSize = sizeSwitcher(size);
    
    var dx = (newSize - oldSize) * windowWidth;
    return dx;
    }
    ```
Also changed this block from:

    ```
    function changePizzaSizes(size) {
        var randPizzaCont = document.getElementsByClassName("randomPizzaContainer")
        for (var i = 0; i < randPizzaCont.length; i++) {
        var dx = determineDx(document.querySelectorAll(".randomPizzaContainer")[i], size);
        var newwidth = (document.querySelectorAll(".randomPizzaContainer")[i].offsetWidth + dx) + 'px';
        document.querySelectorAll(".randomPizzaContainer")[i].style.width = newwidth;
        }
    }
    ```
    
    to
    
    ```
    function changePizzaSizes(size) {
        var randPizzaCont = document.getElementsByClassName("randomPizzaContainer")
        for (var i = 0; i < randPizzaCont.length; i++) {
            //var dx = determineDx(randPizzaCont[i], size);
            //var newwidth = (randPizzaCont[i].scrollWidth + dx) + 'px';
            randPizzaCont[i].style.width = newSize + '%';
        }
    }
    ```
This change reduced the time it takes to change the size of pizza from *300-400 ms* to *0-1 ms*.
