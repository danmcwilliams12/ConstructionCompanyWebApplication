# Live Project
## Introduction

During the last two weeks of my time at the tech academy, I worked with fellow students to build an ASP.NET MVC web application for a local construction company using Entity Framework Code First.  When I entered the project it was a work in progress about a month away from delivery to the client.  At first it was overwhelming to look at a codebase I was unfarmiliar with but over the course of the project I became more and more comfortable with it.  The project was a great opportunity for me to learn new technologies on the fly and implement them in my code very rapidly.  I also had an opportunity to see how to communicate effectively with fellow programmers using project management software Azure DevOps and Slack.  This experience has given me the confidence that I posses the skills to be an effective junior developer that can work with a team to create high quality software.  Below are descriptions of some of the stories I worked on and the code implimentation as well.

### Shift Time Checker Method

Toward the end of my time on the project, the model for jobs was updated to include a ShiftTime class that included the days of the week and the start time for each shift.  In this story, I was tasked with creating a method in the jobs controller that would check to see if the job passed into it possesed any non default shift times, set all default times to null, save changes to the DB and return a bool value.

