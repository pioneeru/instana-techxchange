---
title: 3. TroubleShoot an Issue
description: Diagnosing Issues in Instana
sidebar_position: 3
---


# Troubleshoot an issue

---


## 

In this section of the lab, you will learn how Instana can help you get to root cause of a problem in a quick and intuitive manner.

In order for the events to show as incident, you have to create a **smart alert** which are already created in the **"Quote of the day"** application. 

On the left side, select **“Applications”** and then click on the **"Quote of the day"** link. 

![applications](images/applications.png)

In the **"Quote of the day"** dashboard, click on the **smart alert** tab as shown in the below image - step 1
Then click the  **"Quote of the day"**  as shown in step 2

![smartAlert](images/smartalerts1.png)

Click the Edit icon as shown in the below image. 

![selectErroneous](images/smartalerts2.png)

Explore the smart alert options on the right side.
**DO NOT Change any settings** This step in the lab is  only for viewing and understanding.

![selectErroneous](images/smartalerts3.png)


## Evaluate the Events via an Incident

**It can take up to 1 hour for the incidents triggered by the smart alert setup in the pevious step to show** 

On the left side, select **“Events”**

![events](images/erroneousBK1.png)

This will open up a panel showing all of the Incidents that are affecting the monitored systems and applications. Incidents are the raw Events that have been correlated via Analytics into an Incident.


Make sure that the time selection in the upper right corner is set to **“Last hour”**

As seen in the previous picture, select the Incident that is triggering on the **“QOTD Calls are slower than normal”**  It indicates that there are large numbers of calls slower than usual.

You will see the details related to raw Events that were correlated via analytics into an Incident..


![incident](images/incident1.png)
![incident](images/incident2.png)

In this case, you should see the Triggering Event and a few Related Events. The Triggering Event is the first event that starts to impact a service or a website. The Related Events are events that Instana determined are related by leveraging analytics including the relationships that exist between the various entities, services, and endpoints.

Within the Incident, you can see when the Incident opened and in some cases, when it closed. And, you can see the Incident Timeline. This is the time when each event opened and closed. If the event is still open, it will be color coded with the event severity.

On the right side, click on the **”+”** symbol to expand the Triggering Event.

You can see in the historical graph that there are periods of time where erroneous (failed) calls are high.

![evenErroneous](images/triggeringevent1.png)

## Analyze the Calls
Next, click on the **”+”** symbol to expand the **First Related Events (X)** and then click on the **“Analyze Calls”** button to analyze the calls.

![Analyze Calls](images/analyzecalls.png)

You will be taken to an analytics screen where you can see all of the failed calls for the endpoint **“CP4I.DEMO.API.Q”**. 

![Select call](images/selectCall.png)

It is the ACE flow. After click on the **”expand”** symbol, You can also see erroneous of the calls is high.

![each call](images/eachcall.png)

Select one of the failing **“PUT CP4I.DEMO.API.Q”** calls

Here you will see the User information of the session and other metadata. on the Right side you will see the detials of the failed call.
Scroll down to the Timeline section. 

![Bad Call](images/badCall.png)
![Bad Call1](images/badCall1.png)

Scroll to the bottom of the page where you see the **“Calls”**. This shows you the timeline of the calls for this single transaction.

You’ll notice that there are plus signs. Click on the plus signs to expand the call stack.

![Call Stack](images/callStack.png)

When you select the **msgFlowTra.** call, the right side of the screen gets updated with context for that call. 

![Msg Flow](images/msgFlow.png)

When you see errors, normally you go to the bottom of the call stack. That’s because the error in the last call is likely affecting the upstream calls.

Click on the last call and examine the information on the right side. You will see the following error message: “IBM MQ call failed with compcode ‘2’ (‘MQCC_FAILED’) reason ‘2053’ (‘MQRC_Q_FULL’)”

!![Call Stack Expand](images/callStackExpand.png)

In your spare time, you can click on other types of transactions in the Call Stack, but for now, let’s focus on the diagnostic scenario.



This error message indicates that the MQ Queue is full and new messages can’t be put on the queue.

At this point, we believe we have a diagnosis for our problem. The MQ Queue is full and the transactions are failing because they can’t post a message to the queue.

## Confirm Your Diagnosis
In many cases, you can confirm the root cause of the problem by analyzing the call stack and stack trace, but sometimes you need to investigate further.

In this case, you may want to examine the queue that is causing this error. Or, you may want to confirm whether the problem has been resolved.

Instana automatically puts a link to the queue in the lower right corner. For other technologies, there are similar links.

Scroll all the way to the bottom on the right side of the screen. You’ll see that there is a link that will link to the MQ Queue named CP4I.DEMO.API.Q. This is the queue that is being used for this application transaction.

![Queue Link](images/queueLink.png)

Click on the link to navigate to the queue details.

![Queue Details](images/queueDetails.png)

On this screen, you’ll notice a few important things.

* In the upper right corner, you’ll notice that the time range is specifically set to the timeframe when the Incident was open. **(1)**
* The queue depth reached 100% **(2)**
* The Messages In rate is zero during the timeframe that the queue is full. **(3)**

Clearly there is a problem with this queue. The queue is full and transactions won’t work properly because new messages can’t be placed on the queue. We need to resolve the queue depth problem in order to fix the application.

From here, if you wanted to investigate further, you could use the **bread crumbs** at the top of the screen or the Stack to navigate to the related resources. For example, examine the **IBM MQ Queue Manager**.

![stack](images/stack.png)

That completes this section of the lab.



## Optional - Simulate failures manually and troubleshoot "Ratings Endpoint Service Failures"
We are going to simulate the following (Ratings Endpoint Failures) failures in the system using Anomaly Generator running at http://10.100.1.60:32001
Please visit the URL: http://10.100.1.60:32001 or navigate to the tab(bookmarked) which was already opened in your firefox browser.

Click on Rating Service Failures as shown below.

![Click_on_Ratings_Endpoint_Failures](images/click_on_ratings_service_failures.png)

Now Click on "Start" whenever you are ready to inject the following failures as shown below in the picture.

![Click_on_Start](images/Click_on_Start.png)

Now the system will take time to simulate the failures. Please give atleast 3-5 minutes for the system to simulate the failures.

All of these failures will be in the qotd-rating service. There are going to be 6 failure scenarios in the qotd-rating service.

1. Rating service failing with 500/404 errors half of the time.
2. Increase memory usage
3. Increase cpu usage 
4. Increase latency in primary GET /ratings/:id to 0.9 seconds
5. Start new independent log - unknown code every 4 seconds in quote service
6. Start new dependent logger on /ratings/:id endpoint.

Our troubleshooting can start with looking at the qotd-rating service in Applications. Please set the time duration to the last 10 minutes and go to Ratings service in Applications view.

Our goal is to show you the steps to troubleshoot this scenario. Please click on Applications as shown in the picture below. 

![Click_On_Applications1](images/Click_On_Applications1.png)

Please click on qotd-rating application as shown in the picture below. 

![Click_on_qotd_rating](images/Click_on_qotd_rating.png)

Now we will surface only the 4XX and 5XX errors from these services as shown in the picture below.

![4XX_5XX_response_codes](images/4XX_5XX_response_codes.png)

Now we will try to drag and drop into the time period where the errors were being produced and the time period on the top right corner will get changed on its own from 10 minutes to the drag and drop time period as shown below. We are trying go back in time to see the errors. 

![Drag_drop_erroneous_spikes](images/Drag_drop_erroneous_spikes.png)

Now we are going to look at qotd-rating application and verify all the simulated failure conditions based on the information we see in the graphs. For example: we can see the latency spike to 0.9 second as shown in Step 2 and Step 3. This proves Condition 2 in the failures section. You can see this from the picture below. 

![Validating_failures_part2](images/Validating_failures_part2.png)

Now we can see all the end points getting errors as shown in the picture below which satisfies Condition 1. 

![see_all_failures_with_GET_Ratings](images/see_all_failures_with_GET_Ratings.png)

Next, we will analyze all the failures in the qotd-rating service by clicking "Analyze Calls" as shown in the picture below.

![GET_Ratings](images/GET_Ratings.png)

In this Analytics screen you can see that we are automatically looking at qotd-rating service and trying to expose only the 4XX and 5XX response codes as shown in the picture below.

![4XX_5XX_Analytics](images/4XX_5XX_Analytics.png)

Next, select the top group which has the most 4XX and 5XX errors. Notice you did not have to type any SQL query to get to this result since Instana automatically knows the context of your troubleshooting scenario.

![Select_Group](images/Select_Group.png)

Now we have all the traces with 4XX and 5XX response codes and we can select any one of them to see what is going on inside our code.

![select_red_color](images/select_red_color.png)

you can see the 500 response code and warning Log message below along with the stack trace which will tell you the exact code with the line number responsible for the problem.

![STACK_TRACE_Error_Logs](images/STACK_TRACE_Error_Logs.png)

Lastly you can see the log message which the simulation injected. 

![Log_message](images/Log_message.png)

## Summary
In this lab, you learned how to diagnose a problem in Instana, determining that the queue was full and preventing transactions from completing.

Also successfully in this lab, you learnt how to successfully troubleshoot some other common issues by simulating failures.

Continue to next Lab of **"End User Monitoring(EUM) / Real-User Monitoring(RUM)"**




