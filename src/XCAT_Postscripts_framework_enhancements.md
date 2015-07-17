[Design_Warning](Design_Warning)


The xCAT postscripts framework needs the following enhancements: 

  * The postscript should not hang, no matter what the configuration error is. 
  * The node should not be in an infinite provisioning loop, no matter what the configuration error is. 
  * Provide more useful messages or other effective ways for debugging 
  * Make the deployment process less dependent on name resolution, and review where the error msgs go when things go wrong. 
  * Use a shell subroutine library to avoid duplicate code in different postscripts 
  * We need to figure out a way in xCAT to make our default postscripts smarter so that we don't try to run the otherpkgs and some other postscripts on every boot of a diskless node, yet still allow explicit use of it when the admin does want to run it on an exception basis. 

  


  
An email from one xCAT customer: 

As you know, I've been working with the xxxx team on the xxxx project. This week, they are hitting some issues provisioning nodes using xCAT. What they've found is that after the node is provisioned, its provision status (nodelist.status) stays stuck in "defined". This usually indicates that something in the post-install failed to run. Based on my experience so far, debugging post-install failures can sometimes be tricky for 2 reasons: 

Issue 1: Not enough log messages are generated 

Inside the kickstart %post section, there's some code that is used to do all the prep work like downloading postscripts, preparing the /xcatpost/mypostscript wrapper script, etc.. This code generates very little or no messages in the MN's syslog. If a failure occurs in any of this prep code, no failure messages are generated to tell us an error has occurred. 

Example: I recently found an issue where I copied a script to the /install/postscripts directory that had the wrong permissions set, and then tried provisioning a node. The node provisioning would fail, and its status would getting "defined". After checking the MN's syslog, I didn't see any errors. But, after manually re-running the %post section script on the failed compute node, I noticed many wget errors that I didn't see in the MN's syslog. These messages helped tell me that the script I copied over had the wrong permissions set. 

  
Issue 2: Post-install failures sometimes lead to a chain reaction of failures 

If some code inside the %post section fails, it may cause other code in the %post section to fail, setting off a chain reaction of failures. This makes debugging tricky because after a node provisions, you may see a bunch of things failing, but the real reason why they failed is because something else failed much earlier in the execution of the %post section. 

Example: Using the same example I used for reason 1), after my node provisioned, I saw a bunch of failures like: not being able to log into the node using passwordless SSH, etc... But, the real reason why these failures were happening was because the wget failure in the %post section prevented the /xcatpost/mypostscript wrapper from getting created properly, which prevented all of the postscripts/postbootscripts from running. 

  


* * *

Suggestions 

I had some thoughts around how to address the 2 issues above. I think addressing these issues can help simplify and speed up the debugging process for xCAT users. I'm not sure if you already have plans to make these improvements, but I thought I'd share them with you to get your thoughts. 

Suggestion 1: Enhance %post section to log more information 

I think the following information would be help users debug issues more quickly We should log messages to indicate what the %post section is doing (e.g., downloading postscripts, creating /xcatpost/mypostscripts wrapper, etc...). This will make it easier to trace the %post script execution. Create a new log to record everything that the %post section runs. This log file would get created on the compute node and would be used to log more detailed/verbose msgs. For example, if %post section runs "wget", we should record all of wget's output/error msgs and exit code in this file. This log file would complement the syslogs that are currently generated. The detailed log should be preserved after the node reboots. Currently, I noticed that the wget output is getting logged to /tmp/wget.log during OS provisioning, but after the node reboots, this file isn't preserved so you lose all of the debug info. I also noticed there's a file called /root/post.log on the compute node which doesn't have anything logged to it. Perhaps, we can use it for this purpose? All of the postscripts (in postscripts table) that are executed in %post section should have their exit status logged. I think this is also something we can add to the /var/log/xcat/xcat.log file for postbootscripts. 

  
Suggestion 2: If something fails in the %post section, fail immediately, and set the node's provision status to "failed" If a failure happens in the %post section, we should abort the provisioning, and set the status to "failed". In most cases, if a failure occurs, there's no reason to continue with the provisioning. Doing this will: help prevent "chain reaction of failures" help report failures faster to the user so they can take action sooner, instead of waiting until the provisioning has finished help speed up debugging because we are stopping the provisioning at the point of failure, so the user will need to analyze fewer log messages 