*****
gontt
*****

A simple Gantt-like charting tool written in Go

how should it work?
===================

``gontt tree`` should output something like the following:

::

   Empty the garage
   ├── move tools out
   │   └── more cleats and shelves in basement
   ├── move bikes into shed
   │   └── install doors on the shed
   └── move lumber somewhere

Although there's an argument to be made for prettier lines:

::

   take out the trash  ─╮
   do the dishes        ├─
   vacuum               ╰─╮
   sweep the house        ├─
   brush the dog          ├─


The model of a **Task**, then, is currently:

.. code-block:: go

   type Task struct {
      downstream  []Task
      name        string
   }

Should I implement the standard "box/whisker" idea for timing? Should I implement timing at all? Deadline, estimated duration, hours already spent on it

What about Resources? People, tools, locations, etc.

What if a Task is used as dependency for multiple upstream Tasks?

::

   has 2 downstream ─╮
   has 1 and 1       ├─╮
   has 1 and 1       ╰─┤
   has 2 upstream      ╰─

And if/when we introduce durations:

::

   has 2 downstream ⊏═⊐╮
   has 1 and 1         ├⊏═══⊐╮
   has 1 and 1         ╰⊏═⊐──┤
   has 2 upstream            ╰⊏═⊐

sticky spots
============

- A depends on B and C, B and C each depend on D (I'll call this "multiple upstream")
