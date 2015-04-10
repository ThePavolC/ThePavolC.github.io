---
layout: post
title: Pivot Game I.
description: Making of Pivot game using Python and Kivy.
categories: pivot game
published: True
---

A while ago I found [Pivot][pivot-app] android game, and even though it is very simple, it is quiet good. Plus there are no ads.

So I decided that I try to make the same or similar game. For the programming language I decide to go with Python, and I am going to use [Kivy framework][kivy], even though I have never used it before.

Plus, I was wondering how much I can do in 2 days.

## Plan
1. Do a simple app in Kivy.
2. Make UI for Pivot game
3. Get the ball moving
4. Get the ball to collect the points

# Simple app

I checked out [Kivy's documentation][kivy-doc] and they have exactly what I was looking for. [Pong game tutorial][kivy-pong] was quick and simple but it had everything I needed. Well, everything to get me started.

# Pivot game UI

For the UI, I need to show some borders around my window, some text labels and ball.

So let start with showing the actual game window. This is starting `pivot.py` file. After running this we should be seeing black window, which is good, it means that our environment is set up correctly.

{% highlight python %}
from kivy.app import App
from kivy.uix.widget import Widget

class PivotGame(Widget):
    pass

class PivotApp(App):
    def build(self):
        return PivotGame()

if __name__ == '__main__':
    PivotApp().run()
{% endhighlight %}

To manage UI we will create `pivot.kv` file. In `pivot.kv` I will create some borders around the window. For that I will create rectangle on each side.
Then I want to have text label in center of window with some kind of instructions, and score label in corner.

{% highlight python %}
#:kivy 1.7.2

<PivotGame>:

    canvas:
        Rectangle:
            pos: 0, 0
            size: 10, self.height
        Rectangle:
            pos: self.width - 10, 0
            size: 10, self.height
        Rectangle:
            pos: 0, 0
            size: self.width, 10
        Rectangle:
            pos: 0, self.height - 10
            size: self.width, 10

    Label:
        font_size: 70
        text: "To Start Press Space"
        center: self.parent.center_x, self.parent.center_y

    Label:
        font_size: 20
        text: "Score: 0"
{% endhighlight %}

So far we should have something like this.

![Pivot Game Window](/assets/pivot-game/pivot1.png)

# Adding the ball

To add the ball I need to create new `PivotBall` class in `pivot.py` file which is extending Widget. Then I'll add `PivotBall` to `pivot.kv`file. I will also add `id` to items in `pivot.kv` file so I can use them from my Python code.

{% highlight python %}
#:kivy 1.7.2

<PivotGame>:
    ball: pivot_ball

    ...

    PivotBall:
        id: pivot_ball
        center: self.parent.center_x, self.parent.center_y

<PivotBall>:
    size: 50, 50

    canvas:
        Ellipse:
            pos: self.x, self.y
            size: self.size
{% endhighlight %}

And `pivot.py` file.

{% highlight python %}
...


class PivotBall(Widget):
    pass

...
{% endhighlight %}

Next thing I need to do is hide text label when I press spacebar and make ball moving. In order to do that I will need to bind key press with method in my Python code. To hide text I decided to just turn `opacity` to 0.

At this point I need to do different actions at different states of game. When game has just `started`, I want to see text label but I don't want to see ball. Then when pressed spacebar, I want to hide text label and I want to see moving ball, so `playing` state.

To move ball in circle, I use `Math.sin()` and `Math.cos()` to calculate x,y coordinates, where I pass angle as argument. So to keep ball moving in circle I'll just need to increase angle number, and if I want the ball to move in opposite direction, instead of adding to angle I subtract from it.

Here is `pivot.py`.

{% highlight python %}
from kivy.app import App
from kivy.uix.widget import Widget
from kivy.properties import NumericProperty, ObjectProperty
from kivy.properties import BooleanProperty, OptionProperty
from kivy.clock import Clock
from kivy.core.window import Window
from math import sin, cos

class PivotGame(Widget):

    # Getting references to widgets from kv file
    ball = ObjectProperty(None)
    menu = ObjectProperty(None)
    score_label = ObjectProperty(None)

    # Game states
    state = OptionProperty("started", options=["playing","killed"])
    # Score counter
    score = NumericProperty(-1)

    def update(self, dt):
    # This method hides/shows widgets on screen and
    # calls method that moves the ball
        if self.state == "started":
            self.menu.canvas.opacity = 1
            self.ball.canvas.opacity = 0
            self.score_label.canvas.opacity = 0
        elif self.state == "playing":
            self.menu.canvas.opacity = 0
            self.ball.canvas.opacity = 1
            self.score_label.canvas.opacity = 1
            self.ball.move(dt)

class PivotApp(App):

    def build(self):
        self.game = PivotGame()

        # schedule_interval() calls PivotGame.update() method 30 times in 1 sec
        Clock.schedule_interval(self.game.update, 1.0/30.0)

        # Binding key press with methods on_keyboard_down(), on_keyboard_up()
        self._keyboard = Window.request_keyboard(self._keyboard_closed, self)
        self._keyboard.bind(on_key_down = self._on_keyboard_down)
        self._keyboard.bind(on_key_up = self._on_keyboard_up)

        return self.game

    def _keyboard_closed(self):
        self._keyboard.unbind(on_key_down = self._on_keyboard_down)
        self._keyboard = None

    def _on_keyboard_down(self, keyboard, keycode, text, modifiers):
        # When spacebar is pressed either change state of game or turn the ball
        if keycode[1] == 'spacebar':
            if self.game.state == "started":
                # If game has just started, then change state
                self.game.state = "playing"
            elif self.game.state == "playing":
                # If game is already going on, then turn the ball
                self.game.ball.turn()

    def _on_keyboard_up(self, *args):
        pass

class PivotBall(Widget):

    # starting properties of ball
    angle = NumericProperty(0)
    r = NumericProperty(5.5)
    was_turn = BooleanProperty(True)

    def move(self, dt):
        # self.was_turn decides if we add or subtract angle, so the ball will
        # be turning in different directions
        if self.was_turn:
            self.angle += 0.1
        else:
            self.angle -= 0.1

        # calculating x,y coordinates
        self.x = self.x + sin(self.angle) * self.r
        self.y = self.y + cos(self.angle) * self.r


    def turn(self):
        # switch turning of the ball in opposite direction
        self.was_turn = not self.was_turn

if __name__ == '__main__':
    PivotApp().run()
{% endhighlight %}

Now we should be able to see moving ball in circles and having ball responding to spacebar press.

![Pivot Game starting and playing windows](/assets/pivot-game/pivot2.png)

# Collecting points and touching borders

Up to this point, the ball is freely moving around. That's nice, but little bit boring. So we will write method which will check if ball is touching any of the sides of the window. Then we should write method which will return ball to initial position.
Checking if ball touched the wall, should be in `update()` method so it is run each time the ball is moved.

Additional code for `PivotBall` class checking if the ball is touching border and also moving it back to center.

{% highlight python %}
...
border = NumericProperty(10)

def reset_position(self):
    self.x = self.parent.center_x - self.size[0]
    self.y = self.parent.center_y
    self.was_turn = BooleanProperty(True)
    self.angle = 0

def is_touching_border(self):
    if (self.x < self.border or
        self.x + self.size[0] > self.parent.width - self.border):
        return True
    elif (self.y < self.border or
        self.y + self.size[1] > self.parent.height - self.border):
        return True
    else:
        return False
...
{% endhighlight %}

Calling of these methods in `update()`.

{% highlight python %}
...
def update(self, dt):
    if self.ball.is_touching_border():
        self.state = "killed"
        self.ball.reset_position()
...
{% endhighlight %}

You can see that in last bit of the code, we change game state to `killed`. That causes the ball to stop in the center of screen after it touches the borders. So now we need define what happens when player is `killed`.

This is the behavior I want for each state:

* state `started`
    * show text label telling player to press spacebar
    * hide everything else
* state `playing`
    * show ball and start moving it
    * hide everything else except score counter
* state `killed`
    * show final score
    * hide everything else

There are two things we need to add, new text label with score which will be shown when player is killed and change actions when spacebar is pressed during different game states.

To add the text label, you can copy the code for already existing text label.

For the Python code, here is `update()` method where I hide and show different labels, depending on game state.

{% highlight python %}
def update(self, dt):
        if self.ball.is_touching_border():
            self.state = "killed"
            self.ball.reset_position()

        if self.state == "started":
            self.menu.canvas.opacity = 1
            self.ball.canvas.opacity = 0
            self.score_label.canvas.opacity = 0
            self.score_label2.canvas.opacity = 0

        elif self.state == "playing":
            self.menu.canvas.opacity = 0
            self.ball.canvas.opacity = 1
            self.score_label.canvas.opacity = 0
            self.score_label2.canvas.opacity = 1
            self.ball.move(dt)

        elif self.state == "killed":
            self.menu.canvas.opacity = 1
            self.ball.canvas.opacity = 0
            self.score_label.canvas.opacity = 1
            self.score_label2.canvas.opacity = 0
{% endhighlight %}

And here is `on_keyboard_down()` method.

{% highlight python %}
def _on_keyboard_down(self, keyboard, keycode, text, modifiers):
        if keycode[1] == 'spacebar':
            if self.game.state == "started":
                self.game.state = "playing"
            elif self.game.state == "killed":
                self.game.score = 0
                self.game.state = "playing"
            elif self.game.state == "playing":
                self.game.ball.turn()
{% endhighlight %}

Now it looks more like the actual game, but the very important part is still missing. And that it the target ball that we want to pick. For that I created new widget `PivotTarget`, which is very similar to our current ball.
Then we need to do two things, move the target randomly inside the window and check if we touched it. When our ball touches the target then we increase the score.

In `move_target()` we change x,y coordinates of target using `randint()` which [returns random integer from the range][randint].

And `got_point()` is checking if coordinates of ball are colliding with coordinates of target.

{% highlight python %}
def move_target(self):
        i = 10
        self.target.x = randint(self.target.size[0] + i,
                                self.parent.width - self.target.size[0] - i)
        self.target.y = randint(self.target.size[0] + i,
                                self.parent.height - self.target.size[0] - i)

    def got_point(self):
        if (self.ball.pos[0] <= self.target.pos[0] + self.target.size[0] and
            self.ball.pos[0] >= self.target.pos[0] - self.target.size[0] and
            self.ball.pos[1] <= self.target.pos[1] + self.target.size[1] and
            self.ball.pos[1] >= self.target.pos[1] - self.target.size[1] ):
            return True
{% endhighlight %}

# Result

My Pivot game looks currently something like this. There is a window, there is a ball moving and score is being counted. So I think I reached my goal for this 2 day "sprint".

For the next sprint I am planning to add moving squares as opponents, same as in actual [Pivot app][pivot-app].

So far I am quiet happy with [Kivy][kivy], it is easy enough if you have some experience with Python, but I will definitely need to have more look into it. Two days is just not enough and project is not that complicated.

[You can find source code for game in PivotGame repository][pivot-repo].

![Pivot Game at the end of 1st iteration](/assets/pivot-game/pivot3.png)

[pivot-app]: https://play.google.com/store/apps/details?id=com.NVS.pivot
[kivy]: http://kivy.org/#home
[kivy-doc]: http://kivy.org/docs/gettingstarted/intro.html
[kivy-pong]: http://kivy.org/docs/tutorials/pong.html
[randint]: https://docs.python.org/2/library/random.html#random.randint
[pivot-repo]: https://github.com/ThePavolC/PivotGame
