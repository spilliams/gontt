*****
gontt
*****

A simple Gantt-like charting tool written in Go

how should it work?
===================

``gontt`` should output something like the following:

::
   chart generated from ./gontt.json

   put the dog outside ⊏⊐╮
   brush the dog         ├⊏═⊐──╮
   sweep the house       ╰⊏══⊐┬┤
   take out the trash         │╰⊏⊐
   vacuum                     ╰⊏═⊐

The model of a **Task**, then, is currently:

.. code-block:: go

   type Task struct {
      name        string
      duration    int
      downstream  []Task
      upstream    []Task
   }

``gontt init`` will generate a stub of a json file in the local directory.

``gontt -i some/other/file.json`` will use an explicit input file location.

more thoughts
=============

1. Should the timing have ties to a calendar? deadlines? Actual duration (vs estimated)?

2. What about Resources? People, tools, locations, etc.

3. Potential ugliness: multiple depencency chains:

   ::
      chart generated from ./gontt.json

      put the dog outside ⊏⊐╮
      brush the dog         ├⊏═⊐┬──╮
      bring the dog in      │   ╰⊏⊐│
      sweep the house       ╰⊏══⊐┬─┤
      take out the trash         │ ╰⊏⊐
      vacuum                     ╰⊏═⊐

   Maybe just tack them on the end?

   ::
      Using gonttfile ./gontt.json

      put the dog outside ⊏⊐╮
      brush the dog         ├⊏═⊐──┬╮
      bring the dog in      │     │╰⊏⊐
      sweep the house       ╰⊏══⊐┬┤
      take out the trash         │╰⊏⊐
      vacuum                     ╰⊏═⊐

   gee, teaching a computer to untangle these lines could be a hard thing.

4. Should ``gontt`` have options for prompting the user for information? Or does the chart rely on the user editing the json file manually? Something like the following commands

   .. code-block:: console

      $ gontt t "brush the dog" 1
      Using gonttfile ./gontt.json
      New task "brush the dog" created, with duration 1

      001 put the dog outside ⊏⊐╮
      002 sweep the house       ╰⊏══⊐┬╮
      004 take out the trash         │╰⊏⊐
      003 vacuum                     ╰⊏═⊐
      005 brush the dog       ⊏═⊐

      $ gontt l 1-5
      Using gonttfile ./gontt.json
      "brush the dog" depends on "put the dog outside"

      001 put the dog outside ⊏⊐╮
      005 brush the dog         ├⊏═⊐
      002 sweep the house       ╰⊏══⊐┬╮
      004 take out the trash         │╰⊏⊐
      003 vacuum                     ╰⊏═⊐

      $ gontt l 5-4
      Using gonttfile ./gontt.json
      "take out the trash" depends on "brush the dog"

      001 put the dog outside ⊏⊐╮
      005 brush the dog         ├⊏═⊐──╮
      002 sweep the house       ╰⊏══⊐┬┤
      004 take out the trash         │╰⊏⊐
      003 vacuum                     ╰⊏═⊐
