
# Crowdsourcing Image Annotations - Part 1
In this tutorial, we are going to use Boto3, the AWS SDK for Python Version 3, to access the Amazon Mechanical Turk API. After installing the Boto SDK, we are going to use Boto3 for croudsourcing image annotations.

This first part shows how to use Boto to create HITs and retrieve results. 
Then, we are going to develop a project that can use MTurk to locate objects in images. We will create a Human Intelligence Task (HIT) that will ask Workers to draw a bounding box around specific objects. Once the HIT is submitted, the Requester will receive a set of coordinates corresponding to the points the Worker has labeled. This information will allow the Requester to have Workers locate objects in images, so that they can then help train machines to perform the same operation.

I have developed these tutorials by migrating previous Amazon tutorials for Boto2 and changing the code and steps as needed, specifically:

- [Tutorial: Getting started with MTurk and Python using Boto](https://blog.mturk.com/tutorial-getting-started-with-mturk-and-python-using-boto-452fb0243a30#.eggez6xwf)

- [Tutorial: Using the MTurk Requester Website together with Python and Boto](https://blog.mturk.com/tutorial-using-the-mturk-requester-website-together-with-python-and-boto-4a7ef0264b7e#.7uz9w4hov)

- [Tutorial: Annotating images with bounding boxes using Amazon Mechanical Turk](https://blog.mturk.com/tutorial-annotating-images-with-bounding-boxes-using-amazon-mechanical-turk-42ab71e5068a#.z0clf9aln)

- [Tutorial: Retrieving bounding box image annotations from MTurk](https://blog.mturk.com/tutorial-retrieving-bounding-box-image-annotations-from-mturk-253b86cb7502#.obytdqw01)

- [Tutorial: Measuring the accuracy of bounding box image annotations from MTurk](https://blog.mturk.com/tutorial-measuring-the-accuracy-of-bounding-box-image-annotations-from-mturk-ad3dfcdf8aa0#.m2fm7ad1q)

- [MTurk Code Samples Python](https://github.com/awslabs/mturk-code-samples/tree/master/Python)

## 1- Making your first successful API call
To verify everything is set up correctly, it is often helpful to start by ensuring you can call to retrieve your MTurk account balance. There is no cost to call this API, and if everything is setup correctly, it should always return a result. To do this, we’ll execute the following code:


```python
import boto3

# Use the Amazon Mechanical Turk Sandbox to publish test Human Intelligence Tasks (HITs) without paying any money.

host = 'https://mturk-requester-sandbox.us-east-1.amazonaws.com'
# Uncomment line below to connect to the live marketplace instead of the sandbox
# host = 'https://mturk-requester.us-east-1.amazonaws.com'


client = boto3.client('mturk',
                      endpoint_url=host)
account_balance = client.get_account_balance()

print ("You have a balance of: " + account_balance['AvailableBalance'])
```

    You have a balance of: 10000.00


The first row imports the Boto3 library so that it can be used, the second tells Python where to find the MTurkConnection client in the Boto library. We then connect to MTurk by providing our URL to the MTurk environment we plan to use. Here we are connecting to the Sandbox environment. Sandbox allows you to test your HIT design and workflow but without incurring costs or exposing your HITs to Workers. To connect to the Production environment instead, we would replace this section of the code above with:


```python
host = 'https://mturk-requester.us-east-1.amazonaws.com'
```

But for now, let’s stick with Sandbox. It will let us test things in a more controlled environment. Once you successfully execute this code in the Sandbox environment, you should see:

##### You have a balance of: $10,000.00

Because the Sandbox environment is used for testing without incurring costs, all Requester accounts have a $10,000 account balance. If you make the same call in the Production environment, you should see your real account balance, such as:

##### You have a balance of: $108.196

## 2- Creating your first HIT
To verify everything is set up correctly, it is often helpful to start by ensuring you can call to retrieve your MTurk account balance. There is no cost to call this API, and if everything is setup correctly, it should always return a result. 

To do this, we start by creating '2_boto3_first_hit_question.XML', an XML format that has your question:

```html
<HTMLQuestion xmlns="http://mechanicalturk.amazonaws.com/AWSMechanicalTurkDataSchemas/2011-11-11/HTMLQuestion.xsd">
  <HTMLContent><![CDATA[
<!DOCTYPE html>
<html>
 <head>
  <meta http-equiv='Content-Type' content='text/html; charset=UTF-8'/>
  <script type='text/javascript' src='https://s3.amazonaws.com/mturk-public/externalHIT_v1.js'></script>
 </head>
 <body>
  <form name='mturk_form' method='post' id='mturk_form' action='https://workersandbox.mturk.com/mturk/externalSubmit'>
  <input type='hidden' value='' name='assignmentId' id='assignmentId'/>
  <!-- This is where you define your question(s) -->
  <h1>Please name the company that created the iPhone</h1>
  <p><textarea name='answer' rows=3 cols=80></textarea></p>
  <!-- HTML to handle submitting the HIT -->
  <p><input type='submit' id='submitButton' value='Submit' /></p></form>
  <script language='Javascript'>turkSetAssignmentID();</script>
 </body>
</html>
]]>
  </HTMLContent>
  <FrameHeight>450</FrameHeight>
</HTMLQuestion>
```

Next, we’ll execute the following code:


```python
import boto3

# Use the Amazon Mechanical Turk Sandbox to publish test Human Intelligence Tasks (HITs) without paying any money.
host = 'https://mturk-requester-sandbox.us-east-1.amazonaws.com'
# Uncomment line below to connect to the live marketplace instead of the sandbox
# host = 'https://mturk-requester.us-east-1.amazonaws.com'


client = boto3.client('mturk',
                      endpoint_url=host)



htmlQuestionFile = open('2_boto3_first_hit_question.xml', 'r')
htmlQuestion = htmlQuestionFile.read()

# Create the HIT
# These parameters define the HIT that will be created
# question is what we defined above
# max_assignments is the # of unique Workers you're requesting
# title, description, and keywords help Workers find your HIT
# duration is the # of seconds Workers have to complete your HIT
# reward is what Workers will be paid when you approve their work
# Check out the documentation on CreateHIT for more details
response = client.create_hit(
    Question=htmlQuestion,
    MaxAssignments = 1,
    Title='Answer a simple question',
    Description='Help research a topic',
    Keywords='question, answer, research',
    LifetimeInSeconds = 600,
    AssignmentDurationInSeconds = 600,
    Reward ='0.50',
)

# The response included several fields that will be helpful later
hit_type_id = response['HIT']['HITTypeId']
hit_id = response['HIT']['HITId']
print("Your HIT has been created. You can see it at this link:")
print("https://workersandbox.mturk.com/mturk/preview?groupId=" + hit_type_id)
print("Your HIT ID is: " + hit_id)
```

    Your HIT has been created. You can see it at this link:
    https://workersandbox.mturk.com/mturk/preview?groupId=3EI427GH2WH4ERMR5SF3L4HIXEDV07
    Your HIT ID is: 3E24UO25QZQLCQRQCD4PFX4AEMFO63


By loading the URL provided, you will be able to preview it in the Sandbox environment. Congratulations! You’ve just loaded your first Human Intelligence Task in MTurk.

## 3- Retrieving your results
Now that you’ve created a HIT as a Requester, let’s show how to retrieve and process Worker answers for the HIT. When using the Production environment, once you have created your HIT, Workers will discover it, work on it, and submit results. Because we were just getting started and testing, we created our HIT in Sandbox environment. Thus, it’s unlikely that a Worker will pick up this task and complete it (after all, this was just a test HIT, and there’s no real money involved). 

Instead, because we are working in the Sandbox environment, you’ll want to play the role of the Worker by accepting the HIT you just created and submitting a response. We will then use Boto to retrieve that response.


To begin, go back and click the link we were shown after we created the HIT earlier. Then, go through the process of clicking to “Accept” the HIT, providing an answer to the question (hint: “Apple” is correct, but really any answer will do since we’re only testing), and clicking to “Submit” your result.


Now, MTurk has captured your answer. In order to retrieve it, you’ll want to execute the following code:


```python
import boto3

client = boto3.client(
    service_name='mturk',
    endpoint_url='https://mturk-requester-sandbox.us-east-1.amazonaws.com'
)

# Uncomment the below to connect to the live marketplace
# Region is always us-east-1
# client = boto3.client(service_name = 'mturk', region_name='us-east-1')

# This HIT id should be the HIT you just created - see the CreateHITSample.py file for generating a HIT
hit_id = '3E24UO25QZQLCQRQCD4PFX4AEMFO63'

hit = client.get_hit(HITId=hit_id)
print ('Hit ' + hit_id + ' status: ' + hit['HIT']['HITStatus'])
response = client.list_assignments_for_hit(
    HITId=hit_id,
    AssignmentStatuses=['Submitted'],
    MaxResults=10
)

assignments = response['Assignments']
print ('The number of submitted assignments is ' + str(len(assignments)))
for assignment in assignments:
    WorkerId = assignment['WorkerId']
    assignmentId = assignment['AssignmentId']
    answer = assignment['Answer']
    print ('The Worker with ID ' + WorkerId + ' submitted assignment ' + assignmentId + ' and gave the answer: ' 
           + answer[answer.find('<FreeText>',0) + 10:answer.find('</FreeText>', 0)])


```

    Hit 3E24UO25QZQLCQRQCD4PFX4AEMFO63 status: Reviewable
    The number of submitted assignments is 1
    The Worker with ID AYS6BR0KW5Z35 submitted assignment 3LPW2N6LKT2MTB93TIBVAHDGD205UH and gave the answer: apple


Congratulations! You’ve retrieved your first result on MTurk with Python. Now, as our last step, we will approve this result and dispose of the HIT. Approving the result ensures that Workers will be paid. In the event you forget to call ApproveAssignments, MTurk will automatically approve all assignments after a period of time you define when you create the HIT, or 30 days (whichever is sooner).

## 4- Cleaning things up
Now that you’ve created a HIT and retrieved a result, the only thing left is to review the result, provide feedback to the Worker, and dispose the HIT. The following code snippet takes care of all of this:


```python
import boto3

client = boto3.client(
    service_name='mturk',
    endpoint_url='https://mturk-requester-sandbox.us-east-1.amazonaws.com'
)

# This HIT id should be the HIT you just created - see the CreateHITSample.py file for generating a HIT
hit_id = '3E24UO25QZQLCQRQCD4PFX4AEMFO63'

hit = client.get_hit(HITId=hit_id)
print ('Hit ' + hit_id + ' status: ' + hit['HIT']['HITStatus'])
response = client.list_assignments_for_hit(
    HITId=hit_id,
    AssignmentStatuses=['Submitted'],
    MaxResults=10
)

assignments = response['Assignments']
print ('The number of submitted assignments is ' + str(len(assignments)))
for assignment in assignments:
    WorkerId = assignment['WorkerId']
    assignmentId = assignment['AssignmentId']
    answer = assignment['Answer']
    print ('The Worker with ID ' + WorkerId + ' submitted assignment ' + assignmentId + ' and gave the answer: ' 
           + answer[answer.find('<FreeText>',0) + 10:answer.find('</FreeText>', 0)])

    # Approve the Assignment
    print('Approve Assignment ' + assignmentId)
    client.approve_assignment(
        AssignmentId=assignmentId,
        RequesterFeedback='good',
        OverrideRejection=False
    )
```

    Hit 3E24UO25QZQLCQRQCD4PFX4AEMFO63 status: Reviewable
    The number of submitted assignments is 1
    The Worker with ID AYS6BR0KW5Z35 submitted assignment 3LPW2N6LKT2MTB93TIBVAHDGD205UH and gave the answer: apple
    Approve Assignment 3LPW2N6LKT2MTB93TIBVAHDGD205UH


That’s it. You have now completed all the steps needed to create, accept, submit, retrieve, approve, and dispose of a HIT in MTurk.
