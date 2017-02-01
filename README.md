# NeoCode-RenderBenchmarkTool
FileMaker does not have a way to benchmark the processing power of a FileMaker connected device.  This is that tool.  It tests client side processing power by rendering layout objects.  It works for *.fp7 files (FileMaker Pro, FileMaker Go, & WebDirect), but is NOT useful for *.fmp12 files-- read the discussion to find out the three reasons why.



## WHY DO YOU NEED THIS?
* to determine the layout-rendering performance benchmark for a FileMaker connected device.


## SETUP
### Requirements NC_RenderBenchmark_V01
    FileMaker 11
    BaseElements Plugin (http://www.goya.com.au/baseelements/plugin)

### Requirements NC_RenderBenchmark_V02
    FileMaker 13+

### Installation
    Open the file locally or upload to your server.  The test will run on open.

### Credentials
Opens automatically with the following credentials.  Admin user can edit file.
* user: "Admin"
* pass: ""
  
  
## DISCUSSION

###PART 1 - FM11 SOLUTION HAS PERFORMANCE ISSUES  
#### THE CONTEXT:
A single file, FileMaker 11, solution is distributed to a group 7 users (Win 7) via FileMake Server 11.  Users are experiencing intermittent performance issues; however, some users report the issue is worse and more frequently a problem compared to others.  The application does use some novel design (ESS, Web Services), so there are plenty of potential application/network/server bottlenecks.  The performance issues can not be replicated in development or test environments.  Users are too busy to troubleshoot.  System/network administration report there is no reason to believe it's a systems problem.  Conclusion, it's the FileMaker application.  Of course it is.  

#### THE PLAN: 
Fix the FileMaker performance issues; but first, build a tool that will allow us to measure & report (to-be-discovered) performance improvements to users.  Because the performance issues vary wildly betweeen users, each user needs their own performance benchmark.  

#### THE PROCESS: 
Users with the worst performance experience are mostly seeing the spinning blue wheel.  This tells us the bottleneck is likely on client side.  Timing scripts reveals that the performance lag extends beyond script run-times-- the CPU is choking on the FileMaker layout.  Our conclusion is that the tool needs to benchmark the FileMaker client's rendering of layout objects.  We build a tool that, using custom functions to set global variables, times how long it takes the machine to evalute & render the conditional formatting for a layout objects.  The tool works by calling a window refresh script step; the refresh causes the conditional formatting for a repeating field evaluate and turn green-- effectively, it's a progress bar that progresses only as fast as the CPU will allow.

#### THE RESULT:
It works great.  Systems administrators and users all understand that performance issues are first and foremost due to out-dated client side hardware.  The secondary issue is the processing load created by conditional formatting load on complex layouts-- something we addressed in future iterations. 



### PART 2 - FM12 VERSION OF TOOL ISN"T WORKING:

#### THE CONTEXT:
The tool was built in FileMaker 11.  We decide to convert the tool for FileMaker 12+; however, the tool doesn't work in FMP12.  There are three problems: (1) the progress bar renders all at once, instead of incrementally; (2) some of the cells in progress bar do not turn green; (3) the time required to render the bar varies wildly.  


##### FMP12 ISSUE #1 - NOT INCREMENTAL:
The progress bar normally renders in the following sequence: the conditional formatting of the first field repetition evaluates (this starts the timer), the evaluation completes, and then the conditional formatting turns the 'cell' green.  The user then watches the process repeat: cell 2,3,4... one after another, they all turn green until the test is complete and we display the time it took.  In FMP12, we see an empty render-bar as FileMaker works (blue spinner) and then it just displays the final rendered result.  It appears to freeze the screen, evaluate the conditional formatting, and then display everything at same time.  This is not what happened in FM11.

##### FMP12 ISSUE #2 - NOT COMPLETE:
The render-bar isn't all green.  There are always some cells that are empty. 

##### FMP12 ISSUE #3 - WILDLY DIFFERENT RESULTS:
In FM11, the tool returned the same render time +/- 0.5 seconds.  In FMP12, the render time is now ranging from less then 1 second to 8 seconds.  The difference in timing is obviously connected to Issue #2 (Not Complete); however, sometimes the tool returns negative results.  So there's something else going on.  

#### THE EXPLANATION:
##### ISSUE #1 EXPLANATION: 
As you may have already deduced, the progress bar doesn't render sequentially because FMP12 layouts use CSS.  In FMP12, FileMaker is basically freezing the layout, running all it's conditional formatting and layout dependent calculations, creating a css document, rendering the css document, and then printing the resulting bitmap on the screen as a FileMaker layout.  This caused us to revise the tool methodology.

##### ISSUED #2 & #3 EXPLANATION:
Issue 2 & 3 have the same root cause.  The tool was designed to start timing when cell #1 is evaluated and stop timing once cell #99 is evaluated.  The problem is that, in FMP12, FileMaker evaluates the conditional formatting in a seemingly 'random' & 'changing' order. Sometimes it evaluate cell #43 first and cell #8 next.  Sometimes it evalutes the last cell first (which results in negative render times).  This random evaluation order explains the wildly different render times and the incomplete, intermittent, rendered cells.  In FileMaker 11, each object is evaluated in the order it was added to the layout... this is not the case in FMP12+.  

#### WHAT IS GOING ON?
1. The evaluation order changes each time FileMaker enters layout mode or the window zoom is changed; the refresh step maintains the order established by the mode/zoom change. 
2. HideState calculations are evaluated first; Conditional formatting calculations are evaluated next. 
3. Within the Hide and formatting conditions, objects are evaluated accordingly:
  1. Tab objects are always evaluated first (according to the object's Z axis or forward/back arrangement).  
  1. Fields, buttons, and other objects are evaluated (in a seemingly random order).
  1. Slider objects are always evaluated last (again, according to the object's Z axis or forward/back arrangement).



### NEXT STEPS:
Frankly, none.  We don't currently have a compelling reason to unravel why FileMaker changes the random order which it evaluates conditional formatting for some layout objects-- at least not one other than curiousity and the off chance that it is has some kind of Silly Putty like utility.
