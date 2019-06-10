# Live Project
## Introduction

During the last two weeks of my time at the tech academy, I worked with fellow students to build an ASP.NET MVC web application for a local construction company using Entity Framework Code First.  When I entered the project it was a work in progress about a month away from delivery to the client.  At first it was overwhelming to look at a codebase I was unfarmiliar with but over the course of the project I became more and more comfortable with it.  The project was a great opportunity for me to learn new technologies on the fly and implement them in my code very rapidly.  I also had an opportunity to see how to communicate effectively with fellow programmers using project management software Azure DevOps and Slack.  This experience has given me the confidence that I posses the skills to be an effective junior developer that can work with a team to create high quality software.  Below are descriptions of some of the stories I worked on with code snippets, including comments.

### Shift Time Checker Method

Toward the end of my time on the project, the model for jobs was updated to include a ShiftTime class that included the days of the week and the start time for each shift.  In this story, I was tasked with creating a method in the jobs controller that would check to see if the job passed into it possesed any non default shift times, set all default times to null, save changes to the DB and return a bool value.  Since the infrastructure needed to use this method was not yet in place I needed to include a job from seed data for testing purposes.

```
[HttpPost]
        public bool HasNonDefultShiftTime()
        {
            //Note: Uncomment the JQuery Ajax caller at bottom of site.js to test

            //Test 1: This job should return true and modify defaults
            var testJob = (from j in db.Jobs where j.JobNumber == "375" select j).FirstOrDefault();

            //Test 2: This job should return false  
            //var testJob = (from j in db.Jobs where j.JobNumber == "425" select j).FirstOrDefault();

            var shiftTime = (from s in db.ShiftTimes where s.ShiftTimeId == testJob.ShiftTimes.ShiftTimeId select s).FirstOrDefault();
            bool hasNonDefault = false;           
            //set all default shiftTimes to null
            if (shiftTime.Monday == shiftTime.Default) { shiftTime.Monday = null; }
            if (shiftTime.Tuesday == shiftTime.Default) { shiftTime.Tuesday = null; }
            if (shiftTime.Wednesday == shiftTime.Default) { shiftTime.Wednesday = null; }
            if (shiftTime.Thursday == shiftTime.Default) { shiftTime.Thursday = null; }
            if (shiftTime.Friday == shiftTime.Default) { shiftTime.Friday = null; }
            if (shiftTime.Saturday == shiftTime.Default) { shiftTime.Saturday = null; }
            if (shiftTime.Sunday == shiftTime.Default) { shiftTime.Sunday = null; }
            db.SaveChanges();
            
            //Check to see if schedule has any non-default shift times
            if (shiftTime.Monday != null || shiftTime.Tuesday != null || shiftTime.Wednesday != null || shiftTime.Thursday != null ||
               shiftTime.Friday != null || shiftTime.Saturday != null || shiftTime.Sunday != null)
            {
                hasNonDefault = true;
            }

            return hasNonDefault;
        }
```
### User Suspension Schedule Managment

Website users that become suspended may have schedules that they are currently on that need to end immediately and future schedules that must be deleted.  This story required a popup to fire when an admin attempted to suspend a user confirming the suspension or unsuspension as well as warning that the user's schedules will be removed.  First, I sorted users in the partial view's GET method to determine what popups would fire for each user.

```
public ActionResult _UserListPartial()
        {
            //Queries to determine what pop-up shows on suspension submission
            var currentFutureUsers = (from u in db.Users
                                      join s in db.Schedules on u.Id equals s.Person.Id
                                      where s.EndDate > DateTime.Now
                                      select u).Distinct().ToList();
            ViewBag.currentFutureUsers = currentFutureUsers;

            var suspendedUsers = (from a in db.Users
                                      where a.Suspended
                                      select a).ToList();
            ViewBag.SuspendedUsers = suspendedUsers;

            return PartialView(db.Users.AsEnumerable());
        }
```
With the users sorted, I assigned the appropriate confirmation message to fire on form submission.

```
@using (Html.BeginForm("Suspend", "Account", FormMethod.Post, new { enctype = "multipart/form-data" }))
        {
            <!--Creates checkbox for Suspend and binds it to the model attribute "Suspended"-->
            @Html.Hidden("Id", item.Id)
            @Html.CheckBoxFor(model => item.Suspended, new { Name = "suspended" })

            // submit button for users with current and future schedules
            if (ViewBag.currentFutureUsers.Contains(item))
            {
                <input type="submit" value="Submit" onclick="return confirm('Suspend user and remove all assigned schedules?')" />
            }
            //submit button for users that are currently suspended
            else if (ViewBag.suspendedUsers.Contains(item))
            {
                <input type="submit" value="Submit" onclick="return confirm('Are you sure you want to unsuspend this user?')" />
            }
            //submit button for unsuspended users without current or future schedules
            else
            {
                <input type="submit" value="Submit" onclick="return confirm('Are you sure you want to suspend this user?')" />
            }
        }
```
Then saved the changes to the DB.
```
[HttpPost]
        public ActionResult Suspend(string id, bool suspended = false)
        {
            //fetch user to suspend or unsuspend
            var user = (from ApplicationUser in db.Users where ApplicationUser.Id == id select ApplicationUser).FirstOrDefault();
            //fetch list of users schedules
            var schedules = (from s in db.Schedules
                             where s.Person.Id == user.Id
                             select s).ToList();
            if (suspended)
            {
                user.Suspended = true;
                
                //if user has schedule items
                if (schedules.Count > 0)
                {
                    foreach (var schedule in schedules)
                    {
                        //if schedule item has not ended yet, end it immediately
                        if (schedule.StartDate < DateTime.Now && schedule.EndDate > DateTime.Now)
                        {
                            schedule.EndDate = DateTime.Now;
                        }
                        //deletes all future schedules
                        if (schedule.StartDate > DateTime.Now)
                        {
                            db.Schedules.Remove(schedule);
                        }
                    }
                }
            }
            if (!suspended)
            {
                user.Suspended = false;                              
            }
            if (ModelState.IsValid)
            {
                db.Entry(user).State = EntityState.Modified;
                db.SaveChanges();
                return RedirectToAction("Index", "Dashboard");
            }
            return RedirectToAction("Index", "Dashboard");
        }
```
