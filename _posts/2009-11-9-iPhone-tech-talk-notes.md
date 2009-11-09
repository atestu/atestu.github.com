---

layout: post

title: iPhone Tech Talk Notes

categories: [iphone]
---





What follows are my notes taken during the iPhone Tech Talk session in Paris in 2009. I'm a lazy person, so there is no editing for now. I'll do it later (which means I'll probably never do it, let's be honest).


These notes are interesting for beginners in iPhone development. You need to know a bit about Cocoa Touch. 


These are raw notes. They're quite personnal and my comments will probably be annoying for some people. If my stupid remarks upset you, please close your tab and go read the official Apple Documentation.


- - -



I'm wearing a name tag. Classical music in the background fades for some bad electronic crap. Everybody sit and pull out their iPhone. I'm trying Simplenote the whole day. I'll take out my MacBook if my thumbs can't keep up. 



We're in room A right now for the kickoff and that's where I'm gonna stay the whole day. My God this music is bad. 

Oh Coldplay. Much better. 



These are live notes so I'll write a lot when I'm bored and not so much if the talk is interesting. In the end it probably won't be very interesting for you to read. We'll see.



Now for some terrible teenage pseudo indie rock. Needless to say that I don't socially interact with anyone.



Katy Perry. Black Eyed Peas. All right maybe this isn't a blind test.



I'm switching to my MacBook on [Notational Velocity](http://notational.net/). Much more comfortable. The talk is about to start. Effective iPhone Application Architecture. 

They have code-level support all day. I'm kind of ashamed of my code. It's a terrible mess. Which means I really should go talk to these engineers. Will do that at the next break.



Some people have crazy things in their docks, like Quicktime or Word. That's weird.



Effective iPhone Application Architecture Part I
===



This talk is about "doing things right". I need this class.

Hilarious slide with comments like `/* no idea why this works */`, `/* pretty sure this  leaks */` and macros `#if FIXME`.



Data Persistence
--

User Data : user controlled => Core Data (YES!), if not : small => XML, pLists, big => Core Data.

Don't use NSDictionary for data storage. Use proper data object (attributes).

Preferences : NSUserDefaults for lightweight settings like navigation states. NOT for heavyweight data like images. NOT for sensitive data (whoops…).

Sensitive ? Keychain. Store for your company = multiple apps.

Cached Data : faster iTunes Backup by storing in proper locations. Locations in NSPatchUtilities.h, NO hardcoded paths (hehe…).

tmp (cleared, not backed up), NSCachesDirectory (saved, not backed up), NSDocumentsDirectory (backed up).



Design decisions
--

Make massive changes easier. use dev target to reach every version of iPhone OS. ALWAYS link against current SDK. DON'T base behaviour based on OS version.

Runtime checking = respondsToSelector (check for Mail 3.0) but CACHE it because it's heavy stuff.

weak linking = tells the dynamic loader that if it can't respond, set it to null. great for checking if 3GS (video) for eg.

Drill down into greater detail, focus on one thing at a time. Recipes example.

A controller for each screen : list, detail, photo.

One screen, one view controller. Always subclass UIViewController. View shouldn't interact with model (duh!) : no sthortcuts! You're doing it wrong, n00b.

Many screens, one model.

App delegate loads the model, hands it off to the 1st view controller.

Editing : parent needs to know (pointer ? BAD IDEA (whoops)). Use delegate! Starting to think I should rewrite my app from scratch…

Dalegate : Declare a protocol (here, one method : didChangeRecipe + property). The list now impements the detailViewControllerDelegate protocol. (Implementation : realoadData). Set itself as the delegate before pushing the detail. Set the delegate to nil in dealloc.

send msg to nil? No problem. Send msg to garbage pointer? BIG problems.

Alternative to delegation : notifications (removeobserver in dealloc), key value observing (KVO).

Separation : Reuse, Reorganize.



Memory mgmt
--

Not sexy, but you have to talk about it.

Identify the problem : intermittent crashes, crashlogs without backtrace, mem warning in the console.

Tools : Instruments (Leaks, ObjectAlloc (the graph has to go up AND DOWN)).

Memory warnings on the simulator. Use instruments with the simulator.

Xcode static analysis (yummy!) : retain/release problems, trust *but verify*.

No need to cache the view controller. One nib per VC. Don't put everything in one nib.

	viewDidUnload

	{

		[myView release];

		myView = nil;

	}memory

Core Data takes care of a lot of memory stuff.





Application Lifecycle
--

Compatibility : Classname prefixes (prevent namespace collisions). Avoid underscores in method names (Apple uses them, things could get ugly).

Use refactor to change all the names (better than search in project I've been using all this time…).

Implement - don't call.

handling interruptions : phone calls, SMS, notifications, etc…

implement applicationWillResignActive : Pause and save.
- user dismisses - applicationDidResumeActive
- user accepts - applicationWillTerminate (same as pressing home button)

Concurrency : performing side tasks:

use asynchronous API where available : networking (NSURLConnection).

NSOperation (not for networking)

Illustrate activity to the user.



Next session : concurrency and more code. I'm definitely staying.



Effective iPhone Application Architecture Part II.
===



Same guy. Works in a small team at Apple. They call it "SWAT Teams".



Basically 1 = code flow (theory) and 2 = implementation realities. 



UITableView
---

composited drawing performance : it's there. Impacts the way we code.

opacity matters (system optimizes it).

Simplify the view (increases performance and usability).

Flatten drawing with custom view.

use setNeedsDisplayInRect when you can.

view.opaque is YES for a reason.

NEVER call drawRect directly.

Implement highlighted and setHighlighted. Trust UIKit to do the right thing (don't poll for highlight state).



UIImage
---

Every app uses these.

imageNamed : immediate decompression (blocks!) and cached. 

imageWithContentsOfFile : decompression on demand, not cached.

use imageNamed for UI Components, images used repeatedly (because cached). Avoid loading many at once (can slow apps or view loading). OS uses imageNamed to load nib referenced images.

use imageWithContentsOfFile : image taht wont be needed immediately, used infrequently.

Don't forget that UII is wrapping CGImage, part of Core Graphics, which provides a much wider and deeper set of functionality (eg scaling)



NSOperation (Threading)
---

Why bother? keep UI responsive and leverage idle resources (even on single core).

intensive computation : parsing, image manipulation, disk IO

instrument before you decide : concurrency may not always be a WIN.

where to avoid : networking (it's already there, dumbass! use async APIS)

traditionally, threading is HARD. NSO makes it a lot easier.

Thread confinement : object access confined to one thread. Forget about locking, signaling and all that crap (thank God!).

Compelling OO encapsulation. Everything is in one object (code, state, dependencies, priority). Easy to add code (re-factor existing functionality). Heavy lifting is done for you.



Subclass NSO and override main.

create the operation, give it data (NOT shared data), create a NSOqueue.

getting the results : delegation! notification

UIKit is not thread safe so if you wanna interact with the UI : performSelectorOnMainThread.



Demo! flicking through a list and images loaded à la Tweetie 2. So THAT'S how he did it. Cancel operation if cell not on screen. We see the code. It looks quite easy to implement but… it always does.



Understand basic thread safety rules : UIKit only on main thread : don't touch at the UI (no realdData) in a thread.

transfer data ownership : DO NOT share.



Working w/ system owned view herarchies
---

alternative to private APIs (we know you do it. Please stop.)

override UIWindow sendEvent: => allows you to insert yourself in the event chain
- call super first
- look but don't touch
- be quick



Demo! Detects only triple taps. 

Demo! Image picker : don't hack! All right, we get it.



All, final word : READ THE FUCKING MANUAL. Know the APIs.



Loved these two session, especially Part I. Now I know what's wrong with my code. Will start from scratch. 


- - -



Lunch
---

Music : Phoenix (!) and trance techno. Very disturbing. Plus it's kinda loud. It's 1:20 and I only have 4 hours remaining on my battery. Oh yeah I forgot to mention : the lunch is free and so was the nice breakfast. Saving the chocolate pie (yup, you read that right) for my goûter.

The next session starts in 10 minutes. I'm still in room A, waiting for "Working with Core Data". Ratatat. Bloc Party. Pixies.


- - - 



Core Data
=========

There's a guy in a suit and a PC.

Core Data w/ batching reduces RAM usage (/2) and 80% faster to launch.

NSManagedO is a "row" of data. Encapsulates data + relationship. Provides validation.

NSManagedOContext is a "scratch pad" of your objects. Tracks changes. Handles actions on your data (insert, delete, save…).

NSPersistentStore : data stored in SQLite, Core Data handles everything for you, no SQL code. 

NSManagedOModel describes the structure of your data : graphically create this in XC.

NSEntityDescription : blueprint for your object.



Structuring your data is the most important step.
--------------------------------------------------

speed vs. space : 

* normalization : perfectly factored, no repetition , space efficient.

* denormalization : repetition of data, faster access (to a point), less space efficient).



Normalize first, then find the best organization that fits your needs. Carefully denormalize. Duplication of info requires extra maintenance.

Keep the UI in mind : what's the user flow, what's the data you need at each step? Additional data? It's OK for your model to evolve : Core Data migration support.

No binary in the DB, except if very small (about 1k or less). ~100k images should be in their own entity with relationship (1 to 1) so you don't have to load it too often. Bigger (MBs) images should be stored on disk.



Fetching efficiently
--------------------

NSFetchRequest : access saved data, flexible & powerful. Tell it what, where, and how.

* set the entity (NSEntityDesc

* set predicate (if applicable)

* fetch!



### Prefetching

get more info w/ each fetch, when you know the data you need, relationship and their attributes. Prefetch rel : just give it an array of strings in setRelationshipKey… (not necessarily a pList). Use carefully.



### Batching

Load only what's on the screen, save memory, only deal w/ what's necessary. Easy to adopt : `[fetch setFetchBatchSize:10];`. One FUCKING LINE. Transparently fetch behind the scene.



### Partial faulting

when you only need a subset of the data. Heavily UI influenced : `[fetch setPropertiresToFetch:[NSArray arrayofstrings]];`.



### Aggregates

sum, count, min, etc… supported natively by Cocoa. Call `countForFetchRequest` instead of a `fetch` + `count`. in-database aggregates : NSExpression.



Integrating with UIKit
----------------------

Managed Objects are just objects! Track notifications : `NSManagedObjectContextDidSaveNotification`

UITableViews : Core Data can help

NSFetchedResultC fetches, orders and sections for your data. caches section info for ultra fast performance.

HowTo? Create it, set the fetch request, predicate, and sort descriptors. Set yourself as its delegate and implement your tableview methods (all of that handled in the template). Specify the keypath to be used for the section name. Computes and persistently caches section info, updates info by observing changes.

Settings on NSFRC are immutable. : fetch request & predicate.

Create separate controllers for different data sets.

Threading : Core Data uses a policy of isolation : one managed object context per thread. Never pass MObjects between threads. Prefer objects IDs if you pass anything. Use different MOCs.









Testing and debugging your iPhone app
=======================================



Finding and fixing bugs
-------------------------

### Unit Testing

avoids reintroducing old bugs, prevents regressions

Framework very test driven : Core Data, WebKit

* Logic tests : runs in the simulator

* Application tests : runs inside your actual app on iPhone

Create a UT Bundle. Logical tests : `assert()`.

Requires a lot of discipline… boy is he right about that. This is a lot of work. So…

### Static Analysis! 

Find bugs you didn't know existed. Easy to read message. Without test cases. Focus on Cocoa API enforcement, memory management, logic erros. Based on Clang (Open source).

Tip : duplicate your Debug configuration to create an Analyze configuration.



Using the organizer
---------------------

Device management : you can download the app data from the orga. Here you can also see all the crash logs, provisioning profiles, 



Beta tests
------------

Let others work for you (always a good thing!).



So this session was by far the least interesting. 



Damn that chocolate pie was good. 



Maximizing iPhone App Performance
===================================

It's all about "Pimpin' you app" here. Sweet.

OK this talk looks awesome.



You **need** to optimize everything.



Instruments

Shark: Time profile, fine tune stuff. Powerful but more difficult than Instruments.

Simulator great for debugging memory. For graphics and timing, use the device.



Drawing
-------

use built-in drawing whenever possible, take advantage of animatable view properties.

Mark views as opaque. Draw minimally : call `setNeedsDisplayInRect:` and check the rectangle passed to `drawRect:`.

cache static drawing objects like images, but be prepared to release them in low-memory conditions.

common drawing bug : invalidate times before creating new timers (when you come back from interruptions).

Use PNG files : optimized at build time.

avoid allocating views during scrolling (it's expensive). Reuse table cells (duh!).

collapse the cell view hierarchy.



**Demo!**

Finding zombies with Instruments. ZOMBIES! Oh that's nice. This is gonna save me a lot of time… Why am I discovering this just now? Shame on me.

Now let's optimize scrolling. The scrolling is OK looking for now, but he wants to make it better. Run it in Instruments : the amount of memory being consumed continues to grow. Apparently it's not a good thing.

Use Core Animation in Instruments to debug views. Green is good (no compositing), Red is bad (compositing). Remember : compositing is when your views aren't set to opaque. Opaque background in UILabel is automatically set to transparent when in highlighted state. Smart.

The scrolling is *very* smooth now.

I love programming.

OK apparently it can be even smoother. Wow. Combine all the views into one SUPERVIEW! (CAPS LOCK mandatory here). And indeed, it is even smoother. Tweetie looks less magic now.



Application launch
--------------------

Be lazy! Don't load everything on launch. Don't perform network operation in applicationDidFinishLaunching. Use small nib files.



Memory Usage
--------------

Virtual : it's there, but no swap file. Writeable pages stay in memory, read-only memory pages can be evicted and reloaded.

Static memory : use Thumb except if using floating points. Always use Thumb-2.

Configuration :

* Avoid C++  exception

* Avoid RTTI RunTime Type Identification

`autorelease` can be expensive. Calling it is not, but autoreleased object don't get cleared right away. Prefer direct `release` : more control.

cache the right objects

avoid atomic properties. access is up to 10 times faster with `nonatomic` (however, *not thread safe*).

layer objects are small but backing store is large, shared with SpringBoard.

Remember that images are decompressed in memory to be displayed.

Memory warnings : 1) Free memory, 2) background apps exit and 3) quit you app.

Don't ignore them : it's normal to have them, you must respond : release stuff.



Files and data 
----------------

use memory mapping for large files:

* `mmap` and `munmap`

* `[NSData initWithContentsOfMappedFile];`

Use Core Data

Don't include unnecessary files (eg svn directories…)

Data in ~/Documents is backed up (user data but not caches)

~/Library/Caches for data that can be reaquired

Use PNGs and Plists for structured data files



Power and battery life
------------------------

Very expensive to send data. You should combine everything in one operation.

* Coalesce data into large chuncks rather than thin streams

* Minimize amount of data transmitted - use compact data formats.

2G between WiFi and 3G.

Core Location:

use the least amount of accuracy : default is "best" (expensive). `distanceFilter` = how often you want to receive location changes

call `soptUpdateingLocation` as soon as you're finished.

Fast code = less CPU = less power

Don't poll a condition! Use notification, iPhone OS is event based! Be reasonable.

Consolidate CPU usage into short burst.

while using OpenGL : pick a fixed framerate (30 fps), don't unnecessarily redraw frames.



All right that's it! I'm outta here to edit these 2,500+ words…