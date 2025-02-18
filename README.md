# PennGrader
Welcome to the PennGrader!  This autograder project was created by Leo Murri at the University of Pennsylvania, and it is currently maintained by the CIS faculty, staff, and students at Penn.

Here at PennGrader, we believe that learning comes from lots of practice...and from making lots of mistakes. 

From creator Leo Murri: "After many years as a student I found myself very frustrated in the following homework timeline: struggle on a homework assignment for weeks, submit something that may or may not be right and then wait a few more weeks to receive any type of feedback, at which point I had forgotten all about the homework. After many years as a TA, I also found myself very frustrated with the common auto-grading tools, the hours and hours of manual grading and the onslaught of re-grade requests that came thereafter."

From these frustrations, the PennGrader was born!

The PennGrader was built to allow students to get instant feedback and many opportunities for re-submission. After all, programming is about making mistakes and learning from feedback! Moreover, we wanted to allow TAs and Instructors to write their homework in any way they pleased, without having to worry about the structure of a specific auto-grader. The examples below are done using Jupyter Notebooks which is the most common use case, but you can use this for normal Python homework as well. 

Here is what a student sees in their Homework Notebook. All a student has to do is write their solution and run the auto-grading cell.

![Sample Question](https://penngrader-wiki.s3.amazonaws.com/sample_question.gif)

Through the magic of AWS Lambdas, the student's answer (in this case the `addition_function` object) is packaged and shipped to the backend where it is checked against the teacher-defined test case. Finally, a score is returned. All students' scores are saved in the backend and are easily accessible to the instructors. If at first they don't succeed, they can learn from their mistakes and try again.  (Yes, if you'd like, can set a maximum number of daily submissions to incentivize students to start early). This "grader" function can easily be used from any Jupyter notebook (even Google Colab), all you have to do is to `pip install penngrader`. See templates below.

Ok, ok, you might be saying to yourself: "That looks easy enough, but what about us TAs, we want something that simple too!" Well, look no further. The TAs/Instructors' experience is just as seamless. All TAs will share a __Teacher_Backend__ notebook, which contains all the test case functions. The logic of how testing is done is simple: whatever Python object gets passed through the `answer` field in the `grade(...)` function (see above) will be the input to the test case function (see below). In the above example, `addition_function` is passed as the answer to a test case named `"test_case_1"`. Therefore, the TAs will need to write a `test_case_1(addition_function)` function in the __Teacher_Backend__ notebook, as follows:

![Sample Question](https://penngrader-wiki.s3.amazonaws.com/sample_update.gif)

As you can see, this function tests that `addition_function(1,2) == 3`, if correct it adds 5 points to the `student_score`. The test must then return an integer tuple `(student_score, max_score)`, which is what will be displayed to the student. As you can see this type of test function gives the Instructor complete flexibility on what to test and how much partial credit to give. Remember that the answer that gets passed to the test case could be anything... a function, a class, a DataFrame, a list, a picture... anything! The PennGrader automatically serializes it and all its dependencies and ships to AWS for grading.

## Configuring the autograder for your course

The PennGrader has a fairly simple setup -- you need to create a UUID and a name for each course offering.

We have developed additionally [infrastructure](https://bitbucket.org/zives/penngrader-gradescope/src/master/) for pulling student grades into Gradescope upon notebook submission. (Email for permissions, since this includes some AWS endpoints.)

1. Run the notebook `Adding a Course.ipynb` in this repo to create a UUID
2. Copy `config.yaml.default` (from the PennGrader-Gradescope repo) to `config.yaml`.  Add your UUID to `config.yaml`, as `secret_id`
3. Log into the AWS Console, go to DynamoDB and add an entry into the `Classes` table.  Set `secret_key` to the UUID, and `course_id` to a unique name for the course.  At Penn you will need to contact Zack Ives to register the key for your course.

Next, go to Colab to set up the homework metadata:

* Using the `Assignment Setup.ipynb` (in the Gradescope-PennGrader repo) and your `config.yaml`, register the homework assignment info (name, deadline, etc.).  To do this, update the `TOTAL_SCORE`, `MAX_DAILY_TEST_CASE_SUBMISSIONS`, and `DEADLINE`.  Beware that the deadline is in GMT.
* Register the test cases for your homework assignment.
* Update the homework assignment to use the **name** of your course, appended with `_HWx` where `x` is your homework number.  For instance, if we use `CIS520_F20` as our course name, we might have `CIS520_F20_HW3`.  Do **not** use the `secret_key` here.

Finally, go to Gradescope.  For each homework:

* Copy in your `config.yaml` from above, and update the `homework_num` to the homework number (0-5).

Your homework specification should tell the students to upload their final notebook to Gradescope, with the `STUDENT_ID` set to their numeric PennID.  From this Gradescope will look up their Penngrader scores (as of the submission date).

The TAs may add additional manual grading, or simply release the scores, as appropriate.

[PennGrader_Homework_Template.ipynb](https://penngrader-wiki.s3.amazonaws.com/PennGrader_Homework_Template.ipynb)

[PennGrader_TeacherBackend.ipynb](https://penngrader-wiki.s3.amazonaws.com/PennGrader_TeacherBackend.ipynb)

Download these two notebooks and launch them via Jupyter. They will show you how to add grading cells in your homework notebook and add write/update test cases via the teacher backend, as well as view student's grades.

## Behind the scenes...
In the following section, I will go into detail about the system implementation. Below is the system design overview we will go into.

![Architecture Design](https://penngrader-wiki.s3.amazonaws.com/design.png)

### Clients
There are two pip installable clients, one for students and one for instructors. You can install these two clients by running `pip install penngrader` in your favorite terminal.  When creating a new homework download the [Homework_Template.ipynb](https://penngrader-wiki.s3.amazonaws.com/PennGrader_Homework_Template.ipynb) and the [TeacherBackend.ipynb](https://penngrader-wiki.s3.amazonaws.com/PennGrader_TeacherBackend.ipynb) notebooks and follow the instructions. More details are presented below.

#### Student's Client: PennGrader

The student's client will be embedded in the homework release notebook. Its main purpose will be to interface the student's homework with the AWS backend. This client is represented by the `PennGrader` class which needs to be initialized by the instructor when writing the homework as follows. Note: the HOMEWORK_ID needs to be filled in before releasing the notebook. The student should only enter his or her's student id. 

```
import penngrader.grader
grader = PennGrader(homework_id = HOMEWORK_ID, student_id = STUDENT_ID)
```

The HOMEWORK_ID is the string obtained when creating new homework via the teacher backend, see below. 

STUDENT_ID is the student defined variable representing their 8-digit PennID. The student will need to run this cell at the beginning of the notebook to initialize the grader. After every question, the Instructor will also need to write a grading cell which the student will run to invoke the grader. A grading cell looks as follows:

```
grader.grade(test_case_id = TEST_CASE_NAME, answer = ANSWER) 
```

TEST_CASE_NAME is the string name of the test case function that will grader the given question. 

ANSWER is the object that needs to be graded. 

For example, you might have a question where you instruct the student to create a DataFrame called `answer_df`. You will need to write `answer_df` as the input answer parameter of the grader function as follows:

```
grader.grade(test_case_id = TEST_CASE_NAME, answer = answer_df) 
```

 That way, when the student runs this cell, the grader will automatically find the test function named TEST_CASE_NAME, serialize the `answer_df` python variable and ship it to AWS. The cool thing about the PennGrader is that you can pass pretty much anything as the answer.

#### Teacher's Client: PennGraderBackend

The teacher client allows instructors to create and edit the test cases function mentioned earlier, as well as define multiple homework metadata parameters. As shown in the template notebooks linked above, you first need to initialize the _PennGraderBackend_ for a specific homework as follows:

```
backend = PennGraderBackend(secret_key = SECRET_KEY, homework_number = HOMEWORK_NUMBER)
```

SECRET_KEY is the string variable obtained when creating a course. 

HOMEWORK_NUMBER identifies which homework number you are planning to write/edit. 

After running the above cell in a Jupyter Notebook, given a correct SECRET_KEY,  the assigned HOMEWORK_ID string will be printed out. This HOMEWORK_ID needs to be copied into the initialization of the PennGrader student client for release. Just the homework id, without the secret key, will not allow students to see any of the test cases, so make sure the secret key does not get out. 

After initialization, the  `backend` can be used to 1) update metadata 2) edit/write test cases.

You can edit the following metadata parmaters by runnin the following code:

```
TOTAL_SCORE = 100
DEADLINE = '2019-12-05 11:59 PM'
MAX_DAILY_TEST_CASE_SUBMISSIONS = 100

backend.update_metadata(DEADLINE, TOTAL_SCORE, MAX_DAILY_TEST_CASE_SUBMISSIONS)
```

`TOTAL_SCORE` represents the total number of points the homework is worth and should equal the sum of all test cases weights.

`DEADLINE` represents the deadline of the homework, with format: _YYYY-MM-DD HH:MM A_.  **Please note this deadline is in UTC**.

`MAX_DAILY_TEST_CASE_SUBMISSIONS` represents the number of allowed submissions per test case per day.

Writing test cases is also just as simple. In a Jupyter Notebook (see Teacher Backend template above), you just need to write test case functions for each gradable question.  Each test case will be identified by a _test_case_id_ which is the name of the test case function. A test case functions needs to follow the following format:

```
def test_case_name(answer_object_name):
	student_score = 0 # Current student score
	max_score = 5 # Number of points this question is worth

	# some kind of grading that adds or subtracts from the student_score #

	return (student_score, max_score)
```

The test case function needs to return an integer tuple representing the student score for their answer and max number of points that can be earned in the given question.

After writing all the test cases you need, simply run the following code in a cell and the PennGraderBackend class will automatically extract all user-defined functions in the current notebook and upload them to AWS.

```
backend.update_test_cases()
```

A success message will print once the operation has succeeded. If loading a lot of external libraries this might take a few minutes.

### Lambdas

AWS Lambda is an incredibly cool, but also quite complex system to set up.  We recommend that you use an existing API Gateway URL and set of Lambda Layers.

* The Gateway URL exports a Python function (from Penngrader) to a given URL, with a REST interface and an API key.
* The Lambda needs to be granted appropriate DynamoDB permissions and logger permissions.
* The Lambda makes use of Python libraries, in our case including Pandas and Numpy.  You can set these up in an S3 volume and add these to a Layer, to make these accessible to the Penngrader.  **However note that simply running `pip install -t package pandas numpy pytz` will be inadequate: you need special multi-Linux versions of the C libraries.  See https://korniichuk.medium.com/lambda-with-pandas-fd81aa2ff25e for more info.**

#### Grader

The _Grader_ lambda gets triggered from an API Gateway URL from the student's PennGrader client. The student's client as defined above will serialize its answer and make a POST request to the lambda with the following body parameters: 

`{'homework_id' : ______, 'student_id' : ________, 'test_case_id' : ________, 'answer' : _______ }`

The lambda will proceed by downloading the correct serialized _test_case_'s and _libraries_ from the _HomeworksTestCases_ DynamoDB table. It will then deserialize these objects and extract the correct test case given the _test_case_id_ . import the correct libraries used by given test case. If the submission is valid the student score will be recorded in the backend. 

#### Grades

The grades lambda interfaces the TeacherBackend and student notebook with the Gradebook. Using this API Gateway triggered lambda the TeacherBackend client can request all grades for a given homework. The payload body used to trigger this lambda will need to include 

`{'homework_id' : ______, 'secret_key' : ________, 'request_type' : ________ }`

Where the _secret_key_ parameter is optional and will allow a properly initialized TeacherBackend to download all grades for the selected homework. Students will also be able to view their scores.

#### HomeworkConfig

The _HomeworkConfig_  lambda interfaces the TeacherBackend with the _HomeworkTestCases_ and _HomeworkMetadata_ DynamoDB Tables (see below for table schema). This lambda can be triggered in two ways 1) to update metadata and 2) to update test cases. After unparsing and deserializing the triggered payload, the inputted data gets written into the appropriate DynamoDB table. 

### DynamoDB Tables & S3 Buckets
As shown in the above schematic we maintain the majority of the data needed for grading and grade storage on DynamoDB. Below we list the information recorded in each table.

**Classes DynamoDB Table**

_Classes_ contains information about all courses currently registered for the PennGrader. The grading protocol is on a per-class basis. Each class that wants to create a course that uses the PennGrader will receive `SECRET_KEY`, this secret key will be passed in the TeacherBackend client to allow instructors to edit test cases. The tables contain the following schema:

`secret_key` : Unique UUID used as a secret identifier for a course.

`course_id`  : Human-readable identifier representing the course number and semester of the class offered i.e. 'CIS545_Spring_2019'. This ID will be the pre-fix of the `homework_id`, which will be used to identify a homework assignment.

**HomeworksMetadata DynamoDB Table**

The _HomeworksMetadata_ is used to maintain updatable information about a specific homework. The information in this table will be editable from the TeacherBackend even after homework release. The tables contain the following schema:

`homework_id` : Unique identifier representing a course + homework number pair. _homeowork_id_ is constructed by taking the _course_id_ defined above and appending the homework number of the assignment in question. For example, given the course id defined above (CIS545_Spring_2019), the _homework_id_ for the first homework will be 'CIS545_Spring_2019_HW1'. This homework_id will be passed into the PennGrader Grader class in the student's homework notebook and will be used to correctly find the correct test cases and store the student scores correctly.

`deadline` : String representing the deadline of the homework in local time i.e. 2019-12-05 11:59 PM (format: YYYY-MM-DD HH:MM A)

`max_daily_submissions` : Number representing the total number of submissions allows per test case per day. For example, if this number is set to 5, it means that all students can submit an answer to a specific test case 5 times a day. Resetting at midnight. 

`max_score` : The total number of points this homework is worth. This number should be equal to the sum of all test case weights. _max_score_ is used to show students how many points they have earned out of the total assignment.

**HomeworksTestCases DynamoDB Table**

The _HomeworksTestCases_ table contains a serialized encoding of the test cases and libraries imports needed to run a student's answer. The tables contain the following schema:

`homework_id` : Same _homework_id_ from the _HomeworksMetadata_ table.

`test_cases` : This field contains a dill UTF-8 string serialization of the test cases defined in the teacher backend. The teacher backend extracts all test case functions from the notebook and creates a dictionary of _name_ -> _function_. This parameter is deserialized when a new grading request is made and the correct test case is extracted and ran. 

`libraries` : Similar to the _test_cases_ field, the libraries are UTF-8 dill serialized list of tuples that contain all libraries and functions imported in the teacher backend notebook and their appropriate short name. This list is used to import all needed libraries to run a specific test case. 


**Gradebook DynamoDB Table**

The _Gradebook_ table contains all grading submissions and student scores. The table contains the following schema:

`homework_id` : Same _homework_id_ from the _HomeworksMetadata_ table.

`student_submission_id` : String representing a student submission. This string is create by appending the student's PennID to the test case name i.e. '12345678_test_case_1'.

`max_score` : Maximum points that can be earned for this test case

`student_score` : Points earned by the student on this test case. 

Note: Currently only the last submission is recorded, thus the latest student score will overwrite all previous scores.


