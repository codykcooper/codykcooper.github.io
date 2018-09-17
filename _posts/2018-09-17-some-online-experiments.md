Online Experiments
------------------

Below is a list of online experiments I built a few years back using
[jsPsych](https://www.jspsych.org/) a few years ago when in grad school.
The code in the linked repo was used to prototype complex online
experiments to study decision making under different contexts (e.g.,
food choice while multitasking). These were all built on an older
version of jsPsych, so they could all likely be improved by being
updated to work with the most recent version of jsPsych.

When I was working on these experiments I was able to embed them with a
Qualtrics survey, however it appears there have been changes to the
Qualtrics engine that broke the old approach. There may be a way to get
these to integrate with the new engine, but I haven't figured it out
yet. The author of jsPsych has provided other tools to help distribute
the experiments.

### [Dynamic Multitasking Choice Experiment](https://codykcooper.github.io/Dynamic-Multitasking/)

This experiment presents respondents with two tasks that must be
completed under time pressure.

1.  The top task presents two 3 x 3 grids containing pairs of numbers or
    letters in each cell. Participants are asked to compare the two
    grids and determine if the grids are an exact *match* or if the
    grids *mismatch* in any of the cells. Respondents have up to 15
    seconds to make a determination.
2.  The bottom tasks presents respondents with two images of food.
    Participants are asked to indicate which of the two options they
    prefer. Participants have up to 10 seconds to determine which food
    they prefer.

### [Continuous Response Measure](https://codykcooper.github.io/Continuous-Response-Measure/)

This is an online implementation of a common measure used in
communication and advertising research, a continuous response measure
(CRM). The basic idea of the CRM is to get a moment by moment indication
of how someone is responding to dynamic content (e.g., a television
show, advertisement, etc.). At any moment during viewing the participant
may move the slider from left to right, and the tool will log any change
in value. Additionally, the tool is synched to the clock on the youtube
video, so the CRM value can be tied to an exact moment/event. This is
also helpful if there are any pauses or buffering issues during viewing.

### [Categorize then Decide Experiment](https://codykcooper.github.io/Categorize-then-Decide/)

This experiment presents respondents with a hypothetical scenario where
they work for NASA. They are going to an alien world and are asked to
interact with the locals of this new planet. During their interaction,
the respondents are asked to determine which “group” of potential
inhabitants a series of faces belongs to, and decide how to act (i.e, be
nice or be defensive).
