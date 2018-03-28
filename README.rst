*****
gontt
*****

A simple Gantt-like charting tool written in Go

how should it work?
===================

``gontt`` should output something like the following:

::

   chart generated from ./gontt.json

   001 put the dog outside ⊏⊐╮
   002 brush the dog         ├⊏═⊐──╮
   004 sweep the house       ╰⊏══⊐┬┤
   003 take out the trash         │╰⊏⊐
   005 vacuum                     ╰⊏═⊐

The model of a **Task**, then, is currently:

.. code-block:: go

   type Task struct {
      id          int
      name        string
      duration    int
      downstream  []Task
      upstream    []Task
   }

``gontt init`` will generate a stub of a json file in the local directory.

``gontt -i some/other/file.json`` will use an explicit input file location.

more thoughts
=============

CLI input
---------

Something like the following commands:

.. code-block:: console

   $ gontt add "brush the dog" 1
   Using gonttfile ./gontt.json
   New task "brush the dog" created, with duration 1

   001 put the dog outside ⊏⊐╮
   002 sweep the house       ╰⊏══⊐┬╮
   004 take out the trash         │╰⊏⊐
   003 vacuum                     ╰⊏═⊐
   005 brush the dog       ⊏═⊐

   $ gontt dep 1-5
   Using gonttfile ./gontt.json
   "brush the dog" depends on "put the dog outside"

   001 put the dog outside ⊏⊐╮
   005 brush the dog         ├⊏═⊐
   002 sweep the house       ╰⊏══⊐┬╮
   004 take out the trash         │╰⊏⊐
   003 vacuum                     ╰⊏═⊐

   $ gontt dep 5-4
   Using gonttfile ./gontt.json
   "take out the trash" depends on "brush the dog"

   001 put the dog outside ⊏⊐╮
   005 brush the dog         ├⊏═⊐──╮
   002 sweep the house       ╰⊏══⊐┬┤
   004 take out the trash         │╰⊏⊐
   003 vacuum                     ╰⊏═⊐

Doneness
--------

"I have a big project with lots of little steps along the way, and I want to track them".

So implement a ``gontt done 4`` that strikes through the task. Hide done things in the normal tree view, and add a flag to show them (as strikethrough)

Filtering
---------

"I have a big project to do but today I want to focus on getting towards this one intermediate goal"

.. code-block:: console

   $ gontt show "sweep the house"
   Using gonttfile ./gontt.json

   001 put the dog outside ⊏⊐╮
   002 sweep the house       ╰⊏══⊐
   $ gontt show 2
   [same]

Block out all downstreams though? or leave one level intact?

Block out all other lines, definitely.

Shot down ideas
===============

.. _realtime:

Any sort of realtime view
-------------------------

No "this is today" line, no deadlines, no weekend shading. The fact of the technology here is that the lines between the tasks take up space, so a task that can start immediately after its upstream finishes is still at least 1 character to the right. In a real Gantt chart they'd line up perfectly, with the dependency lines working around that graphical constraint.

Tracked Time?
-------------

A way of recording that I've spent 4 hours on this task already, when it was supposed to take 3.

See :ref:`realtime`.

Resources
---------

MVP, yo

Other Dependencies
------------------

Task dependency is limited to end-start now. What about the others? Let's go over all of them:

-  end-start: "this task can't start until the other ends". the most common one.
-  start-start: "this task can't start until the other starts". if the other task delays, this task has to too.
-  start-end: "this task can't end until the other starts". if the upstream one delays, this one does too. And here the downstream one happens presumably *before* the upstream one, illustrating the need for "upstream/downstream" rather than "precedent/antecedent"
-  end-end: "this task can't end until the other ends". If the upstream has to take longer, the downstream changes too.

I'm inclined to shoot this one down outright. It would make all the lines too ugly. It kills "simple".

Lag and Lead Times
------------------

meh. See :ref:`realtime`.

Constraints
-----------

"Start/Finish no earlier/later than ..." And thus "solving", where the computer tells you "whoops, you won't complete this in time without changing duration, constraint, or dependency". meh. See :ref:`realtime`.

WIP
===

whatever algorithm is ordering the rows...

I have 2 downstreams here (005, 002). What order should they be in?

1. do any of them have no downstreams? no
2. which has fewer downstreams?
3. after I ignore all the downstreams they share, which has fewer?

::

   001 put the dog outside ⊏⊐╮
   005 brush the dog         ├⊏═⊐──╮
   002 sweep the house       ╰⊏══⊐┬┤
   004 take out the trash         │╰⊏⊐
   003 vacuum                     ╰⊏═⊐

   001 put the dog outside ⊏⊐╮
   002 sweep the house       ├⊏══⊐┬╮
   005 brush the dog         ╰⊏═⊐──┤
   004 take out the trash         │╰⊏⊐
   003 vacuum                     ╰⊏═⊐

But wait, is that so ugly? I dunno, maybe there's a case where it could get real ugly. I guess I'll just have to build it and play with it to find out.
