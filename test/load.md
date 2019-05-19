tkinter 3.0 multiple keyboard events together
janislaw wicijowski at gmail.com 
Sun Dec 28 21:34:04 CET 2008
Previous message (by thread): tkinter 3.0 multiple keyboard events together
Next message (by thread): tkinter 3.0 multiple keyboard events together
Messages sorted by: [ date ] [ thread ] [ subject ] [ author ]
On 28 Gru, 09:43, Pavel Kosina <g... at post.cz> wrote:
> well, I am working on a tutorial for youngster (thats why i need to stay
> the code as easy as possible). In this game you are hunted by robots. I
> could use key"7" on numeric keypad for left-up moving but seems to me,
> that "4"+"8" is much more standard for them.
>
> > Hmmm. Maybe you'd like to hook into Tkinter event loop, i.e. by custom
> > events if you don't like periodical polling.
>
> No, that would be even more difficult. I already have a code that use
> your idea:
>
> from Tkinter import *
> root = Tk()
>
> pressedKeys=set()
>
> def onKeyPress(event):
>     pressedKeys.add(event.keysym)
>
> def onKeyRelease(event):
>     pressedKeys.remove(event.keysym)
>
> def move():
>     print list(pressedKeys)
>     root.after(100,move)
>
> root.bind("<KeyPress>", onKeyPress)
> root.bind("<KeyRelease>", onKeyRelease)
> root.after(100,move)
> root.mainloop()
>
> well, I thought that gui´s have such a problem already built-in - so
> that i am not pressed to code it. You know, its not exactly about me -
> if I do it for myself, for my program, that there is no problem, but I
> need to explained it to begginners ..... And I do not want, as might be
> more usual, do my module, that would cover this "insanity" (look at it
> with 13-years old boy eyes ;-) ). Do you like to say me, that this is
> not a standard "function" neither in tkinter, not say gtk or the others,
> too?

Ok, I get it then! Your goal is very humble :)

In the piece of code we have came together to, there is a
functionality you already want - the pressedKeys. I suppose, that
you'd like to hide from the further implementors (children). You can
write another module, which could contain this piece of code and could
be imported by kids to have the desired features.

Meaning only getting this list of keys.

But what you write here:

> I would expect something like this:
>
> def onKeyTouch(event):
>     print (event.keysymAll)
>
> root.bind("<KeyPress>", onKeyTouch)

...is a bit misleading. There is no such thing as simultaneous events
- for that the event dispatcher loop is designed - to simplyfy all
usual
stuff with parallel paradigsm such as interrupts, thread safety
etc. An event is in my eyes a response to a short action, let's call
it
atomic. There's no need to pretend, that clicking a few keys
simultanously is sth immediate and the software could by any chance
guess that a few keys were actually clicked. Even the keyboard driver
does some sequential scanning on the keys, and the key codes are then
send to the board sequentially as well. Try to run this code:

#----------------
#Tkinter multiple keypress
#Answer http://groups.google.com/group/comp.lang.python/browse_thread/thread/88788c3b0b0d51d1/c4b7efe2d71bda93
import Tkinter
import pprint
import time

tk = Tkinter.Tk()
f = Tkinter.Frame(tk, width=100, height=100)
msg = Tkinter.StringVar()
msg.set('Hello')
l = Tkinter.Label(f,  textvariable=msg)
l.pack()
f.pack()

keys = set()

lastTime = None

def keyPressHandler(event):
    keys.add(event.char)
    global lastTime
    now = time.time()
    if lastTime:
        difference = now - lastTime
        if difference < 0.1: #seconds
            print difference, 's'
    lastTime = now
    display()

def keyReleaseHandler(event):
    keys.remove(event.char)
    display()

def display():
    msg.set(str(keys))

f.bind_all('<KeyPress>', keyPressHandler)
f.bind_all('<KeyRelease>', keyReleaseHandler)

f.mainloop()
#--------------------------------


... and find out, how 'simultaneous' they are. For me it's 0.0,
meaning that the event handlers were executed one after another, but
sometimes there is sth like 0.0309998989105 s - proabably an USB
latency, or an operating system context switch - hard to tell.

If you'd like to have two key clicks to generate a single event, you'd
have to write some timeout code (threading.Timer or tk stuff is handy)
to check how 'simultaneous' they actually were. Search web for events
in <<doubled>> inequality signs if you really want this.

I don't know gtk, but hey, pygame has pygame.key.get_pressed if you'd
like to switch:

http://www.pygame.org/docs/ref/key.html

Have luck
Jan

Previous message (by thread): tkinter 3.0 multiple keyboard events together
Next message (by thread): tkinter 3.0 multiple keyboard events together
Messages sorted by: [ date ] [ thread ] [ subject ] [ author ]
More information about the Python-list mailing list
