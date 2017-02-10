# coursebuilder-sendgrid-integration
Using sendgrid api to send emails rather than the in built mail api due to daily limit quotas on the in built mail api.

##Important:

To use this code, you will need a sendgrid account and to have created sendgrid api token which is used to authenticate your account when using the API. It is free to set up a sendgrid account at https://sendgrid.com/,
and you get 12000 emails for free per month. Anymore than that, and you will have to refer to sendgrids price plan.

##Modifications to lib directory

-Added python-http-client.zip

-Added sendgrid-python-2.2.1.zip

-Added smtpapi-python-0.3.1.zip

The above libraries were added so that the course could use sendgrid for sending emails instead of the in built mail api.
This is because the in built mail api has a strict quota of 100 emails a day, and the course can send many more than that in a few hours during a busy period.
Also, sendgrid allows the course to send 12000 emails for free per month. Above that, please refer to the sendgrid price plan.

##Modifications to appengine_config.py

**Lines #111-114** - importing these extra libraries from the lib directory above into coursebuilder, so we can use them in the code.

```
#sendgrid
_Library('sendgrid-python-2.2.1.zip'),
_Library('smtpapi-python-0.3.1.zip'),
_Library('python-http-client.zip'),
```

##Modifications to the code at models/models.py

###Additional imports:

**Line #32**

`import sendgrid`

The above import allow the code to use sendgrid for sending emails.

###Additional static variables:

**Line #109** - `SENDGRID_API_KEY = '<SENDGRID_API_KEY_HERE>'` - The sendgrid api token for your sendgrid account, so the api can authenticate and send the emails on your behalf. Replace the value with your own.

###Additional code to existing methods:

**Lines #1079-1080** in method `_send_welcome_notification` in class `StudentProfileDAO`

```
origin = student.additional_fields
preferred_email = strip_name_from_additional_fields(origin, 'EmailAddress')
```

Gets the preferred email address from the additional_fields for the student.

**Lines #1091-1096** in method `_send_welcome_notification` in class `StudentProfileDAO`

```
if student.name == None or not student.name or student.name.isspace():
    gn = strip_name_from_additional_fields(origin, 'GivenName')
    fn = strip_name_from_additional_fields(origin, 'FamilyName')
    fullname = gn + ' ' + fn
else:
    fullname = student.name
```

Gets the name of the student from their additional_fields.

**Line #1099** in method `_send_welcome_notification` in class `StudentProfileDAO`

`'student_name': fullname,`

Sets the variable fullname in the context rather than student.name, as we do not use this in the course.

**Lines #1124-1140** in method `_send_welcome_notification` in class `StudentProfileDAO`

```
#student.email
#services.notifications.send_async(
#    preferred_email, sender, WELCOME_NOTIFICATION_INTENT,
#    body, subject, audit_trail=context,
#)

#sendgrid implementation
logging.info('sendgrid 2.2.1')
sg = sendgrid.SendGridClient(SENDGRID_API_KEY)
message = sendgrid.Mail()
message.set_subject(subject)
message.set_text(body)
message.set_from(sender)
message.add_to(preferred_email)
status, msg = sg.send(message)
logging.info(status)
logging.info(msg)
```

Commented out section is how the email was sent using the in built API. Below is how the welcome email is now sent using sendgrid.

###Additional methods in models.py

**Line #1224** - `strip_name_from_additional_fields` - gets the value of a property from the additional_fields from the student's record in the datastore.

##Modifications to the code at modules/invitation/invitation.py

###Additional imports:

**Line #45**

`import sendgrid`

The above import allow the code to use sendgrid for sending emails.

###Additional static variables:

**Line #86** - `SENDGRID_API_KEY = '<SENDGRID_API_KEY_HERE>'` - The sendgrid api token for your sendgrid account, so the api can authenticate and send the emails on your behalf. Replace the value with your own.

###Additional methods in class `InvitationEmail`

**Line #144** - `sendgrid` - sends the email using the sendgrid API.

###Additional methods in invitation.py

**Line #366** - `strip_name_from_additional_fields` - gets the value of a property from the additional_fields from the student's record in the datastore.

###Additional code to existing methods:

**Lines #135-142** in method `send` in class `InvitationEmail`

```
        self.sendgrid()
#        notifications.Manager.send_async(
#            self.recipient_email,
#            self.sender_email,
#            INVITATION_INTENT,
#            self.body,
#            self.subject
#        )
```

Call the sendgrid method, rather than go through the in built mail API, which is commented out using the #'s.

**Lines #227-235** in method `get` in class `InvitationHandler`

```
if student.name == None or not student.name or student.name.isspace():
    origin = student.additional_fields
    gn = strip_name_from_additional_fields(origin, 'GivenName')
    fn = strip_name_from_additional_fields(origin, 'FamilyName')
    fullname = gn + ' ' + fn
else:
    fullname = student.name

invitation_email = InvitationEmail(self, user.email(), fullname)
```

Get the student's name from the additional_fields, and put it into the InvitationEmail initialisation method as fullname.

**Lines 341-349 #** in method `post` in class `InvitationRESTHandler`

```
if student.name == None or not student.name or student.name.isspace():
    origin = student.additional_fields
    gn = strip_name_from_additional_fields(origin, 'GivenName')
    fn = strip_name_from_additional_fields(origin, 'FamilyName')
    fullname = gn + ' ' + fn
else:
    fullname = student.name

InvitationEmail(self, email, fullname).send()
```

Get the student's name from the additional_fields, and put it into the InvitationEmail initialisation method as fullname, and call the send method from InvitationEmail class to send the email.

##Modifications to the code at modules/rating/rating.py

###Additional imports:

**Line #27**

`import sendgrid`

The above import allow the code to use sendgrid for sending emails.

###Additional static variables:

**Line #59** - `SENDGRID_API_KEY = '<SENDGRID_API_KEY_HERE>'` - The sendgrid api token for your sendgrid account, so the api can authenticate and send the emails on your behalf. Replace the value with your own.

###Additional methods in class `RatingHandler`

**Line #186** - `sendgrid` - sends the email using the sendgrid API.

**Line #201** - `strip_name_from_additional_fields` - gets the value of a property from the additional_fields from the student's record in the datastore.

###Additional code to existing methods:

**Line #164-177** in method `post` in class `RatingHandler`

```
#send email about new rating.
origin = student.additional_fields
if student.name == None or not student.name or student.name.isspace():
    gn = self.strip_name_from_additional_fields(origin, 'GivenName')
    fn = self.strip_name_from_additional_fields(origin, 'FamilyName')
    fullname = gn + ' ' + fn
else:
    fullname = student.name

mainemail = self.strip_name_from_additional_fields(origin, 'EmailAddress')
cbm = self.strip_name_from_additional_fields(origin, 'ContctByMail')
sm = self.strip_name_from_additional_fields(origin, 'SendMail')

self.sendgrid(fullname, rating, additional_comments, key, mainemail, cbm)
```

This segment sends an email when a new rating/feedback is received from the rating module.

It gets the student's name and email who submitted the rating, and whether they have given permission to be contacted by email from the additional_fields for the setudent.

Then it uses the sendgrid method to send the email, passing in the student's name, email, permission to contact, the key which is used to identify the lesson where the rating was submitted and
any additional comments they may have entered for the rating.