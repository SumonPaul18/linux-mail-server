#
### Monitoring & Managing Postfix Email Queue
#
<b><i>Reference:</b></i>

Manage Postfix Mail Server Queues like a Pro <br>
https://mailmum.io/posts/manage-postfix-mail-server-queues/


How to clean Zimbra mail queue as root user <br>
https://www.geekytuts.net/tag/mail-queue/

Postfix - how to retry delivery of mail in queue <br>
https://serverfault.com/questions/279803/postfix-how-to-retry-delivery-of-mail-in-queue

How to Flush the Mail Queue in Postfix <br>
https://sendgrid.com/en-us/blog/how-to-flush-the-mail-queue-in-postfix#:~:text=If%20an%20email%20gets%20queued,to%20take%20any%20immediate%20action.

Postfix Queue Management <br>
https://easyengine.io/tutorials/mail/postfix-queue/

Requeue Emails in Postfix on Linux <br>
https://www.faqforge.com/linux/how-to-requeue-emails-in-postfix-on-linux/

#

View queue mail from postfix log <br>
From user: expdochk@apparellinkhk.com <br>
To user: shipping@laisherui.com <br>
####
    less /var/log/maillog |egrep "expdochk@apparellinkhk.com|shipping@laisherui.com"

####
    postqueue -f | grep 'from=.*@debonairgroupbd.com' | awk '{print $1}' | xargs postsuper -q

Check the Mail Queue
####
    mailq 
####
    postqueue -p 

Read a particular message, by ID 
####
    postcat -vq DA80E24A0A

Attempt to send all email from mail queue <br>
flushing entire queue immediately.
####
    postqueue -f 

Delete All Deferred Mails
####
    postsuper -d ALL deferred

Requeue all active messages 
####
    postsuper -r active 

Delete a single mail from queue
####
    postsuper -d DA80E24A0A

Delete all mail from queue
####
    postsuper -d ALL 

Read single message in Postfix queue by ID
####
    postcat -q A705238B4C
####
    postcat -bh -q $id

Details Option <br>
-b 	Show body content. <br>
-h 	Show message header content. <br>
-q 	Search the Postfix queue for the named files instead. <br>

#
View queue mail from postfix log <br> 
From user: expdochk@apparellinkhk.com <br>
To user: shipping@laisherui.com <br>
####
    less /var/log/maillog |egrep "expdochk@apparellinkhk.com|shipping@laisherui.com"

#
####
    postqueue -f | grep 'from=.*@debonairgroupbd.com' | awk '{print $1}' | xargs postsuper -q

#
Monitor the Mail Log
####
    tail -f /var/log/maillog

Check the Mail Queue
####
    postqueue -p 

Requeue mail in Postfix
####
    postsuper -r ALL

Requeue Specific Emails
####
    postsuper -r A705238B4C

Troubleshooting <br>
Checking or Configuration <br>
####
    /etc/postfix/main.cf and /etc/postfix/master.cf

#
Remove to spaming user mail from queue in postfix
####
    mailq | tail -n +2 | awk 'BEGIN { RS = "" } /Useraddress/ { print $1 }' | tr -d '*!' | sudo postsuper -d -
####
    mailq | tail -n +2 | awk 'BEGIN { RS = "" } /@domain.com/ { print $1 }' | tr -d '*!' | sudo postsuper -d -
####
    mailq | tail -n +2 | awk 'BEGIN { RS = "" } /@reverie-bd.com/ { print $1 }' | tr -d '*!' | sudo postsuper -d -
#
Script to monitor the queue size
####
    /usr/bin/mailq | /usr/bin/tail -n1 | /usr/bin/gawk '{print $5}'

Create a Shell Script for notify queue list to email
####

    nano queuemail.sh
####
    #!/bin/bash

    mailq_count="$(/usr/bin/mailq | /usr/bin/tail -n1 | /usr/bin/gawk '{print $5}')"

    # If variable is empty, then the queue is empty -> set it to zero
    if [ -z "$mailq_count" ]; then
      mailq_count=0
    fi
    
    if [ "$mailq_count" -gt 0 ]; then
      echo "Mail count on mail.myserver.local is ${mailq_count}" | /usr/bin/mail -s "mail.myserver.local queue size: ${mailq_count}" notify.me@somemail.local
    fi

#
