---
title: A realization of Perlin Noise - By Yunxiang Zhang
date: 2016-12-11 18:17:23
tags: 
---

# About Perlin Noise
Perlin noise is a type of gradient noise developed by Ken Perlin in 1983 as a result of his frustration with the "machine-like" look of computer graphics at the time. (From Wikipedia)  

In general, the function of perlin noise is to make a noise which looks more natural. It is often used in games and visual media such as the generation of terrain and fires.  

The realization of perlin noise consists of the following steps:  
1. Generate a random value and gradient for each integral point.  
2. For points between the points, use interpolation to generate the value.  
3. Using ease curve to make the result looks more natural.  

<!-- more -->

# Code
```html
<!DOCTYPE html>
<html lang="en">
<body>
    <canvas id="canvas" width="1000" height="1000" style="float:left"></canvas>
    <script type="text/javascript">
```
```javascript
        var ctx = canvas.getContext('2d');
        var myImageData = ctx.createImageData(canvas.width, canvas.height);
        var gradientArray, valueArray;
        initPerlin([100, 100]);
        var myImageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
        for (var i = 0; i < canvas.width; i++)
            for (var j = 0; j < canvas.height; j++) {
                var color = (fractal( 4,[i / 200, j / 200]) + 1) / 2 * 255;
                for (var k = 0; k < 4; k++)
                    myImageData.data[i * canvas.height * 4 + j * 4 + k] = (k == 3 ? 255 : color);
            }
        ctx.putImageData(myImageData, 0, 0);
        function pointCopy(point) {
            var newPoint = new Array();
            for (var i = 0; i < point.length; i++) newPoint[i] = point[i];
            return newPoint;
        }
        function getValue(point, array) {
            return function _getValue(dimension, point, array) {
                return dimension == point.length ?
                    array : _getValue(dimension + 1, point, array[point[dimension] % array.length]);
            }(0, point, array);
        }
        function initArray(size) {
            return function _initArray(dimension, size) {
                if (dimension == size.length) return Math.random();
                var array = new Array();
                for (var i = 0; i < size[dimension]; i++) array[i] = _initArray(dimension + 1, size);
                return array;
            }(0, size);
        }
        function initGradient(size) {
            var gradient_size = pointCopy(size);
            gradient_size[size.length] = size.length;
            var array = initArray(gradient_size);
            (function format(dimension, array) {
                if (dimension == size.length) {
                    for (var i = 0; i < size.length; i++) array[i] = array[i] * 2 - 1;
                    normalize(array);
                }
                else for (var i = 0; i < size[dimension]; i++) format(dimension + 1, array[i]);
            })(0, array);
            return array;
        }
        function initPerlin(size) {
            gradientArray = initGradient(size);
            valueArray = initArray(size);
        }
        function s_curve(t) { return t * t * t * (t * (t * 6 - 15) + 10); }
        function lerp(t, a, b) { return a + t * (b - a); }
        function fractal(step, point) {
            var newPoint = pointCopy(point);
            var accumulate = point[0];
            for (var j = 0; j < step; j++) {
                accumulate = accumulate + Math.abs(perlinNoise(newPoint)) / Math.pow(2, j);
                for (var i = 0; i < point.length; i++) newPoint[i] = 2 * newPoint[i];
            }
            return Math.sin(accumulate*Math.PI/2);
        }
        function normalize(vector) {
            var s = 0;
            for (var i = 0; i < vector.length; i++) s += vector[i] * vector[i];
            s = Math.sqrt(s);
            for (var i = 0; i < vector.length; i++) vector[i] = vector[i] / s;
        }
        function perlinNoise(point) {
            var floor = new Array(), ceil = new Array(), s_curve_value = new Array();
            for (var i = 0; i < point.length; i++) {
                floor[i] = Math.floor(point[i]);
                ceil[i] = Math.ceil(point[i]);
                s_curve_value[i] = s_curve(point[i] - floor[i]);
            }
            return function dimensionLerp(dimension, pointCorner) {
                var pointFloor = pointCopy(pointCorner);
                pointFloor[dimension - 1] = floor[dimension - 1];
                var pointCeil = pointCopy(pointCorner);
                pointCeil[dimension - 1] = ceil[dimension - 1];
                if (dimension == 1) {
                    var gradientFloor = getValue(pointFloor, gradientArray);
                    var gradientCeil = getValue(pointCeil, gradientArray);
                    var u = getValue(pointFloor, valueArray) * 2 - 1;
                    var v = getValue(pointCeil, valueArray) * 2 - 1;
                    for(var i=0; i<point.length; i++) u=u+(point[i]-pointFloor[i])*gradientFloor[i]*2;
                    for(var i=0; i<point.length; i++) v=v+(point[i]-pointCeil[i])*gradientCeil[i]*2;
                    return lerp(s_curve_value[0], u, v);
                }
                return lerp(s_curve_value[dimension - 1],
                    dimensionLerp(dimension - 1, pointFloor),
                    dimensionLerp(dimension - 1, pointCeil));
            }(point.length, point);
        }
```
```html
    </script>
</body>
</html>
```
# What I’ve done
I choose HTML + JavaScript to write the code so you can easily see what is the result by copying the code into an html file and see the result.
Instead of the array perlin used in his essay, I generate the value and gradient in a totally random way. This may take more spaces, but the result is better and looks more natural.
Also, I use a recursion way to write the code so it can be extended to any dimension, not only 2D, you can generate a 3D, 4D… perlin noise. Of course, I generate a 2D perlin noise as the example for the limitation of 100 lines.

# Result
1. The original perlin noise looks like below  
https://accrt.github.io/personalHTML/examples/perlin/test_origin.html:
![figure 1](http://ohvmg8dgt.bkt.clouddn.com/%E5%9B%BE%E7%89%871.png)

2. when we accumulate the original result of perlin noise, which is a fractal, we can see the result below  
https://accrt.github.io/personalHTML/examples/perlin/test_fractral.html:
![figure 2](http://ohvmg8dgt.bkt.clouddn.com/%E5%9B%BE%E7%89%872.png)
https://accrt.github.io/personalHTML/examples/perlin/test_fractral2.html:
![figure 7](http://ohvmg8dgt.bkt.clouddn.com/perlin_fractral2.png)

3. If using sin to the accumulation result, that will be  
https://accrt.github.io/personalHTML/examples/perlin/test_sin.html:
![figure 3](http://ohvmg8dgt.bkt.clouddn.com/%E5%9B%BE%E7%89%873.png)
This is just what I wrote in the code.

4. An easy way to make the picture looks like fire is using reflex. You can reflex the black to white color to the color strip showed below.  
![figure 4](http://ohvmg8dgt.bkt.clouddn.com/%E5%9B%BE%E7%89%876.png)
https://accrt.github.io/personalHTML/examples/perlin/test_sin_fire.html: 
![figure 5](http://ohvmg8dgt.bkt.clouddn.com/%E5%9B%BE%E7%89%874.png)
https://accrt.github.io/personalHTML/examples/perlin/test_fractral2_fire.html: 
![figure 8](http://ohvmg8dgt.bkt.clouddn.com/perlin_fractral2_fire.png)

5. Many wood texture are generated by perlin noise as well 
https://accrt.github.io/personalHTML/examples/perlin/test_wood.html:
![figure 6](http://ohvmg8dgt.bkt.clouddn.com/%E5%9B%BE%E7%89%875.png)

6. Of course, when it comes to a 3D perlin noise you can use one dimension as time line, to make the texture changes with time  
https://accrt.github.io/personalHTML/examples/perlin/test_wood_changes.html.

# Some other things
I have put this page on my friend’s personal blog. You can see it on
[A realization of Perlin Noise](https://yanjiasen4.github.io/2016/12/11/A-realization-of-Perlin-Noise)
# Source Pages
http://flafla2.github.io/2014/08/09/perlinnoise.html  
https://en.wikipedia.org/wiki/Perlin_noise