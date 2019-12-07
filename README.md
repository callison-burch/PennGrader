# PennGrader
Welcome to the PennGrader!

Here at PennGrader we believe that learning comes from lots of practice...and from making lots of mistakes. 

After many years as a student I found myself very frustrated in the following homework flow: struggle on a homework for weeks, submit something that may or may not be right and then wait weeks for any type of feedback, by which I had forgotten all about the homeowork. After many years as a TA I also found myself very frustrated with the common autograding tools, the hours and hours of manual grading and the onslaught of re-grade requests that came thereafter.

From these frustrations the PennGrader was born!

The PennGrader was built to allow students to get instant feedback and many opportunities for re-submission. Moreover, we wanted to allow TAs and Instructors to write homeworks in any way they pleased, without having to worry structuing it for a specific autograder. The examples below are done using Jupyter Notebooks which is the most common use case, but you can use this for a normal Python homework as well. 

Here is what a student sees in his Homework Notebook. All a student has to do is write his solution and run the autograding cell.









---------------------------
It was made for Jupyter Notebooks in mind but can be outfitted into a non-Jupyter python grading script. We want to be able to tell students if they did it write or wrong right away. I heard a lot of people saying "oh the students knowing if they are getting it correct or not are just going to keep trying until they get it right" as a counterargument for this approach. And let me tell you, I think that is exactly the best argument to be made for this approach of grading. A motivated student will be able to realize he is doing something wrong and then go back study more, practice more, learn more and figure out how to fix it. If a student writes an answer and has no way of checking if it is correct or not it and say is compleltely wrong, that learning opportunity is just going to go to waste. The students will realize a few weeks down the road when he gets the grade back that he did it wrong, but by then he will have 100 more assignemtns and most likely have forgetten what that question was even about. PennGrader wants to give students the opportuntiy to learn from their mistakes and helped them figure out what needs to be studied and learned. And while we are at it, we wanted to make the most seemless grading user experience grading that there is ever been (for students and even more for TAs). 

## Create a Homework for your class!
Follow the instructions below to create a homework for your class.

## Behind the scenes...
In the following section I will go into details about the system implementation. Below is the system design overview we will go into.

![Architecture Design](https://penngrader-wiki.s3.amazonaws.com/design.png)
