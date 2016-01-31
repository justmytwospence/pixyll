---
author: Spencer Boucher
layout: post
summary:
title: Super-power your keyboard with Karabiner
---

If you live inside Emacs (or the terminal), properly tuning your key bindings is
paramount. Unfortunately every keyboard is different and, even more than that,
pretty much *every* keyboard is now different that what was being used when the
oldest and most useful tools around were being developed.

## Vim ##

Vim becomes much more fluid when you have easy access to the `ESC` key. Because
Vim is modal, it uses modifier keys very rarely, which means every time you
change from inserting text to any other type of operation, you have to press
`ESC` first (although you *should* be spending most of your time in normal mode
and dropping into insert, rather than vice-versa). The only problem with this?
`ESC` is **so** very far away from the home row. Its hard to picture a more
inconvenient place to put the `ESC` key, at least on a Mac.

In console Vim, you can use `CTRL-[` because most terminal emulators recognize
this keystroke as identical to escape. This doesn't work in graphical environments,
however. Therefore, many people map their `Caps Lock` key to `ESC`, at least on Macs.
This is super convenient, because your left pinky requires almost no effort to move
over a centimeter. This has the added benefit of making the keyboard aesthetically
symmetric (at least in my opinion): left pinky goes out to escape, right pinky goes
out to enter. Yin and Yang, if you will.

## Emacs ##

Unlike Vim, Emacs is not is *not* inherently modal (although some might say that
Emacs just has dozens and dozens of smaller modalities instead of a few
all-encompassing ones), and therefore *does* make heavy use of modifier keys:
`CTRL`, `META`, etc. I personally try to stay away from `META` bindings as much
as possible.

By far the most common way of making `CTRL` more accessible on a Mac keyboard is
remapping `Caps Lock` to `CTRL`. After all, pretty much everyone agrees that the
size:usefulness ratio of `Caps Lock` is way off the rails[^2].

[^2]: The only people who can justify the size of `Caps Lock` are internet
    trolls, and you are definitely note an internet troll, right?

## Karabiner ##

I use *both* Vim and Emacs style key bindings, via the Evil-mode Emacs package.
package. Therefore I want to figure out how to remap my keyboard in such a way
as to make both `CTRL` *and* `ESC` easily accessible, and obviously I can't bind
them *both* to `Caps Lock`, right?

Wrong! [Karabiner][] allows us to be more nuanced about your key bindings. We can set
`Caps Lock` to be like `ESC` when we press it by itself (perfect for Vim), but act
like `CTRL` when we use it as a modifier key (perfect for Emacs)! Even better, we
can do the exact same thing for the `RET` key, making `CTRL` twice as accessible
and maintaining that oh so sweet symmetry.

I'm really happy with the setup, and would recommend it to anyone who uses Emacs and
Vim regularly.

[Karabiner]: https://pqrs.org/osx/karabiner/
